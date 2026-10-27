# Working with the S3 SCRATCH_BUCKET

## What is object storage?

Object storage keeps data as **objects** in a flat container called a **bucket**. Each object holds three things:

- **the data**, such as a miniSEED file, a CSV table, or an image file;
- **a key**, which is the object's unique name within the bucket, such as `waveforms/2024/IU.ANMO.mseed`;
- **metadata**, such as size, content type, and the time it was last modified.

You read and write objects over HTTP using an API. Amazon S3 (Simple Storage Service) is the most widely used object storage service, and many other providers support the same API. A full object address looks like `s3://bucket-name/key`.

## Prefixes: folders that aren't folders

A bucket has no directories. Every object sits at the same level, and its key is a single string. A key like `waveforms/2024/IU.ANMO.mseed` is not a file inside a `2024` folder inside a `waveforms` folder. It's one name that happens to contain slashes.

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
- **Prefixes spread the load.** S3 scales request throughput per prefix (thousands of reads and writes per second for each one), so spreading many parallel workers across several prefixes avoids throttling.

## How it differs from file storage

Your GeoLab home directory (`/home/jovyan`) is **file storage**: a disk attached to your server, organized as a tree of folders, where programs open, read, and edit files in place. Object storage works differently:

| | File storage (home directory, `/tmp`) | Object storage (S3) |
| --- | --- | --- |
| **Organization** | A hierarchy of directories | A flat bucket of keys. The `/` in a key only looks like a folder path; S3 has no real directories |
| **Access** | The operating system's file calls (`open`, `ls`, `cp`) | HTTP requests through an API (`aws s3`, `fsspec`, `boto3`, `s3fs`) |
| **Editing** | Change part of a file in place | Objects are replaced whole; to change one byte you rewrite the object |
| **Scale** | Limited by the size of the disk (50 GB for a GeoLab home directory) | Effectively unlimited; you pay for what you store |
| **Sharing** | Attached to one server | Readable by any machine with network access and permission, including many Dask workers at once |
| **Speed** | Low latency for small, frequent reads and writes | Higher latency per request, but high throughput for large or parallel reads |

## Using the SCRATCH_BUCKET in GeoLab

Object storage suits large datasets and parallel computing. Many workers can read different objects at the same time, and the data stays in the cloud next to the compute. That is why many public geoscience datasets are published in S3, and why GeoLab provides a temporary scratch bucket (`$SCRATCH_BUCKET`) for your own large intermediate results.

Two habits change when you work with object storage:

- **Prefer cloud-optimized formats.** Formats such as Zarr or Parquet split data into many objects, so you can read only the pieces you need instead of downloading a whole file.
- **Write once, read many.** Because objects can't be edited in place, write finished results rather than appending to a file over time.

**Objects in the scratch bucket are deleted automatically.** Each object is deleted 7 days after it was last created or modified, so each one expires on its own schedule. Because S3 objects can't be edited in place, "modified" means the object was rewritten, for example by uploading it again or by `cp`, `mv`, or `sync`. Reading or downloading an object doesn't reset its 7 days. The date shown by `aws s3 ls` is the date the 7 days count from. Copy anything you want to keep to your home directory or to [your own S3 bucket](#using-your-own-s3-bucket-in-us-east-2) before it expires.

## Using the AWS CLI

The AWS Command Line Interface (CLI) is Amazon's command-line tool for working with AWS services, including S3. It's installed in GeoLab, and your server is set up with credentials for the scratch bucket, so you can run it from a Terminal without any configuration. Run `which aws` to confirm it's available.

The CLI has two sets of S3 commands:

- **`aws s3`** provides high-level commands that feel like the file commands you already know: `ls`, `cp`, `mv`, `rm`, and `sync`. They take `s3://bucket/key` paths, upload large files in parallel parts automatically, and handle many objects at once with `--recursive`. Use these for everyday work.
- **`aws s3api`** maps one-to-one to the S3 API operations, such as `put-object`, `get-object`, and `list-objects-v2`. It takes the bucket and key as separate arguments and returns JSON, which is useful for scripting or for options the high-level commands don't expose.

`aws s3` is the recommended command to use because it mirrors file system Linux or UNIX file system commands.

> In GeoLab, `$SCRATCH_BUCKET` is an alias for a shared bucket plus your username. Use `echo` to see the full bucket path:
>
> ```bash
>echo $SCRATCH_BUCKET
>s3://earthscope-scratch/auth0|67e45f270c99cc5fb3358bd8
>```

### AWS CLI examples

The examples below use `aws s3`. For more commands and options, run `aws s3 help`.

- To add a file to the SCRATCH_BUCKET:

  ```bash
  aws s3 cp ./IU.ANMO.mseed $SCRATCH_BUCKET/waveforms/2024/IU.ANMO.mseed
  ```

- To add every file in a local directory, including its subdirectories, add `--recursive`. Each file's path relative to the directory becomes part of its key:

  ```bash
  aws s3 cp ./waveforms/2024/ $SCRATCH_BUCKET/waveforms/2024/ --recursive
  ```

- To list objects in the SCRATCH_BUCKET:

  ```bash
  aws s3 ls $SCRATCH_BUCKET/waveforms/2024/
  ```

- By default, `ls` shows one level and groups deeper keys as folders (`PRE 2024/`). To list every object under a prefix with its full key, add `--recursive`. Adding `--human-readable --summarize` shows sizes in KiB/MiB/GiB and ends with the total object count and size:

  ```bash
  aws s3 ls $SCRATCH_BUCKET/waveforms/ --recursive --human-readable --summarize
  ```

- To copy an object from the SCRATCH_BUCKET to a data directory in the file system:

  ```bash
  aws s3 cp $SCRATCH_BUCKET/waveforms/2024/IU.ANMO.mseed ~/data
  ```

- To copy every object under a prefix to a local directory, add `--recursive`. The CLI creates local subdirectories to match the rest of each key:

  ```bash
  aws s3 cp $SCRATCH_BUCKET/waveforms/2024/ ~/data/2024/ --recursive
  ```

- To delete an object from the SCRATCH_BUCKET:

  ```bash
  aws s3 rm $SCRATCH_BUCKET/waveforms/2024/IU.ANMO.mseed
  ```

- To delete every object under a prefix, add `--recursive`. There's no undo, so run it with `--dryrun` first to see what would be deleted:

  ```bash
  aws s3 rm $SCRATCH_BUCKET/waveforms/2024/ --recursive --dryrun
  aws s3 rm $SCRATCH_BUCKET/waveforms/2024/ --recursive
  ```

  Always include the full prefix and the trailing slash. `aws s3 rm $SCRATCH_BUCKET/waveforms/2024 --recursive` (no slash) also deletes keys that only start with that text, such as `waveforms/2024_old/...`. Because all GeoLab users share the scratch bucket's credentials, a mistyped prefix could delete other users' files.

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
  aws s3 sync $SCRATCH_BUCKET/waveforms/2024/ ~/data/2024/
  ```

  By default, `sync` never deletes anything at the destination. Add `--delete` to also remove files at the destination that no longer exist at the source, so the two match exactly. Preview it with `--dryrun` first, because it removes data:

  ```bash
  aws s3 sync ./waveforms/2024/ $SCRATCH_BUCKET/waveforms/2024/ --delete --dryrun
  ```

  You can limit what's synced with `--exclude` and `--include`. For example, to sync only miniSEED files:

  ```bash
  aws s3 sync ./waveforms/2024/ $SCRATCH_BUCKET/waveforms/2024/ --exclude "*" --include "*.mseed"
  ```

### Python Examples

[fsspec](https://filesystem-spec.readthedocs.io) provides one file-system interface for many kinds of storage, including local disks, S3, Google Cloud Storage, and HTTP.

fsspec takes the same `s3://` paths as the CLI, so `$SCRATCH_BUCKET` works as-is, and it has a `recursive` option for working with a whole prefix. Many scientific Python libraries, including xarray, pandas, and Dask, use fsspec to read `s3://` paths directly.

Create a file system object and read the scratch bucket path from the environment:

```python
import os
from pathlib import Path
import fsspec

fs = fsspec.filesystem("s3")   # uses the GeoLab server's credentials
SCRATCH_BUCKET = os.environ["SCRATCH_BUCKET"]
```

- To add a file to the SCRATCH_BUCKET:

  ```python
  fs.put("./IU.ANMO.mseed", f"{SCRATCH_BUCKET}/waveforms/2024/IU.ANMO.mseed")
  ```

- To add every file in a local directory, including its subdirectories, add `recursive=True`:

  ```python
  fs.put("./waveforms/2024/", f"{SCRATCH_BUCKET}/waveforms/2024/", recursive=True)
  ```

- To list objects in the SCRATCH_BUCKET. Like `aws s3 ls`, `ls` shows one level, with deeper keys listed as folders:

  ```python
  fs.ls(f"{SCRATCH_BUCKET}/waveforms/2024/")
  ```

- To list every object under a prefix with its full key, use `find`. `du` gives the total size in bytes, like `--summarize`:

  ```python
  for key, info in fs.find(f"{SCRATCH_BUCKET}/waveforms/", detail=True).items():
      print(f"{info['size'] / 1024**2:8.2f} MiB  {key}")

  print(f"Total: {fs.du(f'{SCRATCH_BUCKET}/waveforms/') / 1024**2:.2f} MiB")
  ```

- To copy an object from the SCRATCH_BUCKET to a data directory in the file system:

  ```python
  fs.get(f"{SCRATCH_BUCKET}/waveforms/2024/IU.ANMO.mseed", os.path.expanduser("~/data/"))
  ```

- To copy every object under a prefix to a local directory, add `recursive=True`:

  ```python
  fs.get(f"{SCRATCH_BUCKET}/waveforms/2024/", os.path.expanduser("~/data/2024/"), recursive=True)
  ```

- To read an object without downloading it first, open it like a local file with `fsspec.open`:

  ```python
  with fsspec.open(f"{SCRATCH_BUCKET}/waveforms/2024/IU.ANMO.mseed", "rb") as f:
      data = f.read()
  ```

- To delete an object from the SCRATCH_BUCKET:

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

Objects in the scratch bucket are deleted 7 days after they were last created or modified. To keep results, or to share them with collaborators, store them in a bucket in your own AWS account.

GeoLab runs in the AWS **us-east-2** (Ohio) region, and it only lets you move data to buckets in us-east-2. AWS charges for data transferred between regions and out to the internet, but not for data moved between S3 and compute in the same region. Keeping your bucket in us-east-2 means copying data between GeoLab and your bucket doesn't add transfer costs.

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

Treat the access key like a password. Don't paste it into a notebook or script, and don't commit `~/.aws/` to a Git repository. If a key is exposed, deactivate it in the IAM console and create a new one.

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

A single command can't copy directly from the scratch bucket to your own bucket, because each command uses only one set of credentials. Copy from the scratch bucket first to the filesystem, then upload with your profile to the bucket:

```bash
aws s3 cp $SCRATCH_BUCKET/results/ ~/results/ --recursive
aws s3 cp ~/results/ s3://my-geolab-results/results/ --recursive --profile my-aws
```
