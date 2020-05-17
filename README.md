
# <a href="https://github.com/jupyter/repo2docker"><img src="https://raw.githubusercontent.com/jupyter/repo2docker/3fa7444fca6ae2b51e590cbc9d83baf92738ca2a/docs/source/_static/images/repo2docker.png" height="40px" /></a>  repo2docker GitHub Action


Trigger [repo2docker](https://github.com/jupyter/repo2docker) to build a Jupyter enabled Docker image from your GitHub repository and push this image to a Docker registry of your choice.  This will automatically attempt to build an environment from configuration files found in your repository in the [manner described here](https://repo2docker.readthedocs.io/en/latest/usage.html#where-to-put-configuration-files).

Read the full docs on repo2docker for more information:  https://repo2docker.readthedocs.io

Images generated by this action are automatically tagged with both `latest` and `<SHA>` corresponding to the relevant [commit SHA on GitHub](https://help.github.com/en/github/getting-started-with-github/github-glossary#commit).  Both tags are pushed to the Docker registry specified by the user. If an existing image with the `latest` tag already exists in your registry, this Action attempts to pull that image as a cache to reduce uncessary build steps.

## What Can I Do With This Action?

- Use repo2docker to pre-cache images for your own [BinderHub cluster](https://binderhub.readthedocs.io/en/latest/zero-to-binderhub/setup-binderhub.html), or for [mybinder.org](https://mybinder.org/).
  - You can use this Action to pre-cache Docker images to a Docker registry that you can reference in your repo.  For example if you have the file `Dockerfile` in the `binder/` directory relative to the root of your repository with the following contents, this will allow Binder to start quickly by pulling an image you have already built:

    Example of the contents of ` binder/Dockerfile`:
    ```dockerfile
    # This is the image that is built and pushed by this Action (replace this with your image name)
    FROM myorg/myimage:latest
    ...
    ```
- Provide a way to Dockerize data science repositories with Jupyter server enabled that you can deploy to VMs, [serverless computing](https://en.wikipedia.org/wiki/Serverless_computing) or other services that can serve Docker containers as-a-service.
- Maximize reproducibility by allowing authors, without any prior knowledge of Docker, to build and share containers.

## Recommended Usage

You must copy the contents of your repository to use this action as illustrated below:

```yaml
name: Build Notebook Container
on: [push] # You may want to trigger this Action on other things than a push.
jobs:
  build:
    runs-on: ubuntu-latest
    steps:

    - name: checkout files in repo
      uses: actions/checkout@master

    - name: update jupyter dependencies with repo2docker
      uses: machine-learning-apps/repo2docker-action@master
      with:
        DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
        DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
```

## Mandatory Inputs

**Exception: if the input parameter `DEBUG` is set to any value, these values become optional.**

- `DOCKER_USERNAME`:
    description: Docker registry username
- `DOCKER_PASSWORD`:
    description: Docker registry password

## Optional Inputs

- `NOTEBOOK_USER`:
    description: username of the primary user in the image. If this is not specified, the username in `$GITHUB_ACTOR` is used.
- `IMAGE_NAME`:
    name of the image.  Example - myusername/myContainer.  If not supplied, this defaults to <DOCKER_USERNAME/GITHUB_REPOSITORY_NAME>
- `DOCKER_REGISTRY`:
    description: name of the docker registry.  If not supplied, this defaults to [DockerHub](https://hub.docker.com/)
- `LATEST_TAG_OFF`:
    Setting this variable to any value will prevent your image from being tagged with `latest`, in additiona to the [GitHub commit SHA](https://help.github.com/en/github/getting-started-with-github/github-glossary#commit).  This is enabled by default.
- `ADDITIONAL_TAG`:
    An optional string that specifies the name of an additional tag you would like to apply to the image.  Images are already tagged with the relevant [GitHub commit SHA](https://help.github.com/en/github/getting-started-with-github/github-glossary#commit).
- `DEBUG`:
    Setting this variable to any value will turn debug mode on.  When debug mode is on, images will not be pushed to a registry.  Furthermore, verbose logging will be enabled.  This is disabled by default.

## Outputs

- `IMAGE_SHA_NAME`
    The name of the docker image, which is tagged with the SHA.
- `DEBUG_STATUS`:
    This will be `true` if debug mode was turned on or `false` otherwhise.

# Examples

## Push Image To A Registry Other Than DockerHub

```yaml
name: Build Notebook Container
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:

    - name: checkout files in repo
      uses: actions/checkout@master

    - name: update jupyter dependencies with repo2docker
      uses: machine-learning-apps/repo2docker-action@master
      with: # make sure username & password matches your registry
        DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
        DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
        DOCKER_REGISTRY: "containers.pkg.github.com"
```

## Change Image Name

When you don't provide an image name your image name defaults to `DOCKER_USERNAME/GITHUB_REPOSITORY_NAME`.  For example if the user [`hamelsmu`](http://www.github.com/hamelsmu) tried to run this Action from this repo, it would be named `hamelsmu/repo2docker-action`.  However, sometimes you may want a different image name, you can accomplish by providing the `IMAGE_NAME` parameter as illustrated below:

```yaml
name: Build Notebook Container
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:

    - name: checkout files in repo
      uses: actions/checkout@master

    - name: update jupyter dependencies with repo2docker
      uses: machine-learning-apps/repo2docker-action@master
      with:
        DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
        DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
        IMAGE_NAME: "hamelsmu/my-awesome-image" # this overrides the image name
```

## Test Image Build

You might want to only test the image build withtout pusing to a registry, for example to test a pull request. You can do this by specifying any value for the `DEBUG` parameter:

```yaml
name: Build Notebook Container
on: [pull_request]

  debug-mode-no-registry:
    runs-on: ubuntu-latest
    steps:  
    - name: Checkout PR
      uses: actions/checkout@v2
      with:
        ref: ${{ github.event.pull_request.head.sha }}

    - name: test build
      uses: machine-learning-apps/repo2docker-action@master
      with:
        DEBUG: 'true'
        IMAGE_NAME: "hamelsmu/repo2docker-test"
```

_When you specify a value for the `DEBUG` parameter, you can omit the otherwhise mandatory parameters `DOCKER_USERNAME` and `DOCKER_PASSWORD` as no images are pushed to a registry in debug mode._
