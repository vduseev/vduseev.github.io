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

## Other ways to calculate directory size 

Let's consider other options to calculate directory size. Here is a couple of modifications of original *os.walk* based approach. They are not really that useful but nevertheless are interesting to consider in the benchmark, just for fun.

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

### Sum up file sizes in each directory using a list comprehension and `sum` function

This should get us the exact same result as in the [basic *os.walk* implementation](#os_walk_getsize_loop), because list comprehension we are doing is just a shorter way to rewrite that internal loop in the original version.

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

Now let's implement a function that will use `os.scandir` to iterate over files in the directory. 
We should take into account that *os.scandir* function is not recursive, which means we need to implement recursion ourselves. 

The call to `DirEntry.stat()` can sometimes throw a `FileNotFoundError` exception. So, with above mentioned, our *scandir* based function would look something like this.

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

### Call `du -sh` in a subprocess to calculate directory size

One more approach we should try is calling the `du` *(disk usage)* tool in Unix to get the directory size. Obviously, this is a platform dependent solution and we are creating an otherwise unnecessary Python wraper around a simple tool, adding an overhead. Since `du` is an external tool we will use *subprocess* Python module to make this call.

Even before we run our benchmark, it is quite obvious, that the `du` based approach will produce the fastest results. But are those results reliable enough? 

Well, they are not. The `du` tool calculates disk usage in terms of occupied file system blocks. By default, that block size is equal to 512 bytes. That means that it is highly unlikely that `du` will produce an output equal to the actual size of the data. 

Of course, on some systems `du` has a `-b` flag that instructs the tool to report an actual size with a byte level precision. But introducing that flag will make our solution even more platform dependent.

<a name = "os_du_subprocess"></a>
{% highlight python %}
def os_du_subprocess(path):
    import subprocess
    size = subprocess.check_output(
        ['du', '-sh', path]
    ).split()[0].decode('utf-8')
    return size
{% endhighlight %}

### Call a subprocess with `find`, `ls`, and `aws` pipelined

It is possible to get an actual directory size using default system tools. If we recursively find all files using `find` command and execute `ls -l` on them, we can feed the results to `awk` tool to sum up bytes in every line and print the final result. But that can't be anywhere as fast as `du -s`.

Here is the command that will be called in a subprocess:

{% highlight bash %}
find $MY_PATH -type f -exec ls -l '{}' \; | awk '{sum+=$5} END {print sum}'
{% endhighlight %}

Transformed into a python function, with a `shell=True` option enabled it looks like this.

{% highlight python %} 
def os_ls_subprocess(path):
    import subprocess
    size = subprocess.check_output(
        'find ' + path + ' -type f -exec ls -l \'{}\' \; | awk \'{sum+=$5} END {print sum}\'',
        shell=True
    ).split()[0].decode('utf-8')
    return size
{% endhighlight %}

## Benchmark results

Benchmark creates a test directory with 4 levels of nesting and 5 files in each directory, including root. After a warmup directory size calculation all approaches are tested several times.

| Rank     |Method | Average (s)    | Best (s)    | Precision |
|-|-|-|-|-|
| 1 | `du` | 0.028 | 0.025 | **No** |
| 2 | `os.scandir` (recursive) | 0.067 | 0.064 | Yes |
| 3 | `os.walk` with `getsize()` | ~0.085 | 0.080 | Yes |
| 4 | `os.walk` with `lstat` | 0.097 | 0.092 | Yes |
| 5 | `find` + `ls -l` + `awk` | 48.906 | 46.606 | Yes |
| | | | |


## Conclusion 

As seen from the table above, `du` is the fastest method to obtain a directory size, taking `0.028` seconds on average in this test. If you don't mind a small precision loss and platform dependability, then it's a perfect solution.

A platform independent and fast solution is the recursive `os.scandir` approach. It is about `35%` faster than `os.walk` based solutions. In tests on Mac OS X *scandir* based method demonstrated 26-48% improvement in speed.

Explicit `os.lstat` calls add an overhead roughly equal to 12%, though I can't completely understand the reason behind it.

## Try for yourself

The benchmark is [published on GitHub](https://github.com/vduseev/python-directory-size-benchmark). Feel free to test and use!


