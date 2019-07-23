# Upgrading

This document, intended for Kubernetes administrators, will walk you through
upgrading Kelda on your Kubernetes cluster.

1. **Download the latest release of Kelda**

    Paste the following into your shell to download the latest Kelda release:

        curl -fsSL https://kelda.io/install.sh | sh

    You should see the following output:

        Downloading the latest Kelda release...
        ################################################################# 100.0%
        The latest Kelda release has been downloaded to /var/folders/8j/wk3ln0fj1nd4sb4pn1bz7j9m0000gn/T/tmp.ZftQtS2I/kelda
        Please install Kelda to your desired location, or use the snippet below to install it to /usr/local/bin.


            sudo cp /var/folders/8j/wk3ln0fj1nd4sb4pn1bz7j9m0000gn/T/tmp.ZftQtS2I/kelda /usr/local/bin


    Either use the provided snippet to upgrade if Kelda is installed in
     `/usr/local/bin`, otherwise copy the binary over to your install location.

    Verify that you have correctly installed the latest version of the Kelda CLI.

        kelda version

        local version:  0.11.0
        minion version: 0.10.0

    The rest of this document assumes that you are running all commands from the
    root of the extracted release of Kelda.

1. **Upgrade the Kelda minion**

    Deploy the latest version of the Kelda minion to your Kubernetes cluster
    with the following command:

        kelda setup-minion --license <path to license>

    The output should look something like this:

        Deploy to context `dev`? (y/N) y
        Deploying Kelda components to the `dev` context....
        Waiting for minion to boot....
        Done!

1. **Verify the new version**

    You can verify the new version by running `kelda version`. The output of
    this command will look something like this:

        kelda version

        local version:  0.11.0
        minion version: 0.11.0

1. **Delete previous namespace**

    You must delete your development namespace whenever you upgrade to a new
    version of the CLI.

        kelda delete

    The output of this command will look something like this:

        Deleting namespace 'user-namespace'.........................

    Executing this command will find the namespace for the current user and
    delete it. This will take a few minutes.

    **Note:** This will only affect the current user's namespace. Each
    developer must run `kelda delete`.

1. **Resume development**

    You can resume development by running `kelda dev`. This will create a new
    namespace using the latest version of Kelda.

## Release Notes

<!---
### Next Release

Notes for the upcoming release are added here when the relevant code is added,
and uncommented when the next release is made.
-->

### 0.12.0

- Added `kelda upgrade-cli`, which makes it easy to upgrade the CLI to match
  the minion's version.
- Simplify the initial Kelda installation process. Kelda can now be installed
  via a shell script downloaded by `curl`.

### 0.11.0

- Added `kelda update`, which updates the container images in the
  development environment to the latest versions available upstream.
- Added support for "init commands" during file syncs. These are commands that
  are only triggered when certain files are changed. For example, this can be
  used to only run `npm install` when `package-lock.json` is changed.

### 0.10.0

- We've changed the way we transmit errors between the minion and CLI. This is
  a **breaking change** and requires updating both the CLI and Minion.
- We now require a license file to install Kelda, and automatically collect
  usage analytics.

### 0.9.1

- Fix bug where changes to Kubernetes manifests wouldn't get deployed.

### 0.9.0

Release 0.9.0 makes it easier to install the Minion, and makes some UX
improvements around error handling.

### 0.8.0

The first release using our new versioning scheme.
