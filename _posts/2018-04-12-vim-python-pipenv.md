---
title:  "Can Vim detect Pipenv environment?"
date:   2018-04-12 08:00:00 +0200
categories: howto vim python environment ycm youcompleteme virtualenv pipenv .vimrc
summary: "How to configure Vim to auto-discover Pipenv environment"
image: https://image.ibb.co/jdwRhS/vim_python_pipenv.jpg
---

**Vim** can be a great IDE. If I could you use just one word to describe it, that word would be "fast". Vim can be easily configured to be a powerful IDE for Python development. However, as times change, as does the official recommended packaging tool for Python – it was **Pip** before, now it's **Pipenv**, a high level wrapper around `pip` and `virtualenv`.

![vim-python-pipenv](https://image.ibb.co/jdwRhS/vim_python_pipenv.jpg)

Setting up a proper integration between Vim and Pipenv may seem like a cumbersome task. It entirely depends on what you want to achieve. This article focuses on integration between **Vim**, **Pipenv**, and **YouCompleteMe** – a fast, fuzzy-search code completion engine for Vim. But some of the options described in the article are suitable for other setups as well.

<!--more-->

## Vim as a Python IDE

I could talk about Vim all day, if I had a chance. Vim requires less resources to run than most of the modern IDEs. It's faster in many ways, even when it comes to syntax verification, file reading, and handling of large repositories. Vim has a tremendously large amount of commands, plugins, and shortcuts which are all configurable and allow you to navigate and edit in a blink of an eye. And last but not least, you can usually find an instance of Vim (or at least `vi`) on any Unix based OS, which proves to be especially useful for DevOps engineers. Same powerful editor on a server and on your workstation.

Here is a small list of plugins you could install to turn Vim into a Python IDE:

{% highlight vim %}
" Smart auto-indentation for Python
Plugin 'vim-scripts/indentpython.vim'
" Auto-completing engine
Plugin 'Valloric/YouCompleteMe'
" Syntax checker
Plugin 'vim-syntastic/syntastic'
' Python backend for "syntastic'
Plugin 'nvie/vim-flake8'
" Status bar (powerline)
Plugin 'vim-airline/vim-airline'
" Awesome staring screen for Vim
Plugin 'mhinz/vim-startify'
" File manager
Plugin 'scrooloose/nerdtree'
" Search bar
Plugin 'kien/ctrlp.vim'
" Theme
Plugin 'crusoexia/vim-monokai'
" Powerful commenting utility
Plugin 'scrooloose/nerdcommenter'
" Rich python syntax highlighting
Plugin 'kh3phr3n/python-syntax'
{% endhighlight %}

Obviously, there is so much more to Vim than just a list of plugins. Vim's configuration file – `.vimrc` – can become quite large, just because there are so many things you can configure in Vim. However, complete configuration of Vim for Python development is a topic of a different article.

## VirtualEnv Support

On of the issues one can discover is that Vim by default has no notion of virtual environment you are running in. It counts on the default python interpreter found in the `PATH` environment variable.

The known classic way to make Vim aware of project's virtual environment is to check the list of environment variables to detect whether we are running inside a virtual environment and activate it. This approach is documented on the [realpython.com](https://realpython.com/vim-and-python-a-match-made-in-heaven/#virtualenv-support):

{% highlight vim %}
"python with virtualenv support
py << EOF
import os
import sys
if 'VIRTUAL_ENV' in os.environ:
project_base_dir = os.environ['VIRTUAL_ENV']
activate_this = os.path.join(project_base_dir, 'bin/activate_this.py')
execfile(activate_this, dict(__file__=activate_this))
EOF
{% endhighlight %}

Unfortunately, this no longer works if we deal with Pipenv based virtual environments. The environment itself might not even be in the same directory, or not even under default path `~/.local/share/virtualenvs/`.

However, we can take advantage of Pipenv's own commands to determine if we are supposed to be running in a virtual environment.

{% highlight bash %}
pipenv --venv
{% endhighlight %}

When we have a PyEnv installed it will detect our call to `pipenv`. Let's take a look at possible situations:

### Directory is not configured for PyEnv or Pipenv

In this case both python interpreter and Pipenv are not configured for the current directory. PyEnv reports that it could, theoretically, run `pipenv`, which is available in the `3.6.5` installation, but would need you to indicate that explicitly.

{% highlight bash %}
# Remove PyEnv configuration file for experiment
$ rm .python-version
$ pipenv --venv
pyenv: pipenv: command not found

The `pipenv` command exists in these Python versions:
3.6.5
{% endhighlight %}
The same behavior will be observed if `pipenv` is not installed as a package in the interpreter of choice. Let's say, we specify a python version `3.4.3` in a `.python-verison` file, which has no `pipenv` installed. Same error will be thrown out by PyEnv in that case.

### Directory is configured for PyEnv, but no virtual environment is created yet

Here, `.python-version` file exists for PyEnv to detect. And the python interpreter that we specified in the file has Pipenv installed as a package. In that case Pipenv will indicate that no virtual environment has been created yet.

{% highlight bash %}
# Create PyEnv configuration file
$ echo "3.6.5" > .python-version
$ pipenv --venv
No virtualenv has been created for this project yet!
{% endhighlight %}

### Directory is configured for PyEnv and virtual environment exists

In this case `pyenv --venv` returns full path to the virtual environment utilized in this project.

{% highlight bash %}
$ pipenv --venv
/Users/vduseev/.local/share/virtualenvs/.aws-MRQhgJfv
{% endhighlight %}

## Making it work in `.vimrc`

We can take advantage of the fact that whenever we call `pipenv --venv` its return code will indicate successful (`sys.exit(0)`) or unsuccessful (`sys.exit(1)`) run.

There is a variable named `shell_error` in Vim, and, as you probably figured out, it contains the last exit code of the executed shell command.

If we make a call to `pipenv --venv` from `.vimrc`, we will either get a correct path, or one of the error messages we observed earlier. In order to make a call we can use the `system()` function of Vim:

{% highlight vim %}
let pipenv_venv_path = system('pipenv --venv')
{% endhighlight %}

Then, simply by checking the value of `shell_error` variable, we can find out if the call was successful.

{% highlight vim %}
if shell_error = 0
  " do one thing
else
  " do another thing
endif
{% endhighlight %}

Since in this article we set up Python's auto-completion for YouCompleteMe engine, we will use YCM's global variable `g:ycm_python_binary_path` to point it to the correct executable of proper virtual environment.

{% highlight vim %}
let venv_path = substitute(pipenv_venv_path, '\n', '', '')
let g:ycm_python_binary_path = venv_path . '/bin/python'
{% endhighlight %}

The final script that we'll add to the `.vimrc` will lok like this:

{% highlight vim %}
" Point YCM to the Pipenv created virtualenv, if possible
" At first, get the output of 'pipenv --venv' command.
let pipenv_venv_path = system('pipenv --venv')
" The above system() call produces a non zero exit code whenever
" a proper virtual environment has not been found.
" So, second, we only point YCM to the virtual environment when
" the call to 'pipenv --venv' was successful.
" Remember, that 'pipenv --venv' only points to the root directory
" of the virtual environment, so we have to append a full path to
" the python executable.
if shell_error == 0
  let venv_path = substitute(pipenv_venv_path, '\n', '', '')
  let g:ycm_python_binary_path = venv_path . '/bin/python'
else
  let g:ycm_python_binary_path = 'python'
endif
{% endhighlight %}

This script can be further modified to support different completion engines or even other types of plugins, but the essence remains the same – determine whether Vim is running in the correctly configured Pipenv directory.