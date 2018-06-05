---
title: "Perfect AWS CLI setup"
date: 2018-04-07 10:00:00 +0200
keywords: howto aws cli env environment version pipenv pyenv isolated installation
description: "How to perfectly set up AWS CLI on a Unix system in a clean and isolated fashion using pyenv and pipenv"
image: https://image.ibb.co/eKxuNo/aws_cli_thumb.jpg
redirect_from: 
  - /aws-cli-isolated/
  - /perfect-aws-cli-setup/

---

I deploy to AWS a lot. I believe it's fine to use AWS GUI when you explore things, but otherwise it is better to write scripts to achieve results. Be it Bash scripts that use AWS CLI or Python scripts that use boto3 library. Writing scripts guarantees that when you forget how to properly deploy a cluster of ElasticSearch instances and shards you will just use your script instead of researching AWS documentation again. AWS CLI is a Python library installed via pip.

![AWS CLI isolated clean installation on Unix]({{ page.image }})

I try to keep the installation of AWS CLI isolated from everything else. Making it possible to have multiple installations with different versions. Here is how I achieve that on MacOS.

<!--more-->

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

    {% highlight bash %}
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
    {% endhighlight %}

  * Make the script executable

    ```
    chmod a+x ~/.aws/bin/aws
    ```

  * Create a symlink to this executable script

    {% highlight bash %}
    sudo ln -s ~/.aws/bin/aws /usr/local/bin
    {% endhighlight %}

* Voilà, the `aws` executable is now available from any directory.

  {% highlight bash %}
  $ pwd
  /Users/user/

  $ aws --version
  aws-cli/1.15.0 Python/3.6.5 Darwin/17.4.0 botocore/1.10.0
  {% endhighlight %}
