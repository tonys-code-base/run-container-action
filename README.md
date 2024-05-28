# Docker Container Action

Basic action to pull an image run as a docker container.

- Supports images sourced from Docker or ECR registry
- Accepts `inputs` for `docker run` [`options`](https://docs.docker.com/reference/cli/docker/container/run/#options) and/or [runtime arguments](https://docs.docker.com/engine/reference/run/#commands-and-arguments)

## Inputs

```yaml
docker-registry-url:
  description: 'Docker registry address'
  required: true
image:
  description: 'Image name'
  required: true
tag:
  description: 'Image tag'
  required: false
  default: latest
options:
  description: 'Docker run options.
    https://docs.docker.com/reference/cli/docker/container/run/#options'
  required: false
runtime_args:
  description: 'Docker runtime arguments'
  required: false
```

## Output

```yaml
outputs:
  docker_run_rc:
    description: 'Value of return code from received from docker run'
```

## Example 1: Pull from Private *Docker Registry* and Run Container

The sample workflow below:

- runs script  `my-script.sh`, which resides in the root of the checkout path, i.e., `${{ github.workspace/my-script.sh }}`

```yaml
...
...
env:
  image_name: alpine
  image_tag: latest
  registry: registry:443

jobs:
  run-container:
    runs-on: [self-hosted]
    steps:
      - name: Checkout repo
        ...

      - name: Authenticate to registry
        id: login
        uses: docker/login-action@v3
        ...

      - name: Run container
        id: run-docker-container
        uses: tonys-code-base/run-container-action@v1.0.0
        with:
          docker-registry-url: ${{ env.registry }}
          image: ${{ env.image_name }}
          tag: ${{ env.image_tag }}
          options: >-
            -v "${{ github.workspace }}":"/myapp/path"
            -e MYENV_VAR=sample-value
          runtime_args: >-
            sh -c "/myapp/path/my-script.sh"

      - name: Get return code of docker run
        id: docker-return-code
        run: |
          echo ${{ steps.run-docker-container.outputs.docker_run_rc }}
```

## Example 2: Pull Image from AWS *ECR Registry* and Run Container

Run the same script in previous example, but with an image sourced from AWS ECR.

```yaml
...
...
env:
  image_name: my-ecr-image
  image_tag: latest
  registry: 123456789012.dkr.ecr.us-east-1.amazonaws.com

jobs:
  run-container:
    runs-on: [self-hosted]
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        id: get-aws-creds
        uses: aws-actions/configure-aws-credentials@v4
        ...
        ...

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
        ...
        ...

      - name: Run container
        id: run-docker-container
        uses: tonys-code-base/run-container-action@v1.0.0
        with:
          docker-registry-url: ${{ env.registry }}
          image: ${{ env.image_name }}
          tag: ${{ env.image_tag }}
          options: >-
            -v "${{ github.workspace }}":"/myapp/path"
            -e MYENV_VAR=sample-value
          runtime_args: >-
            sh -c "/myapp/path/my-script.sh"

      - name: Get return code of docker run
        id: docker-return-code
        run: |
          echo ${{ steps.run-docker-container.outputs.docker_run_rc }}
```
