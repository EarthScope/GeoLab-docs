# Glossary of Cloud Computing Terms

**[bucket](https://cloudian.com/blog/object-storage-care/)**
In this context, the term "bucket" is used to denote any general cloud-based object storage system. A bucket usually has a "policy" that determines who can access the bucket, and what actions you can perform (list, read, copy, etc) on the bucket and its contents. Buckets have many features that make them distinct from file-based / directory storage system.

**[binder](https://mybinder.org)**
A free service for deploying reproducible compute environments.

**[conda](https://docs.conda.io)**
A package and environment management system that can be used for any software language, including Python, R, Java, JavaScript, FORTRAN, etc

**container**
Light, standalone, executable bundles of software that include code, system tools, and libraries. "Container" is often used interchangeably with "image;" the difference is that the image is the template, whereas the container is a live, running instance of that template.

**CPU**
Central Processing Unit: the thing that does the computational work

**Dask**
A flexible open-source python library for parallel computing

**[Docker](https://www.docker.com)**
Docker is a widely used platform that allows developers to package applications and their dependencies together into portable, isolated packages called containers.

**[EFS](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html)**
The Amazon Elastic File Storage (EFS) is an AWS service that offers scalable, durable file storage. This is the default storage type associated with everyone's home directory in GeoLab.

**ephemeral**
Temporary, non-persistent resources that disappear at the end of your session. Examples include temporary data storage, and the code environments (installed software) on a remote machine. 
When working in the cloud, think of ephemeral resources as being 'rented' -- they are configured to your specifications when you request them, and then all software, data, etc are deleted when you return the resource at the end of your session. Many cloud systems are built on ephemeral resources, because you only "pay" for resources while you're using them. This is ideal for intermediate copies of datasets, or large compute resources you may only need for a brief analysis (such as GPU) 

**[git](https://git-scm.com)**
A widely used version control system used by many open-source projects to manage and share their code base. See [Using Git](../getting_started/using_git.md).

**[GitHub](https://github.com)**
A web-based platform that enables software developers to remotely collaborate on software development using the git command line tool. GitHub is a huge resource for open-source collaboration where users can store, manage, and contribute to not only their own projects, but any public-facing project on the platform. See [Using Git](../getting_started/using_git.md)

**GPU**
Graphical Processing Unit; a specialized circuit designed specifically to speed computer graphics and image processing. Many parallelized compute operations and machine learning tasks can be executed faster on GPU vs CPU. 

**instance**
The virtual server or isolated machine running on a cloud provider's physical hardware. Instances are ephemeral and independent, meaning you can create or terminate them on-demand. 

**image**
A standardized, static, and immutable snapshot of a virtual machine's (VM) operating system, applications, configuration, and data, stored as a single file. It serves as a blueprint for deploying new server instances quickly, functioning as a "template" that ensures consistency across multiple virtual machine deployments. "Image" is often used interchangeably with "container;" the difference is that the image is the template, whereas the container is a live, running instance of that template.

**[Jupyter Book](https://jupyterbook.org/intro.html)**
Jupyter Book is a command-line tool that allows users to create publication-quality PDFs or web-based interactive books from Markdown documents and Jupyter Notebooks. This documentation site is built using MyST.

**[Jupyter Notebook](https://jupyter-notebook-beginner-guide.readthedocs.io/en/latest/what_is_jupyter.html)**
A Jupyter Notebook is a human-readable document that allows users to create and share reproducible workflows. The notebook may contain some combination of code (typically Python), equations, images, or narrative text that illustrate the data processing steps that were taken.

**[JupyterHub](https://jupyterhub.readthedocs.io)**
Developed by the open-source Jupyter community, JupyterHub allows you to deploy an application that provides remote data science environments to different users.

**kernel**
A kernel is the core, foundational component of a computer's operating system that acts as the primary interface between software applications and physical hardware. It holds complete control over system resources—such as CPU and memory. It manages processes and allocates memory to different applications so they don't interfere with one another. In a Jupyter Notebook, the kernel stores all variables and functions you've defined. Restarting the kernel clears that memory to give you a fresh slate to start from.

**[S3](https://aws.amazon.com/s3/)**
The Amazon Simple Storage Service (S3) is the AWS version of object storage we most commonly use. Usually a more cost-effective option than EFS.

**server**
A server is a computer or software program that provides data, resources, or services to other computers (called "clients") over a network