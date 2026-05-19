# File Systems and Data Storage

GeoLab offers several file storage options described below. These storage locations are for storing data processing results, intermediate data products, notebooks, etc. These locations are not for long term storage. We strongly recommend using data in EarthScope repositories directly through programmatic access without making a copy.

## User home directories (aka folders)

Your username is ``jovyan`` (a resident of the Jupyter ecosystem), and your home directory is ``/home/jovyan``.
This is the same for all users, but no one else can see or access the files in *your* home directory.

``/home/jovyan`` is a persistent network-attached drive. Any files you put there will be there when you
log out and log back into the JupyterHub.

The easiest way to move files in and out of your home directory is via the JupyterLab web interface.
Drag a file into the file browser to upload, and right-click to download back out.
You can also open a terminal via the JupyterLab launcher and use this to ssh / scp / ftp to remote systems.

The ``/home/jovyan`` space is intended only for notebooks and code. It is **not** an appropriate place to store or manipulate
datasets, as it can get really expensive (and slow) when used that way. If you must save data to long-term storage, do it in small batches and proactively delete the data once finished with your analysis. 

Please note the following storage limits on ``/home/jovyan``:

* Store up to a maximum of 50 GB
* Accounts that have not accessed GeoLab for a period of six months may have all contents in their home directory removed.

To see how much disk space you are using, use the command below from any Terminal window:

`du -sh --exclude="shared*" ~/`
Alternatively, you can naviate to File -> Hub Control Panel, select the 'Usage' tab from the top right, and explore your storage as a [Quota-enforced](./quotas.md) resource.

If you exceed the 50 GB limit, your current session will remain active but your processes will not be able to write to storage. This may result in your server instance crashing. To remedy this, start another server instance and immediately reduce your disk usage. If you are unable to restart a server after multiple attempts, please reach out to the GeoLab admins by sending an email to help@earthscope.org for assistance.

If your account is inactive and data are removed from your home directory, this does not affect your access to GeoLab or your EarthScope User account. If you log into GeoLab after a period of inactivity and subsequent data removal, you will start with an empty home directory.

## The `shared` storage

The `shared` storage is a read-only file directory. This storage is located at:
* `shared` in the File Browser, and
* `/home/jovyan/shared/` on the file system

Shared storage is useful for common notebooks and (smallish) data sets for workshops or group exercises.  Contact us at help@earthscope.org if you would like us to consider temporarily adding files to this location.

## The `/tmp` directory
Any directory in Github outside of /home/jovyan is ephemeral. This means if you add data or scripts under a writeable directory like /tmp/myfile.txt _it will not be there when you log out and log back in._

Nevertheless, /tmp is a convenient location for storing data temporarily because it is a fast SSD drive. The space available depends on your server but will generally be much larger than /home/jovyan (50-100s of GB). We recommend using `/tmp` for intermediate datasets. 

## The `scratch` bucket storage
Through GeoLab, you have access to a temporary `scratch` bucket. This is an AWS S3 cloud storage bucket designed for temporary storage of large datasets, or data which you need to access over a short period of time but across more than one session. Data in scratch buckets will be automatically deleted after 7 days.

You can check the name of your scratch bucket by opening a Terminal in your hub and running the command:

```bash
$ echo $SCRATCH_BUCKET
s3://2i2c-aws-us-scratch-showcase/<username>
```

There are many ways to interact with S3 buckets, e.g., through the command line, or with python libraries like boto3. See [2i2c's Cloud Object Storage Documentation](https://docs.2i2c.org/user/data/object-storage/) for more details on getting started. 
 _Please note that to keep GeoLab free for users, EarthScope does not offer persistent buckets._ See below for details on bringing your own data bucket. 

## Bring your own S3 bucket
Users can create their own S3 bucket and use it from GeoLab. If you want to create your own bucket we recommend setting it up in the `us-east-2` AWS region, which is where GeoLab operates, to minimize transfer latency and costs. More details on configuring an AWS account can be found here: https://aws.amazon.com/pm/serv-s3/ 
