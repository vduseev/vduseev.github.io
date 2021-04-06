---
title:  "Getting started with Wutch"
date:   2021-04-04 22:00:00 +0200
keywords: wutch watchdog watcher liveserver sphinx jekyll gulp live server shell cli
description: "Demonstration of running wutch as live server, shell command executor, and more."
image: https://i.ibb.co/x72ZZSv/with-page-pic.png
---

![Wutch Demo](https://github.com/vduseev/wutch/raw/master/docs/_static/wutch-demo.gif)

Wutch is a python based live server that observes changes in the given directories and
executes a shell command on each change. It also, optionally, renders the results in the
default web browser, automatically refreshing each web page after every change.

You can use wutch with Sphinx, Jekyll, and other static site generators. On the GIF above
you can see how wutch builds its own documentation. It behaves just like a live server.
Adding any change causes a subprocess shell to run Sphinx (doc site generator used
by wutch) to rebuild its docs in real time.

<!--more-->

### Why wutch?

If you've ever worked with live servers popular among frontend developers you know how convenient
they are. Problem is you should either rely on the Live Server extension such as
[this one](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) or
create a build pipeline using something like `gulp`.

We wanted to create a command line tool that is able to rebuild documentation or website
on each change and automatically refresh browser's webpage afterwards.

### Features

* Watches multiple directories and glob file patters for changes
* Able to ignore multiple directories and glob file patters
* Runs given shell command on each change
* Starts a live server (with an option to bind to a given port and address)
* (optionally) Opens a browser when Wutch starts pointing to the build directory
* Automatically refreshes web page on each build
* Configurable timeout between rebuilds
* Configurable binging port and host (can be used as a development web server)

### Hot to install wutch?

At the moment, wutch is distributed as a Python package for Python >= 3.8. In the future
a Homebrew formula, as well as `.deb` and `.rpm` distributions will be added.

You can install wutch like this:

{% highlight bash %}
pip install wutch
{% endhighlight %}

Or, to install for the current user only:

{% highlight bash %}
pip install --user wutch
{% endhighlight %}

### How to use wutch?

After installation `wutch` binary becomes available. Now you can start a wutch watcher process.
By default, it will do the following

* watch current directory `./` for changes
* start a live server
* open a web browser pointing to the `./_build/index.html`
* run `sphinx-build` shell command on each change
* auto-refresh web page on each build

{% highlight bash %}
$ wutch
{% endhighlight %}

How to configure wutch to perform different actions? Wutch supports multiple sources of
configuration settings. They are loaded in a priority given below. Let's take
a `command` parameter that specifies what shell command to run as an example here.

* command line options

  {% highlight bash %}$ wutch --command "bundle exec jekyll build"{% endhighlight %}

* environment variables starting with `WUTCH_`

  {% highlight bash %}export WUTCH_COMMAND="bundle exec jekyll build"{% endhighlight %}

* configuration file `wutch.cfg` present in the current directory

  {% highlight json %}{ "command": "bundle exec jekyll build" }{% endhighlight %}

* default variables hardcoded into `wutch` itself

  {% highlight bash %}sphinx-build{% endhighlight %}

Below is the full list of **supported parameters**:

{% highlight bash %}
-h, --help            show this help message and exit
-c COMMAND, --command COMMAND
                      Shell command executed in response to file changes. Defaults to: sphinx-build.
-p [PATTERNS ...], --patterns [PATTERNS ...]
                      Matches paths with these patterns (separated by ' '). Defaults to: ['*'].
-P [IGNORE_PATTERNS ...], --ignore-patterns [IGNORE_PATTERNS ...]
                      Ignores file changes in these patterns (separated by ' '). Defaults to: [].
-d [DIRS ...], --dirs [DIRS ...]
                      Directories to watch (separated by ' '). Defaults to: ['.'].
-D [IGNORE_DIRS ...], --ignore-dirs [IGNORE_DIRS ...]
                      Ignore file changes in these directories (separated by ' '). Defaults to: ['_build', 'build'].
-w WAIT, --wait WAIT  Wait N seconds after the command is finished before refreshing the web page. Defaults to: 3.
-b BUILD, --build BUILD
                      Build directory containing files to render in the browser. Defaults to: _build.
-I [INJECT_PATTERNS ...], --inject-patterns [INJECT_PATTERNS ...]
                      Patterns of files to inject with JS code that refreshes them on rebuild (separated by ' '). Defaults to: ['*.htm*'].
-i INDEX, --index INDEX
                      File that will be opened in the browser with the start of the watcher. Defaults to: index.html.
--host HOST           Host to bind internal HTTP server to. Defaults to: localhost.
--port PORT           TCP port to bind internal HTTP server to. Defaults to: 5010.
-B NO_BROWSER, --no-browser NO_BROWSER
                      Do not open browser at wutch launch. Defaults to: False.
-S NO_SERVER, --no-server NO_SERVER
                      Do not start the webserver, just launch the shell command. Defaults to: False.
{% endhighlight %}

#### Building Sphinx documentation with Wutch

Wutch relies on itself when building and rendering its own documentation written with
Sphinx (see [wutch docs sources](https://github.com/vduseev/wutch/tree/master/docs)).

In order to set up a Sphinx doc development using wutch's live server it's enough
to place a `wutch.cfg` file into the project folder and then simply run wutch binary.

{% highlight json %}
{
    "dirs": ["docs"],
    "patterns": ["*.rst", "*.py"],
    "command": "make -C docs build",
    "build": "docs/_build/html",
}
{% endhighlight %}

With these settings wutch will look for the changes happening in `*.rst` and `*.py`
files in the `./docs` directory. For each change, wutch will run
`make -C docs build` command from its [Makefile](https://github.com/vduseev/wutch/blob/master/docs/Makefile).
It will then open the `./docs/_build/html/index.html` file in the default browser
and will auto-refresh the browser page every rebuild.

#### Building Jekyll with Wutch

Jekyll already comes with the `jekyll serve` command that creates a server which
rebuilds the website for every change. However, jekyll is not able to refresh the
page in the browser automatically.

Imagine you have a pretty standard Jekyll project structure. Below is a structure
of this blog.

{% highlight bash %}
drwxr-xr-x  _drafts
drwxr-xr-x  _includes
drwxr-xr-x  _layouts
drwxr-xr-x  _posts
drwxr-xr-x  _site
drwxr-xr-x  assets
-rwxr-xr-x  index.md
drwxr-xr-x  pages
-rw-r--r--  robots.txt
-rwxr-xr-x  404.html
-rwxr-xr-x  Gemfile
-rw-r--r--  Gemfile.lock
-rwxr-xr-x  _config.yml
{% endhighlight %}

Here is how to build and run a live server for it with Wutch.
For this example we will be using command line options instead of the config file.

{% highlight bash %}
$ wutch --ignore-dirs _site --build _site --command "bundle exec jekyll build"
{% endhighlight %}

#### Using Wutch as live server for frontend builds

Even though there are plenty of options in frontend to implement a live server that
watches for changes and rebuilds CSS and HTML, still, wutch is capable of that too.

Imagine you have a Gulp based build pipeline for your website and you can run a build
using `gulp build` command. Then all results of the build are dropped into the `build`
directory of the project.

In that case we'd need to look for the changes in the current directory, but ignore
the changes in the `build` folder.

{% highlight bash %}
$ wutch --ignore-dirs "build" --build "build" --command "gulp build"
{% endhighlight %}

### More about Wutch

* Documentation: [wutch.readthedocs.io](https://wutch.readthedocs.io)
* Repo: [github.com/vduseev/wutch](https://github.com/vduseev/wutch)
* PyPI package: [pypi.org/project/wutch](https://pypi.org/project/wutch/)
