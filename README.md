# Glide: Vendor Package Management for Golang

*Manage your vendor and vendored packages with ease.* Glide is a tool for
managing the `vendor` directory within a Go package. This feature, first
introduced in Go 1.5, allows each package to have a `vendor` directory
containing dependent packages for the project. These vendor packages can be
installed by a tool (e.g. glide), similar to `go get` or they can be vendored and
distributed with the package.

[![Build Status](https://travis-ci.org/Masterminds/glide.svg)](https://travis-ci.org/Masterminds/glide) [![Go Report Card](http://goreportcard.com/badge/Masterminds/glide)](http://goreportcard.com/report/Masterminds/glide)

### Features

* Ease dependency management
* Support **versioning packages** including [Semantic Versioning
  2.0.0](http://semver.org/) support. Any constraint the [`github.com/Masterminds/semver`](https://github.com/Masterminds/semver)
  package can parse can be used.
* Support **aliasing packages** (e.g. for working with github forks)
* Remove the need for munging import statements
* Work with all of the `go` tools
* Support the VCS tools that Go supports:
    - git
    - bzr
    - hg
    - svn
* Support custom local and global plugins (see docs/plugins.md)
* Repository caching including reuse of packages in the `$GOPATH`
* Flatten dependencies resolving version differences and avoiding the inclusion
  of a package multiple times.

## How It Works

The dependencies for a project are listed in a `glide.yaml` file. This can
include a version, VCS, repository location (that can be different from the
package name), etc. When `glide up` is run it downloads the packages (or updates)
to the `vendor` directory. It then recursively walks through the downloaded
packages looking for those with a `glide.yaml` file (or Godep, gb, or GPM config
file) that don't already have a `vendor` directory and installing their
dependencies to their `vendor` directories.

A projects is structured like this:

```
- $GOPATH/src/myProject (Your project)
  |
  |-- glide.yaml
  |
  |-- main.go (Your main go code can live here)
  |
  |-- mySubpackage (You can create your own subpackages, too)
  |    |
  |    |-- foo.go
  |
  |-- vendor
       |-- github.com
            |
            |-- Masterminds
                  |
                  |-- ... etc.
```

*Take a look at [the Glide source code](http://github.com/Masterminds/glide)
to see this philosophy in action.*

## Install

On Mac OS X you can install the latest release via [Homebrew](https://github.com/Homebrew/homebrew):

```
$ brew install glide
```

[Binary packages](https://github.com/Masterminds/glide/releases) are available for Mac, Linux and Windows.

To build from source you can:

1. Clone this repository and change directory into it
2. Run `make bootstrap`

This will leave you with `./glide`, which you can put in your `$PATH` if
you'd like. (You can also take a look at `make install` to install for
you.)

The Glide repo has now been configured to use glide to
manage itself, too.

## Usage

```
$ glide create                            # Start a new workspaces
$ open glide.yaml                         # and edit away!
$ glide get github.com/Masterminds/cookoo # Get a package and add to glide.yaml
$ glide install                           # Install packages and dependencies
# work, work, work
$ go build                                # Go tools work normally
$ glide up                                # Update to newest versions of the package
```

Check out the `glide.yaml` in this directory, or examples in the `docs/`
directory.

### glide create [optional package name]

Initialize a new workspace. Among other things, this creates a stub
`glide.yaml`

```
$ glide create
[INFO] Initialized. You can now edit 'glide.yaml'
```

If an optional package name is specified, Glide will add it to
glide.yaml as the name of your project.

### glide get [package name]

You can download package to your `vendor` directory and have it added to your
`glide.yaml` file with `glide get`.

```
$ glide get github.com/Masterminds/cookoo
```

### glide guess and glide pin

To help with the creating and managing your `glide.yaml` files there are two
more helper commands. The `glide guess` command will look over your project,
read the imports, attempt to intelligently guess at the ones you need to list,
and create the text for a `glide.yaml` file.

There are times you need to pin a dependency to a version, such as when you're
preparing to deploy to production. For that case there is the `glide pin` command
that will pin each dependency in the `glide.yaml` file to the current commit id.

### glide up (aliased to update and install)

Download or update all of the libraries listed in the `glide.yaml` file and put
them in the `vendor` directory. It will also recursively walk through the
dependency packages doing the same thing if no `vendor` directory exists.

```
$ glide up
```

This will recurse over the packages looking for other projects managed by Glide,
Godep, gb, and GPM. When one is found those packages will be installed as needed.

## glide novendor (aliased to nv)

When you run commands like `go test ./...` it will iterate over all the
subdirectories including the `vendor` directory. When you are testing your
application you may want to test your application files without running all the
tests of your dependencies and their dependencies. This is where the `novendor`
command comes in. It lists all of the directories except `vendor`.

    $ go test $(glide novendor)

This will run `go test` over all directories of your project except the
`vendor` directory.

## glide name

When you're scripting with Glide there are occasions where you need to know
the name of the package you're working on. `glide name` returns the name of the
package listed in the `glide.yaml` file.

### glide rebuild

Re-run `go install` on the packages in the `glide.yaml` file. This
(along with `glide install` and `glide update`) pays special attention
to the contents of the `subpackages:` directive in the YAML file.

```
$ glide rebuild
[INFO] Building dependencies.
[INFO] Running go build github.com/kylelemons/go-gypsy/yaml
[INFO] Running go build github.com/Masterminds/cookoo/cli
[INFO] Running go build github.com/Masterminds/cookoo
```

This is useful when you are working with large 3rd party libraries. It
will create the `.a` files, which can have a positive impact on your
build times.

### glide tree

Glide includes a few commands that inspect code and give you details
about what is imported. `glide tree` is one such command. Running it
gives data like this:

```
$ glide tree
github.com/Masterminds/glide
	github.com/Masterminds/cookoo   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/cookoo)
		github.com/Masterminds/cookoo/io   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/cookoo/io)
	github.com/Masterminds/glide/cmd   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/cmd)
		github.com/Masterminds/cookoo   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/cookoo)
			github.com/Masterminds/cookoo/io   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/cookoo/io)
		github.com/Masterminds/glide/gb   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/gb)
		github.com/Masterminds/glide/util   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/util)
			github.com/Masterminds/vcs   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/vcs)
		github.com/Masterminds/glide/yaml   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/yaml)
			github.com/Masterminds/glide/util   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/util)
				github.com/Masterminds/vcs   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/vcs)
			github.com/Masterminds/vcs   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/vcs)
			gopkg.in/yaml.v2   (/Users/mfarina/Code/go/src/gopkg.in/yaml.v2)
		github.com/Masterminds/semver   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/semver)
		github.com/Masterminds/vcs   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/vcs)
		github.com/codegangsta/cli   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/codegangsta/cli)
	github.com/codegangsta/cli   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/codegangsta/cli)
	github.com/Masterminds/cookoo   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/cookoo)
		github.com/Masterminds/cookoo/io   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/cookoo/io)
	github.com/Masterminds/glide/gb   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/gb)
	github.com/Masterminds/glide/util   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/util)
		github.com/Masterminds/vcs   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/vcs)
	github.com/Masterminds/glide/yaml   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/yaml)
		github.com/Masterminds/glide/util   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/util)
			github.com/Masterminds/vcs   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/vcs)
		github.com/Masterminds/vcs   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/vcs)
		gopkg.in/yaml.v2   (/Users/mfarina/Code/go/src/gopkg.in/yaml.v2)
	github.com/Masterminds/semver   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/semver)
	github.com/Masterminds/vcs   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/Masterminds/vcs)
	github.com/codegangsta/cli   (/Users/mfarina/Code/go/src/github.com/Masterminds/glide/vendor/github.com/codegangsta/cli)
```

This shows a tree of imports, excluding core libraries. Because
vendoring makes it possible for the same package to live in multiple
places, `glide tree` also prints the location of the package being
imported.

### glide list

Glide's `list` command shows an alphabetized list of all the packages
that a project imports.

```
$ glide list
github.com/Masterminds/cookoo (Location: vendored)
github.com/Masterminds/cookoo/io (Location: vendored)
github.com/Masterminds/glide/cmd (Location: gopath)
github.com/Masterminds/glide/gb (Location: gopath)
github.com/Masterminds/glide/util (Location: gopath)
github.com/Masterminds/glide/yaml (Location: gopath)
github.com/Masterminds/semver (Location: vendored)
github.com/Masterminds/vcs (Location: vendored)
github.com/codegangsta/cli (Location: vendored)
gopkg.in/yaml.v2 (Location: vendored)
```

The possible locations for `list` are `vendored`, `gopath`, and `missing` (if
the package is not installed anywhere accessible).

### glide help

Print the glide help.

```
$ glide help
```

### glide --version

Print the version and exit.

```
$ glide --version
glide version 0.7.0
```

### glide.yaml

The `glide.yaml` file does two critical things:

1. It names the current package
2. It declares external dependencies

A brief `glide.yaml` file looks like this:

```yaml
package: github.com/Masterminds/glide
import:
  - package: github.com/Masterminds/semver
  - package: github.com/Masterminds/cookoo
    vcs: git
    version: ^1.2.0
    repo: git@github.com:Masterminds/cookoo.git
```

The above tells `glide` that...

1. This package is named `github.com/Masterminds/glide`
2. That this package depends on two libraries.


The first library exemplifies a minimal package import. It merely gives
the fully qualified import path.

When Glide reads the definition for the second library, it will get the repo
from the source in `repo`, checkout the latest version between 1.2.0 and 2.0.0,
and put it in `github.com/Masterminds/cookoo` in the `vendor` directory. (Note
that `package` and `repo` can be completely different)

**TIP:** The version is either VCS dependent and can be anything that can be checked
out or a semantic version constraint that can be parsed by the [`github.com/
Masterminds/semver`](https://github.com/Masterminds/semver) package.
For example, with Git this can be a branch, tag, or hash. This varies and
depends on what's supported in the VCS.

**TIP:** In general, you are advised to use the *base package name* for
importing a package, not a subpackage name. For example, use
`github.com/kylelemons/go-gypsy` and not
`github.com/kylelemons/go-gypsy/yaml`.

### Controlling package and subpackage builds

In addition to fetching packages, Glide builds the packages with `go
install`. The YAML file can give special instructions about how to build
a package. Example:

```yaml
package: github.com/technosophos/glide
import:
  - package: github.com/kylelemons/go-gypsy
    subpackage: yaml
  - package: github.com/Masterminds/cookoo
    subpackage:
      - .
      - cli
      - web
  - package: github.com/crowdmob/amz
    subpackage: ...
```

According to the above, the following packages will be built:

1. The `go-gypsy/yaml` package
2. The `cookoo` package (`.`), along with `cookoo/web` and `cookoo/cli`
3. Everything in `awz` (`...`)

See the `docs/` folder for more examples.

## Supported Version Control Systems

The Git, SVN, Mercurial (Hg), and Bzr source control systems are supported. This
happens through the [vcs package](https://github.com/masterminds/vcs).

## Troubleshooting

#### Q: bzr (or hg) is not working the way I expected. Why?

These are works in progress, and may need some additional tuning. Please
take a look at the [vcs package](https://github.com/masterminds/vcs). If you
see a better way to handle it please let us know.

#### Q: Should I check `vendor/` into version control?

That's up to you. It's not necessary, but it may also cause you extra
work and lots of extra space in your VCS. There may also be unforeseen errors
([see an example](https://github.com/mattfarina/golang-broken-vendor)).

#### Q: How do I import settings from GPM, Godep, or gb?

There are two parts to importing.

1. If a package you import has configuration for GPM, Godep, or gb Glide will
   recursively install the dependencies automatically.
2. If you would like to import configuration from GPM, Godep, or gb to Glide see
   the `glide import` command. For example, you can run `glide import godep` for
   Glide to detect the projects Godep configuration and generate a `glide.yaml`
   file for you.

Each of these will merge your existing `glide.yaml` file with the
dependencies it finds for those managers, and then emit the file as
output. **It will not overwrite your glide.yaml file.**

You can write it to file like this:

```
$ glide import godep > new-glide.yaml
```

#### Q: Can Glide fetch a package based on OS or Arch?

A: Yes. Using the `os` and `arch` fields on a `package`, you can specify
which OSes and architectures the package should be fetched for. For
example, the following package will only be fetched for 64-bit
Darwin/OSX systems:

```yaml
- package: some/package
  os:
    - darwin
  arch:
    - amd64
```

The package will not be fetched for other architectures or OSes.

## LICENSE

This package is made available under an MIT-style license. See
LICENSE.txt.

## Thanks!

We owe a huge debt of gratitude to the [GPM and
GVP](https://github.com/pote/gpm) projects, which
inspired many of the features of this package. If `glide` isn't the
right Go project manager for you, check out those.

The Composer (PHP), npm (JavaScript), and Bundler (Ruby) projects all
inspired various aspects of this tool, as well.

## The Name

Aside from being catchy, "glide" is a contraction of "Go Elide". The
idea is to compress the tasks that normally take us lots of time into a
just a few seconds.
