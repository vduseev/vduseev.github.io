---
title: Work Environment
permalink: /work-environment/
layout: page
sitemap: false
---

## MacBook and workstation

* **SmcFanControl**

## Homebrew

## Terminal

**tldr: iTerm2 + Fish + Fonts + OMF + BobTheFish theme**

## Vim

* [MacVim](https://github.com/macvim-dev/macvim) – the editor of choice.

  MacOS comes preinstalled with Vim. However, I recommend installing MacVim for better compatibility with plugins. Installation instructions can be found in MacVim READMEs. After installing MacVim App I create symbolic link to the `mvim` binary.

  ```
  sudo ln -s /Applications/MacVim.app/Contents/bin/mvim /usr/local/bin/vim
  ```

  This will override system's Vim symbolic link which is located in `/usr/bin/` directory.

* [This article](https://realpython.com/vim-and-python-a-match-made-in-heaven/) tells how to turn Vim into a Python IDE

* YouCompleteMe – autocompletion plugin

  - Install via Vundle (takes some time since repository is 200+ MB)
  - Then run `python3 install.py` in `~/.vim/bundle/YouCompleteMe`

## Python: PyEnv & Pipenv

I work so much with Python that having a clean and manageable Python installation is crucial for me. I broke system's installation of Python on Linux and MacOS numerous times. So the best recipe is to keep away from system's Python as far as possible. Here is the recipe I finally employed after trial and error and a presentation at PyData Warsaw 2017.

* PyEnv – is a manager of python installations, forked from rbenv. It is important to remember that Pyenv is only able to control which Python interpreter will be invoked in a particular directory or project. Pyenv can also install Python interpreters for you. However, it has nothing to do with Python library installations, virtual environments, or with Pip. Pyenv keeps all Python installations in `~/.pyenv/` directory.

* Pipenv - official package manager for Python.

## Ruby: Rbenv

## Java: SdkMan

## AWS CLI

I deploy to AWS a lot. I believe it's fine to use AWS GUI when you explore things, but otherwise it is better to write scripts to achieve results. Be it Bash scripts that use AWS CLI or Python scripts that use boto3 library. Writing scripts guarantees that when you forget how to properly deploy a cluster of ElasticSearch instances and shards you will just use your script instead of researching AWS documentation again. AWS CLI is a Python library installed via pip.

I try to keep the installation of AWS CLI isolated from everything else. Making it possible to have multiple installations with different versions. Here is how I achieve that:

* `~/.aws/` directory. This directory will be automatically created when you configure your AWS CLI with `aws configure` command. However, I prefer to create it ahead of time, because the installation will be kept here.
* `~/.aws/.python-version` file that contains Python version that will be used for AWS CLI. As of now, this file contains `3.6.5`. This, of course, assumes that you use Pyenv to manage Python installations. Also the version that you chose must have Pipenv installed in it.
* `awscli` library. Once you are in `~/.aws/` directory just run
  ```
  pipenv install awscli
  ```
  This will create a dedicated virtual environment and install AWS CLI there so that it's exclusively available only from that place. Pipenv will know which virtual environment to use in this directory thanks to the `Pipfile.lock` file.
* We have an isolated and clean installation of AWS CLI. Now, how can we make it available across the system? At the moment the only way to start AWS CLI is to fire up Pipenv from the directory with Pipfile.
  ```
  pipenv run aws
  ```
  Instead, let's make a symlink that will redirect any requests to `aws` command into our dedicated virtual environment. It's pretty easy.
  * Create `~/.aws/bin/` directory.
  * Create `~/.aws/bin/aws` file there with the following script inside:
    ```
    #!/usr/bin/env bash
    # The line above ensures cross compatibility in MacOS

    # Set ENV variable for PyEnv to know which interpreter to use
    # This will not work if you no longer have 3.6.5 version in Pyenv!
    export PYENV_VERSION=3.6.5

    # Set PIPENV location ENV variable to tell Pipenv where to look for virtual environment.
    export PIPENV_PIPFILE=~/.aws/Pipfile

    # When this command is executed following happens:
    # 1. Pyenv starts and uses shims to look for the pipenv module in the
    #    3.6.5 installation of Python. Then it starts pipenv
    # 2. Pipenv reads Pipfile location from environment variable of the
    #    shell that we just set and finds the aws executable
    #    in the dedicated virtual evnironment.
    # 3. We pass all the arguments in our script to aws executable using
    #    "$@" bash directive.
    pipenv run aws "$@"
    ```
  * Make the script executable
    ```
    chmod a+x ~/.aws/bin/aws
    ```
  * Create a symlink to this executable script
    ```
    sudo ln -s ~/.aws/bin/aws /usr/local/bin
    ```
* Voilà, the `aws` executable is now available from any directory.
  ```
  $ pwd
  /Users/user/

  $ aws --version
  aws-cli/1.15.0 Python/3.6.5 Darwin/17.4.0 botocore/1.10.0
  ```


## Atom

## Pass

## JetBrains

