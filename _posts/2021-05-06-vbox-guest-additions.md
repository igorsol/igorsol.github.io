---
layout: post
title:  "VirtualBox guest additions issue"
date:   2021-05-06 17:26:00 +0300
tags: ubuntu virtualbox
categories: issue solution
---
# Issue

After upgrading VirtualBox to version 6.1.22 installation of guest additions in Ubuntu 16.04 has suddenly failed.

# Investigation

I tried to rollback to VirtualBox 6.1.20 and 6.1.18 but the issue persisted.

Terminal suggested to look into some log file:

    VirtualBox Guest Additions: Look at /var/log/vboxadd-setup.log to find out what went wrong

But that log file only contained two lines of useless information (something about X.Org server).

Then I found somewhere you can install guest additions with this command:

{% highlight bash %}
$ sudo rcvboxadd setup
{% endhighlight %}

This also failed and suggested to look into the same file `/var/log/vboxadd-setup.log`. Now this
file contained much more useful information, for example this part:
{% highlight bash %}
echo >&2 "  ERROR: Kernel configuration is invalid.";           \
echo >&2 "         include/generated/autoconf.h or include/config/auto.conf are missing.";\
echo >&2 "         Run 'make oldconfig && make prepare' on kernel src to fix it.";      \
{% endhighlight %}
plus some gcc command lines with their error output.

gcc error was about `inode_change_ok` fucntion in the `utils.c` file in the `vboxsf` module.
So i found virtualbox's repository at <https://www.virtualbox.org/browser/vbox/trunk#src/VBox/Additions/linux>
I also found this block of code in the `utils.c`:
{% highlight cpp %}
#if (RTLNX_VER_RANGE(3,16,39,  3,17,0)) || RTLNX_VER_MIN(4,9,0) || (RTLNX_VER_RANGE(4,1,37,  4,2,0))
# if RTLNX_VER_MIN(5,12,0)
    rc = setattr_prepare(ns, dentry, iattr);
# else
    rc = setattr_prepare(dentry, iattr);
# endif
#else
    rc = inode_change_ok(pInode, iattr);
#endif
{% endhighlight %}
Obviously if my Ubuntu has kernel version 4.4.0 then `inode_change_ok` will be selected. But on some site I found that the correct branch is `setattr_prepare(dentry, iattr)`.

# Solution

Well it was possible to try to modify utils.c but source code had a clue: it should work with newer kernel version.
So I decided to upgrade a kernel in my Ubuntu 16.04 virtual machine (see links below). This resolved
guest additions compilation issue.

# External Links

[Guest Additions chapter in VirtualBox documentation](https://www.virtualbox.org/manual/ch04.html)

[Ticket describing the issue on VirtialBox's issue tracker](https://www.virtualbox.org/ticket/20325)

[Ubuntu 16.04 - Upgrade Kernel (unofficial)](https://blog.programster.org/ubuntu-16-04-upgrade-kernel)

[Official Ubuntu documentation on kernel upgrades](https://wiki.ubuntu.com/Kernel/LTSEnablementStack#Ubuntu_16.04_LTS_-_Xenial_Xerus)
