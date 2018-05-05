---
title:  "Fastest way to get directory size in Python"
date:   2018-04-25 22:00:00 +0200
keywords: howto python benchmark fasters directory size os walk scandir listdir
description: "Fastest way to get directory size in Python"
image: https://image.ibb.co/jdwRhS/vim_python_pipenv.jpg
---

> What is the fastest way to calculate directory size in Pyhton?

Is it better to do `os.walk` or to retreat to the custom recursive function based on `os.scandir`? Or maybe a platform dependent solution such as `du -s` on Unix is the best? Let's benchmark each approach and find out.

**Note**: `os.scandir` has been included into Python 3.5 and replaced `os.listdir` in the standard `os.walk` implementation. 

<!--more-->

I have stumbled upon this question while I was working on [`s3push`](https://github.com/vduseev/s3push) project that facilitates directory uploading to AWS S3 storage. One of the [features](https://github.com/vduseev/s3push/issues/12) present in the tool is to display a progress bar in the terminal while uploading the files to S3. In order to implement this, one must calculate the total size of the files before uploading them. Sounds easy, right? 

<a name = "os_walk_getsize_loop"></a>
{% highlight python %}
import os


def os_walk_getsize_loop(path):
    size = 0
    from os.path import join, getsize
    for root, dirs, files in os.walk(path):
        for filename in files:
            fullname = join(root, filename)
            size += getsize(fullname)
    return size
{% endhighlight %}

However, that implies that the tree has to be traversed two times: first, for the total size calculation, and then a second time, during the actual process of file uploading. Which brings up a question: *Can we speed up a directory size calculation?*

Previously, before version 3.5, Python has been using `os.listdir` inside the `os.walk` function, and it made `os.walk` pretty slow to the point where people started questioning the nature of Python's slow directory size calculation (see this amazing discusssion on [StackOverflow.com](https://stackoverflow.com/questions/2485719/very-quickly-getting-total-size-of-folder/)). 
The reason behind it is that `os.listdir` calls OS functions to get the names of the files in the directory, and then separately makes additional calls to determine whether the file is a directory or a file. 

[Ben Hoyt](http://benhoyt.com/) investigated this and wrote the [`scandir`](https://pypi.org/project/scandir/) package which is now included into core Python as a part of the `os` module. I highly recommend reading the [full story](http://benhoyt.com/writings/scandir/) behind creation and development of *os.scandir* as a demonstration of what improving a piece of Python could look like and how much effort it takes to get the improvement accepted into the Python codebase.

As Ben [points out](http://benhoyt.com/writings/scandir/#why-is-scandir-needed), these days, Windows, Linux, and Mac actually return the file type info along with the list of files in the directory. In contrast to the *listdir* where file type was requested through a separate `stat` system call. 
Moreover, Windows even returns the corresponding sizes of the files. `os.scandir` takes advantage of this fact; thus, it replaced *listdir* in *os.walk* function as a more efficient solution.

Can we take an advantage of the fact that `os.scandir` might already have the sizes of the files without making additional system calls? Even though both `os.walk` and `os.scandir` are now basically the same with sheer difference between them being the recursive nature of *os.walk*, we can still try. However, we can't obtain file sizes from results provided by *os.walk*, which yileds a tuple consisting of current invistigated path, list of directories in it, and a list of file names in it. In contrast, *os.scandir* yields `DirEntry` objects that contain a lot of additional information.

We will compare `os.scandir` based approach with several modifications of `os.walk` to find out which one will deliver the fastest results. We should also consider system level caching, when, for example, i-nodes get added to the block cache. To exclude block cache warmup we perform an initial run on the directory and then do the real measurement.

Here is a couple of modifications of original *os.walk* based approach. They are not really useful but nevertheless are interesting to consider in the benchmark, just for fun.

### Call `os.lstat` explicitly to get file size

Here we are making an explicit call to operating system's `stat` function through `os.lstat()` method.

<a name = "os_walk_lstat"></a>
{% highlight python %}
def os_walk_lstat(path):
    size = 0
    import stat
    for root, dirs, files in os.walk(path):
        for filename in files:
            fullname = os.path.join(root, filename)
            st = os.lstat(fullname)
            if not stat.S_ISLNK(st.st_mode):
                size += st.st_size
    return size
{% endhighlight %}

### Sum up sizes of files in each directory using a list comprehension

This should get us the exact same result as in the [basic *os.walk* implementation](#os_walk_getsize_loop), because list comprehension we are doing is just a shorter way to write down that internal loop in the original version.

<a name = "os_walk_getsize_sum"></a>
{% highlight python %}
def os_walk_getsize_sum(path):
    size = 0
    from os.path import join, getsize
    for root, dirs, files in os.walk(path):
        size += sum([
            getsize(join(root, name)) for name in files
        ])
    return size
{% endhighlight %}

### Call `os.scandir` recursively to calculate directory size

Now let's implement a function that will use `os.scandir` to list the files in the directory. We should take into account that *os.scandir* is not recursive, which means we need to implement recursion ourselves. Also the call to `DirEntry.stat()` can sometimes throw a `FileNotFoundError` exception. I noticed this when testing *os.scandir* in a working directory of some *Ruby* based tool. Turns out the tool has been creating symbolink links to store some of its configurations data. That is not forbidden in Unix, because you can store virtually anything in symlink, and it will have the size equal to the byte size of information stored in it, be it a redirection path or any data you put in it.
With above mentioned our *scandir* based function would look something like this.

<a name = "os_scandir_recursive"></a>
{% highlight python %}
def os_scandir_recursive(path):

    def scandir_recursive(path):
        for entry in os.scandir(path):
            if entry.is_dir(follow_symlinks=False):
                yield from scandir_recursive(entry.path)
            else:
                yield entry

    size = 0
    for entry in scandir_recursive(path):
        try:
            size += entry.stat().st_size
        except FileNotFoundError:
            continue
    return size
{% endhighlight %}

One more approach we should try is calling the `du` *(disk usage)* tool in Unix to get the directory size. Obviously, this is a platform dependent solution and we are creating an otherwise unnecessary Python wraper around a simple tool, adding an overhead. Since `du` is an external tool we will use *suprocess* Python module to make this call.

**Call `du -sh` in a subprocess**

<a name = "os_du_subprocess"></a>

```
2018-05-03 05:20:14,224 dummy.py     INFO Creating test benchmark directory at benchmark_tree
2018-05-03 05:22:08,279 dummy.py     INFO Priming the system's cache...
2018-05-03 05:22:36,859 dummy.py    DEBUG Benchmarking iteration #1/10...
2018-05-03 05:25:02,995 dummy.py    DEBUG Benchmarking iteration #2/10...
2018-05-03 05:27:27,380 dummy.py    DEBUG Benchmarking iteration #3/10...
2018-05-03 05:29:57,638 dummy.py    DEBUG Benchmarking iteration #4/10...
2018-05-03 05:32:22,316 dummy.py    DEBUG Benchmarking iteration #5/10...
2018-05-03 05:34:46,547 dummy.py    DEBUG Benchmarking iteration #6/10...
2018-05-03 05:37:10,902 dummy.py    DEBUG Benchmarking iteration #7/10...
2018-05-03 05:39:34,326 dummy.py    DEBUG Benchmarking iteration #8/10...
2018-05-03 05:42:07,593 dummy.py    DEBUG Benchmarking iteration #9/10...
2018-05-03 05:44:55,726 dummy.py    DEBUG Benchmarking iteration #10/10...
2018-05-03 05:47:34,497 dummy.py     INFO os_walk_lstat method; best: 32.608; average: 34.670
2018-05-03 05:47:34,497 dummy.py     INFO os_walk_getsize_loop method; best: 32.280; average: 33.608
2018-05-03 05:47:34,497 dummy.py     INFO os_walk_getsize_sum method; best: 32.521; average: 34.245
2018-05-03 05:47:34,497 dummy.py     INFO os_scandir_recursive method; best: 30.202; average: 32.375
2018-05-03 05:47:34,497 dummy.py     INFO os_du_subprocess method; best: 14.282; average: 14.860
2018-05-03 05:47:34,497 dummy.py     INFO Removing test benchmark directory at benchmark_tree
```

