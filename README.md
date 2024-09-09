# Prefect Helm Charts

This repository contains the official Prefect Helm charts for installing and configuring Prefect on Kubernetes. These charts support multiple use cases of Prefect on Kubernetes depending on the values provided and chart selected including:

## [Prefect worker](charts/prefect-worker/)

[Workers](https://docs.prefect.io/latest/concepts/work-pools/#worker-overview) are lightweight polling services that retrieve scheduled runs from a work pool and execute them.

Workers each have a type corresponding to the execution environment to which they will submit flow runs. Workers are only able to join work pools that match their type. As a result, when deployments are assigned to a work pool, you know in which execution environment scheduled flow runs for that deployment will run.

## [Prefect server](charts/prefect-server/)

[Prefect server](https://docs.prefect.io/latest/guides/host/) is a self-hosted open source backend that makes it easy to observe and orchestrate your Prefect flows. It is an alternative to using the hosted [Prefect Cloud](https://docs.prefect.io/latest/cloud/) platform. Prefect Cloud provides additional features including automations and user management.

## [Prometheus Prefect Exporter](charts/prometheus-prefect-exporter/)

The Prometheus Prefect Exporter is a tool to pull relevant Prefect metrics from a hosted Prefect Server instance

## Prefect agent

Prefect Agents have been deprecated. `prefect-helm` version `2024.8.30163822` and earlier will contain the `prefect-agent` chart.

## Usage

[Helm](https://helm.sh) must be installed to use the charts.
Please refer to Helm's [documentation](https://helm.sh/docs/) to get started.

### TL;DR

```bash
helm repo add prefect https://prefecthq.github.io/prefect-helm
helm search repo prefect
helm install my-release prefect/prefect-<chart>
```

### Installing released versions

Charts are automatically versioned and released together. The `appVersion` and `prefectTag` version are pinned at package time to the current release of Prefect 2.

The charts are hosted in a [Helm repository](https://helm.sh/docs/chart_repository/) deployed to the web courtesy of GitHub Pages.

1. Add the Prefect Helm repository to Helm and list available charts and versions:

    ```bash
    helm repo add prefect https://prefecthq.github.io/prefect-helm
    helm repo update
    helm search repo prefect
    ```

2. Install the Helm chart

    Each chart will require some specific configuration values to be provided, see the individual chart README's for more information on configuring and installing the chart.

    If chart installation fails, run the same command with `--debug` to see additional diagnostic information.

    Refer to the [Helm `install` documentation](https://helm.sh/docs/helm/helm_install/) for all options.

The helm charts are tested against Kubernetes 1.26.0 and newer minor versions.

### Installing development versions

Development versions of the Helm chart will always be available directly from this Github repository.

1. Clone repository

2. Change to this directory

3. Download the chart dependencies

    ```bash
    helm dependency update
    ```

4. Install the chart

    ```bash
    helm install . --generate-name
    ```

### Upgrading

1. Look up the name of the last release

    ```bash
    helm list
    ```

2. Run the upgrade

    ```bash
    # Choose a version to upgrade to or omit the flag to use the latest version
    helm upgrade prefect-server prefect/prefect-server --version 2023.09.07
    ```

    For development versions, make sure your cloned repository is updated (`git pull`) and reference the local chart

    ```bash
    helm upgrade prefect-server .
    ```

3. Upgrades can also be used enable features or change options

   ```bash
   helm upgrade prefect-server prefect/prefect-server --set newValue=true
   ```

#### Important notes about upgrading

- Updates will only update infrastructure that is modified.
- You will need to continue to set any values that you set during the original install (e.g. `--set namespaceOverride=prefect` or `--values path/to/values.yaml`).
- If you are using the postgresql subchart with an autogenerated password, it will complain that you have not provided that password for the upgrade. Export the password as the error asks then set it within the subchart using:
  ```bash
  helm upgrade ... --set postgresql.postgresqlPassword=$POSTGRESQL_PASSWORD
  ```

## Options:

See comments in `values.yaml`.

### Security context

By default, the worker, and server run as an unprivileged user with a read-only root filesystem. You can customize the security context settings for both the worker and server in the `values.yaml` file for your use case.

If you need to install system packages or configure other settings at runtime, you can configure a writable filesystem and run as root by configuring the pod and container security context accordingly:

```yaml
podSecurityContext:
  runAsUser: 0
  runAsNonRoot: false
  fsGroup: 0

containerSecurityContext:
  runAsUser: 0
  # this must be false, since we are running as root
  runAsNonRoot: false
  # make the filesystem writable
  readOnlyRootFilesystem: false
  # this must be false, since we are running as root
  allowPrivilegeEscalation: false
```

If you are running in OpenShift, the default `restricted` security context constraint will prevent customization of the user. In this case, explicitly configure the `runAsUser` settings to `null` to use OpenShift-assigned settings:

```yaml
podSecurityContext:
  runAsUser: null
  fsGroup: null

containerSecurityContext:
  runAsUser: null
```

The other default settings, such as a read-only root filesystem, are suitable for an OpenShift environment.

## Version support policy

Prefect follows the [upstream Kubernetes support policy](https://kubernetes.io/releases/version-skew-policy/), meaning that we test against the three most recent minor version releases of Kubernetes. The charts may be compatible with older releases of Kubernetes, however, we do not test against those versions and may choose to reject issues or patches to add support.

## Troubleshooting

### The database deploys correctly but other services fail with "bad password"

If you are using the subchart deployed database with persistence enabled, it is likely the password for the user has persisted between deployments in the PVC for the database but the secret has been regenerated by the Helm chart and does not match. Deploy and set the 'postgresql.auth.existingSecret' option or set a constant password at `postgresql.auth.password`.

## Contributing

Contributions to the Prefect Helm Charts are always welcome! We welcome your help - whether it's adding new functionality, echo 'eval "$(~/.local/bin/mise activate zsh)"' >> ~/.zshrctweaking documentation, or anything in between. In order to successfully contribute, you'll need to fork this repository and commit changes to your local prefect-helm repo. You can then open a PR against this upstream repo that the team will review!

To get started, ensure you have the required dependencies installed:

```shell
mise install
```

Be sure to run `pre-commit install` before starting any development. [`pre-commit`](https://pre-commit.com/)
will help catch simple issues before committing.

### Documentation

Please make sure that your changes have been linted & the chart documentation has been updated.  The easiest way to accomplish this is by installing [`pre-commit`](https://pre-commit.com/).

### Testing & validation

Make sure that any new functionality is well tested!  You can do this by installing the chart locally, see [above](https://github.com/PrefectHQ/prefect-helm#installing-development-versions) for how to do this.

You can also create and run test suites via [helm-unittest](https://github.com/helm-unittest/helm-unittest).
Related test files are stored under `./charts/<chart>/tests/*_test.yaml`.
Refer to the `helm-unittest` repository for more information.

The following helper script will run the tests via the `helm-unittest` Docker image in case you don't have the binary installed locally:

```shell
./scripts/helm_unittest.sh
```

When `helm-unittest` is available via the [`mise` registry](https://mise.jdx.dev/registry.html), we'll add it to `.mise.toml`
for easy local installation.

### Opening a PR

A helpful PR explains WHAT changed and WHY the change is important. Please take time to make your PR descriptions as helpful as possible. If you are opening a PR from a forked repository - please follow [these](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/allowing-changes-to-a-pull-request-branch-created-from-a-fork) docs to allow `prefect-helm` maintainers to push commits to your local branch.
