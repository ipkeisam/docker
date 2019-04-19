2. Enable and start Docker
Docker is available in Linux distributions based on Red Hat Package Manager (RPM), such as CentOS, and Red Hat Enterprise Linux.

Since you are using the course client VM, Docker is already installed.

If you needed to install Docker you could do so as root: yum -y install docker.
The whole lab needs to be run as root. Switch to the root user:

sudo -i
After the installation Docker is inactive. Enable and start Docker as a service:

systemctl enable docker.service
systemctl start docker.service
3. Search for Images
Docker provides the ability to search for existing images. By default, Docker searches its publicly available cloud-based registry called Docker Hub. Docker Hub contains many base operating system images, which are often used as the base image on which additional functionality, such as a Java Runtime Environment, is layered. In addition to base OS images, many developers have contributed containerized applications. These containerized applications are layered on a base OS image, such as the centos image.

The Docker client can also be configured to search other Docker private registries. In this section of the lab, you begin by searching for base OS images.

Search Docker Hub for the base OS image for CentOS:

docker search centos
Sample Output
NAME     DESCRIPTION                    STARS OFFICIAL AUTOMATED
centos   The official build of CentOS.  3882  [OK]
Observe that you did not need to specify a server address or URL for the Docker daemon when using the docker command. By default the docker command connects to the Docker daemon on your local machine.
Search for images for Red Hat Enterprise Linux 7, which is the host operating system for the VM provided with this course:

docker search registry.access.redhat.com/rhel7.4
Sample Output
NAME      DESCRIPTION                                   STARS OFFICIAL AUTOMATED
rhel7.4   This platform image provides a minimal run... 0
Because this image requires a subscription, it is located in the Red Hat registry and not on Docker Hub.
Search for Wildfly images:

docker search wildfly
Sample Output
NAME           DESCRIPTION                       STARS OFFICIAL AUTOMATED
jboss/wildfly  WildFly application server image  353            [OK]
4. Download Image from Red Hat Registry
In order to run an image as a container, it needs to be downloaded first. This happens the first time a container is run. But sometimes it can be useful to download an image first to speed up the creation of a container. For example, if JBoss EAP 7 is the preferred runtime for applications in your organization, it makes sense to "pre-pull" the image to every OpenShift node that might run your application to speed up first runs.

Download the latest JBoss EAP 7.0 image:

docker pull registry.access.redhat.com/jboss-eap-7/eap70-openshift:latest
This image is about 700MB, so this may take a while.

Sample Output
latest: Pulling from jboss-eap-7/eap70-openshift
26e5ed6899db: Already exists
66dbe984a319: Already exists
873afb69ae61: Already exists
2e179c33b917: Already exists
fa72cea25fc3: Pull complete
05044da43e99: Pull complete
Digest: sha256:b34abeaef0ddf1eb03afdf37f24c08ab1bd6928074fbddff3d128e2b284244f7
Status: Downloaded newer image for registry.access.redhat.com/jboss-eap-7/eap70-openshift:latest
5. Manage BusyBox Container Life Cycle
In this section, you create a container from a publicly available Docker image, and run, view, stop, restart, inspect, and delete the container.

This section uses the freely available BusyBox Docker image. BusyBox is an open source project with a long history and dubbed by its authors as "the Swiss Army knife of Embedded Linux." The BusyBox Docker image includes all of the operating system tools required for this exercise, such as the Bourne shell, in a minuscule download of 1.5 MB.

BusyBox is not suitable for developing business applications. Instead you need tools provided by an enterprise Docker container, such as one based on Red Hat Enterprise Linux.
5.1. Run BusyBox Container
This section of the lab involves downloading the BusyBox image and running a container created from this image in your ocplab lab environment (or your local machine). The Docker installation includes a local Docker repository that stores images you download or create.

The BusyBox image is not included in your local Docker repository. It is, however, available from the Docker Hub registry.

To download the BusyBox image and run a container from it, execute the following command in your environment where Docker is installed:

docker run -d busybox /bin/sh -c "while true; do echo 'I love Linux containers'; sleep 5s; done"
Sample Output
Unable to find image 'busybox:latest' locally
latest: Pulling from library/busybox
0ffadd58f2a6: Pull complete
Digest: sha256:bbc3a03235220b170ba48a157dd097dd1379299370e1ed99ce976df0355d24f0
Status: Downloaded newer image for busybox:latest
0cdcea369824c03d2c0c7b3378ba702f84ac13825fd6040bd20529b3658b122b
Even with a slow network connection, expect the 1.5 MB download to complete quickly.
The BusyBox Docker image consists of two layers that are downloaded to your local Docker repository.

Once the BusyBox image has downloaded, Docker automatically starts a running container from the image. The last string of characters in the output is the ID of the newly created and running container.

Verify that the BusyBox-based container is running:

docker ps -a
Scan the output for an entry detailing the running BusyBox container:

Sample Output
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
ccef7ecc4fd2        busybox             "/bin/sh -c 'while tr"   13 seconds ago      Up 12 seconds                           adoring_mcclintock
The entry includes important information such as the CONTAINER ID and STATUS.

Make a note of the specific container ID (in the example above ccef7ecc4fd2) - you will need it for the next few steps.

5.2. View Console Log
Docker allows you to display logs from running or stopped containers. Any output directed to stdout is captured in a log file and can be viewed with the docker logs command.

View the logs (using the container ID from the output of the previous command):

docker logs -f <containerId>
Sample Output
I love Linux containers
I love Linux containers
I love Linux containers
I love Linux containers
I love Linux containers
I love Linux containers
The BusyBox container is executing a single task in the background.

When ready to continue, press Ctrl+C to return to the command prompt.

As an enterprise software developer, you can expect to be asked to containerize more sophisticated applications than that executed by the BusyBox container. Keep in mind, however, that the principle of containerizing a single operating system task still applies. While it is technically possible to run multiple services within a container such as supervisor, this is generally not considered a best practice.
5.3. Stop BusyBox Container
Stopping a running container requires knowing the container’s ID. You use the container’s ID to check the status of the container.

Check the status of your running containers:

docker ps -a
Stop the BusyBox container by substituting the container ID for <containerId>:

docker stop <containerId>
Check the status of the container and expect to see that it changed to Exited:

docker ps -a
5.4. Restart BusyBox Container
Previously, you created a new container from the BusyBox image with the docker run command. You can continue to use docker run to create more new containers from this image, or use the docker start command to restart an already existing container that is currently stopped.

Restart the previously created BusyBox container:

docker start <containerId>
Check the container’s status:

docker ps -a
5.5. Inspect BusyBox Container
You may want to know the state of your container including network settings, storage, startup command parameters, and environment variables passed to the container.

Inspect the running state of the container:

docker inspect <containerId> | more
Press Spacebar to advance through all of the details.
5.6. Inspect Changes to Container’s File System
You may also want to view a listing of the file changes that occurred in the container’s file system.

View the list of changes to the file system:

docker diff <containerId>
The BusyBox container includes very few (if any) changes to its file system.

Examine the following example, but do not execute the command, as it is for the purpose of illustration only:

Example
# docker diff hungry_swartz
C /run
A /run/secrets
C /tmp
A /tmp/hsperfdata_nexus
A /tmp/hsperfdata_nexus/1
A /tmp/jetty-0.0.0.0-8081-nexus-_-any-
A /tmp/outreach
A /tmp/outreach/c49c0150284bfb884aa5c0409f1ac31f0914568f
A /tmp/outreach/c49c0150284bfb884aa5c0409f1ac31f0914568f.metadata
A /tmp/outreach/d329f892a68d1d12b1be5584e05d5cd3432bce6f
A /tmp/outreach/d329f892a68d1d12b1be5584e05d5cd3432bce6f.metadata
A /tmp/nexus-ehcache
A /tmp/nexus-ehcache/shiro-activeSessionCache.data
"A" indicates a file added to the file system.

"D" indicates a file deleted from the file system.

"C" indicates a changed file.

This example provides an illustration of file system changes made to a Docker container that runs a Node.js application.

5.7. Delete BusyBox Container
If a container is no longer needed, it can be permanently deleted.

Delete the BusyBox container:

docker rm -f <containerId>
Purge all stopped containers:

docker rm $(docker ps -aq)
You may see an error message if all containers are already removed.

6. Manage Image Life Cycle
You use Docker to manage the life cycle of a Linux container on your workstation. In this section, you execute docker commands that are useful when working with images.

6.1. List Images in Local Repository
View the list of images in a local Docker repository:

docker images
Sample Output
REPOSITORY  TAG     IMAGE ID      CREATED      SIZE
busybox     latest  6ad733544a63  2 weeks ago  1.13 MB
centos      7       d123f4e55e12  2 weeks ago  197 MB
Note the image ID included in the entry for each image in the local repository.

6.2. Delete Image
As long as a container based on a particular image does not currently exist in your local Docker environment, that image can be deleted.

If a container based on the image that you want to delete does exist, delete the container first.
Delete the BusyBox image:

docker rmi busybox
Sample Output
Untagged: busybox:latest
Untagged: busybox@sha256:bbc3a03235220b170ba48a157dd097dd1379299370e1ed99ce976df0355d24f0
Deleted: sha256:6ad733544a6317992a6fac4eb19fe1df577d4dec7529efec28a5bd0edad0fd30
Deleted: sha256:0271b8eebde3fa9a6126b1f2335e170f902731ab4942f9f1914e77016540c7bb
