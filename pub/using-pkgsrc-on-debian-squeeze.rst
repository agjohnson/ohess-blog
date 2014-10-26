.. post:: Apr 15, 2012
    :tags: pkgsrc, linux, admin

    Directions for setting up pkgsrc on Debian Squeeze

Using pkgsrc on Debian Squeeze
==============================

After hitting some issues with glibc while `setting up pkgsrc on Centos`_, I
revisited setting up a Debian squeeze server with pkgsrc. Here are some notes on
setting pkgsrc up to build.

First, get build requirements for bootstrapping::

    apt-get install cvs build-essential

Next, checkout and bootstrap pkgsrc::

    cd /usr
    cvs -d anoncvs@anoncvs.NetBSD.org:/cvsroot checkout -rpkgsrc-2011Q1 pkgsrc

Make sure to pay attention to the current pkgsrc release.

You should have ``/usr/pkgsrc`` now, start by bootstraping::

    cd /usr/pkgsrc/bootstrap
    export SH=/bin/bash
    ./bootstrap

The second command is a work around for the following, otherwise tripped,
error::

    a:/usr/pkgsrc/bootstrap# ./bootstrap
    ERROR: Your shell's echo command is not BSD-compatible.
    ERROR: Please select another shell by setting the environment
    ERROR: variable SH.

As with any pkgsrc install, make sure ``/usr/pkg/{bin,sbin}`` is in your PATH
env variable (assuming your shell is bash)::

    export PATH=/usr/pkg/bin:/usr/pkg/sbin:$PATH

Building a Package
------------------

Much like the last attempt at using pkgsrc on Centos, you will need the
ncurses development headers::

    apt-get install libncurses5-dev

Without going into much detail, because my boxes are virtualized on Xen with
Rackspace's cloud, I can assume they will be running the same kernel version.
Using a cluster of Debian servers with the same release, on the same
kernel allows me to use one server as a pkgsrc build server, pulling in
binary packages from the other servers.

So, here, you would build from pkgsrc, I'll just install from a binary package
now that we're bootstrapped.

Setting up rc.subr
------------------

Also, `as mentioned`_, sysv-rc is far more compatible with pkgsrc init scripts
and although some manual work is required, init scripts are easily maintained.

First, install ``rc.subr``, this will still be used by pkgsrc init scripts::

    cd /usr/pkgsrc/pkgtools/rc.subr
    bmake
    bmake install

Not a requirement, but I like to move the rc.d directory location to
``/usr/pkg/etc/rc.d`` to differentiate between the system init.d and pkgsrc rc.d
paths. I add the following to ``/usr/pkg/etc/mk.conf``::

    RCD_SCRIPTS_DIR = /usr/pkg/etc/rc.d

Once built and installed, ``rc.subr`` requires one change::

    --- /etc/rc.subr~       2011-04-23 06:02:34.000000000 +0000
    +++ /etc/rc.subr        2011-04-23 06:03:01.000000000 +0000
    @@ -56,7 +56,7 @@
     _RCCMD_rcs="/usr/bin/rcs"
     _RCCMD_rm="/bin/rm"
     _RCCMD_sh="/bin/sh"
    -_RCCMD_su="/usr/bin/su"
    +_RCCMD_su="/bin/su"
     _RCCMD_systrace="/bin/systrace"
     _RCCMD_whoami="/usr/bin/whoami"

Starting with init
------------------

Using postfix as the example still, install the example init script
into your ``rc.d`` path, mine is set to ``/usr/pkg/etc/rc.d``::

    cp /usr/pkg/share/examples/rc/postfix /usr/pkg/etc/rc.d

Edit ``/usr/pkg/etc/rc.d/postfix`` and add the sysv-rc headers, something
similar to::

    ### BEGIN INIT INFO
    # Provides:          postfix
    # Required-Start:    $local_fs $remote_fs
    # Required-Stop:     $local_fs $remote_fs
    # Default-Start:     2 3 4 5
    # Default-Stop:      0 1 6
    # Short-Description: Postfix daemon.
    ### END INIT INFO

Lastly, link the script to ``/etc/init.d`` and run ``update-rc.d``::

    ln -s /usr/pkg/etc/rc.d/postfix /etc/init.d/postfix
    update-rc.d postfix defaults

Now, set up ``/etc/rc.conf`` as normal, and postfix should boot.

Footnotes
---------

.. _setting up pkgsrc on Centos: pkgsrc_centos
.. _as mentioned: pkgsrc_centos#init

