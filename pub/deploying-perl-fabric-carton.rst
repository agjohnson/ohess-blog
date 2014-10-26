.. post:: Oct 12, 2012
    :tags: perl, admin

    An example of deploying Perl applications with Fabric and Carton, with slides from a panel talk at pdx-pm.

Deploying Perl with Fabric and Carton
=====================================

.. raw:: html

    <div class="row">
      <div class="col col-2">

Carton, similar to Python's pip or Node's npm, allows for module version
locking for more predictable deployments. Using the bundle mode, a local
cache can be used for deployments, or using the standard deployment mode
the carton.lock file will be used to install modules.

Combined with Carton, Fabric can be used to deploy exact copies of module
dependencies to multiple servers, even in parallel. Fabric could also be used to
rsync the local module cache for bundle installs.

.. raw:: html

      </div>
      <div class="col col-1">
        <script
          async
          class="speakerdeck-embed"
          data-id="5061752bc491a200020226f0"
          data-ratio="1.3333333333333333"
          src="http://speakerdeck.com/assets/embed.js"
        ></script>
      </div>
    </div>

Fabric and Carton Glue
----------------------

Here is a working example of a Carton deployment using Fabric. Using
App::local::lib::helper as the shell in the fabfile, bootstrap cpanminus and
Carton and then deploy the application with Carton.

.. code-block:: python

    '''Deployment'''

    from fabric.api import *
    from fabric.contrib.project import rsync_project
    from contextlib import contextmanager

    import os.path

    env.www_host = 'prod@server'
    env.build_path = '/srv/www/site'
    env.env_path = '/srv/www/envs/bootstrap'

    @task
    @hosts(env.www_host)
    def bootstrap():
        '''Bootstrap environment'''
        if not exists(env.build_path):
            run('mkdir -p {build_path}'.format(**env))
        if not exists(env.env_path):
            run('mkdir -p {env_path}/{{lib,bin}}'.format(**env))
        with cd(env.env_path):
            env.cpanm = '{env_path}/bin/cpanm'.format(**env)
            if not exists(env.cpanm):
                run('curl -Lko {cpanm} "http://cpanmin.us"'.format(**env))
                run('chmod +x {cpanm}'.format(**env))
            run('{cpanm} -L . App::cpanminus'.format(**env))
            run('{cpanm} -L . App::local::lib::helper'.format(**env))
            run('{cpanm} -nL . carton'.format(**env))

    @task
    @hosts(env.www_host)
    def deploy():
        '''Deploy master'''
        rsync_project(
            env.build_path,
            './',
            exclude=[
                '*.pyc',
                '.git',
                'local/',
            ]
        )
        with localenv():
            with cd(env.build_path):
                run('carton install')

    @contextmanager
    def localenv():
        '''Context manager for local::lib'''
        with settings(shell='{env_path}/bin/localenv /bin/sh -c'.format(**env)):
            yield

    def exists(path):
        '''Same as the contrib.files.exists, only with a different test'''
        with settings(hide('everything'), warn_only=True):
            return not run('test -e "%s"' % path).failed
    </pre>
