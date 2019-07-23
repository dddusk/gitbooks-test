# Configuration

This document, intended for Kubernetes administrators, will provide details on configuring your own project to work with Kelda. In order to get started with the initial configuration you should download Kelda and the Magda example project using the instructions below.

**Note:** If you've already completed the Quickstart, then you have already deployed the Kelda Minion and installed the CLI. You can skip to the [Configuration Files](configuration.md#configuration-files) portion of this documentation to configure Kelda for your own project.

## Installing Kelda Quick

You should install the latest version of Kelda using the following command:

```text
curl -fsSL https://kelda.io/install.sh | sh
```

You should see the following output:

```text
Downloading the latest Kelda release...
################################################################# 100.0%
The latest Kelda release has been downloaded to /var/folders/8j/wk3ln0fj1nd4sb4pn1bz7j9m0000gn/T/tmp.ZftQtS2I/kelda
Please install Kelda to your desired location, or use the snippet below to install it to /usr/local/bin.


    sudo cp /var/folders/8j/wk3ln0fj1nd4sb4pn1bz7j9m0000gn/T/tmp.ZftQtS2I/kelda /usr/local/bin
```

Either use the provided snippet to upgrade if Kelda is installed in `/usr/local/bin`, otherwise copy the binary over to your install location.

## Downloading Magda

Use the following commands to download the Magda example project:

```text
curl -Lo magda-demo.tar.gz https://update.kelda.io/?file=demo&release=latest&token=install

tar -xvzf magda-demo.tar.gz

cd magda
```

The rest of this guide assumes you are running commands from the root of the `magda` folder.

## Components

Kelda consists of two components:

* The **Minion** is a pod that runs in the Kubernetes cluster. It is responsible

  for creating and managing Kelda development namespaces.

* The **CLI** is a program, installed on the developer's workstation, which

  allows them to interact with their Kelda development namespace.

The `magda` folder contains the following files:

```text
├── magda-kelda-config
│   ├── ...
├── magda-gateway
│   ├── ...
├── magda-scala
│   ├── ...
├── magda-web-server
│   ├── ...
└── README.md
```

* Each of the `magda-gateway`, `magda-scala`, and `magda-web-server` directories

  contains an open source microservices application that you can use as a

  reference to structure your own configuration.

### Deploying the Minion

Kelda uses the credentials stored in `~/.kube/config` for communicating with the Kubernetes cluster. Before deploying Kelda, make sure that you have a valid `kubeconfig` file.

You can deploy the Minion by executing `kelda setup-minion`. The output of this command will look something like this:

```text
kelda setup-minion --license <path to license>

Deploy to context `dev`? (y/N) y
Deploying Kelda components to the `dev` context....
Waiting for minion to boot....
Done!
```

You can verify that it booted with the following command:

```text
kubectl logs -l service=kelda -n kelda
```

The output should be something like this:

```text
time="2019-05-19T22:43:37Z" level=info msg="Listening for connections.." address="0.0.0.0:9000"
```

## Configuration Files

This section describes all of the configuration files that are used by Kelda.

### User Configuration

**Note:** If you've already completed the Quickstart guide, then you already have a `~/.kelda.yaml` file. The only part of the file that should be changed for your own project is the path to the workspace file since it is likely different than the one you used during the Quickstart.

Each developer's workstation requires a configuration file located at `~/.kelda.yaml`.

```text
# The version of the configuration format.
version: v1alpha1

# A unique namespace that will be used for the development namespace.
namespace: user

# The Kubernetes cluster to use for development. The name of the current
# cluster can be retrieved through `kubectl config current-context`.
context: dev-cluster

# The path to the configuration for deploying the application.
# More explanation on the workspace.yaml is below.
workspace: ~/kelda-workspace/workspace.yaml
```

This configuration can be generated using the Kelda CLI.

1. Ensure that the current Kubernetes context is set to the development

   cluster. You can verify this with `kubectl config current-context`.

2. `cd` into the Kelda workspace directory.
3. Run `kelda config`.

   The `config` command is used to set up a user-specific configuration for Kelda. Running this command will ask you a couple of questions.

   ```text
    Unique identifier for development namespace [user]:
   ```

   The _unique identifier_ defaults to the username for the computer that you executed the `config` command from. You can change this to any string if you wish, but it should be unique to the Kubernetes cluster.

   ```text
    Kubernetes context for development cluster [dev-cluster]:
   ```

   The _Kubernetes context_ refers to the `kubectl` context that you wish to use with this cluster. This will default to the current `kubectl` context. You can change this to any other valid context if you wish.

   ```text
    Path to Kelda Workspace file [~/kelda-workspace/workspace.yaml]:
   ```

   The _Kelda Workspace_ file refers to the Kelda configuration that you wish to use for this cluster. It will default to the `workspace.yaml` file that it locates in the current directory.

   After completing these configuration prompts, a Kelda configuration file will be written to your home directory.

   ```text
    Wrote config to /home/user/.kelda.yaml
   ```

### Workspace Configuration

The `workspace.yaml` file specifies what services and tunnels should be created in the development namespace. You can refer to `magda-kelda-config/workspace.yaml` in the release directory for a reference of how to write this file.

```text
# Sample workspace.yaml file that defines three services, and proxies
# ports from one service to the local workstation.

version: "v1alpha1"

tunnels:
- serviceName: "gateway"
    remotePort: 80
    localPort: 8080

services:
- name: "authorization-api"
- name: "web-server"
- name: "content-api"
```

```text
# Sample workspace directory strucuture. Each service defined in the
# workspace.yaml file must have a corresponding directory.

├── workspace.yaml
├── authorization-api
│   ├── deployment-authorization-api.yaml
│   └── service-authorization-api.yaml
├── content-api
│   ├── configmap-scss-compiler-config.yaml
│   ├── deployment-content-api.yaml
│   └── service-content-api.yaml
└── web-server
    ├── configmap-web-app-config.yaml
    ├── deployment-web.yaml
    └── service-web.yaml
```

**Tunnels** are ports that you want to forward from the development cluster. This allows you to access remote resources directly from your local workstation. In the example above, we are proxying port 80 on the remote `gateway` service to port 8080 on our local workstation.

**Services** are named references to the specific microservices that compose your application. We expect that each service referenced in `workspace.yaml` has a directory which contains the Kube YAML that will be deployed. In the example above we have defined three services, and each service that is defined in the `workspace.yaml` file has its own directory which defines the Kubernetes configuration for that service.

### Kubernetes YAML Configuration

Writing the Kubernetes YAML from scratch is outside of the scope of this document. However, we have some general recommendations if you are migrating over from standard Kubernetes YAML, Docker Compose, or a Helm configuration.

* Right now, `kelda logs`, `kelda ssh`, and `kelda dev` only work for services

  that have exactly one pod. Services with more than one pod can still be deployed,

  but these commands will not function correctly.

#### Migrating from Standard Kubernetes YAML

* Group YAML into services so that each service has one pod.
* Decrease counts for Replica Sets.
* Remove services that do not make sense to run in the development environment such as

   scheduled tasks.

#### Migrating from Docker Compose

You can use a tool like [Kompose](http://kompose.io/) to generate a Kubernetes configuration from an existing Docker Compose file. Before running this tool, we recommend making the following adjustments.

* Edit the Docker Compose file so that it doesn't have Docker-specific volume configuration

  since this is handled differently in Kubernetes.

  * `volumes_from` for data containers will use the `InitContainer` and shared volume

     approach described below.

  * All syncing will use Kelda's `kelda.yaml` file to define which files need to be synced.
  * Configuration files should use Kubernetes primitives such as `ConfigMap`, be synced

    over via `kelda.yaml`, or be baked directly into the container.

#### Migrating from Helm

If you are using Helm, you should run the [`helm template`](https://helm.sh/docs/helm/#helm-template) command to generate the raw Kubernetes configuration and then follow the recommended practices for Standard Kubernetes YAML described above.

#### Registry Credentials

The `regcred` secret in the `kelda` namespace is automatically copied to each development namespace. Kubernetes manifests can then reference this secret as an ImagePullSecret. You can create this secret by running the following command:

```text
kubectl create secret docker-registry regcred -n kelda \
    --docker-server https://index.docker.io/v1/ \
    --docker-username=USERNAME \
    --docker-password=PASSWORD \
    --docker-email=EMAIL
```

**Note:** Since every developer will have access to this secret, we recommend using a service account instead of a user-specific Dockerhub account.

#### Data Volumes

A common pattern is to run mock copies of databases in the development environment. We suggest building a container with mocked data and deploying it to the development cluster. A sample Kube YAML file that follows this pattern for elasticsearch is shown below:

```text
containers:
- image: elasticsearch:1
name: elasticsearch
volumeMounts:
- mountPath: /usr/share/elasticsearch/data
    name: data
initContainers:
- image: data-es-geoservice:latest
name: elasticsearch-data
command: [ "sh", "-c", "cp -r /usr/share/elasticsearch/data/* /usr/share/elasticsearch/data_dst" ]
volumeMounts:
- mountPath: /usr/share/elasticsearch/data_dst
    name: data
volumes:
- name: data
    emptyDir: {}
```

#### Cloud Services

If you depend on cloud services in production, such as Amazon RDS, we suggest using a containerized version in your development environment. Although it's possible to connect directly to the cloud service, using a containerized version makes it easy to enforce isolation between developers.

### Service Configuration

The directory from which `kelda dev` is run must have a configuration file named `kelda.yaml` that specifies how to run the service in development mode.

```text
# REQUIRED. The version of the configuration format.
version: "v1alpha1"

# REQUIRED. Name of the service. Must match the name in workspace.yaml.
name: SERVICE_NAME

# REQUIRED. The files to sync from the local machine to the container.
sync:
- from: SRC_ON_LAPTOP
  to: DST_IN_CONTAINER
  except: ['.git']
- from: SRC
  to: DST
  triggerInit: true

# OPTIONAL. The image to run.
# If not set, Kelda uses the same image as when not developing.
image: DEV_IMAGE

# OPTIONAL. The command to run in the development container after each file sync.
# If not set, Kelda uses the same command as when not developing.
command: ['DEV_COMMAND']

# OPTIONAL. The command to run for sync rules that have `triggerInit` set
# to true (e.g. the second rule above).
initCommand: ['INIT_COMMAND']
```

#### Deciding What to Synchronize

When syncing code changes you can choose to compile your code on your local workstation and sync the resulting files and binaries directly to Kubernetes, or sync the code changes to Kubernetes and have the compilation step occur there. There are some tradeoffs to both approaches. For the best experience we recommend the following:

* For interpreted languages, you should synchronize the source files, and install the dependencies

  in the remote container to avoid platform issues.

* For JVM languages, you should build the JAR locally and sync the result over. Kelda will automatically

  sync the artifacts, but triggering the compilation step it outside of the scope of Kelda.

* For other compiled languages, you should cross-compile locally. Compiling in the remote container

  is also an option, but it has not been thoroughly tested.

## Troubleshooting Common Issues

### File Sync Limitations

Kelda opens a file descriptor for each directory/subdirectory that's being synced, so the number of files that can be synced are limited by the system's maximum number of open files. If the limit is too low, `kelda dev` will error with `too many open files in system`.

On OSX, the default maximum number of open files is 256. This can be increased by editing the `limit.maxfiles` as described in this blog post: [https://medium.com/mindful-technology/too-many-open-files-limit-ulimit-on-mac-os-x-add0f1bfddde](https://medium.com/mindful-technology/too-many-open-files-limit-ulimit-on-mac-os-x-add0f1bfddde).

