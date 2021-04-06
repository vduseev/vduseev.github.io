---
title:  "How to create macOS Mojave VM for VirtualBox"
date:   2018-09-28 22:00:00 +0200
keywords: vm macos mojave virtualbox howto
description: "Step by step instruction on creation of macOS mojave virtual machine to run under VirtualBox."
image: https://image.ibb.co/doDhso/twitter_architecture.jpg
---

## Environment

This is the environment I had when creating the virtual machine:
- macOS Mojave 10.14
- VirtualBox Version 5.2.18
- At least 4GB of host memory. The more, the better.
- At least ~20GB of space for the virtual machine.

## Create an ISO installation file for VirtualBox

* Download macOS Mojave from the Apple App Store.

* Create a virtual USB flash drive

{% highlight bash %}
$ hdiutil create -o /tmp/Mojave -size 8G -layout SPUD -fs HFS+J -type SPARSE
created: /tmp/Mojave.sparseimage
{% endhighlight %} 

* Mount the image of the flash drive.

{% highlight bash %}
$ hdiutil attach /tmp/Mojave.sparseimage -noverify -mountpoint /Volumes/install_build
/dev/disk3              Apple_partition_scheme
/dev/disk3s1            Apple_partition_map
/dev/disk3s2            Apple_HFS                       /Volumes/install_build
{% endhighlight %}

* Use the `createinstallmedia` tool embedded into the Installation app to copy the files to the virtual drive.

{% highlight bash %}
$ sudo /Applications/Install\ macOS\ Mojave.app/Contents/Resources/createinstallmedia --volume /Volumes/install_build/
Password:
Ready to start.
To continue we need to erase the volume at /Volumes/install_build/.
If you wish to continue type (Y) then press return: y
Erasing disk: 0%... 10%... 20%... 30%... 100%
Copying to disk: 0%... 10%... 20%... 30%... 40%... 50%... 100%
Making disk bootable...
Copying boot files...
Install media now available at "/Volumes/Install macOS Mojave"
{% endhighlight %}

* Unmount the disk image to free the resources.

{% highlight bash %}
$ hdiutil detach /Volumes/Install\ macOS\ Mojave/
"disk3" ejected.
{% endhighlight %}

* Convert the disk image into an ISO file.

{% highlight bash %}
$ hdiutil convert /tmp/Mojave.sparseimage -format UDTO -o /tmp/Mojave.iso
Reading Driver Descriptor Map (DDM : 0)…
Reading Apple (Apple_partition_map : 1)…
Reading  (Apple_Free : 2)…
Reading disk image (Apple_HFS : 3)…
........................................................................
Elapsed Time: 44.440s
Speed: 184.3Mbytes/sec
Savings: 0.0%
created: /tmp/Mojave.iso.cdr
{% endhighlight %}

* Move the file to the directory of your choice and remove the `.cdr` part of the extension.

{% highlight bash %}
$ mv /tmp/Mojave.iso.cdr ~/Downloads/Mojave.iso
{% endhighlight %}

* Remove the sparseimage from the `/tmp` firectory.

{% highlight bash %}
$ rm /tmp/Mojave.sparseimage
{% endhighlight %}

