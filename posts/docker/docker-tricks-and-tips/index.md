In a previous [post](../dockerizing-a-build-system/), I covered an approach for dockerizing a software build system. In this article, I discuss some techniques I've found useful while iterating on a Dockerfile to get it just right. For example, if the Dockerfile involves downloading and installing a 5GB file, each iteration of "`docker image build`" could take a lot of time even with good network speeds.

<!-- TEASER_END -->

On the surface, creating a Dockerfile for a build system seems like a straightforward exercise: simply implement the same steps in a Dockerfile that you'd perform if you were installing the items directly. Unfortunately, I've found that it usually doesn't quite work that way, and a few "tricks" are handy for such DevOps exercises.

In the [tutorial repository](https://github.com/ravi-chandran/dockerize-tutorial) from the previous post, I've added a [folder](https://github.com/ravi-chandran/dockerize-tutorial/tree/master/tutorial2_docker_tricks) with an example covering some of these tricks which I'll walk through in this post.


## Organize Build Tool I/O
The build inputs and outputs, and the scripts that configure and invoke the tools should be outside the image and the eventually running container. These inputs and outputs are best accessed by setting up docker volumes. I covered this extensively in a previous [post](../dockerizing-a-build-system/) but wanted to emphasize this as it's been a useful convention for my work.


## Saving Time On Docker Image Build Iterations
Using a local HTTP server is useful to avoid downloading large files multiple times from the internet during "`docker image build`" iterations. To illustrate this by example, let's say we need to create a docker image with Anaconda 3 under Ubuntu 18.04. The Anaconda 3 installer is a ~0.5GB file, so I'll use this as our "large" file for this example.

Note that I don't want to use the docker `COPY` instruction as it creates a new layer. I want to delete the large installer after using it to minimize the docker image size. One could use [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/), but I've found this approach sufficient and quite effective.

The basic idea is to use a Python-based HTTP server locally to serve the large file(s) and have the Dockerfile `wget` the large file(s) from this local server. Let's explore the details of how to set this up effectively. The full example is provided [here](https://github.com/ravi-chandran/dockerize-tutorial/blob/master/tutorial2_docker_tricks/).


The necessary contents of the folder `tutorial2_docker_tricks/` in this example repository are outlined here:
```bash
tutorial2_docker_tricks/
├── build_docker_image.sh                   # builds the docker image
├── run_container.sh                        # instantiates a container from the image
├── install_anaconda.dockerfile             # Dockerfile for creating our target docker image
├── .dockerignore                           # used to ignore contents of the installer/ folder from the docker context
├── installer                               # folder with all our large files required for creating the docker image
│   └── Anaconda3-2019.10-Linux-x86_64.sh   # from https://repo.anaconda.com/archive/Anaconda3-2019.10-Linux-x86_64.sh
└── workdir                                 # example folder used as a volume in the running container
```

The key steps of the approach are:

- Place the large file(s) in the `installer/` folder. In this example, we have the large Anaconda installer file `Anaconda3-2019.10-Linux-x86_64.sh`. Note that you won't get this file if you clone my [git repository](https://github.com/ravi-chandran/dockerize-tutorial/). But you can download the installer from [here](https://repo.anaconda.com/archive/Anaconda3-2019.10-Linux-x86_64.sh) to follow along with the example. Note that only you, as the docker image creator, needs this source file. The end users of the docker image don't.

- Create the `.dockerignore` file and have it ignore the `installer/` folder to avoid Docker from copying all the large files into the build context.

- In a terminal, `cd` into the `tutorial2_docker_tricks/` folder and execute the build script as "`./build_docker_image.sh`".

- In `build_docker_image.sh`, we start the Python HTTP server to serve any files from the `installer/` folder. 
```python
cd installer
python3 -m http.server --bind 10.0.2.15 8888 &
cd ..
```

- If you're wondering about the strange IP address, I'm working with a VirtualBox Linux VM, and `10.0.2.15` shows up as the address of the Ethernet adapter when I run `ifconfig`. This IP seems to be the convention used by VirtualBox. If your setup is different, you'll need to update this IP address in `build_docker_image.sh` and `install_anaconda.dockerfile` appropriately. The server's port number is set to 8888 for this example.


- As the HTTP server is set to run in the background, I stop the server near the end of the script with the "`kill -9`" command using a cool approach I found [here](https://stackoverflow.com/a/37214138).
```bash
kill -9 `ps -ef | grep http.server | grep 8888 | awk '{print $2}'
```

- You will note that I also have this same "`kill -9`" earlier in the script before starting the HTTP server. In general, when I iterate on any build script which I might deliberately interrupt, this ensures a clean start of the HTTP server each time.

- In the [Dockerfile](https://github.com/ravi-chandran/dockerize-tutorial/blob/master/tutorial2_docker_tricks/install_anaconda.dockerfile), there is a "`RUN wget`" instruction that downloads the Anaconda installer from the local HTTP server. It also deletes the installer file and cleans up after the installation all within the same layer to keep the image size to a minimum.
```dockerfile
# install Anaconda by downloading the installer via the local http server
ARG ANACONDA
RUN wget --no-proxy http://10.0.2.15:8888/${ANACONDA} -O ~/anaconda.sh \
    && /bin/bash ~/anaconda.sh -b -p /opt/conda \
    && rm ~/anaconda.sh \
    && rm -fr /var/lib/apt/lists/{apt,dpkg,cache,log} /tmp/* /var/tmp/*
```

- After the build is complete, you should see a docker image `anaconda_ubuntu1804:v1` present. (You can list the images with "`docker image ls`").

- You can instantiate a container from this image using `./run_container.sh` at the terminal while in the folder `tutorial2_docker_tricks/`. You can verify that Anaconda is installed as follows:
```bash
$ ./run_container.sh 
$ python --version
Python 3.7.5
$ conda --version
conda 4.8.0
$ anaconda --version
anaconda Command line client (version 1.7.2)
```

- You'll note that `run_container.sh` sets up a volume `workdir`. In this example repository, the folder `workdir/` is empty. This is a convention I use to set up a volume where I can have my Python and other scripts that are independent of the docker image.


## Non-Root User
An important aspect of I/O concerns the ownership of the build tool output files. By default, since Docker runs as `root`, the output files would be owned by `root` which is unpleasant. We typically want to work as a non-`root` user. Changing the ownership after the build output is generated can be done with scripts but is an additional unnecessary step. It's best to set the [`USER`](https://docs.docker.com/engine/reference/builder/#user) argument in the Dockerfile at the earliest point possible.
```dockerfile
ARG USERNAME
# other commands...
USER ${USERNAME}
```
The `USERNAME` can be passed in as a build argument (`--build-arg`) when executing the "`docker image build`". You can see an example of this in the example [Dockerfile](https://github.com/ravi-chandran/dockerize-tutorial/blob/master/tutorial2_docker_tricks/install_anaconda.dockerfile) and corresponding [build script](https://github.com/ravi-chandran/dockerize-tutorial/blob/master/tutorial2_docker_tricks/build_docker_image.sh).

Some portions of the tools may also need to be installed as a non-`root` user. So the sequence of installations in the Dockerfile may need to be different from the way it's done if you were installing manually and directly under Linux.


## Minimizing Image Size
Each `RUN` command is equivalent to executing a new shell, and each `RUN` command creates a layer. The naive approach of mimicking installation instructions with separate `RUN` commands may eventually break at one or more steps, and will also result in a larger image. Chaining multiple installation steps in one `RUN` command, and including the `autoremove`, `autoclean` and `rm` commands as in the example below is useful to minimize the size of each layer.
```dockerfile
RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
       apt-get -y --quiet --no-install-recommends install \
       # list of packages being installed go here \
    && apt-get -y autoremove \
    && apt-get clean autoclean \
    && rm -fr /var/lib/apt/lists/{apt,dpkg,cache,log} /tmp/* /var/tmp/*
```

Besides this, ensure that you have a `.dockerignore` file in place to ignore items that don't need to be sent to the Docker build context.


## Non-Interactive Installation
I've found the `DEBIAN_FRONTEND=noninteractive apt-get -y --quiet --no-install-recommends` options for the `apt-get install` instruction (as in the example above) necessary to prevent the installer opening dialog boxes. Note that these options should be used as part of the `RUN` instruction. The `DEBIAN_FRONTEND=noninteractive` should not be set as an environment variable (`ENV`) in the Dockerfile as explained [here](https://docs.docker.com/engine/faq/) as it will be inherited by the containers.



## Logging Build And Run Output
Save a typescript of everything that happened during the docker image build or container run session using a simple `tee` in the bash scripts. In other words, just add "`|& tee $BASH_SOURCE.log`" to the end of the "`docker image build`" and the "`docker image run`" commands in your scripts. See the examples in the [image build](https://github.com/ravi-chandran/dockerize-tutorial/blob/master/tutorial2_docker_tricks/build_docker_image.sh) and [container run](https://github.com/ravi-chandran/dockerize-tutorial/blob/master/tutorial2_docker_tricks/run_container.sh) scripts. 

What this `tee`-ing technique does is generate a file with the same name as the bash script but with a "`.log`" extension appended to it so that you know which script it originated from. Everything you see printed to the terminal when running the script will get logged to this file with a similar name.

This is especially valuable for users of your Docker images to report issues to you when something doesn't work. You can ask them to send you the log file to help diagnose the issue. Many tools generate so much output as to easily overwhelm the default size of the terminal's buffer. Relying only on the terminal's buffer capacity to copy-paste error messages may not be sufficient for diagnosing issues if the cause of the issue can only be seen much earlier in the output.

I've also found this to be useful even in the Docker image-building scripts especially when using the Python-based HTTP server discussed above. The server generates so many lines during a download that it typically overwhelms the terminal's buffer.


## Default Shell Selection
The default shell assumed by Docker is `sh` for Linux. I'm more familiar with `bash` and so I have a tendency to override the default with `bash` early on in the Dockerfile like this:
```dockerfile
SHELL ["/bin/bash", "-c"]
```
I don't think there is a significant impact for the types of commands I use in Dockerfiles, so this is just precaution. The script invoked at the Dockerfile entrypoint should have the appropriate [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) anyway.
