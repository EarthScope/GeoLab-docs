# Using S3 Buckets

GeoLab gives every user a temporary scratch bucket in Amazon S3 for large intermediate results. This page introduces object storage and shows you how to use the scratch bucket from the Terminal and from Python. It also covers connecting your own S3 bucket for data you want to keep. For an overview of all storage options, see [File Systems and Data Storage](../getting_started/user_storage.md).

## What Is Object Storage?

Object storage works much like requesting waveforms from a seismic data center. You ask for data by its identifier, such as `IU.ANMO.00.BHZ`, without knowing where it's stored on the data center's disks. When data are corrected, the data center replaces the record instead of editing it.

In object storage, each stored item is an **object**, and objects are kept in a **bucket**. A bucket is a flat container with no folders inside it; you find each object by its name, not by a directory path. Each object holds three things:

- **the data**, such as a miniSEED file, a CSV table, or an image file;
- **a key**, the object's unique name within the bucket, such as `waveforms/2024/IU.ANMO.mseed`. Like a SEED identifier, it's what you use to request the object;
- **metadata**, such as size, content type, and the time it was last modified.

You don't open objects the way you open files on a disk. Instead, every action, such as uploading, downloading, or listing, is a web request sent over HTTP to the storage service's API. Tools like the AWS CLI and fsspec send these requests for you, so in practice you work with commands and paths, not raw requests.

Amazon S3 (Simple Storage Service) is the most widely used object storage service, and GeoLab's scratch bucket is an S3 bucket. Many other providers support the same API, so the same tools work with them too.

## Prefixes: Folders That Aren't Folders

A bucket has no directories. Every object sits at the same level, and its key is a single string. A key like `waveforms/2024/IU.ANMO.mseed` is not a file inside a `2024` folder inside a `waveforms` folder. It's one name that happens to contain slashes.

A full object address combines the bucket name and the key: `s3://bucket-name/key`, for example `s3://my-bucket/waveforms/2024/IU.ANMO.mseed`. Everything after the bucket name is the key, slashes included.

A **prefix** is any leading part of a key. `waveforms/` and `waveforms/2024/` are all prefixes of `waveforms/2024/IU.ANMO.mseed`. Tools use prefixes, together with the `/` **delimiter**, to make a bucket look like a directory tree:

```text
s3://my-bucket/
├── waveforms/2024/IU.ANMO.mseed
├── waveforms/2024/IU.COLA.mseed
├── waveforms/2025/IU.ANMO.mseed
└── stations.csv
```

Listing the prefix `waveforms/` with the `/` delimiter returns the "subfolders" `2024/` and `2025/`, because S3 groups every key that continues past the next `/`. The AWS web console and `aws s3 ls` show these groups as folders, but they exist only because some object's key starts with them.

This has a few practical consequences:

- **The trailing slash matters.** `aws s3 ls s3://my-bucket/waveforms/` lists what's under that prefix. Without the slash, `aws s3 ls s3://my-bucket/waveforms` matches the prefix `waveforms` itself.
- **Folders can't be empty.** A prefix disappears when you delete the last object that uses it. You don't need to create a folder before writing to it; writing `results/run1/output.csv` creates the key, and the "folders" appear with it.
- **Renaming a folder means rewriting every object.** There's no rename operation. Moving `results/` to `archive/results/` copies each object to a new key and deletes the old one, which can be slow for prefixes with many objects.
- **Use prefixes to organize and to filter.** Because listing and copying work by prefix, a consistent scheme such as `<project>/<year>/<network>/<station>` lets you grab a subset with one command, for example `aws s3 cp --recursive s3://my-bucket/waveforms/2024/ ./2024/`.
- **Prefixes spread the load.** S3 handles about 3,500 writes and 5,500 reads per second for each prefix, and slows down requests beyond that. A single notebook won't reach this limit, but many parallel workers, such as a [Dask Gateway cluster](./dask.md), can. Giving each worker its own prefix, such as `results/station-A/` and `results/station-B/`, gives each one its own limit. For example, 10 workers each writing 500 objects per second under `results/` make 5,000 writes per second on one prefix, which is over the limit. Split across 10 prefixes, that's 500 per prefix, well under each limit.

## How It Differs From File Storage

Your GeoLab home directory (`/home/jovyan`) is **file storage**, the same kind of storage as the disk in your laptop. It's a network-attached disk connected to your server and organized as a tree of folders. Programs can open a file, jump to any position in it, read or change a few bytes, and append to it. Renaming a folder is instant, because only the folder's name changes.

Object storage gives up those conveniences in exchange for scale. You can't edit part of an object or rename a folder in place; you read and write whole objects through requests. In return, a bucket can hold far more data than a disk, and many machines can read from it at the same time. In GeoLab, this means using your home directory for notebooks and code, `/tmp` for fast working files, and S3 for large datasets and for data your Dask workers need to reach.

The table summarizes the differences:

| | File storage (home directory, `/tmp`) | Object storage (S3) |
| --- | --- | --- |
| **Organization** | A hierarchy of directories | A flat bucket of keys. The `/` in a key only looks like a folder path; S3 has no real directories |
| **Access** | The operating system's file calls (`open`, `ls`, `cp`) | HTTP requests through an API (`aws s3`, `fsspec`, `boto3`, `s3fs`) |
| **Editing** | Change part of a file in place | Objects are replaced whole; to change one byte you rewrite the object |
| **Scale** | Limited by the size of the disk ([50 GB](../getting_started/quotas.md) for a GeoLab home directory) | Effectively unlimited; you pay for what you store. The scratch bucket is managed by EarthScope and data is kept for only seven days to keep minimize costs and to keep access free to users. |
| **Sharing** | Attached only to your GeoLab server. | The **scratch bucket** is not private and is stored under your user ID prefix, which you can share with other GeoLab users. **Your own bucket** can be shared by granting users IAM permissions in the AWS console. |
| **Speed** | Low latency for small, frequent reads and writes | Higher latency per request, but high throughput for large or parallel reads |

## Using the Scratch Bucket in GeoLab

Object storage suits large datasets and parallel computing. Many workers can read different objects at the same time, and the data stays in the cloud next to the compute. That is why many public geoscience datasets are published in S3, and why GeoLab provides a temporary scratch bucket (`$SCRATCH_BUCKET`) for your own large intermediate results.

We recommend reading data from NSF NGF repositories directly rather than copying it into the scratch bucket. Use the scratch bucket for results you produce, especially when:

- **the results are too large for your server.** Intermediate products can quickly outgrow your 50 GB home directory and your server's `/tmp`.
- **many workers need to read or write them.** Dask Gateway workers run on separate machines and can't see your server's disks, but they can all reach the scratch bucket.
- **a workflow has more than one stage.** One step writes results, and a later step, possibly in another session, reads them back.

For example:

- **Ambient noise seismology.** You compute daily cross-correlations between every pair of stations in a regional network over several years. A Dask cluster processes one day per task, and each worker writes its correlations to the scratch bucket under a prefix such as `xcorr/2024-03-15/`. A second step reads all the daily correlations back and stacks them for each station pair. The daily files may total hundreds of gigabytes, but you only need the final stacks, which are small enough to keep in your home directory or your own bucket.
- **GNSS time series.** You process RINEX observation files for hundreds of GNSS stations to estimate daily positions, writing each station's results to `positions/<station>/`. A later step reads the position time series for all stations to fit velocities and remove seasonal signals. Once you have the velocity field, the daily positions can expire with the rest of the scratch bucket.

### How Object Storage Changes Your Workflow

When you move a workflow from files on a disk to the scratch bucket, two habits change:

- **Prefer cloud-optimized formats.** Formats such as Zarr or Parquet split data into many objects, so you can read only the pieces you need instead of downloading a whole file.
- **Write once, read many.** Because objects can't be edited in place, write finished results rather than appending to a file over time.

:::{warning}
**Objects in the scratch bucket are deleted automatically.** Each object is deleted 7 days after it was last created or modified, so each one expires on its own schedule. Because S3 objects can't be edited in place, "modified" means the object was rewritten, for example by uploading it again or by `cp`, `mv`, or `sync`. Reading or downloading an object doesn't reset its 7 days. The date shown by `aws s3 ls` is the date the 7 days count from. Copy anything you want to keep to [your own S3 bucket](#using-your-own-s3-bucket-in-us-east-2) before it expires. Your home directory has a 50 GB limit and isn't meant for datasets.
:::

## Using the AWS CLI to Manage S3 Buckets

The AWS Command Line Interface (CLI) is Amazon's command-line tool for working with AWS services, including S3. It's installed in GeoLab, and your server is set up with credentials for the scratch bucket, so you can run it from a Terminal (**File → New → Terminal**, see [Using the Terminal in GeoLab](../getting_started/geolab_terminal.md)) without any configuration. Run `which aws` to confirm it's available.

The CLI has two sets of S3 commands:

- **`aws s3`** provides high-level commands that feel like the file commands you already know: `ls`, `cp`, `mv`, `rm`, and `sync`. They take `s3://bucket/key` paths, upload large files in parallel parts automatically, and handle many objects at once with `--recursive`. Use these for everyday work.
- **`aws s3api`** maps one-to-one to the S3 API operations, such as `put-object`, `get-object`, and `list-objects-v2`. It takes the bucket and key as separate arguments and returns JSON, which is useful for scripting or for options the high-level commands don't expose.

We recommend `aws s3` because its commands work like the Linux file commands you already use.

:::{note}
In GeoLab, `$SCRATCH_BUCKET` is an alias for a shared bucket plus your username. Use `echo` to see the full bucket path:

```bash
$ echo $SCRATCH_BUCKET
s3://earthscope-scratch/<username>
```
:::

### AWS CLI Examples

The examples below use `aws s3`. For more commands and options, run `aws s3 help`.

- To add a file to the scratch bucket as an object:

  ```bash
  aws s3 cp ./IU.ANMO.mseed $SCRATCH_BUCKET/waveforms/2024/IU.ANMO.mseed
  ```

- To add every file in a local directory, including its subdirectories, add `--recursive`. Each file's path relative to the directory becomes part of its key:

  ```bash
  aws s3 cp ./waveforms/2024/ $SCRATCH_BUCKET/waveforms/2024/ --recursive
  ```

- To list objects in the scratch bucket:

  ```bash
  aws s3 ls $SCRATCH_BUCKET/waveforms/2024/
  ```

- By default, `ls` shows one level and groups deeper keys as folders (`PRE 2024/`). To list every object under a prefix with its full key, add `--recursive`. Adding `--human-readable --summarize` shows sizes in KiB/MiB/GiB and ends with the total object count and size:

  ```bash
  aws s3 ls $SCRATCH_BUCKET/waveforms/ --recursive --human-readable --summarize
  ```

- To copy an object from the scratch bucket to a data directory in the file system:

  ```bash
  aws s3 cp $SCRATCH_BUCKET/waveforms/2024/IU.ANMO.mseed /tmp/data/
  ```

- To copy every object under a prefix to a local directory, add `--recursive`. The CLI creates local subdirectories to match the rest of each key:

  ```bash
  aws s3 cp $SCRATCH_BUCKET/waveforms/2024/ /tmp/data/2024/ --recursive
  ```

- To delete an object from the scratch bucket:

  ```bash
  aws s3 rm $SCRATCH_BUCKET/waveforms/2024/IU.ANMO.mseed
  ```

- To delete every object under a prefix, add `--recursive`. There's no undo, so run it with `--dryrun` first to see what would be deleted:

  ```bash
  aws s3 rm $SCRATCH_BUCKET/waveforms/2024/ --recursive --dryrun
  aws s3 rm $SCRATCH_BUCKET/waveforms/2024/ --recursive
  ```

  :::{warning}
  Always include the full prefix and the trailing slash. `aws s3 rm $SCRATCH_BUCKET/waveforms/2024 --recursive` (no slash) also deletes keys that only start with that text, such as `waveforms/2024_old/...`. Because all GeoLab users share the scratch bucket's credentials, a mistyped prefix could delete other users' files.
  :::

- To move an object to another prefix:

  ```bash
  aws s3 mv $SCRATCH_BUCKET/waveforms/2024/IU.ANMO.mseed $SCRATCH_BUCKET/archive/waveforms/2024/IU.ANMO.mseed
  ```

- To move every object under a prefix to another prefix, add `--recursive`. Keep the trailing slashes so the keys keep their structure under the new prefix:

  ```bash
  aws s3 mv $SCRATCH_BUCKET/waveforms/2024/ $SCRATCH_BUCKET/archive/waveforms/2024/ --recursive
  ```

S3 has no rename operation, so `mv` copies each object to its new key and then deletes the original. For a prefix with many objects this can take a while. Add `--dryrun` to preview what will be moved without changing anything.

- To keep a local directory and a prefix in step, use `sync`. Unlike `cp --recursive`, which copies everything every time, `sync` only copies files that are new or whose size or modification time has changed, so it's fast to rerun. It's always recursive, so you don't need `--recursive`:

  ```bash
  # Upload new and changed files from a local directory
  aws s3 sync ./waveforms/2024/ $SCRATCH_BUCKET/waveforms/2024/

  # Download new and changed objects to a local directory
  aws s3 sync $SCRATCH_BUCKET/waveforms/2024/ /tmp/data/2024/
  ```

  By default, `sync` never deletes anything at the destination. Add `--delete` to also remove files at the destination that no longer exist at the source, so the two match exactly. Preview it with `--dryrun` first, because it removes data:

  ```bash
  aws s3 sync ./waveforms/2024/ $SCRATCH_BUCKET/waveforms/2024/ --delete --dryrun
  ```

  You can limit what's synced with `--exclude` and `--include`. For example, to sync only miniSEED files:

  ```bash
  aws s3 sync ./waveforms/2024/ $SCRATCH_BUCKET/waveforms/2024/ --exclude "*" --include "*.mseed"
  ```

### Using Python to Work with S3 Buckets

We recommend [fsspec](https://filesystem-spec.readthedocs.io) for working with S3 from Python. It provides one file-system interface for many kinds of storage, including local disks, S3, Google Cloud Storage, and HTTP.

fsspec takes the same `s3://` paths as the CLI, so `$SCRATCH_BUCKET` works as-is, and it has a `recursive` option for working with a whole prefix. Many scientific Python libraries, including xarray, pandas, and Dask, use fsspec to read `s3://` paths directly.

You may also see two other Python libraries for S3:

- [s3fs](https://s3fs.readthedocs.io) is the package fsspec uses for `s3://` paths. You can use it directly with `s3fs.S3FileSystem()`, but it has the same methods as the examples below and only works with S3, so there's no advantage to using it instead of fsspec.
- [boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) is the AWS SDK for Python. It gives access to every S3 feature, such as bucket settings and permissions, but common tasks take more code. It takes the bucket name and key separately rather than an `s3://` path, has no `recursive` option, so you loop over objects yourself, and returns at most 1,000 objects per listing, so you have to page through larger prefixes.

Create a file system object and read the scratch bucket path from the environment:

```python
import os
from pathlib import Path
import fsspec

fs = fsspec.filesystem("s3")   # uses the GeoLab server's credentials
SCRATCH_BUCKET = os.environ["SCRATCH_BUCKET"]
```

- To add a file to the scratch bucket as an object:

  ```python
  fs.put("./IU.ANMO.mseed", f"{SCRATCH_BUCKET}/waveforms/2024/IU.ANMO.mseed")
  ```

- To add every file in a local directory, including its subdirectories, add `recursive=True`:

  ```python
  fs.put("./waveforms/2024/", f"{SCRATCH_BUCKET}/waveforms/2024/", recursive=True)
  ```

- To list objects in the scratch bucket (like `aws s3 ls`, `ls` shows one level, with deeper keys listed as folders):

  ```python
  fs.ls(f"{SCRATCH_BUCKET}/waveforms/2024/")
  ```

- To list every object under a prefix with its full key, use `find`. `du` gives the total size in bytes, like `--summarize`:

  ```python
  for key, info in fs.find(f"{SCRATCH_BUCKET}/waveforms/", detail=True).items():
      print(f"{info['size'] / 1024**2:8.2f} MiB  {key}")

  print(f"Total: {fs.du(f'{SCRATCH_BUCKET}/waveforms/') / 1024**2:.2f} MiB")
  ```

- To copy an object from the scratch bucket to a data directory in the file system:

  ```python
  fs.get(f"{SCRATCH_BUCKET}/waveforms/2024/IU.ANMO.mseed", "/tmp/data/")
  ```

- To copy every object under a prefix to a local directory, add `recursive=True`:

  ```python
  fs.get(f"{SCRATCH_BUCKET}/waveforms/2024/", "/tmp/data/2024/", recursive=True)
  ```

- To read an object without downloading it first, open it like a local file with `fsspec.open`:

  ```python
  with fsspec.open(f"{SCRATCH_BUCKET}/waveforms/2024/IU.ANMO.mseed", "rb") as f:
      data = f.read()
  ```

- To delete an object from the scratch bucket:

  ```python
  fs.rm(f"{SCRATCH_BUCKET}/waveforms/2024/IU.ANMO.mseed")
  ```

- To delete every object under a prefix, add `recursive=True`. fsspec has no dry-run option, so list what's there with `find` first. As with the CLI, always include the trailing slash:

  ```python
  prefix = f"{SCRATCH_BUCKET}/waveforms/2024/"
  print(fs.find(prefix))           # preview what will be deleted
  fs.rm(prefix, recursive=True)
  ```

- To move an object to another prefix:

  ```python
  fs.mv(f"{SCRATCH_BUCKET}/waveforms/2024/IU.ANMO.mseed",
        f"{SCRATCH_BUCKET}/archive/waveforms/2024/IU.ANMO.mseed")
  ```

- To move every object under a prefix to another prefix, move each object found under the source prefix, keeping the rest of its key. As with the CLI, each object is copied and then deleted. Avoid `fs.mv(..., recursive=True)` here: it also tries to copy the "folders", which don't exist as objects in S3, so it fails partway and leaves the original objects in place:

  ```python
  source_prefix = f"{SCRATCH_BUCKET}/waveforms/2024/"
  target_prefix = f"{SCRATCH_BUCKET}/archive/waveforms/2024/"

  for key in fs.find(source_prefix):   # keys come back without "s3://"
      fs.mv(f"s3://{key}", target_prefix + key.removeprefix(source_prefix.removeprefix("s3://")))
  ```

- fsspec has no `sync` command. This function uploads only the local files that are missing from the prefix or whose size has changed. It compares sizes only, not modification times, so an edited file that stays the same size isn't uploaded again:

  ```python
  def sync_up(local_dir, s3_prefix, pattern="*"):
      """Upload new or changed files from local_dir to s3_prefix."""
      local_dir = Path(local_dir).expanduser()
      s3_prefix = s3_prefix.rstrip("/") + "/"
      # Sizes of objects already in the bucket, keyed by path relative to the prefix
      remote = {
          key.removeprefix(s3_prefix.removeprefix("s3://")): info["size"]
          for key, info in fs.find(s3_prefix, detail=True).items()
      }
      for path in local_dir.rglob(pattern):
          if not path.is_file():
              continue
          rel = path.relative_to(local_dir).as_posix()
          if remote.get(rel) != path.stat().st_size:
              print(f"upload: {rel}")
              fs.put(str(path), s3_prefix + rel)

  sync_up("./waveforms/2024/", f"{SCRATCH_BUCKET}/waveforms/2024/")

  # Sync only miniSEED files, like --exclude "*" --include "*.mseed"
  sync_up("./waveforms/2024/", f"{SCRATCH_BUCKET}/waveforms/2024/", pattern="*.mseed")
  ```

## Using Your Own S3 Bucket in us-east-2

Objects in the scratch bucket are deleted 7 days after they were last created or modified. To keep results, or to share them with collaborators, you can store them in a bucket in your own AWS account. Note that you pay for storage; however, there are AWS programs for research and a Free Tier for trying out AWS services. See [Cloud Resources for Students and Researchers](#cloud-resources-for-students-and-researchers) to learn about these programs.

GeoLab runs in the AWS **us-east-2** (Ohio) region, and it only lets you move data to buckets in us-east-2. AWS charges for data transferred between regions and out to the internet, but not for data moved between S3 and compute in the same region. Keeping your bucket in us-east-2 means copying data between GeoLab and your bucket doesn't add transfer costs.

Your AWS account still pays for some things, because AWS bills the bucket owner:

- **Storage.** You pay for the data stored in your bucket.
- **Requests.** You pay a small fee for each request on your bucket, such as an upload, download, or listing, including requests made from GeoLab. The fee is per request, not per byte: reading a 1 KB file and a 1 GB file into GeoLab each count as one download request. Moving the data itself between your bucket and GeoLab is free because both are in us-east-2. See the **Storage and Requests** tab on the [Amazon S3 pricing page](https://aws.amazon.com/s3/pricing/) for current rates.
- **Downloads outside AWS.** You pay a transfer fee when data leaves AWS, for example when you download results from your bucket to your laptop. See the **Data transfer** tab on the [Amazon S3 pricing page](https://aws.amazon.com/s3/pricing/) for current rates.

The NSF National Geophysical Facility covers the cost of GeoLab servers and the scratch bucket, so using the scratch bucket is free. To keep GeoLab free for all users, EarthScope does not offer persistent buckets.

### Adding Your Own AWS Credentials

The credentials your GeoLab server comes with only give access to the scratch bucket. To use a bucket in your own AWS account, you need to add credentials for that account.

1. In the [IAM console](https://console.aws.amazon.com/iam/) for your AWS account, create an access key for an IAM user. Give that user permission to use only the buckets it needs. Don't create access keys for your account's root user.
2. In a GeoLab Terminal, save the key as a named **profile**. A profile keeps these credentials separate from the scratch bucket's credentials:

   ```bash
   aws configure --profile my-aws
   ```

   Enter your access key ID and secret access key when prompted, `us-east-2` for the default region, and `json` for the output format. The CLI saves them in `~/.aws/credentials` and `~/.aws/config` in your home directory.

3. Add `--profile my-aws` to any command that uses your own bucket. Commands without it keep using the scratch bucket's credentials:

   ```bash
   aws s3 ls s3://my-geolab-results/ --profile my-aws
   ```

   In Python, pass the profile when you create the file system:

   ```python
   my_fs = fsspec.filesystem("s3", profile="my-aws")
   my_fs.ls("s3://my-geolab-results/")
   ```

:::{warning}
Treat the access key like a password. Don't paste it into a notebook or script, and don't commit `~/.aws/` to a Git repository. If a key is exposed, deactivate it in the IAM console and create a new one.
:::

### Creating the Bucket

You can create the bucket in the [S3 console](https://console.aws.amazon.com/s3/): choose **Create bucket** and set **AWS Region** to **US East (Ohio) us-east-2**. Or create it with the AWS CLI, using your profile:

```bash
aws s3 mb s3://my-geolab-results --region us-east-2 --profile my-aws
```

Bucket names must be unique across all of AWS, 3 to 63 characters long, and use only lowercase letters, numbers, hyphens, and dots. New buckets block public access by default.

If you create the bucket with `aws s3api create-bucket` instead, any region other than us-east-1 must also be given as a location constraint:

```bash
aws s3api create-bucket --bucket my-geolab-results --region us-east-2 \
    --create-bucket-configuration LocationConstraint=us-east-2 --profile my-aws
```

To check the region of an existing bucket:

```bash
aws s3api get-bucket-location --bucket my-geolab-results --profile my-aws
```

The output should show `"LocationConstraint": "us-east-2"`. If your bucket is in any other region, GeoLab won't move data to it, so create a new bucket in us-east-2 instead.

### Copying Files From Scratch to Your S3 Bucket

A single command can't copy directly from the scratch bucket to your own bucket, because each command uses only one set of credentials. Copy from the scratch bucket to `/tmp` first, then upload to your bucket with your profile. `/tmp` is fast but is cleared when your server stops, so upload before you log out:

```bash
aws s3 cp $SCRATCH_BUCKET/results/ /tmp/results/ --recursive
aws s3 cp /tmp/results/ s3://my-geolab-results/results/ --recursive --profile my-aws
```

### Example: Backing Up a Private Dataset and Using It in GeoLab

If you have a collection of miniSEED files on an external hard drive, you can upload it to your bucket. You can use it as a backup and read it from GeoLab whenever you need it. Unlike the scratch bucket, your data isn't deleted automatically.

1. On your own computer, install the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and save your access key as a profile, the same way as in GeoLab:

   ```bash
   aws configure --profile my-aws
   ```

2. Upload the dataset from the drive to your bucket. On macOS, external drives appear under `/Volumes`; on Windows they have a drive letter, such as `E:\`. Use `sync` rather than `cp --recursive`, so that if the upload is interrupted, rerunning the command only uploads the files that are still missing:

   ```bash
   aws s3 sync /Volumes/MyDrive/my-dataset/ s3://my-geolab-results/my-dataset/ --profile my-aws
   ```

   Uploading into AWS has no transfer fee, but each file is one request. If the dataset has many small files, consider combining them into fewer, larger files first.

3. In GeoLab, list the dataset to check that it arrived:

   ```bash
   aws s3 ls s3://my-geolab-results/my-dataset/ --recursive --human-readable --summarize --profile my-aws
   ```

4. Read the files directly from your bucket in Python. You don't need to copy them to your home directory or to the scratch bucket first:

   ```python
   import fsspec
   from obspy import read

   my_fs = fsspec.filesystem("s3", profile="my-aws")

   for key in my_fs.glob("s3://my-geolab-results/my-dataset/**/*.mseed"):
       with my_fs.open(f"s3://{key}", "rb") as f:
           st = read(f)
       print(st)
   ```

   Each file you read is one download request on your bucket. Because your bucket and GeoLab are both in us-east-2, there's no transfer fee, however large the files are.

5. If you add or change files on the drive later, rerun the `sync` command from step 2 on your computer. It uploads only the new and changed files.

## Cloud Resources for Students and Researchers

You will need an AWS account to host your own S3 bucket which is billed to you. There are two AWS programs can help cover that cost.

### AWS Cloud Credit for Research

The [AWS Cloud Credit for Research](https://aws.amazon.com/government-education/research-and-technical-computing/cloud-credit-for-research/) program awards AWS Promotional Credit that you can use for AWS services, including S3 storage. It supports projects that:

- build publicly available, cloud-hosted applications, software, or tools for your research community;
- run proof-of-concept or benchmark tests of moving research workloads or open datasets to the cloud;
- train others to use the cloud for research through workshops or tutorials.

Program requirements and benefits:

- **Who can apply:** full-time faculty and full-time research staff at an accredited research institution, and graduate, postgraduate, or PhD students enrolled at one.
- **Award amounts:** amounts depend on your proposal and the usage it describes. Student awards are capped at $5,000; faculty and staff awards have no cap.
- **Review time:** applications are reviewed year-round, and a review typically takes 90 to 120 days. Expedited reviews aren't available, so apply well before you need the credit.
- **Limits:** for EC2, credit covers only On-Demand and Spot Instances. Credit expires, so plan your project timeline around its duration.
- **Contact:** messages about applications come from `aws-research-credit@amazon.com`. Check your spam folder.

To apply, use the [application form](https://pages.awscloud.com/aws-cloud-credit-for-research.html). Before applying, read the [FAQs](https://aws.amazon.com/government-education/research-and-technical-computing/cloud-credit-for-research/faqs/) and the [AWS Promotional Credit terms](https://aws.amazon.com/awscredits/). You can also browse [previous recipients](https://aws.amazon.com/government-education/research-and-technical-computing/cloud-credit-for-research/previous-recipients/) for examples of funded projects. If you're new to the cloud, [AWS Educate](https://aws.amazon.com/education/awseducate/) offers free training.

### AWS Free Tier

New AWS accounts can try AWS at no cost through the [AWS Free Tier](https://aws.amazon.com/free/). It's a way to test a personal bucket before committing to paying for one.

Benefits:

- **Credits:** new accounts get $100 in credits at sign-up and can earn up to $100 more by completing activities in the AWS console.
- **Free plan:** you aren't charged unless you upgrade to the Paid plan. Most new customers don't need a payment method to sign up.
- **Always Free services:** more than 30 AWS services are free up to monthly usage limits, on both the Free and Paid plans. See the [Free Tier page](https://aws.amazon.com/free/) for each service's limits.

Limitations:

- **New customers only:** you aren't eligible if you have or have had an AWS account. Credits can't be transferred.
- **Short duration:** the Free plan ends 6 months after you open the account, or sooner if your credits run out.
- **Data is deleted:** when the Free plan ends, AWS suspends the account. AWS keeps your data for 90 days and then permanently deletes the account and its contents unless you upgrade to the Paid plan. Don't keep data you need on a Free plan account.
- **Limited services:** the Free plan includes only some AWS services, and it's meant for evaluation, not for sensitive data.
- **Monthly limits:** Always Free allowances reset each month, and unused amounts don't carry over.
- **Charges on the Paid plan:** after you upgrade, standard rates apply to usage beyond your credits. Set a monthly spend limit to avoid surprise bills.

For details, see the [Free Tier FAQs](https://aws.amazon.com/free/free-tier-faqs/) and the [Free Tier terms](https://aws.amazon.com/free/terms/). The FAQs and terms disagree on when credits expire, so check the expiry date in your account's billing settings.
