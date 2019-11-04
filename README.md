![Actions Status](https://github.com/machine-learning-apps/repo2docker-action/workflows/Tests/badge.svg)

# repo2docker GitHub Action

Trigger [repo2docker](https://github.com/jupyter/repo2docker) to turn your GitHub repository into a Jupyter enabled docker image.  https://repo2docker.readthedocs.io

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

    - name: try-local-build
      uses: machine-learning-apps/repo2docker-action@master
      with:
        DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
        DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
        IMAGE_NAME: "your_username/your_image_name"
```


## Mandatory Inputs

- `DOCKER_USERNAME`:
    description: Docker registry username
 -`DOCKER_PASSWORD`:
    description: Docker registry password
- `IMAGE_NAME`:
    description: name of the image.  Example - myContainer

## Optional Inputs

- `DOCKER_REGISTRY`:
    description: The uri of the docker registry
    default: "registry.hub.docker.com"

## Outputs


- `IMAGE_SHA_NAME`:
    description: 'The name of the docker image, which is tagged with the SHA.'

