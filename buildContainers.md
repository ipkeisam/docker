2. Create Docker Project
The whole lab needs to be run as root. Switch to the root user:

sudo -i
Create a directory in your file system called busyBox_project and change to it:

cd $HOME
mkdir busyBox_project
cd busyBox_project
In the busyBox_project directory, create a shell script called loveLXC.sh:

#!/bin/sh

# If command line parameter has been included, use it as our $ENTITY variable
if [ "$#" -eq 0 ]; then
   ENTITY="Red Hat"
else
   ENTITY=$@
fi

# If LOVE_LXC_DIR env var is not set, then set default value
if [ "x$LOVE_LXC_DIR" = "x" ]; then
   LOVE_LXC_DIR=/tmp
fi

OUTPUT_FILE="super_top_secret_log.log"

while true; do

  # Write to standard output
  echo "$ENTITY loves linux containers";

  # Write to output file
  echo "$ENTITY loves linux containers" >> $LOVE_LXC_DIR/$OUTPUT_FILE

  sleep 5s;
done
Make the shell script executable:

chmod 755 loveLXC.sh
Create a Dockerfile that instructs Docker to add your shell script to the BusyBox base image and execute that shell script on container startup:

Create a text file called Dockerfile in the busyBox_project directory.

Add the following contents to Dockerfile:

# Example of very simple Dockerfile based on minimalistic BusyBox base image
# Please read through each comment to gain a better understanding of what this Dockerfile does

# Docker will pull from docker.io if base image does not already exist in local Docker repo
FROM busybox

# Feel free to modify this
MAINTAINER "Red Hat Global Partner Technical Enablement <open-program@redhat.com>"

# The LABEL instruction adds metadata to an image. A LABEL is a key-value pair.
LABEL version="1.0" \
      description="This is our first project that creates a custom Docker image based on the extremely small BusyBox base image"

# Define two environment variables: NAME and LOVE_LXC_DIR
# Value of env variables can be modified at container startup using -e parameter
ENV NAME="My enterprise IT shop" \
    LOVE_LXC_DIR="/var/lovelxc"

# Docker is to execute all subsequent commands in this file as BusyBox's root user
USER root

# /usr/local/bin/ is included in the root user's $PATH
# copy our shell script to this directory
COPY loveLXC.sh /usr/local/bin/loveLXC.sh

# Ensure that shell script added to image is set as executable
RUN chmod 755 /usr/local/bin/loveLXC.sh

# Create directory and define it as a container mount point
# Our shell script will write output to local filesystem at this mount point
CMD mkdir $LOVE_LXC_DIR
VOLUME $LOVE_LXC_DIR

# At startup, this container should run a single command
# Our script uses the command line parameters passed as part of $NAME env var
CMD /usr/local/bin/loveLXC.sh $NAME
Read through each comment to better understand what this Dockerfile does.

Review the two simple text files of your completed Docker project:

ls -lF
Sample Output
total 16
-rw-r--r--  1 root  root  1521 Oct 11 14:26 Dockerfile
-rwxr-xr-x  1 root  root   516 Oct 11 14:25 loveLXC.sh*
2.1. Build Image
While still in the busyBox_project directory, build the Docker image:

docker build . --rm=true -t rhtgptetraining/lovelxc
By including . in the command, the Docker build happens in the current directory.
Sample Output
Sending build context to Docker daemon 4.608 kB
Step 1 : FROM busybox
latest: Pulling from library/busybox

56bec22e3559: Pull complete
Digest: sha256:29f5d56d12684887bdfa50dcd29fc31eea4aaf4ad3bec43daf19026a7ce69912
Status: Downloaded newer image for busybox:latest
 ---> e02e811dd08f
Step 2 : MAINTAINER "Red Hat Global Partner Technical Enablement <open-program@redhat.com>"
 ---> Running in c415a4dda97e
 ---> 2b202677d426
Removing intermediate container c415a4dda97e
Step 3 : LABEL version "1.0" description "This is our first project that creates a custom Docker image based on the extremely small BusyBox base image"
 ---> Running in c4175fdd8433
 ---> 1bf67c12bf45
Removing intermediate container c4175fdd8433
Step 4 : ENV NAME "My enterprise IT shop" LOVE_LXC_DIR "/var/lovelxc"
 ---> Running in 82aaff34bca8
 ---> 0eaa5efe0ba8
Removing intermediate container 82aaff34bca8
Step 5 : USER root
 ---> Running in 4616361a4938
 ---> 049da725c8bf
Removing intermediate container 4616361a4938
Step 6 : COPY loveLXC.sh /usr/local/bin/loveLXC.sh
 ---> 27d3e01f9123
Removing intermediate container ecc68f04ad58
Step 7 : RUN chmod 755 /usr/local/bin/loveLXC.sh
 ---> Running in 65bd02024843
 ---> 0c02c45ec58d
Removing intermediate container 65bd02024843
Step 8 : CMD mkdir $LOVE_LXC_DIR
 ---> Running in 3e93fdf3de49
 ---> e30766977bdf
Removing intermediate container 3e93fdf3de49
Step 9 : VOLUME $LOVE_LXC_DIR
 ---> Running in dd7e168462e0
 ---> cf1828ff010e
Removing intermediate container dd7e168462e0
Step 10 : CMD /usr/local/bin/loveLXC.sh $NAME
 ---> Running in bf66dc06d131
 ---> b35337a5761c
Removing intermediate container bf66dc06d131
Successfully built b35337a5761c
Note that Docker packages up the entire directory and sends it over to the Docker daemon. This means that you can build on a remote Docker server if necessary.

Execute docker images and note the presence of your new custom Docker image.

docker images
Sample Output
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
rhtgptetraining/lovelxc   latest              1eb5774495e0        37 seconds ago      1.15 MB
docker.io/busybox         latest              8c811b4aec35        6 weeks ago         1.15 MB
2.2. Run Container
In this section, you run a container based on your new custom Docker image.

Run a new container (without passing an environment variable) from your custom image:

docker run -d --name="gpte-lovelxc" rhtgptetraining/lovelxc
The --name parameter gives your running container a name. This way you can refer to the container by name rather than by container ID.

Tail the standard output of the container:

docker logs -f gpte-lovelxc
Review the standard output and press Ctrl+C when you are finished.

Delete the existing container and re-instantiate using an environment variable:

docker rm -f gpte-lovelxc
docker run -d --name="gpte-lovelxc" -e NAME="The world" rhtgptetraining/lovelxc
Again, tail the standard output:

docker logs -f gpte-lovelxc
Review the output of this new container and press Ctrl+C when finished.

3. Examine Mounted Volume
Containers are ephemeral. Any state is lost when the container stops. When a container is deleted, any data written to the container’s file system is deleted as well. In OpenShift, containers are never restarted. Instead, new containers are spun up to replace old containers when needed. The ramifications of this behavior are significant.

Because OpenShift creates new containers to replace old ones, persistent storage volumes mounted on containers are critical for maintaining state for files like configuration files and data files.

Recall that your simple Dockerfile defines a persistent volume, using the VOLUME directive. The volume defines a mount point to an external volume on the native host or other containers. Often in production environments, this volume is mapped to a mount point on a clustered file system.

In this final section of the lab, you look at the file written to your host operating system by using the simple shell script. The writing of this file is made possible by having defined a volume in Dockerfile with the VOLUME directive.

Execute the docker inspect command and retrieve the path on the host operating system to the container volume:

docker inspect -f '{{json .Mounts}}' gpte-lovelxc | jq
Sample Output
[
  {
    "Type": "volume",
    "Name": "81979527ae6c9c5af7a0dbae05e55346d1a21c6f2e54080fd358a6ad3faa7a3c",
    "Source": "/var/lib/docker/volumes/81979527ae6c9c5af7a0dbae05e55346d1a21c6f2e54080fd358a6ad3faa7a3c/_data",
    "Destination": "/var/lovelxc",
    "Driver": "local",
    "Mode": "",
    "RW": true,
    "Propagation": ""
  }
]
docker inspect returns the entire definition of the container image as a big JSON string. You can limit the output to just the volume mounts that you are interested in by using the {{json .Mounts}} filter. You can then use jq to make this output easier to read.
View the contents of the file written by the script to this external volume on the host operating system:

Make sure that you use the value of your particular source and do not copy/paste this example verbatim.
cat /var/lib/docker/volumes/81979527ae6c9c5af7a0dbae05e55346d1a21c6f2e54080fd358a6ad3faa7a3c/_data/super_top_secret_log.log  | more
Sample Output
The world loves linux containers
The world loves linux containers
The world loves linux containers
Delete the running container:

docker rm -f gpte-lovelxc
3.1. Specify Volume to Mount
When running a container, it is possible to specify a local directory to be mounted into a container. This way you do not have to inspect the image where your log file is written. But more importantly, this local directory persists even though containers may be deleted and re-created. In the OpenShift environment, the "directory" is provided dynamically by the OpenShift Container Platform and a developer simply requests a persistent volume claim.

Create a directory in your current project directory:

mkdir -p $HOME/busyBox_project/pv
Now run the container again, but specify the -v flag to tell Docker to use this directory for the volume:

docker run -d --name="gpte-lovelxc-persistent" -e NAME="Persistence" -v $HOME/busyBox_project/pv:/var/lovelxc rhtgptetraining/lovelxc
Observe that you are mapping the $HOME/busyBox_project/pv directory into the location /var/lovelxc in the container.

By doing this, everything written to /var/lovelxc in the container is instead written to $HOME/busyBox_project/pv.

Examine the contents of the directory:

ls $HOME/busyBox_project/pv
Display the contents of the log file in the pv directory and note that it now says Persistence loves linux containers:

tail $HOME/busyBox_project/pv/super_top_secret_log.log
Stop and remove the container:

docker rm -f gpte-lovelxc-persistent
Start a new container pointing to the same volume mount but with different parameters:

docker run -d --name="gpte-lovelxc-persistent" -e NAME="Repeat Persistence" -v $HOME/busyBox_project/pv:/var/lovelxc rhtgptetraining/lovelxc
Examine the log file and see that the new text is appended at the end of the existing log file rather than written in a whole new log file as before:

tail $HOME/busyBox_project/pv/super_top_secret_log.log
4. Clean Up Environment
Delete all running containers:

docker rm -f $(docker ps -aq)
Delete the image you built:

docker rmi rhtgptetraining/lovelxc
Delete the BusyBox base image:

docker rmi busybox
