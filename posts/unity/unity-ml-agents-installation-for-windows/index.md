Here are some notes on how to set up [Unity ML Agents](https://github.com/Unity-Technologies/ml-agents) under Windows 10. The instructions are consolidated from multiple locations. They've been cleaned up or updated to make sense for the current Unity 2019.30f3 version, the current [`ml-agents`](https://github.com/Unity-Technologies/ml-agents) and Windows 10.

<!-- TEASER_END -->

These instructions do not use [Anaconda](https://www.anaconda.com/) but just the "standard" Python from `www.python.org`. I prefer to use the standard Python together with `venv` based virtual environments where possible. Anaconda installations together with `conda` virtual environments could be used just as well but require a lot more disk space and installation time.

These notes follow the instructions [here](https://github.com/Unity-Technologies/ml-agents/blob/master/docs/Installation.md) and [here](https://github.com/Unity-Technologies/ml-agents/blob/master/docs/Using-Virtual-Environment.md#python-version-requirement-required) but are customized for my conventions:

- Use folder `D:\repos` for all `git` repositories.
- Use folder `D:\repos\zz-python-envs` to store `venv` virtual environments across many projects.
- Use batch files to activate environments for convenience (no need to remember the weird paths for activation)
- Extra attention to using a 64-bit Python installer. (I've inadvertently used the 32-bit Python installer before and it resulted in unexplainable errors.)
- Use "`python -m pip install`" rather than just "`pip install`" as the former is safer and is the [officially recommended approach](https://docs.python.org/3/installing/index.html#basic-usage).

If you don't have a `D:` drive, you can perform the same setup under the `C:` drive instead, or in any other folder.

## Python and ML-Agents Toolkit Setup
At the time of writing this, the ML-Agents Toolkit [did not support Python 3.8](https://github.com/Unity-Technologies/ml-agents/blob/master/docs/Using-Virtual-Environment.md#python-version-requirement-required). So we'll use Python 3.7.6 which is the latest at this time.

- Download the Python 3.7.6 (not 3.8) installer from [here](https://www.python.org/downloads/). Ensure that it is 64-bit version. [This](https://www.python.org/ftp/python/3.7.6/python-3.7.6-amd64.exe) is the direct path for the 64-bit version of Python 3.7.6.

- Verify the installer's MD5 checksum. For more info on `certutil`, refer to this [article](https://www.lifewire.com/validate-md5-checksum-file-4037391).

<pre>
D:\Downloads><b>certutil -hashfile python-3.7.6-amd64.exe MD5</b>
MD5 hash of python-3.7.6-amd64.exe:
cc31a9a497a4ec8a5190edecc5cdd303
CertUtil: -hashfile command completed successfully.
</pre>

- Install Python 3.7.6 by running the downloaded executable. Use the default install option but check the box for "**Add Python 3.7 to `PATH`**"

- Verify the installation by running `python` from a command prompt and making sure the version is right and `AMD64` is indicated as shown below.
<pre>
D:\repos><b>python</b>
Python 3.7.6 (tags/v3.7.6:43364a7ae0, Dec 19 2019, 00:42:30) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
</pre>

- Create a batch file in `D:\repos\activatemlagents.bat` with lines:
```bat
cd D:\repos
zz-python-envs\mlagents\Scripts\activate
```

- Set up virtual environment and upgrade pip. Note that we are **not** installing `mlagents` from PyPi.
```bat
cd D:\repos
md zz-python-envs
python -m venv zz-python-envs\mlagents
activatemlagents
python -m pip install --upgrade pip
```

- Clone the ML-Agents Toolkit repository, and switch to the latest stable release.
```bat
cd D:\repos
git clone https://github.com/Unity-Technologies/ml-agents.git
cd ml-agents
git checkout latest_release
```

- Install `mlagents` from the cloned repo rather than from PyPi.
```bat
cd D:\repos
activatemlagents
cd D:\repos\ml-agents\ml-agents-envs
python -m pip install -e ./
cd ..
cd ml-agents
python -m pip install -e ./
```

- Verify installation by opening a new command prompt and executing the following:
```bat
cd D:\repos
activatemlagents
mlagents-learn --help
```

## Using ML-Agents Toolkit With Unity
For Unity 2019.30f3 or later, here are the steps to add ML-Agents to a project. These are adapted from [here](https://github.com/Unity-Technologies/ml-agents/blob/master/docs/Learning-Environment-Create-New.md#set-up-the-unity-project).

- In Unity 2019.30f3 or later, create a new 3D project or open an existing project.
- In the Unity project, install the Barracuda package: Use the `Window menu`, select `Package Manager`, click `Advanced` dropdown menu to the left of the search bar and ensure "`Show preview packages`" is checked. Search for the Barracuda package and install it.
- From the Windows file system window, navigate to the folder containing your cloned ML-Agents repository, i.e. `D:\repos\ml-agents\UnitySDK\Assets`
- Drag the `ML-Agents` folder from `D:\repos\ml-agents\UnitySDK\Assets` to the Unity Editor Project window. 
 