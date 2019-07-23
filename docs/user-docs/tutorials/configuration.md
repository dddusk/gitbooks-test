# Writing a Kelda Configuration

In this tutorial we will walk through creating a complete Kelda configuration
step by step. Similar to the [Quickstart](/) we will be working with an open
source microservices application called Magda. Each release of Kelda includes
a working configuration for Magda that you can use to learn about the key concepts
of Kelda.

If you have not already, download and unzip the release of Kelda that was
provided to you and enter the unzipped directory.

## Workspace

The [`workspace.yaml`](/administrator/configuration/#workspace-configuration)
file is used to define the services and tunnels in your cluster.

In your release of Kelda you should see a directory called `magda-kelda-config`.
This directory includes additional sub-directories that include the Kubernetes
configuration files for the various microservices in our Kubernetes cluster.
You will also see an existing `workspace.yaml` file in this directory.
You can ignore it for now because we are going to write it again from scratch.

The contents of this directory are shown below:

    ├── magda-kelda-config
    │   ├── authorization-api
    │   │   ├── deployment-authorization-api.yaml
    │   │   └── service-authorization-api.yaml
    │   ├── combined-db
    │   │   ├── service-authorization-db.yaml
    │   │   ├── service-content-db.yaml
    │   │   ├── service-registry-db.yaml
    │   │   ├── service-session-db.yaml
    │   │   └── statefulset-combined-db.yaml
    │   ├── content-api
    │   │   ├── configmap-scss-compiler-config.yaml
    │   │   ├── deployment-content-api.yaml
    │   │   └── service-content-api.yaml
    │   ├── elasticsearch
    │   │   ├── daemonset-es-ds.yaml
    │   │   ├── deployment-kibana.yaml
    │   │   ├── service-elasticsearch-discovery.yaml
    │   │   ├── service-elasticsearch.yaml
    │   │   ├── service-kibana.yaml
    │   │   └── statefulset-es-data.yaml
    │   ├── gateway
    │   │   ├── configmap-gateway-config.yaml
    │   │   ├── deployment-gateway.yaml
    │   │   └── service-gateway.yaml
    │   ├── indexer
    │   │   ├── deployment-indexer.yaml
    │   │   └── service-indexer.yaml
    │   ├── preview-map
    │   │   ├── deployment-preview-map.yaml
    │   │   └── service-preview-map.yaml
    │   ├── registry-api
    │   │   ├── deployment-registry-api-full.yaml
    │   │   ├── service-registry-api-read-only.yaml
    │   │   └── service-registry-api.yaml
    │   ├── search-api
    │   │   ├── deployment-search-api.yaml
    │   │   └── service-search-api.yaml
    │   ├── secrets
    │   │   ├── secret-auth-secrets.yaml
    │   │   └── secret-db-passwords.yaml
    │   ├── setup
    │   │   └── regcred.yaml
    │   ├── web-server
    │   │   ├── configmap-web-app-config.yaml
    │   │   ├── deployment-web.yaml
    │   │   └── service-web.yaml

### Writing a Workspace Configuration

1. Enter the `magda-kelda-config` directory.

        cd magda-kelda-config

1. Make a backup of the existing `workspace.yaml` file to use for reference in
   case something goes wrong.

        mv workspace.yaml workspace.yaml.bak

1. Create a new file called `workspace.yaml` and open it in your favorite text
   text editor.

        touch workspace.yaml
        vim workspace.yaml

1. Specify the format version of this configuration file.

    The current format version of `workspace.yaml` is `v1alpha1`.

        version: "v1alpha1"

1. Define your services.

    First we will define all of the services that our application consists of.
    Each service that is defined in the `workspace.yaml` file needs to have
    a corresponding directory at the same level which contains the necessary
    Kubernetes YAML configuration that is used to configure, deploy, and run
    the service.

    The schema for defining a service looks like this:

        services:
          - name: "service-name"

    Be sure that "service-name" matches the name of the directory that contains
    the service configuration files. Go ahead and add a service for each
    of the services in Magda. The `workspace.yaml` file should look like
    this now:

        version: "v1alpha1"

        services:
          - name: "secrets"
          - name: "combined-db"
          - name: "elasticsearch"
          - name: "authorization-api"
          - name: "preview-map"
          - name: "gateway"
          - name: "indexer"
          - name: "registry-api"
          - name: "search-api"
          - name: "web-server"
          - name: "content-api"

1. Define your tunnels.

    We are going to create a tunnel for the `gateway` and `web-server` services
    from the Kubernetes cluster to our local workstation. The schema for defining
    a tunnel looks like this:

        tunnels:
          - serviceName: "<service-name>"
            remotePort: <remote-port>
            localPort: <local-port>

    * `serviceName` is a quoted string which refers to the name of the service
    that you wish to create the tunnel for. Note that this name should be the
    same as the name that you created in the services configuration.

    * `remotePort` is an integer which refers to the port that the services is
    listening on in your Kubernetes cluster.

    * `localPort` is an integer which refers to the port that you wish to
    forward the service to on your local workstation.

    In your open `workspace.yaml` file add the following contents to configure
    your tunnels.

        tunnels:
        - serviceName: "gateway"
            remotePort: 80
            localPort: 8080
        - serviceName: "web-server"
            remotePort: 9229
            localPort: 9229

1. Final `workspace.yaml` file.

    This completes the `workspace.yaml` configuration. Double check to make
    sure that the final version of your file looks like this:

        version: "v1alpha1"

        services:
          - name: "secrets"
          - name: "combined-db"
          - name: "elasticsearch"
          - name: "authorization-api"
          - name: "preview-map"
          - name: "gateway"
          - name: "indexer"
          - name: "registry-api"
          - name: "search-api"
          - name: "web-server"
          - name: "content-api"

        tunnels:
          - serviceName: "gateway"
            remotePort: 80
            localPort: 8080
          - serviceName: "web-server"
            remotePort: 9229
            localPort: 9229

1. Update the `.kelda.yaml` configuration file.

    If you have completed the quickstart, then you should already have a
    `~/.kelda.yaml` configuration file that points to the correct kelda
    workspace configuration. If you do not, please repeat the steps shown
    in [Step 5 of the quickstart](/#setup).

1. Test out the workspace.

    You can test out your `workspace.yaml` configuration by running Kelda
    with the `--no-sync` flag.

        kelda dev --no-sync

    You should see something like this in your terminal:

    ![Kelda Running with No Sync](/assets/no-sync.png)

    If everything booted up correctly then you have a valid Kelda configuration!
    You can press `Ctrl + C` to exit this screen.

## Service Configuration

Each service that you will be developing with Kelda needs to have its own
`kelda.yaml` configuration file which defines how the service should be run.

### Writing a Service Configuration

1. From the root of the Kelda release, enter the `magda-web-server` directory.

        cd magda-web-server

1. Make a backup of the existing `kelda.yaml` file in case something goes wrong.

        mv kelda.yaml kelda.yaml.bak

1. Create a new file called `kelda.yaml` and open it in your favorite text editor.

        touch kelda.yaml
        vim kelda.yaml

1. Write a configuration.

    The schema for the `kelda.yaml` file looks like this:

        version: "v1alpha1"
        name: "service-name"
        sync:
          - from: "local_directory"
            to: "remote_directory"

    Note that all values in this file must be quoted strings.

    In your opened `kelda.yaml` file, specify the configuration format version
    and add the name of the service, which in this case is `web-server`. Note
    that this name needs to be identical to the name of the service that we
    added in the previous `workspace.yaml` configuration.

        version: "v1alpha1"
        name: "web-server"

    Next, define which directories you would like to sync between your local
    workstation and the remote Kubernetes cluster.

        sync:
          - from: "src"
            to: "dist"

    In this case, we are going to be syncing from our local `src` directory to
    the remote `dist` directory. Any changes that we made locally will be synced
    up to the remote cluster and the service will automatically be restarted.
    This makes it so that any changes are seamlessly reflected in our remote cluster.

## Testing Everything Out

Now that we have a valid `workspace.yaml` and `kelda.yaml` configuration we
can test everything out. We can use `kelda dev` to deploy our application
to the cluster, make changes locally, and see them reflected in
the cluster immediately.

    kelda dev
