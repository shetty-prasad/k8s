
# Volumes in Docker 

When Docker is installed, it organizes data within the **/var/lib/docker** directory. This folder contains several subdirectories such as aufs, containers, images, and volumes. Each subdirectory serves a specific role in Docker’s architecture:

**containers**: Stores all files related to running containers.
**images**: Contains stored images.
**volumes**: Holds data for persistent storage created by containers.

A Docker image consists of several read-only layers:

**Base Layer**: The Ubuntu operating system.
**Packages Layer**: APT packages installed on top of Ubuntu.
**Dependencies Layer**: Python packages such as Flask.
**Source Code Layer**: Your application code included in the image.
**Entry Point Layer**: The layer that sets the container’s entry point.

* Running a container from this image creates a new writable layer on top, which stores changes such as logs, temporary files, or modifications made during runtime. 
* This mechanism is known as copy-on-write. 
* Even if you modify a read-only file from the image, Docker creates a separate copy in the writable layer before applying the changes.

<img width="528" height="330" alt="image" src="https://github.com/user-attachments/assets/aafb7ea6-7b70-48ec-9a65-f35b3669855d" />

**Remember, when you remove a container, its writable layer and any associated changes will be lost. The original image remains unchanged unless it is rebuilt.**

**Persisting Data with Volumes**

For data persistence outside the container’s ephemeral writable layer—such as database storage—use Docker volumes.

**Creating and Using Volumes**

First, create a volume:
```
docker volume create data_volume
```

Then, mount the created volume when launching a container. For example, to store MySQL data in the volume, run:
```
docker run -v data_volume:/var/lib/mysql mysql
```

If a specified volume does not exist, Docker automatically creates it. To inspect the volumes, you can list the contents of **/var/lib/docker/volumes**.

**Bind Mounts**
Alternatively, if you prefer using external storage (for instance, storing database files in /data/mysql on your host), you can use bind mounts:
```
docker run -v /data/mysql:/var/lib/mysql mysql
```

# Volumes in Kubernetes



