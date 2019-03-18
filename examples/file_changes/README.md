# Detect that a file has changed.

One of the simplest uses for Make outside its
traditional uses is reporting files that have
changed since it was last run. In this example's
makefile, the set of files to report on has been
written out explicitly.

This example is an especially good starting point
for learning about Make since it demonstrates how
changes in a rule's prerequisites cause Make to
run the rule's recipe. This behavior is central to
any use of Make.

## Project setup

Often projects have a `configure` script that
performs setup tasks required for Make to run
correctly in the current environment. For this
project, there isn't much setup: it simply makes a
directory to store files that register when Make
last reported that a file changed.

```
$ ls | grep track # No track directory
$ ./configure # Configure the project
$ ls | grep track # The track directory has been made
track
```

## Use the project

Shipped with this project are three example text
files that we'll track with Make; they're stored
at `files/{a,b,c}.txt`. The way the `Makefile` is
written, Make hasn't reported on any files yet
(the track directory is empty), so that it'll
run the recipes required to build those "reports"
on its first run.

```
$ ls track # The track directory is empty
$ make -s # See which files have changed since Make was last run
files/a.txt
files/b.txt
files/c.txt
```

Make now has a report time for each file `files/<letter>.txt`
stored in the last-modified timestamp of the file
`track/<letter>.txt`. On future runs, Make can use this information
to determine if one of our files has changed since the last run. If
we don't change any files, then Make has nothing to report.

```
$ ls track # The track directory contains files with report times
$ make -s # Nothing has changed since Make was last run
$
```

Now here's where Make becomes useful. Let's change the contents
of one of the files we're tracking. I'm going to open up
`files/c.txt` and change its contents to this:

```
Cerulean is a shade of blue.
```

Running Make again, it correctly reports `files/c.txt` as having
changed since the last time it was run.

```
$ make -s
files/c.txt
$ make -s # Reports are once-only; back to nothing again
$ 
```

And now we can play this game forever. Changing `files/{a,b}.txt`:

`files/a.txt`:

```
Athabasca is a lake in Canada.
```

`files/b.txt`:

```
Bosphorus is a strait exiting the Black Sea.
```

Running Make:

```
$ make -s
files/a.txt
files/b.txt
```

Having seen the list of files change between runs, let's take a moment to
step through what Make does to determine if it should report a file:
* If `track/<letter>.txt` doesn't exist, report it.
* Otherwise, compare the last-modified timestamps of `track/<letter>.txt` and `files/<letter>.txt`.
* If the timestamp for `files/<letter>.txt` is more recent than the timestamp for `track/<letter>.txt`, report it.
* Update the timestamp for `track/<letter>.txt` to the current time.

Compare that to how Make processes a rule in general:
* If a rule's target doesn't exist, then run its recipe (to build its target).
* Otherwise, if one of the rule's prerequisites has been updated since the target was last updated, run its recipe (to build its target).

The general case assumes that rules exist in order to build their targets,
and that targets should be rebuilt if the input used to build them has
changed. However, Make doesn't require this; it's just convention (look up
phony targets). Likewise, Make doesn't require that your targets be "built"
in the traditional sense; any file can be a target, including
[empty files](https://www.gnu.org/software/make/manual/html_node/Empty-Targets.html).
For this example, the target was an empty file that we "built" by updating
its last-modified timestamp with the `touch` command. We
used the empty file `track/<letter>.txt` to record the last time we checked
for changes to `files/<letter>.txt`.

## Reset the project

Most projects come with a way to reset them.
The most common one is the `clean` target,
which is the standard way to clean up or fix
a project after Make has been run a number of
times. For this project, I've defined `clean`
to remove all the report times from the track
directory, so that the next call to Make will
behave as though Make is reporting on the
tracked files for the first time.

```
$ ls track # The track directory contains files with report times
$ make clean # Reset to after ./configure was run
$ ls track # The track directory is empty
$ make -s
files/a.txt
files/b.txt
files/c.txt
```

Another, harsher version of project clean up
is the `distclean` target, which is the standard
way to clean up everything the project has done
in the working directory since it was downloaded.
Specifically, only files that existed in the
original distribution of the project should exist
after `distclean` is run. For this project, I've
defined `distclean` to remove the directory that
the `configure` script made.

```
$ ls | grep track # The track directory exists
$ make distclean # Reset to before ./configure was run
$ ls | grep track # The track directory no longer exists
$ ./configure # Can treat this like a brand new project now
$ make -s
files/a.txt
files/b.txt
files/c.txt
```

## Notes

* The [configure script](https://www.gnu.org/software/automake/manual/html_node/GNU-Build-System.html) and the [clean and distclean targets](https://www.gnu.org/prep/standards/html_node/Standard-Targets.html) are standard aspects of a Make project whose contents and purposes are probably more rigidly defined than they are in this example.
* This and other examples make heavy use of [empty target files](https://www.gnu.org/software/make/manual/html_node/Empty-Targets.html). While more substantive targets should be used when they're available (e.g. don't use an empty target to register when a *.o file has changed), they seem to be really useful for extending the use of Make beyond code builds.
