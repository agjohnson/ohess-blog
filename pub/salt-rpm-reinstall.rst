.. post:: Feb 1, 2013
    :tags: salt, centos, rpm, redhat, admin

    Using Salt Stack to upgrade an already install RPM.

Salt - Reinstall Package from RPM
=================================

I hit a small issue trying to install a new version of Node.js on our web
servers this week. We had previously installed Node v0.6 from a repo that is now
defunct. We wanted to upgrade our web cluster to v0.8 through salt, but ran into
problems trying to purge the old package. So, here is a shortcut for
reinstalling a new package from RPM::

    # Node

    {% set node_ver = '0.8.18-1' %}

    node-pkg:
      pkg.installed:
        - sources:
          - nodejs: salt://node/nodejs-{{ node_ver }}.{{ grains['cpuarch'] }}.rpm
        - watch:
          - module: nodejs-old

    nodejs-old:
      cmd.run:
        - name: /bin/true
        - onlyif: "rpm -q nodejs && ! rpm -q nodejs-{{ node_ver }}"
      module.wait:
        - name: pkg.purge
        - pkgs: nodejs
        - watch:
          - cmd: nodejs-old

In this example, ``cmd.run`` is only run if ``nodejs`` is installed and not the
version we are trying to install. If ``cmd.run`` is triggered, it runs
``/bin/true``, which causes the ``module.wait`` watch to trigger, thus purging the
package from the system. The install check is performed last, which installs
Node.
