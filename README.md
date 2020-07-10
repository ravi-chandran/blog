# RC Notebook

These notes are for my blog setup in an Ubuntu 18.04 VM as well as Windows.

For the Ubuntu VM, differences in setup between VirtualBox and WSL2 VMs are noted where applicable.


## Installation
- For `wsl2` with `Ubuntu-1804`, install the following:
```bash
sudo apt update
sudo apt install python3-venv
```

- Create GitHub repository called `blog`.

- Then clone the `blog` repository and install Nikola in its root folder.
```bash
cd ~/repos/blog
git clone git@github.com:ravi-chandran/blog.git
cd blog
python3 -m venv nikola
source ./nikola/bin/activate
python3 -m pip install --upgrade pip setuptools wheel
python3 -m pip install --upgrade "Nikola[extras]"
```

- For Windows:
```bat
cd D:\repos\blogging\blog
python -m venv nikola
nikola\Scripts\activate
python -m pip install --upgrade pip setuptools wheel
python -m pip install --upgrade "Nikola[extras]"
```

- Add these to `.bash_aliases` for convenience in a VirtualBox VM:
```bash
alias blog="cd ~/repos/blog"
alias nik="source ./nikola/bin/activate"
```

- Add these to `.bash_aliases` for convenience in a WSL2 VM:
```bash
alias blog="cd /mnt/d/repos/blogging/blog"
alias nik="pushd . > /dev/null; cd ; source ./nikola/bin/activate; popd > /dev/null"
```

- Add these to `.bashrc` for convenience in a WSL2 VM (Note VcXsrv also needs appropriate installation/configuration):
```bash
# X11 Forwarding
#export DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf 2>/dev/null):0
# WSL2 is a VM that uses a virtual network switch to connect the VM to Windows 10.
# To use X11 forwarding with WSL2, the DISPLAY environment variable cannot point to localhost or 127.0.0.1.
# Instead, DISPLAY must point to the Windows 10 machine's IP address on the virtual network switch which changes on each boot.
# Fortunately, /etc/resolv.conf has this Windows 10 machine's IP address in the line with nameserver.
# We can grab this IP and set it to the DISPLAY variable with the following one-liner:
export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0.0

# add Google Chrome (or whatever browser you use) as default browser
export BROWSER='/mnt/c/Program Files (x86)/Google/Chrome/Application/chrome.exe'

# get the eth0 IP address and save it for use with nikola serve -b -a $MYIPADDR
export MYIPADDR=$(ifconfig eth0 | grep 'inet ' | awk '{print $2}')
```

- If it's the first time, initialize the repository as a Nikola site:
```bash
cd ~/repos/blog
source ./nikola/bin/activate
nikola init .
```

## Usage

### Create New Post
```bash
blog  # gets to my repo
nik   # activates the nikola virtual environment

# create a folder for the category if it doesn't already exist
mkdir -p posts/somecategory

# create post in the appropriate folder under posts - each folder represents a category
nikola new_post -t "This is my title" posts/somecategory
```

### Build and Review Post
```bash
nikola build
nikola serve -b -a $MYIPADDR    # for WSL2
# nikola auto -b -a $MYIPADDR   # alternate option
```

### Publish
1. Commit the post's files to `git`

2. Deploy the post to the blog:
```bash
nikola github_deploy
```