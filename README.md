# Skaffold Github Action

> Code in [src](/src/) is autogenerated as build outcome from [@hiberbee/actions](https://github.com/hiberbee/actions) repository, which combines various DevOps Github Actions. If you would like to contribute, create PR in that repository.

<p align="center">
  <img src="https://img.shields.io/github/license/hiberbee/github-action-minikube?style=flat-square" alt="License">
  <img src="https://img.shields.io/github/workflow/status/hiberbee/github-action-minikube/CI?label=github-actions&style=flat-square" alt="GitHub Action Status">
  <img src="https://img.shields.io/github/v/tag/hiberbee/github-action-minikube?label=hiberbee%2Fgithub-action-minikube&style=flat-square" alt="GitHub Workflow Version">
</p>

Skaffold is a command line tool that facilitates continuous development for Kubernetes applications. You can iterate on your application source code locally then deploy to local or remote Kubernetes clusters. Skaffold handles the workflow for building, pushing and deploying your application. It also provides building blocks and describe customizations for a CI/CD pipeline.

This action allows you to execute skaffold commands as Github Action. Repository is self-testable, so you can refer to [Skaffold manifest](test/skaffold.yaml), [Container Structure tests](test/structure-test.yaml) and [Github workflow](.github/workflows/ci.yml)

## Installed versions

- skaffold 1.38.0
- container-structure-test 1.11.0

## Inputs

### Required

### Optional

| Name                               | Description                                                                                    | Default                 |
|------------------------------------|------------------------------------------------------------------------------------------------|-------------------------|
| `skaffold-version`                 | Set Skaffold version                                                                           | 1.39.1                  |
| `container-structure-test-version` | Set Container Structure Test version                                                           | 1.11.0                  |
| `kubectl-version`                  | Set Kubernetes CLI version                                                                     | 1.24.3                  |
| `working-directory`                | Set current working directory similar to Github Actions run                                    | ${{ github.workspace }} |
| `filename`                         | Path or URL to the Skaffold config file                                                        | skaffold.yaml           |
| `command`                          | Default command for Skaffold to execute                                                        | diagnose                |
| `file-output`                      | Filename to write build images to                                                              | n/a                     |
| `repository`                       | Default repository value (overrides global config)                                             | n/a                     |
| `insecure-registries`              | Target registries for built images which are not secure                                        | n/a                     |
| `image`                            | Set Skaffold profile name                                                                      | n/a                     |
| `tag`                              | The optional custom tag to use for images which overrides the current Tagger configuration     | n/a                     |
| `push`                             | Push the built images to the specified image repository                                        | n/a                     |
| `concurrency`                      | Number of concurrently running builds. If equals 0 (default) - will run all builds in parallel | n/a                     |
| `kube-context`                     | Deploy to this Kubernetes context                                                              | n/a                     |
| `kubeconfig`                       | Path to the kubeconfig file to use for CLI requests                                            | n/a                     |
| `namespace`                        | Run deployments in the specified namespace                                                     | n/a                     |
| `profile`                          | Activate profiles by name                                                                      | n/a                     |
| `output`                           | Format output with go-template                                                                 | n/a                     |
| `skip-tests`                       | Whether to skip the tests after building                                                       | true                    |
| `cache`                            | Set to false to disable default caching of artifacts                                           | true                    |
| `cache-file`                       | Specify the location of the cache file                                                         | n/a                     |

## Outputs

| Name   | Description      | Payload                                           |
|--------|------------------|---------------------------------------------------|
| builds | Built image tags | ``` [{ "imageName": "string", tag: "string"}] ``` |

### Example

#### Build Docker images

```yaml
jobs:
  pipeline:
    name: Skaffold Docker build
    runs-on: ubuntu-20.04
    steps:
      - name: Build Docker images
        uses: hiberbee/github-action-skaffold@1.20.0
        with:
          command: build
          repository: ghcr.io/hiberbee/docker
          image: nodejs
          tag: ${{ github.sha }}
```

See more complex example with build, test & deployment simple Helm chart from Dockerfile to local K8S mini cluster.

```yaml
name: Skaffold
on:
  push:
    paths:
      - src/**
      - .github/workflows/ci.yml
      - action.yml
jobs:
  pipeline:
    name: Skaffold Pipeline
    runs-on: ubuntu-20.04
    steps:
      - name: Checkout sources
        uses: actions/checkout@v3

      - name: Setup Minikube
        uses: hiberbee/github-action-minikube@1.5.0

      - name: Setup Helm
        uses: hiberbee/github-action-helm@1.3.0
        with:
          repository-config: test/repositories.yaml

      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          registry: ${{ secrets.DOCKER_REGISTRY }}
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Run Skaffold pipeline as action
        uses: hiberbee/github-action-skaffold@1.19.0
        with:
          command: run
          repository: ghcr.io/${{ github.repository }}

      - name: Get Helm releases
        run: helm list

```

## CLI usage

You can use that action just to set up Skaffold and then perform actions manually. Here is code sample:
