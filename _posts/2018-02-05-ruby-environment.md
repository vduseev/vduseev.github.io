---
title: "Choosing between Ruby environment managers: rbenv vs. RVM"
date: 2018-02-05 02:29:01 +0200
keywords: ruby environment mac os x rvm rbenv rubymine bundler
description: "RVM vs. rbenv: why RVM is bad and rbenv is good. Easy setup instructions for MacOS X."
image: https://image.ibb.co/nHjNQH/rbenv_vs_rvm.jpg
---
{%- assign rbenv = '[`rbenv`][rbenv_website]' -%}
{%- assign rvm = '[`RVM`][rvm_website]' -%}
{%- assign gems = '[gems][what_is_gem]' -%}
{%- assign gemsets = '[gemsets][what_is_gem]' -%}
{%- assign pyenv = '[`pyenv`][pyenv]' -%}
{%- assign bundler = '[Bundler][bundler]' -%}

Using a system's default Ruby interpreter to develop projects and install required {{gemsets}} is a sure path to dependency nightmare. Each project needs its own environment independent from others. At the same time, the system's installation of Ruby should be left intact.

This article gives a quick overview of the popular Ruby environment managers and provides a concise installation instruction for a painless setup on MacOS X.

![rbenv vs. RVM](https://image.ibb.co/nHjNQH/rbenv_vs_rvm.jpg)

<!--more-->

*To quickly jump to the environment manager installation click [here](#macos-x-installation).*

## Overview

There are two main options for environment management when it comes to Ruby. First one is the {{rvm}}. RVM was the first decent environment manager which could manage multiple interpreters and per-project based {{gemsets}}, it is still very popular among Ruby developers. Although, not only the installation process of it is somewhat unusual for these days, but is also messes up with a lot of things in the system in a magic and incomprehensible way. For instance, {{rvm}} ensures that a correct version of Ruby will be utilized in a project by replacing original `cd` command in `*nix` systems, which is way too much for nothing else but Ruby environment manager.

The other known option is called {{rbenv}}. It's a much more simpler and relieable solution, and it does not mess the system up, keeping everything local and manageable.

### RVM and its downsides

So, what's wrong with the {{rvm}}? The installation of {{rvm}} is quite bizarre for anyone without a background in Linux systems administration, and project's homepage leaves even more for a reader to guess regarding RVM's internal logic and implementation. {{rvm}} gives an impression of the tool that mingles with the system in an irreversible way. Obviously, it's a tested, trusted, and community verified tool that does not perform any malicious actions with your system, but the first impression it gives doesn't look like something you would expect from a user friendly environment manager.

The installation requires you to add a pair of public GPG keys of project's maintainers to your GPG keychain and then download and run a 1000 lines long bash script that will do the installation for you.

*First, adding GPG keys:*

{% highlight bash %}
# Add public GPG key of the RVM's authors to your GPG keychain
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

# Result:
gpg: key 105BD0E739499BDB: public key "Piotr Kuczynski <piotr.kuczynski@gmail.com>" imported
gpg: key 3804BB82D39DC0E3: 82 signatures not checked due to missing keys
gpg: key 3804BB82D39DC0E3: public key "Michal Papis (RVM signing) <mpapis@gmail.com>" imported
gpg: no ultimately trusted keys found
gpg: Total number processed: 2
gpg:               imported: 2
{% endhighlight %}

Next thing that RMV's website will suggest is to run the following command to continue installation:
{% highlight bash %}
\curl -sSL https://get.rvm.io | bash -s stable
{% endhighlight %}
This line downloads and runs RVM's bash installation script.

Before running any unknown script I usually stop and suspend the installation process. You never know what that script is going to do to your system unless your read through it. RVM skips the "read" part right away. I'd call such installation process a bad practice. How difficult will it be to remove RVM from the system and clean up the traces? Was it to hard for creators to enable a Homebrew based installation? You can't tell until you spend a considerable amount of time searching for the answers. And, again, doing a deep research of what's supposed to be a simple Ruby environment manager is not something you would expect from a user friendly tool.

Interestingly enough, if prior to going all in and running an unknown script you think ahead and decide to check the official website for the **uninstalling instructions, you won't find them**. There is a single mention of `implode` command on the [RVM's CLI usage][rvm_cli_usage] page. But according to that page:
```
implode   - removes all ruby installations it manages, everything in ~/.rvm
```
`implode` does not uninstall the RVM itself. Basically, there is no way to automatically uninstall {{rvm}} other than manually cleaning up everything that the installation did to your system. And that's a huge downside.

> The only way to uninstall {{rvm}} is to manually clean up everything it did to your system.

If you proceed with the installation and run the second step from RVM's homepage, the bash script will be downloaded and executed.

{% highlight bash %}
# Download RVM's installation script and install it
\curl -sSL https://get.rvm.io | bash -s stable

# Result:
Downloading https://github.com/rvm/rvm/archive/1.29.3.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.3/1.29.3.tar.gz.asc
gpg: Signature made Sun Sep 10 22:59:21 2017 CEST
# ...
Installing RVM to /Users/vduseev/.rvm/
    RVM PATH line found in /Users/vduseev/.mkshrc.
    RVM PATH line not found for Bash or Zsh, rerun this command with '--auto-dotfiles' flag to fix it.
    Adding rvm loading line to /Users/vduseev/.profile /Users/vduseev/.bash_profile /Users/vduseev/.zlogin.
Installation of RVM in /Users/vduseev/.rvm/ is almost complete:
# ...
Searching for binary rubies, this might take some time.
No binary rubies available for: osx/10.13/x86_64/ruby-2.4.1.
# ...
Installing Ruby from source to: /Users/vduseev/.rvm/rubies/ruby-2.4.1, this may take a while depending on your cpu(s)...
# ...
  * To start using RVM you need to run `source /Users/vduseev/.rvm/scripts/rvm`
    in all your open shell windows, in rare cases you need to reopen all shell windows.
{% endhighlight %}

If you are as impatient as me, you would most likely hope that at least the usage of RVM is clear and simple after all that unknown compilation-installation-mess. I would argue that it's not. Let's think about a first time Ruby user who managed to install {{rvm}}. After spending around 10 minutes reading, trying to understand, and installing {{rvm}} the user will search the homepage for the link that describes some usage examples or the basics:

![RVM homepage documentation links](https://image.ibb.co/hwr9Cn/rvm_doc_links.jpg)

Did you find a page that explains the [usage][rvm_basics]? Or maybe, [CLI usage][rvm_cli_usage] would be a better page to look at for the  first time user? You would certainly hope that at least the basics are short and concise, because an environment manager is not the final goal of the whole process. The final goal is to develop using Ruby in a clean and isolated fashion.

The usage instruction of RVM is muddy and unclear. The understanding of correct way to use the tool might require even more time than its installation. So, if you have installed RVM by mistake, here is the way to clean everything up and uninstall {{rvm}} completely.

### Cleaning up after RVM

RVM can be uninstalled in a several steps. Here are two topics on StackOverflow covering the uninstallation of {{rvm}}: [How can I remove RVM (Ruby Version Manager) from my system?][rvm_uninstall_stackoverflow_1] and [Howto Uninstall RVM \[duplicate\]][rvm_uninstall_stackoverflow_2].

#### {% increment rvm_remove_step %}. Delete RVM installed rubies

If you have already installed and integrated {{rvm}} into your shell, you will need to start with the `rvm implode` command.
{% highlight bash %}
# Check that RVM exists in your path
echo $PATH | grep rvm

# If grep does not return anything, then RVM is not integrated into your shell
# yet. Otherwise, run "rvm implode" to uninstall ruby interpreters
# installed by RVM
rvm implode
{% endhighlight %}

#### {% increment rvm_remove_step %}. Removing the ~/.rvm directory

Some size stats before removing the directory
{% highlight bash %}
# Check disk usage of the ~/.rvm directory
# -s displays an entry for each specified file or directory
# -h prints the result in a "Human-readable" format.
#    e.g., Byte, Kilobyte, Megabyte, Gigabyte, Terabyte and Petabyte.
du -sh ~/.rvm
# Result:
181M	/Users/vduseev/.rvm
{% endhighlight %}

The pure default installation of {{rvm}} on MacOS takes 181 Mb.

Remove RVM's directory
{% highlight bash %}
rm -rf ~/.rvm
{% endhighlight %}

#### {% increment rvm_remove_step %}. Cleaning shell configuration files
Check the following files for the references to RVM and remove the RVM line from each of them:
```
~/.bashrc
~/.bash_profile
~/.profile
~/.zshrc
~/.zlogin
```

References look like this:
{% highlight bash %}
[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*
{% endhighlight %}

We can do a quick `grep` to check if {{rvm}} put itself there.
{% highlight bash %}
cat ~/.bashrc | grep rvm
# Result:
[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*
{% endhighlight %}

#### {% increment rvm_remove_step %}. Removing unnecessary GPG keys

To remove public GPG keys of RVM maintainers we will need their user names. We can find them in the full list of public keys stored in GPG.
{% highlight bash %}
gpg --list-keys
# Result:
# ...
pub   rsa4096 2016-11-11 [SC]
      7D2BAF1CF37B13E2069D6956105BD0E739499BDB
uid           [ unknown] Piotr Kuczynski <piotr.kuczynski@gmail.com>
sub   rsa4096 2016-11-11 [E]

pub   rsa4096 2014-10-28 [SC]
      409B6B1796C275462A1703113804BB82D39DC0E3
uid           [ unknown] Michal Papis (RVM signing) <mpapis@gmail.com>
uid           [ unknown] Michal Papis <michal.papis@toptal.com>
uid           [ unknown] [jpeg image of size 5015]
sub   rsa2048 2015-11-02 [E]
sub   rsa4096 2014-10-28 [S] [expires: 2019-03-09]
# ...
{% endhighlight %}

You can use any representation of user name in any `uid` row listed in a key to delete it. I will use email address.
{% highlight bash %}
# Delete key
gpg --delete-key piotr.kuczynski@gmail.com
# Result:
gpg (GnuPG) 2.2.3; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

pub  rsa4096/105BD0E739499BDB 2016-11-11 Piotr Kuczynski <piotr.kuczynski@gmail.com>

Delete this key from the keyring? (y/N) y
# Piotr's key is removed now

# Delete Michal's key
gpg --delete-key mpapis@gmail.com
gpg (GnuPG) 2.2.3; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

pub  rsa4096/3804BB82D39DC0E3 2014-10-28 Michal Papis (RVM signing) <mpapis@gmail.com>

Delete this key from the keyring? (y/N) y
# Both public keys are now removed
{% endhighlight %}

### rbenv

{{rbenv}} is short for `Ruby Environment`. It's a different command line tool that enables you to quickly and easily switch between different rubies installed on your system.

What I like about {{rbenv}} is it's simplicity and obviousness. It might look especially familiar for those coming from Python background, because rbenv, in fact, looks a lot like {{pyenv}} — a python version manager forked from rbenv and modified for Python.

`rbenv` is easy in development. All you need is to create a `.ruby-version` file in a directory and from that moment the version specified in this file will be used in this particular directory.

It bundles greatly with {{bundler}}. In fact, `bundler` takes care of *gem* management completely. Each installation of *Ruby* gets its own installation of bundler. Installation can be configured so that it reuses gems for projects that utilize same Ruby version. Otherwise, gemsets are installed on per-project basis.

`rbenv` works by introducing a directory full of small executables called `shims` into your path. The path to this directory looks like this: `/Users/vduseev/.rbenv/shims`. Each *shim* is a tiny Bash script that has the exact same name as any Ruby interpreter based tool in your system.

For example, before calling the actual `Jekyll` executable, a shim will check the `.ruby-version` files and `RBENV_VERSION` environmental variable as well as global `~/.rbenv/version` file if other options did not work. *Shim* will then pull out a correct version of Jekyll according to your specified Ruby version. To be able to do that, the directory with shims is placed first in the `$PATH`. This way any call to `Jekyll` in our case will be at first directed to a *shim*.

Let's take a look at how a user would use `rbenv`.

#### Checking what Ruby versions are available

{% highlight bash %}
rbenv install --list

# Alternatively: rbenv install -l

# Result:
Available versions:
  1.8.5-p52
  1.8.5-p113
  1.8.5-p114
  1.8.5-p115
  1.8.5-p231
  1.8.6
  1.8.6-p36
# ...
  2.5.0-rc1
  2.5.0
  2.6.0-dev
  2.6.0-preview1
# ...
  jruby-9.1.15.0
  jruby-9.1.16.0
  jruby-9.2.0.0-dev
# ...
  ree-1.8.7-2012.01
  ree-1.8.7-2012.02
  topaz-dev
{% endhighlight %}

The whole list is much longer. You can see that beside the versions of the standard interpreter there is a whole bunch of other implementations.

#### Installing Ruby versions

Any version is installed into the `~/.rbenv directory`. You can later specify which installed version you want to use in your project.

{% highlight bash %}
rbenv install 2.5.0
{% endhighlight %}

#### Install bundler

{% highlight bash %}
gem install bundler
{% endhighlight %}

#### Install gems

Now, if you have a `Gemfile` ready, you can perform an installation of all required gems.

{% highlight bash %}
bundle install
{% endhighlight %}

#### Choose Ruby version for project

The order in which Ruby version is checked by `rbenv` is perfectly described in the docs. It is hard to rephrase it better:

> When you execute a shim, rbenv determines which Ruby version to use by reading it from the following sources, in this order:
>
> The `RBENV_VERSION` environment variable, if specified. You can use the rbenv shell command to set this environment variable in your current shell session.
>
> The first `.ruby-version` file found by searching the directory of the script you are executing and each of its parent directories until reaching the root of your filesystem.
>
> The first `.ruby-version` file found by searching the current working directory and each of its parent directories until reaching the root of your filesystem. You can modify the .ruby-version file in the current working directory with the rbenv local command.
>
> The global `~/.rbenv/version` file. You can modify this file using the rbenv global command. If the global version file is not present, rbenv assumes you want to use the "system" Ruby—i.e. whatever version would be run if rbenv weren't in your path.

## Rbenv installation on MacOS X <a name="macos-x-installation"/>
A proper installation of ruby environment manager on MacOS X requires only few steps. Most importantly, `rbenv` has to be installed through **Homebrew**, and `rbenv init` command has to be called.

### Step 1
Install `rbenv`
```
brew install rbenv
```
Note that this also installs ruby-build, so you'll be ready to install other Ruby versions out of the box.

### Step 2
Initialize `rbenv` on your OS.
```
rbenv init
```

### That's it!

Don't forget to call `rbenv rehash` after installing new gems to give rbenv a change to generate new *shims*.

[rbenv_website]: https://github.com/rbenv/rbenv
[rvm_website]: https://rvm.io/
[rvm_cli_usage]: https://rvm.io/rvm/cli
[rvm_basics]: https://rvm.io/rvm/basics
[what_is_gem]: http://guides.rubygems.org/what-is-a-gem/
[rvm_uninstall_stackoverflow_1]: https://stackoverflow.com/questions/3558656/how-can-i-remove-rvm-ruby-version-manager-from-my-system
[rvm_uninstall_stackoverflow_2]: https://stackoverflow.com/questions/3950260/howto-uninstall-rvm
[pyenv]: https://github.com/pyenv/pyenv
[bundler]: http://bundler.io/
