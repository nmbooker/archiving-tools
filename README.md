archiving-tools
===============

Tools to help with archiving old files.

# Tarball Archiving

One is where I just want to bundle up a directory of files inside a tarball
so the files don't have to be synced piecemeal and I can move them elsewhere
easily.

For that I've created ```archive.to_tar```.

# Tree-based Archiving

I also have another archiving method, for when I want to move files I no
longer need to use frequently into a single place rather than keeping all
those files in sync across all of my computers.

For that I have two separate 'root' directories: a working_root and an archive_root.

All my working files, that I'm likely to need in the near future, go in
my working_root, while old files that I'm keeping for reference or are
unlikely to be needed soon are kept in my archive_root.

Other than the location of the parent directories, I keep the same
folder (directory) structure in both of the directories.

I have a rule that no file will appear under the same path under both roots,
or that the two roots' contents are disjoint.
The exception is directories.

The scripts ```archiving.comm``` and ```archiving.mv``` are the results
so far.

# Why not use mv, tar or a graphical file manager?

Because if I do I get in a complete mess.  I have to keep two file manager
windows in sync if I use a GUI file manager, and create parent directories
manually.
In a typical archiving pass, I'm looking at files at different levels of
the directory hierarchy in both roots at the same time, and moving files
between them is error prone, especially if I'm tired.

The basic answer is that mv, tar and my graphical file manager don't have
any real idea of what I'm trying to do, and so no way of ensuring I don't
shoot myself in the foot or filling in the relevant bits of the paths for
me to smooth out my workflow.

The tools I've written here have a series of pre-flight checks that make
sure that, assuming something doesn't change between the checks happening
and the operation taking place, no data is going to get lost.
So the archiving.mv and archive.to_tar scripts make sure, for example, that
the target file or directory doesn't already exist, and will exit early
with an error if they do (rather than doing half the job and then failing).

They also have a 'dry run' mode which plays out on screen what the program
would try to do, without actually trying to do it.

```archiving.comm``` checks for file paths in common between the two roots,
to check that the disjointness condition hasn't been violated.

# Implementation
The scripts are written in Python.  Some bits use Python standard library
features, while some call standard Unix utilities (some relying on GNU specific stuff I'm afraid).

```archive.to_tar``` for example uses both GNU commands 'tar' and 'rm', as
I would if I was doing it manually:
 cd parent/of/directory/to/archive
 tar czf directory.tar.gz directory
 rm -rf directory
except that the cd to the parent directory is done automatically, as are
checks such as whether the .tar.gz file already exists.

```archiving.comm``` uses the GNU 'find' command to get the lists of files, then uses
Python's set data structure and operators to find common entries.

```archiving.mv``` is currently pure Python as I write this README.

# archiving.mv

Moves paths from the working_root into the equivalent subdirectory of the
archive_root.

The locations of the roots can be specified either with the -w and -a options,
or with the environment variables:
 ARCHIVING_WORKING_ROOT
 ARCHIVING_ARCHIVE_ROOT

The latter are used in small wrapper scripts I use to call archiving.mv, one for each pair of trees (I have two such pairs of trees at the moment - one for personal files and one for work files).

I used environment variables because it allows you to set them once a session and call the archiving.mv script lots of times quite easily, rather than having to specify -a and -w every time you call it.

Here's an example:
 cd /data/sfnick/home/some/subdir
 export ARCHIVING_WORKING_ROOT=/data/sfnick/home
 export ARCHIVING_ARCHIVE_ROOT=/net/nas/sfnick/archive
 archiving.mv path/to/file/i/want/to/archive
 archiving.mv -r path/to/directory/i/want/to/archive

Requiring a -r for directories is just an extra check to prevent accidentally moving a whole tree instead of an individual file.

Supplying -d gives a dry run - the pre-flight checks are done, but no filesystem-changing commands are run.

Supplying -v specifies that before each mkdir or move operation the operation
(with full paths) is printed on screen (to stderr).

-d and -v together provides a good way to find out what would happen for an invocation without committing to doing it.

# archiving.comm

Checks that two trees are disjoint.

archive.comm /path/to/working_root /path/to/archive_root

The paths in common are printed to stdout, and are relative to either of
the roots rather than being absolute paths or relative to your current working directory.

# archive.to\_tar

Archive a directory into a tarball of the same name + .tar.gz, and optionally
remove that directory afterwards.

gzip compression is enabled by default.

-R will cause the directory to be
removed recursively once tar has exited successfully,
and adding -f to that will cause -f to be passed to the rm command.

-u causes the gzip compression to be disabled, and the output will be to
a basic tarball called basename.tar (note no .gz)

-v causes the output of 'tar' to be verbose.

-D causes a dry run.  The pre-flight checks are done, and assuming they
succeeded, the lists of command line arguments that would be run are
listed as usual, but they're not actually listed.

usage: archive.to_tar [-h] [-u] [-v] [-R] [-f] [-D] path

archive a directory of files to a tar file

positional arguments:
  path                 path to directory to archive

optional arguments:
  -h, --help           show this help message and exit
  -u, --uncompressed   disable gzip compression
  -v, --verbose-tar    make tar verbose
  -R, --then-remove    remove path afterwards
  -f, --force-removal  remove path with -f (only with -R)
  -D, --dry-run        don't actually do anything - just print what would
                       happen
