---
title: "Choosing between Ruby environment managers: rbenv vs. RVM"
date: 2018-02-05 02:29:01 +0200
categories: dev ruby environment macos mac os x rvm rbenv rubymine bundler
---
{%- assign rbenv = '[`rbenv`][rbenv_website]' -%}
{%- assign rvm = '[`RVM`][rvm_website]' -%}
{%- assign gems = '[gems][what_is_gem]' -%}

Using a system's default Ruby interpreter to develop projects and install required {{gems}} is a sure path to dependency nightmare. Each project needs its own environment independent from other projects. At the same time, the system's installation of Ruby should be left intact.

This article gives a quick overview of the popular Ruby environment managers and provides a concise installation instruction for a painless setup on MacOS X.

![rbenv-help](https://image.ibb.co/jxxdhn/rbenv_help.jpg)

*To quickly jump to the environment manager installation click [here](#macos-x-installation).*

<!--more-->

## Overview

There are two main options for environment management when it comes to Ruby. First one, is the {{rvm}}. Being the first decent environment manager which could manage multiple interpreters and per-project based {{gems}}, it is still very popular among Ruby developers. Although, not only the installation process of it is somewhat unusual for these days, but is also messes up a lot of things in a magic and incomprehensible way.

The other option is called {{rbenv}}. It's a much more simpler and relieable solution, and it does not mess the system up, keeping everything local and manageable.

### RVM and its downsides

So, what's wrong with the {{rvm}}? The installation of {{rvm}} is quite bizarre for anyone without a background in Linux systems administration, and project's homepage leaves even more for a reader to guess regarding RVM's internal logic and implementation. {{rvm}} gives an impression of the tool that mingles with the system in an irreversible way. Obviously, it's a tested, trusted, and community verified tool that does not perform any malicious actions with your system, but the first impression it gives doesn't look like something you would expect from a user friendly environment manager.

The installation requires you to add a pair of public GPG keys of project's maintainers to your GPG keychain and then download and run a 1000 lines long bash script that will do the installation for you.

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

Next thing that RMV's website will suggest is to run this command for installation: `\curl -sSL https://get.rvm.io | bash -s stable`. This line downloads and runs RVM's installation Bash script.

Before running any unknown script I usually stop and suspend the installation process. You never know what that script is going to do to your system unless your read through it. RVM skips the "read" part right away. I'd call such installation process a bad practice. How difficult will it be to remove RVM from the system and clean up the traces? Was it to hard for creators to enable a Homebrew based installation? You can't tell until you spend a considerable amount of time searching for the answers. And, again, doing a deep research of what's supposed to be a simple Ruby environment manager is not something you would expect from a user friendly tool.

Interestingly enough, if prior to going all in and running an unknown script you think ahead and decide to check the official website for the **uninstalling instructions, you won't find them**. There is a single mention of `implode` command on the [RVM's CLI usage][rvm_cli_usage] page. But according to that page:
```
implode   - removes all ruby installations it manages, everything in ~/.rvm
```
`implode` does not uninstall the RVM itself. Basically, there is no way to automatically uninstall {{rvm}} other than manually cleaning up everything that the installation did to your system. And that's a huge downside.

> The only way to uninstall {{rvm}} is to manually clean up everything it did to your system.

If you proceed with the installation and run the second step from RVM's homepage, the bash script will be downloaded and executed. It takes exactly **6 minutes** on my MacBook Pro with 2.6 GHz Intel Core i5 to make a default `-s stable` installation.

{% highlight bash %}
# Download RVM's installation script and install it
\curl -sSL https://get.rvm.io | bash -s stable

# Result:
Downloading https://github.com/rvm/rvm/archive/1.29.3.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.3/1.29.3.tar.gz.asc
gpg: Signature made Sun Sep 10 22:59:21 2017 CEST
gpg:                using RSA key E206C29FBF04FF17
gpg: Good signature from "Michal Papis (RVM signing) <mpapis@gmail.com>" [unknown]
gpg:                 aka "Michal Papis <michal.papis@toptal.com>" [unknown]
gpg:                 aka "[jpeg image of size 5015]" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 409B 6B17 96C2 7546 2A17  0311 3804 BB82 D39D C0E3
     Subkey fingerprint: 62C9 E5F4 DA30 0D94 AC36  166B E206 C29F BF04 FF17
GPG verified '/Users/vduseev/.rvm/archives/rvm-1.29.3.tgz'

Installing RVM to /Users/vduseev/.rvm/
    RVM PATH line found in /Users/vduseev/.mkshrc.
    RVM PATH line not found for Bash or Zsh, rerun this command with '--auto-dotfiles' flag to fix it.
    Adding rvm loading line to /Users/vduseev/.profile /Users/vduseev/.bash_profile /Users/vduseev/.zlogin.
Installation of RVM in /Users/vduseev/.rvm/ is almost complete:

  * To start using RVM you need to run `source /Users/vduseev/.rvm/scripts/rvm`
    in all your open shell windows, in rare cases you need to reopen all shell windows.
Ruby enVironment Manager 1.29.3 (latest) (c) 2009-2017 Michal Papis, Piotr Kuczynski, Wayne E. Seguin

Searching for binary rubies, this might take some time.
No binary rubies available for: osx/10.13/x86_64/ruby-2.4.1.
Continuing with compilation. Please read 'rvm help mount' to get more information on binary rubies.
Checking requirements for osx.
Certificates bundle '/usr/local/etc/openssl@1.1/cert.pem' is already up to date.
Requirements installation successful.
Found user configured '-j' flag in 'rvm_make_flags', please note that RVM can detect number of CPU threads and set the '-j' flag automatically if you do not set it.
Installing Ruby from source to: /Users/vduseev/.rvm/rubies/ruby-2.4.1, this may take a while depending on your cpu(s)...
ruby-2.4.1 - #downloading ruby-2.4.1, this may take a while depending on your connection...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 11.9M  100 11.9M    0     0  1487k      0  0:00:08  0:00:08 --:--:-- 1736k
ruby-2.4.1 - #extracting ruby-2.4.1 to /Users/vduseev/.rvm/src/ruby-2.4.1....
ruby-2.4.1 - #applying patch /Users/vduseev/.rvm/patches/ruby/2.4.1/random_c_using_NR_prefix.patch.
ruby-2.4.1 - #configuring...................................................................
ruby-2.4.1 - #post-configuration.
ruby-2.4.1 - #compiling..............................................................
ruby-2.4.1 - #installing.......
ruby-2.4.1 - #making binaries executable..
ruby-2.4.1 - #downloading rubygems-2.6.14
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  751k  100  751k    0     0   648k      0  0:00:01  0:00:01 --:--:--  648k
No checksum for downloaded archive, recording checksum in user configuration.
ruby-2.4.1 - #extracting rubygems-2.6.14....
ruby-2.4.1 - #removing old rubygems.........
ruby-2.4.1 - #installing rubygems-2.6.14...........................
ruby-2.4.1 - #gemset created /Users/vduseev/.rvm/gems/ruby-2.4.1@global
ruby-2.4.1 - #importing gemset /Users/vduseev/.rvm/gemsets/global.gems...............................................
ruby-2.4.1 - #generating global wrappers........
ruby-2.4.1 - #gemset created /Users/vduseev/.rvm/gems/ruby-2.4.1
ruby-2.4.1 - #importing gemsetfile /Users/vduseev/.rvm/gemsets/default.gems evaluated to empty gem list
ruby-2.4.1 - #generating default wrappers........
ruby-2.4.1 - #adjusting #shebangs for (gem irb erb ri rdoc testrb rake).
Install of ruby-2.4.1 - #complete
Ruby was built without documentation, to build it run: rvm docs generate-ri
Creating alias default for ruby-2.4.1...

  * To start using RVM you need to run `source /Users/vduseev/.rvm/scripts/rvm`
    in all your open shell windows, in rare cases you need to reopen all shell windows.
{% endhighlight %}

If you are as impatient as me, you would most likely hope that at least the usage of RVM is clear and simple after all that unknown compilation-installation-mess. I would argue that it's not. Let's think about a first time Ruby user who managed to install {{rvm}}. After spending around 10 minutes reading, trying to understand, and installing {{rvm}} the user will search the homepage for the link that describes some usage examples or the basics:

![RVM homepage documentation links](https://image.ibb.co/hwr9Cn/rvm_doc_links.jpg)

Did you find a page that explains the [usage][rvm_basics]? Or maybe, [that][rvm_cli_usage] would be a better page to look at for the  first time user? You would certainly hope that at least the basics are short and concise, because an environment manager is not the final goal of the whole process. The final goal is to develop using Ruby in a clean and isolated fashion.

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

#### {% increment rvm_remove_step %}. Removing unnecessary GPG keys

### rbenv



## MacOS X Installation
A proper installation of ruby environment manager on MacOS X requires only few steps. Most importantly, `rbenv` has to be installed through Homebrew, and `rbenv init` command has to be called.

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

## Install Ruby versions



[rbenv_website]: https://github.com/rbenv/rbenv
[rvm_website]: https://rvm.io/
[rvm_cli_usage]: https://rvm.io/rvm/cli
[rvm_basics]: https://rvm.io/rvm/basics
[what_is_gem]: http://guides.rubygems.org/what-is-a-gem/
[rvm_uninstall_stackoverflow_1]: https://stackoverflow.com/questions/3558656/how-can-i-remove-rvm-ruby-version-manager-from-my-system
[rvm_uninstall_stackoverflow_2]: https://stackoverflow.com/questions/3950260/howto-uninstall-rvm