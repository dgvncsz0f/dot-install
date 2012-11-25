=============
 dot-install
=============

This is a very minimalist package manager that I use to manage my dot
files.

For very long time I've been using a shell script that took care of
installing my files. That worked really well, the only problem begin
that files or configuration specific to one machine would not go to
the git repository. I have a personal notebook, a workstation at the
work and a dozen of servers that I deploy my dot files to.

From that comes my motivation: deploying dot files to heterogeneous
environments and making sure everything is in the git repository.

This project attempt to solve this problem using *bundles* and
*modules*. Things are organized in modules and bundles group those
modules. That way I could create a bundle for my machine at the office
and another one for my personal notebook, or just install a specific
module into a server.

The good thing is that by solving this problem I also solved a small
inconvenient. Sharing was not possible with the previous scheme as the
script would overwrite everything with no mercy. Obviously that was
not the driver for writing this, but it may very well suits others.

Running
=======

Given you have a git repository that follows the *dot layout* (as
described in this document)::

  $ dot-install bundle=foo

This will install the *foobar* bundle. If you want just a module
instead:

  $ dot-install module=bar

To use a different repository:

  $ dot-install bundle=foobar repo=git://foobar.git

You may also combine the two options [*bundle* and *module*]. If you
invoke `dot-install` with no arguments, it will ask you for
a *repository* and a *bundle* interactively:

  $ dot-install
  repository: ...
  bundle: ...

Invoke the script without arguments and you will get all available
options. Most importantly, `root`, which changes the directory to
install the files and `dryrun` that do not change anything but let you
see what will be done.

The dot layout
==============

The repository this script is able to deal with must have the
following structure::

   repository
   +- modules
      +- ... 
   +- bundles
      +- ...

Modules
-------

Modules are directories with a `dist` directory inside and an optional
`hook` directory. That is the only requirement.

The `dist` directory holds the files that will get installed by that
module, with the exact same path. If there are *symlinks*, they will
be followed. In other words, if you need *symlink* you have to use the
`pre`/`post` hooks.

The `hook` directory may contain a `pre` and/or `post` file. This must
be a regular unix script, with the usual shebang at the beginning. It
also must have the execution permission bits set. That means you can
use any script language that is available on the target during the
install.

Bundles
-------

Bundles simply group modules. I usually create *symlinks* for the
modules I want, but you may have modules inside as well.

How it works
============

`dot-install` is a shell script. This, hopefully, make it easier to
run it to deploy files to servers and fresh new machines.

From a birds-eye, the scripts performs three functions:

1. clone the git repository;

2. stage the files;

3. install the files;

The first stage will clone the git repository [#]_ into the following
directory::

  ~/.dot/repositories/

The name of the repository will be the md5sum of the URL. That way you
can `dot-install` from multiple repositories, one at a time.

At this phase, it will also initialize/update any git submodule
defined [something I use a lot].

Past fetching the files and the modules have been figured, files will
be staged into a temporary location [#]_. The actual directory is
defined by the `mktemp` program.

The pre/post hooks are invoked in this stage. At this point nothing
have been changed, but everything that will be done is available at
the staging directory.

And then comes the installing phase. Here, two operations are
performed:

1. Remove the directories that will be modified [#]_;

2. Copy the files to the final location [#]_;

The remove step is necessary as the script don't keep track of what
have been installed. After this is done, is it just a matter of
copying the files and we are done.

.. [#] https://github.com/dgvncsz0f/dot-install/blob/v0.0.1/dot-install#L254
.. [#] https://github.com/dgvncsz0f/dot-install/blob/v0.0.1/dot-install#L336
.. [#] https://github.com/dgvncsz0f/dot-install/blob/v0.0.1/dot-install#L378
.. [#] https://github.com/dgvncsz0f/dot-install/blob/v0.0.1/dot-install#L386

LICENSE
=======

GPLv3