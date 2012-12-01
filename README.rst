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
instead::

  $ dot-install module=bar

To use a different repository::

  $ dot-install bundle=foobar repo=git://foobar.git

You may also combine the two options [*bundle* and *module*]. If you
invoke `dot-install` with no arguments, it will ask you for
a *repository* and a *bundle* interactively::

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

Modules are directories with a `dist` directory inside. That is the
only requirement. There other special files/directories,aas follows::

  module
  +- dist
  +- dist-root
  +- hook
     + pre
     + post
  +- save

:dist: This directory holds the files that will get installed by that
       module, with the exact same path. If there are *symlinks*, they
       will be followed. In other words, if you need *symlink* you
       have to use the `pre`/`post` hooks.

:hook: The `hook` directory may contain a `pre` and/or `post`
       file. This must be a regular unix script, with the usual
       shebang at the beginning. It also must have the execution
       permission bits set. That means you can use any script language
       that is available on the target during the install.

:save: Tells the installer which files to backup and restore after
       installing the modules. One file name per line. Good candidates
       are auto generated files, like .ssh/known_hosts. Example::

         $ cat save
         /.ssh/known_hosts
         /.emacs.d/bookmarks

:dist-root: Similar to dist, these files will get installed onto the
            filesystem. The difference is that these files are not
            relative to the `root` variable, instead they are relative
            to the `/` directory.

            This will require the `sudo` program to be available, and
            by default it will double check each action before
            installing the files.

Hooks
~~~~~

As stated above, any script file will do. There are three environment
variables defined:

:dot_root: The location where files will be installed. If the hook
            itself needs to create/modify any file it must relative to
            this directory;

:dot_module: The name of the module;

:dot_bundle: The name of the bundle, if any;

:dot_hook: Either `pre` or `post`;

The current directory will be set to the current module being
installed prior invoking the script.

One last important thing about hooks. If the exit status of the *pre*
script is non zero, the module is not installed.

Bundles
-------

Bundles simply group modules. I usually create *symlinks* for the
modules I want, but you may have modules inside as well.

How it works
============

`dot-install` is a shell script. This, hopefully, make it easier to
run it to deploy files to servers and fresh new machines.

From a birds-eye view, the scripts performs five functions [#]_:

1. clone the git repository;

2. stage the files;

3. backup files that modules decide to keep;

4. install the files;

5. restore the backup;

The first stage will clone the git repository into the following
directory::

  ~/.dot/repositories/

The name of the repository will be the md5sum of the URL. That way you
can `dot-install` from multiple repositories, one at a time.

At this phase, it will also initialize/update any git submodule
defined [something I use a lot].

Past fetching the files and the modules have been figured, files will
be staged into a temporary location. The actual directory is defined
by the `mktemp` program.

The pre/post hooks are invoked in this stage. At this point nothing
have been changed, but everything that will be done is available at
the staging directory.

Now the script will look for save files and copy all files that need
not to be kept intact. They will be restored later, in the end.

And then comes the installing phase. Here, two operations are
performed:

1. Only when *purge=true*, remove the top level directories that
   exists in *stgroot* from *distroot*. In other words, when it is
   supposed to install any files under *~/.emacs.d/*, it will first
   remove ~/.emacs.d. The **default is purge=false**;

2. Copy the files to the final location;

The remove step is necessary as the script don't keep track of what
have been installed. After this is done, is it just a matter of
copying the files into the right directories.

Something important to notice is that the remove step is only
performed for files inside the `dist` directory. Files that are under
`dist-root` are only copyied, no cleanup is done. You are not
completely safe, though. We use `tar` to perform the copy. So if you
are copying a file, and currently there is a directory, the directory
will be completely removed and you will get the file instead.

However, the default is to confirm every action. Using this you can
carefully review what will be done. Lastly, if `sudo' can't be find it
just ignores the `dist-root` directory.

Finally, it will retore any files that have been put into the
backup. Jobs done!

.. [#] https://github.com/dgvncsz0f/dot-install/blob/master/dot-install#L719

LICENSE
=======

GPLv3
