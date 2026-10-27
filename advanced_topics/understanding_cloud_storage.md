# Understanding Cloud Storage in GeoLab

GeoLab uses AWS' Simple Storage Service (S3) and Elastic File System (EFS).

# Types of Storage in GeoLab

## Object Storage

In this case, the company's data lake requires scalability, security, durability, and low latency. Object storage is ideal for any application with these requirements for several reasons. First, object storage uses a flat address space that allows it to grow as more data is added. Second, object storage uses extensible metadata, which allows cloud providers to implement security and compliance models to protect the data. Third, data is written across multiple nodes for redundancy, meaning that if one node is dropped, there are copies of data, making object storage durable. Finally, object storage supports the transfer of data at 100 gbps within the same region.

## Elastic File System Storage

File storage is the best storage solution in this case, offering a hierarchical system of files and directories that the majority of software currently support. File storage is ideal for common office workloads because employees are familiar with file storage, which works like a desktop computer's file system or NAS. Unlike local file systems, however, cloud file storage is a fully managed solution deployable at a global scale. Cloud file storage is available in all time zones and built on top of block storage, which provides the durability, scalability, and low latency required by business operations. File sharing protocols, such as SMB and NFS, are included with cloud file systems, and applications natively connect to cloud storage.
But the hierarchical design of file storage places limitations on the size of files, the number of files stored, and directory depth, because exceeding those limits would degrade performance and increase latency. Files systems scale horizontally by adding more systems to increase storage. This differs from block or object storage, which scale vertically by adding resources to an instance.