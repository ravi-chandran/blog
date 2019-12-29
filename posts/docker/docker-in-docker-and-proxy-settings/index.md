Docker-in-Docker is explained by its author [here](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/). In a nutshell, the recommendation is to not use "true" Docker-in-Docker. Instead, use:
```bash
docker container run --volume /var/run/docker.sock:/var/run/docker.sock ...
```

The above volume mapping provides the container with access to the Docker socket with which it can start more "sibling" containers. But the approach is not a "true" Docker-in-Docker (which is not recommended) and avoids its corresponding issues. The nice thing about this approach is it can work with any Docker image (with some additional items installed as explained in this post).

Here's a compilation of useful proxy settings that may be required if you work in an organization using proxies and require Docker-in-Docker for your work. In addition, I also outline how to update your Dockerfile so that the corresponding image can support this Docker-in-Docker approach. 

<!-- TEASER_END -->

# Dockerfile Changes To Support Docker-in-Docker
To add Docker-in-Docker, you'll need to update your Dockerfile to include the following. Note that `docker-ce` and `containerd.io` don't have to be installed. 
```dockerfile
# corkscrew is used for the git protocol through a HTTP CONNECT proxy
# Per https://docs.docker.com/install/linux/docker-ce/ubuntu/ :
#     apt-transport-https, ca-certificates, curl, gnupg-agent, software-properties-common
RUN apt-get update \
    && apt-get install -y \
        apt-transport-https \
        ca-certificates \
        corkscrew \
        dos2unix \
        gnupg-agent \
        software-properties-common \
    && apt-get -y autoremove \
	&& apt-get clean autoclean \
	&& rm -rf /var/lib/apt/lists/{apt,dpkg,cache,log} /tmp/* /var/tmp/*

# Install docker
# https://github.com/docker/docker/blob/master/project/PACKAGERS.md#runtime-dependencies
RUN apt-get update \
    && apt-get install -y \
        iptables \
        procps \
        e2fsprogs \
        xfsprogs \
        xz-utils \
        git \
        kmod \
    && apt-get -y autoremove \
	&& apt-get clean autoclean \
	&& rm -rf /var/lib/apt/lists/{apt,dpkg,cache,log} /tmp/* /var/tmp/*

# Reference: https://docs.docker.com/install/linux/docker-ce/ubuntu/
# Add Docker’s official GPG key, verify fingerprint manually
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - \
    && apt-key fingerprint 0EBFCD88 \
    && add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) \
        stable" \
    && apt-get update \
    && apt-get install -y \
        docker-ce-cli \
    && apt-get -y autoremove \
	&& apt-get clean autoclean \
	&& rm -rf /var/lib/apt/lists/{apt,dpkg,cache,log} /tmp/* /var/tmp/*
# Note: For docker-in-docker, don't install docker-ce, containerd.io

# create non-root user:group, and generate a home directory to support SSH
ARG USER_NAME
RUN adduser --disabled-password --gecos '' ${USER_NAME} \
    && adduser ${USER_NAME} sudo \
    && echo "${USER_NAME} ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# run container as non-root user from this point forward
# so that build output owner is not root
USER ${USER_NAME}

# if you need to use your SSH keys
VOLUME /home/${USER_NAME}/.ssh

# contains docker-entrypoint.sh and setproxies.sh
VOLUME /scripts

ENTRYPOINT ["/scripts/docker-entrypoint.sh"]
```

# Running The Docker-in-Docker Container
Let's assume that the Docker image is named `myproject:v1`. To start a container based on this image, run the following Bash script:
```bash
docker container run                                    \
    --volume /var/run/docker.sock:/var/run/docker.sock  \
    --volume $(pwd)/common/scripts:/scripts             \
    --volume /home/${USER}/.ssh:/home/${USER}/.ssh      \
    --user $(id -u ${USER}):$(id -g ${USER})            \
    --rm -it --name my_shell myproject:v1               \
    |& tee $BASH_SOURCE.log
```

The volume mapping "`--volume /var/run/docker.sock:/var/run/docker.sock`" provides the container with access to the Docker socket to potentially create "sibling" containers.

We need to store multiple files for proxy configuration purposes. Let's place these files in the folder `$(pwd)/common/scripts`. The volume mapping "`--volume $(pwd)/common/scripts:/scripts`" maps this folder to `/scripts` inside the container. The files we will store in the folder `$(pwd)/common/scripts` are:

  - `docker-entrypoint.sh`: container entrypoint script
  - `setproxies.sh`: proxy configuration script sourced by `docker-entrypoint.sh`
  - `01proxy`: `apt-get` proxy configuration file
  - `git-proxy.sh`: `corkscrew` configuration file

If you need to access your private repositories via SSH from the container, then the volume mapping for your SSH keys is needed.

I've explained the pipe "`|& tee $BASH_SOURCE.log`" before in a [previous post](../docker-tricks-and-tips/) containing multiple Docker tips and tricks. This line creates an output log from the container execution.

# Entrypoint Script (`docker-entrypoint.sh`)
When the docker container is run, the script `docker-entrypoint.sh` specified in the Dockerfile `ENTRYPOINT` instruction is executed. This script should include the following:

```bash
#!/usr/bin/env bash

# proxy settings
source /scripts/setproxies.sh

# add the user to the docker group
DOCKER_SOCKET=/var/run/docker.sock
DOCKER_GROUP=docker
USER=$(whoami)

if [ -S ${DOCKER_SOCKET} ]; then
    DOCKER_GID=$(stat -c '%g' ${DOCKER_SOCKET})
    sudo addgroup --gid ${DOCKER_GID} ${DOCKER_GROUP}
    sudo usermod --append --groups ${DOCKER_GROUP} ${USER}
fi

newgrp ${DOCKER_GROUP}
```

The script invokes the `setproxies.sh` script which will be explained shortly. It then adds the user to the `docker` group. Finally, it uses the `newgrp` command to log the user into the `docker` group. In essence, we're executing these [Docker post-install steps](https://docs.docker.com/install/linux/linux-postinstall/) here. I haven't found a way to do all of these steps within the Dockerfile itself which would have been more convenient.

# Proxy Settings Script (`setproxies.sh`)
In the `docker-entrypoint.sh` script above, we source `setproxies.sh`. As its name implies, the script specifies the proxy configurations for some commonly used applications which don't seem to use the proxy settings from the environment (why not?). At least these were the ones I found were needed for the build system I was working on.

```bash
#!/usr/bin/env bash

# Proxy info - update as appropriate based on your company
MY_PROXY_ADDR=http://whatever_company_proxy.com
MY_PROXY_PORT=999
MY_PROXY=$MY_PROXY_ADDR:$MY_PROXY_PORT

# environment proxy settings
echo "Setting Proxies:"
echo "- environment"
export http_proxy=$MY_PROXY
export https_proxy=$MY_PROXY
export ftp_proxy=$MY_PROXY
export no_proxy="localhost,127.0.0.1"
export HTTP_PROXY=$MY_PROXY
export HTTPS_PROXY=$MY_PROXY
export FTP_PROXY=$MY_PROXY
export NO_PROXY="localhost,127.0.0.1"
export ALL_PROXY=$MY_PROXY
export all_proxy=$MY_PROXY

# wget proxy configuration
echo "- wget"
touch ~/.wgetrc
echo "https_proxy = $MY_PROXY" > ~/.wgetrc
echo "http_proxy = $MY_PROXY" >> ~/.wgetrc
echo "ftp_proxy = $MY_PROXY" >> ~/.wgetrc
echo "no_proxy = \"localhost,127.0.0.1\"" >> ~/.wgetrc
echo "use_proxy = on" >> ~/.wgetrc

# npm proxy
echo "- npm"
npm config set proxy $MY_PROXY
npm config set https-proxy $MY_PROXY

# apt-get proxy
echo "- apt-get"
sudo cp /scripts/01proxy /etc/apt/apt.conf.d/01proxy

# using corkscrew
echo "- git proxies via corkscrew"
chmod +x /scripts/git-proxy.sh
git config --global core.gitProxy /scripts/git-proxy.sh

# cleanup
unset MY_PROXY_ADDR
unset MY_PROXY_PORT
unset MY_PROXY
```

## Environment, `wget` and `npm` Proxy Settings
We first initialize script variables for the proxy environment. These are `unset` at the end of the script to minimize "environmental pollution". Then we set `wget` and `npm` proxies.

## Proxy Setting For `apt-get`
For `apt-get`, I store the proxy settings in a file `01proxy` and copy it into `/etc/apt/apt.conf.d/01proxy`. Obviously, the contents of `01proxy` will depend on your organization's proxy settings. Typically, this file will have one or more lines of the format:
```
Acquire::http::Proxy "http://yourproxyaddress:proxyport/";
```
If you have intra-organization mirrors for repositories, then you'll also need a line such as:
```
Acquire::http::Proxy::your_mirror_url DIRECT;
```

## Proxy Setting For `git`
I found the method for git proxy configuration with the aid of `corkscrew` [here](https://wiki.archlinux.org/index.php/HTTP_tunneling). `corkscrew` is a tool for tunneling SSH through HTTP proxies. You'll need to create a file `git-proxy.sh` as shown below with your proxy information. The "`git config --global core.gitProxy /scripts/git-proxy.sh`" line in `setproxies.sh` specifies `git-proxy.sh` to be used for `corkscrew`-ing through the proxy to reach URLs starting with `git://`.
```bash
#!/usr/bin/env bash
# reference: https://wiki.archlinux.org/index.php/HTTP_tunneling
corkscrew http://yourproxyaddress proxyport "$@"
```

Besides `corkscrew`, other options for ways for handling `git` access exist. For example, if the `git://` protocol doesn't work and a GitHub mirror exists for the repository, you can directly specify substitutions using `git config` commands. Here are a couple of examples I've used:

```bash
git config --global url.https://github.com/qemu/capstone.git/.insteadOf git://git.qemu.org/capstone.git
git config --global --add url.https://github.com/qemu/keycodemapdb.git/.insteadOf git://git.qemu.org/keycodemapdb.git
```
