# Run Docker Container Action

Runs a Docker container.

The action,

- Pulls image from a Docker registry
- Accepts optional `inputs` for `docker run` [options](https://docs.docker.com/reference/cli/docker/container/run/#options) and/or [runtime arguments](https://docs.docker.com/engine/reference/run/#commands-and-arguments)
- Executes `docker run` using the inputs provided

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

## Example 1: Pull from Private Docker Registry and Run

To run a script with name `my-script.sh`, which resides in GitHub repository root path.

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
        uses: actions/checkout@v4

      - name: Authenticate to registry
        id: login
        uses: docker/login-action@v3
        with:
          docker-registry-url: ${{ env.registry }}
          username: ${{ secrets.UNAME }}
          password: ${{ secrets.PASSWD }}

      - name: Run container
        uses: tonys-code-base/run-container-action@v1.0.0
        with:
          docker-registry-url: registry:443
          image: alpine
          tag: latest
          options: >-
            -v ${{ github.workspace }}:/myapp/path
            -e MYENV_VAR=sample-value
          runtime_args: >-
            /myapp/path/my-script.sh
```

## Example 2: Pull from AWS ECR Registry and Run

Run the same script mentioned in previous example, but this time, the image is sourced from AWS ECR.

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
        with:
          mask-password: 'true'
          ...
          ...

      - name: Run container
        uses: tonys-code-base/run-container-action@v1.0.0
        with:
          docker-registry-url: ${{ env.registry }}
          image: ${{ env.image_name }}
          tag: ${{ env.image_tag }}
          options: >-
            -v ${{ github.workspace }}:/myapp/path
            -e MYENV_VAR=sample-value
          runtime_args: >-
            /myapp/path/my-script.sh
```

