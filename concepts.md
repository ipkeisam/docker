1. Containers and Images
The basic unit of OpenShift applications is the container. A Linux container provides a lightweight mechanism for isolating running processes so that they interact only with their designated resources. Many application instances can be running in containers on a single host without visibility into each other’s processes, files, networks, and other resources. Typically, each container provides a single service, often called a microservice. Examples include a web server or a database, although containers can be used for arbitrary workloads.

While the Linux kernel has provided support for container technologies for years, the Docker project more recently has developed a convenient management interface for Linux containers on a host. OpenShift and Kubernetes add the ability to orchestrate Docker containers across multiple hosts.

Docker containers are based on Docker images. A Docker image is a binary file that includes all of the requirements for running a single Docker container, as well as metadata describing its needs and capabilities. Think of it as a packaging technology. Docker containers have access only to the resources defined in the image, unless you give the container additional access when creating it. By deploying the same image in multiple containers across multiple hosts and load-balancing between them, OpenShift provides redundancy and horizontal scaling for a service packaged into an image.

You can use Docker directly to build images. OpenShift also supplies builders that assist with creating an image by adding your code or configuration to existing images.

2. Pods and Services
OpenShift leverages the Kubernetes concept of a pod. A pod is a collection of one or more containers deployed together on one host, and the smallest compute unit that can be defined, deployed, and managed.

Pods are the rough equivalent of OpenShift gears, with containers the rough equivalent of cartridge instances. Each pod is allocated its own internal IP address, therefore owning its entire port space, and containers within pods can share their local storage and networking.

Pods have a life cycle: They are defined, then assigned to run on a node, and then run until their containers exit or they are removed for some other reason. Pods, depending on policy and exit code, may be removed after their containers exit, or they may be retained to enable access to their containers' logs.

OpenShift treats pods as largely immutable—changes cannot be made to a pod definition while the pod is running. OpenShift implements changes by terminating an existing pod and re-creating it with modified configurations or base images. Pods are also treated as expendable, and do not maintain state when destroyed or re-created. Therefore, pods are usually managed by higher-level controllers rather than directly by users.

A Kubernetes service acts as an internal load balancer. It identifies a set of replicated pods in order to proxy the connections it receives to them. Backing pods can be added to or removed from a service arbitrarily while the service remains available, enabling anything that depends on the service to refer to it at a consistent internal address.

Services are assigned an IP address and port pair that, when accessed, proxy to an appropriate backing pod. A service uses a label selector to find all of the running containers that provide a certain network service on a certain port.

3. Projects and Users
A project is a Kubernetes namespace with additional annotations, and is the central vehicle for managing regular users' access to resources. A project allows a community of users to organize and manage their content in isolation from other communities. Users must be given access to projects by administrators—unless they are given permission to create projects, in which case they automatically have access to their own projects.

Projects can have separate name, displayName, and description attributes:

The mandatory name is a unique identifier for the project.

The optional displayName is how the project is displayed in the web console. It defaults to the same value as name.

The optional description is a more detailed description of the project and is also visible in the web console.

Interaction with OpenShift is associated with a user. An OpenShift user object represents an actor. The actor may be granted permissions in the system by assigning roles to the user or any of the user’s groups.

A number of types of users exist:

Regular users: The way most interactive OpenShift users are represented. Regular users are created automatically in the system upon first login, or created using the API. Regular users are represented with the User object, such as joe or alice.

System users: Many of these are created automatically when the infrastructure is defined, mainly for the purpose of enabling the infrastructure to interact securely with the API. They include a cluster administrator (with access to everything), a per-node user, users for use by routers and registries, and various other users. There is also an anonymous system user that is used by default for unauthenticated requests. Examples of system users include system:admin, system:openshift-registry, and system:node:node1.example.com.

Service accounts: These are special system users associated with projects. Some are created automatically when the project is first created. More can be created by project administrators for the purpose of defining access to the contents of each project. Service accounts are represented with the ServiceAccount object. Examples include: system:serviceaccount:default:deployer and system:serviceaccount:foo:builder.

Every user must authenticate to access OpenShift. API requests with missing or invalid authentication are treated as requests from the anonymous system user. Once authenticated, policy determines what the user is authorized to do.

4. Builds
A build is the process of transforming input parameters into an object. Most often, the process is used to transform source code into a runnable image. A BuildConfig object is the definition of the entire build process.

The OpenShift build system provides extensible support for build strategies that are based on selectable types specified in the build API. There are three build strategies available:

Docker build: Allows for creation of a build from a Dockerfile.

Source-to-Image (S2I) build: Allows for creation of a build from application source code using a builder image for common languages like Java, PHP, Ruby or Python.

Custom build

Docker builds and S2I builds are supported by default.

The resulting object of a build depends on the builder used to create it. For Docker and S2I builds, the resulting objects are runnable images. For custom builds, the resulting objects are whatever the builder image author has specified.

5. Image Streams
An image stream is similar to a Docker image repository in that it contains one or more Docker images identified by tags. An image stream presents a single virtual view of related images, as it may contain images from:

Its own image repository in OpenShift’s integrated Docker registry

Other image streams

Docker image repositories from external registries

OpenShift stores complete metadata about each image, including example command, entry point, and environment variables. Images in OpenShift are immutable.

OpenShift components such as builds and deployments can watch an image stream, receive notifications when new images are added, and then, for example, react by performing a build or a deployment.

6. Templates
A template describes a set of objects that can be parameterized and processed by OpenShift. The objects include anything that users have permission to create within a project—for example, services, build configurations, and deployment configurations. A template may also define a set of labels to apply to every object defined in the template.

The set of objects defined in a template collectively define a desired state. Desired state is an important concept in the Kubernetes/OpenShift model. It is Kubernetes/OpenShift’s responsibility to make sure that the current state matches the desired state.

A template contains a set of definitions and parameters for the objects to be created together. For example, an application might consist of a front-end web application backed by a database. Each consists of a service object and deployment configuration object, and they share a set of credentials (parameters) for the front end to authenticate to the back end. Specifying parameters in a template allows all of the objects instantiated by that template to see consistent values for these parameters when the template is processed. Template parameters can be specified directly or generated automatically. An example of an automatically generated template parameter is a unique database password.

Templates can be processed from a definition in a file or from an existing OpenShift API object. Cluster administrators can define standard templates in the API that are available for all users to process. Users can define their own templates within their own projects.

Administrators and developers can interact with templates using the CLI and web console.

7. Deployments
A deployment in OpenShift is a replication controller based on a user-defined template called a deployment configuration. Deployments are created manually or in response to triggered events.

The deployment system provides:

A deployment configuration, which is a template for deployments

Triggers that drive automated deployments in response to events

User-customizable strategies to transition from the previous deployment to the new deployment

Rollbacks to a previous deployment

Manual replication scaling

The deployment configuration contains a version number that is incremented each time a new deployment is created from that configuration. In addition, a description of the reason for the last deployment is added to the configuration.

8. Routes
An OpenShift route exposes a service as a host name, such as "www.example.com". As a result, external clients are able to reach it by name.

DNS resolution for a host name is handled separately from routing. Your administrator may have configured a cloud domain that always correctly resolves to the OpenShift router. Or, if you are using an unrelated host name, you may need to modify its DNS records independently to resolve to the router.

Tools to create routes are evolving. While the web console can display routes, it is not yet able to create them. Using the CLI, you can create only an unsecured route using the sample command as shown here. The new route inherits the name from the service unless you specify one.

$ oc expose service/<name> --hostname=www.example.com
