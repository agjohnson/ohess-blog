.. post:: Apr 15, 2012
    :tags: pkgsrc, linux, admin

    Directions for setting up pkgsrc on Centos 5.5

Using pkgsrc on Centos 5.5
==========================

Beginning with pkgsrc on Centos is rather simple. Before checking out pkgsrc,
a base install of Centos is missing several requirements::

    yum install gcc gcc-c++ kernel-devel cvs

Once these requirements are met, you can checkout and bootstrap pkgsrc::

    cd /usr
    CVS_RSH=ssh cvs -d anoncvs@anoncvs.NetBSD.org:/cvsroot checkout -rpkgsrc-2011Q1 pkgsrc

Of course, the pkgsrc release might need to be changed, this example is using
the current release as of now. I needed the ``CVS_RSH`` for this to work, as
note the example is assuming your current shell is bash.

After cvs finishes up, you should have ``/usr/pkgsrc`` -- time to bootstrap::

    cd /usr/pkgsrc/bootstrap
    ./bootstrap

This will build the base package management utilities, bmake, et al.

The last step before you get going is making sure ``/usr/pkg`` is higher
priority in your PATH env variable. This would be done in your bash/csh/etc
profile script normally. Setting it on the command line is fine for now (again,
depending on your shell, I'm using the base install of bash here)::

    export PATH=/usr/pkg/bin:/usr/pkg/sbin:$PATH

Building a Package
------------------

For this step, I'll build ``mail/postfix``::

    cd /usr/pkgsrc/mail/postfix
    bmake

You'll likely hit a snag::

    > ERROR: No usable termcap library found on the system.

This is resolved by install ncurses development files::

    yum install ncurses-devel

Restart the build and things should compile cleanly, you can install the package
normally.

Using rc.subr and init
----------------------

Using pkgsrc on sysv-rc based servers is a little easier, as sysv-rc follows the
same dependency type model that pkgsrc's rc.subr does. This allows you to use
pkgsrc scripts by simply copying the init script to /etc/rc.d and making some
small changes to the init script header.

Centos does not use sysv-rc however, and so a little more hand holding is
required.  To tie pkgsrc init scripts in with Centos' init, you'll need to edit
the scripts and decide boot priority yourself.

First, install ``rc.subr``, this will still be used by pkgsrc init scripts::

    cd /usr/pkgsrc/pkgtools/rc.subr
    bmake
    bmake install

There was one change I needed to make on my Debian servers::

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

Using postfix as the example still, install the init script from pkgsrc::

    cd /etc/init.d
    cp /usr/pkg/share/examples/rc/postfix .

The init script will require changes here, and the normal changes to
``/etc/rc.conf`` are required as well.

That is, very briefly, how to maintain the init scripts installed by pkgsrc
packages. There is a lot more to configuring the scripts however, I will try
to expand with a later article.
