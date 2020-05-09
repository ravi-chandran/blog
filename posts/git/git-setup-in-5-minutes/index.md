I've had these notes regarding `git` setup in Linux for a while and decided it would be good to add it to the blog for future reference. Setting up `git` and SSH is a little tricky. For example, if you don't have all the file permissions just right for your SSH setup, things will just not work. Setup involves piecing together information from multiple web searches, so collecting it all here is useful.

Some of the notes also explain how you can copy `git` and SSH configurations from Windows to a Linux VM (e.g. WSL2 or VirtualBox-based).

<!-- TEASER_END -->

## Installation
Install `git`. Note that `gitk` is optional. I like `gitk` for quickly reviewing the changes from each commit in a file.

- Install `git` and `gitk`:
```bash
sudo apt update
sudo apt upgrade
sudo apt install git gitk
```

- If you already use `git` on Windows, you can find your `config` file and keys from there. On Windows, you can get to the `.ssh` folder using `cd %homedrive%%homepath%\.ssh` from a command prompt.

## SSH Setup
- Create folder for SSH setup:
```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cd ~/.ssh
nano config  # create git config file
```

- Save the following in a file named `~/.ssh/config`:
```bash
Host github.com
  User git
  Hostname github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/github_id_rsa
```

- I'm assuming you already have private and public SSH keys named `github_id_rsa` and `github_id_rsa.pub` for your GitHub account. If not, create them (e.g. see [here](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)).

- Copy `github_id_rsa` and `github_id_rsa.pub` into `~/.ssh`.

- Set the appropriate file permissions:
```bash
cd ~/.ssh
cp /media/sf_sharelinux/github_id_rsa .
cp /media/sf_sharelinux/github_id_rsa.pub .
touch authorized_keys
touch known_hosts

chmod 600 github_id_rsa
chmod 644 github_id_rsa.pub
chmod 644 config
chmod 644 authorized_keys
chmod 644 known_hosts
```

- Verify that the files and ownership are similar to the following:
```bash
rc@bb:~/.ssh$ ll
total 20
drwx------  2 rc rc 4096 Nov 23 13:54 ./
drwxr-xr-x 16 rc rc 4096 Nov 23 13:17 ../
-rw-r--r--  1 rc rc    0 Nov 23 13:54 authorized_keys
-rw-r--r--  1 rc rc  122 Nov 23 13:22 config
-rw-------  1 rc rc 1675 Nov 23 13:54 github_id_rsa
-rw-r--r--  1 rc rc  401 Nov 23 13:54 github_id_rsa.pub
-rw-r--r--  1 rc rc    0 Nov 23 13:54 known_hosts
```
- If the user and group don't both match your username, use `chown` to set them that way, e.g. `chown rc.rc filename`

## Git Configuration
- If you already have `.gitconfig` under Windows (in the `%homedrive%%homepath%` folder), you can copy it into the Linux home directory. Alternatively, set up the username and email for `git` as follows. Note that the no-reply email is used (which can be configured in [GitHub](https://help.github.com/en/github/setting-up-and-managing-your-github-user-account/setting-your-commit-email-address)):
```bash
git config --global user.email 48181110+ravi-chandran@users.noreply.github.com
git config --global user.name ravi-chandran
```

- You can check your `git` configuration, especially any non-global options with:
```bash
git config --list --show-origin
```

## Verify Setup
- Verify SSH connection to GitHub with `ssh -T git@github.com` which should respond with a success message.

- Additional checks: Clone a repository to verify configuration. Verify that you can push as well.
