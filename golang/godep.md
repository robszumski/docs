## Using Godep

This is a brief guide describing how we manage third-party dependencies using [`godep`](https://github.com/tools/godep). 
We're going to use the [rkt](https://github.com/coreos/rkt) repository as an example.

The [build script](https://github.com/coreos/rkt/blob/master/build) is crafted to make this transparent to most users (i.e. if you're just building rkt from source, or modifying any of the codebase without changing dependencies, you should have no need to interact with godep).
But occasionally the need arises to either a) add a new dependency or b) update/remove an existing dependency.
At this point, the ramblings below from an experienced Godep victim^Wenthusiast might prove of use...

### Update godep

Step zero is generally to ensure you have the **latest version** of `godep` available in your `PATH`.

### Having the right directory layout (i.e. `GOPATH`)

To work with `godep`, you'll need to have the repository (i.e. `github.com/coreos/rkt`) checked out in a valid `GOPATH`.
If you use the [standard Go workflow](https://golang.org/doc/code.html#Organization), with every package in its proper place in a workspace, this should be no problem.
As an example, if one was obtaining the repository for the first time, one would do the following:

```
$ export GOPATH=/tmp/foo               # or any directory you please
$ go get -d github.com/coreos/rkt/...  # or 'git clone https://github.com/coreos/rkt $GOPATH/src/github.com/coreos/rkt'
$ cd $GOPATH/src/github.com/coreos/rkt
```

If, however, you instead prefer to manage your source code in directories like `~/src/rkt`, there's a problem: `godep` doesn't like symbolic links (which is what the rkt `build` script [uses to create a self-contained GOPATH](https://github.com/coreos/rkt/blob/master/build#L8)).
Hence, you'll need to work around this with bind mounts, with something like the following:
```
$ export GOPATH=/tmp/foo        # or any directory you please
$ mkdir -p $GOPATH/src/github.com/coreos/rkt
$ sudo mount --bind ~/src/rkt $GOPATH/src/github.com/coreos/rkt
$ cd $GOPATH/src/github.com/coreos/rkt
```

One benefit of this approach over the single-workspace workflow is that checking out different versions of dependencies in the `GOPATH` (as we are about to do) is guarnteed to not affect any other packages in the `GOPATH`.
(Using [gvm](https://github.com/moovweb/gvm) or other such tomfoolery to manage `GOPATH`s is an exercise left for the reader.)

### Restoring the current state of dependencies

Now that we have a functional `GOPATH`, use `godep` to restore the full set of vendored dependencies to their correct versions.
(What this command does is essentially just loop over the set of dependencies codified in `Godeps/Godeps.json`, using `go get` to retrieve and then `git checkout` (or equivalent) to set each to their correct revision.)

```
$ godep restore # might take a while if it's the first time...
```

At this stage, your path forks, depending on what exactly you want to do: add, update or remove a dependency.
But in _all three cases_, the procedure finishes with the [same save command](#saving-the-set-of-dependencies).

#### Add a new dependency

In this case you'll first need to retrieve the dependency you're working with into `GOPATH`.
As a simple example, assuming we're adding `github.com/fizz/buzz`:
```
$ go get -d github.com/fizz/buzz
```

Then add your new dependency into `godep`'s purview by simply importing the standard package name in one of your sources:

```
$ vim $GOPATH/src/github.com/coreos/rkt/some/file.go
...
import "github.com/fizz/buzz"
...
```

Now, GOTO [saving](#saving-the-set-of-dependencies)

#### Update an existing dependency

In this case, assuming we're updating `github.com/foo/bar`:

```
$ cd $GOPATH/src/github.com/foo/bar
$ git pull   # or 'go get -d -u github.com/foo/bar/...' 
$ git checkout $DESIRED_REVISION
$ cd $GOPATH/src/github.com/coreos/rkt
$ godep update github.com/foo/bar/...
```

Now, GOTO [saving](#saving-the-set-of-dependencies)

#### Removing an existing dependency

This is the simplest case of all: simply remove all references to a dependency from the source files.

Now, GOTO [saving](#saving-the-set-of-dependencies)

### Saving the set of dependencies

Finally, here we are, the magic command, the holy grail, the ultimate conclusion of all `godep` operations.
Provided you have followed the preceding instructions, regardless of whether you are adding/removing/modifying dependencies, this command will cast the necessary spells to solve all of your dependency worries:
```
$ godep save -r ./...
```

## Finishing up

At this point, you should be good to PR.
As well as a simple sanity check that the code actually builds and tests pass, here are some things to look out for:
- `git status Godeps/` should show only a minimal and relevant change (i.e. only the dependencies you actually intended to touch).
- `git diff Godeps/` should be free of any changes to import paths within the vendored dependencies
- `git diff` should show _all_ third-party import paths prefixed with `Godeps/_workspace`
- If something looks awry, restart, pray to your preferred deity, and try again.
