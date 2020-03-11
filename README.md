# docker-colab-local
[![Docker Cloud Build Status](https://img.shields.io/docker/cloud/build/aecampos/colab-local.svg?label=build)](https://hub.docker.com/r/aecampos/colab-local)

A Docker container to act as a local runtime for [Google Colab](https://colab.research.google.com) or private jupyter server with BERT preinstalled.

## Get the Image

```
docker pull sorokine/docker-colab-local:latest
```

`docker run` can download the image automatically but `docker pull` will ensure that you have the latest build.

## Run
```bash
docker run \
  --runtime=nvidia \
  -it --rm -p 8081:8081 \
  sorokine/docker-colab-local:latest
```
Or, to mount a volume so that it's accessible to colab:
```bash
docker run \
  --runtime=nvidia \
  -it --rm -p 8081:8081 \
  -v "$PWD":/opt/colab \
  sorokine/docker-colab-local:latest
```

where `"$PWD"` is a full path on your host machine to put your notebooks (other than the ones hosted on colab) and other files created by the notebooks (e.g., computed indices).

To make downloaded BERT models persistent run as:

```bash
docker run \
  --runtime=nvidia \
  -it --rm -p 8081:8081 \
  -v "$PWD":/opt/colab \
  -v $HOME/.cache/torch:/root/.cache/torch \
  sorokine/docker-colab-local:latest
```

This will use the same cache directory as the models run on the host (if your host is Linux or Mac).

## Making Nvidia Work

* on some docker versions replace `--runtime=nvidia` with `--gpus all`.
* if you do not have a GPU or Nvidia drivers installed completely omit all nvidia flags from docker command.  The notebook will work but it will be slow.
* if you do not have the latest CUDA installed on your system check which [tags](https://hub.docker.com/r/sorokine/docker-colab-local/tags) are available in docker hub repo and use the one for your version of CUDA, e.g.:

```bash
docker run \
  --runtime=nvidia \
  -it --rm -p 8081:8081 \
  -v "$PWD":/opt/colab \
  -v $HOME/.cache/torch:/root/.cache/torch \
  sorokine/docker-colab-local:10.1
```

* To find the version of CUDA on your system run `nvidia-smi`.  

## Connecting

If the container isn't running on your local machine, you'll need to forward port 8081.  Run this command from the system where you are runing your browser:
```
ssh MACHINE_WHERE_DOCKER_IS_RUNNING -L 8081:localhost:8081
```

In Colaboratory, click the "Connect" button and select "Connect to local runtime...". Enter the port 8081 step in the dialog that appears and click the "Connect" button. (from [colaboratory](https://research.google.com/colaboratory/local-runtimes.html)).  Only `localhost` hostname is accepted (no numeric IPs).  Replace the token in the dialog box with the token that is shown in the terminal after starting docker container.  The connection string should look like `http://localhost:8081/?token=abcdef123456....`. 

If your recieve an error like `0.0.0.0:8081 failed: port is already allocated.`. try running container on a diferent port, for example for port 8082:

```bash
docker run \
  --runtime=nvidia \
  -it --rm -p 8082:8081 \
  -v "$PWD":/opt/colab \
  -v $HOME/.cache/torch:/root/.cache/torch \
  sorokine/docker-colab-local:latest
```

Colab notebooks expect the port in connection and on the server to be the same.  To fix this problem without rebuilding the image forward the port with ssh: 
```
$ ssh MACHINE_WHERE_DOCKER_IS_RUNNING -L 8081:localhost:8082
```

## Running on Remote System 

This is a single command to run the image on a remote system with the GPU enabled and using the specified port:

```
COLAB_PORT=8082 ssh REMOTE_HOST_NAME -L 8081:localhost:8082 \
  docker run \
  --gpus=all \
  -p $COLAB_PORT:$COLAB_PORT \
  -v "$PWD":/opt/colab \
  -v $HOME/.cache/torch:/root/.cache/torch \
  --env -e COLAB_PORT=$COLAB_PORT \
  -it --rm \
  sorokine/docker-colab-local:latest
```

## Notes

* if some packages are missing install them with `!pip install` in your notebook.  This has to be repeated on kernel restart.

# TODO

plans and pressing ussues will go here
