# Debian packaging/PPAs

- [Debian packaging/PPAs](#debian-packagingppas)
  - [Versioning](#versioning)
  - [Introduction to source packages](#introduction-to-source-packages)
    - [Dowloading source packages](#dowloading-source-packages)
  - [Building a source package](#building-a-source-package)
  - [Package metadata (files under `debian/`)](#package-metadata-files-under-debian)
    - [`debian/control`](#debiancontrol)
    - [`debian/rules`](#debianrules)
    - [`debian/changelog`](#debianchangelog)
  - [Patch systems](#patch-systems)
  - [Running commands during installation/removal](#running-commands-during-installationremoval)
  - [Tools](#tools)
    - [Cowbuilder (pbuilder)](#cowbuilder-pbuilder)
    - [Packaging with a Version Control System](#packaging-with-a-version-control-system)
  - [Building source packages](#building-source-packages)
    - [Extra steps not mentioned in the Ruby packaging article](#extra-steps-not-mentioned-in-the-ruby-packaging-article)
    - [Issues](#issues)
  - [References](#references)

## Versioning

```
1.2.1.1-5
\--+--/ +--- debian version
   |
   upstream version
```

## Introduction to source packages

A single source package can create more binary packages.

The `.dsc` file is the main one.

"Native" are packages for apt/dpkg; "Quilt" is a source format.

- 1.0 native: `package_version.tar.gz`
- 3.0 Quilt:
  - `pkg_ver.orig.tar.gz`: upstream source
  - `pkg_debver.debian.tar.gz` Debian changes

The directory with the source package content must be `<lowercase_name>-<version>`

Sample structure of a downloaded package source:

```
dash-0.5.8/
dash_0.5.8-2.1ubuntu2.diff.gz
dash_0.5.8-2.1ubuntu2.dsc
dash_0.5.8.orig.tar.gz
```

### Dowloading source packages

In the apt configuration (`/etc/apt`) files, source repositories are simplly prefixed with `deb-src` instead of `deb`.

When adding a PPA via `add-apt-repository`, the option `-s` will automatically enable the corresponding source repository.

## Building a source package

For the tutorial, see the article in the blog, and the script in the `openscripts` repository.

## Package metadata (files under `debian/`)

Main files:

-  control – meta-data about the package (dependencies, etc.)
-  rules – specifies how to build the package
-  copyright – copyright information for the package
-  changelog – history of the Debian package

### `debian/control`

Most important - lots of metadata.

### `debian/rules`

This is a Makefile. Generally, use the `dh` (`Debhelper 7`) packaging helper.

### `debian/changelog`

Edit manually, or via `dch`.

Installed as `/usr/share/doc/<package>/changelog.Debian.gz`.

Information about the package version (therefore, the package filename) is gathered from this file.

## Patch systems

Patches stored in `debian/patches` (for example, `grep` has a `debian/patches/series` with the list of patches).

Patches are applied and unapplied during build.

There are different systems; the recommended is [Quilt](https://pkg-perl.alioth.debian.org/howto/quilt.html).

Patches have a standard header [format](http://dep.debian.net/deps/dep3).

## Running commands during installation/removal

Defined in the "maintainer" scripts: `preinst`, `postinst`, `prerm`, `postrm`.

Prompting the user must be done with `debconf`.

## Tools

Base tools:

- `debuild`: build, test with `lintian`, sign with GPG
- `pbuilder`: build packages in chrooted environment
- `dput`: upload package

### Cowbuilder (pbuilder)

Cowbuilder is a wrapper around pbuilder+cowdancer, which makes building faster:

```sh
sudo cowbuilder --create                  # stored as `/var/cache/pbuilder/base.cow/`
sudo cowbuilder --update
sudo cowbuilder --build $package_name.dsc # creates `/var/cache/pbuilder/result/$package_name.deb`

debsign /var/cache/pbuilder/result/$package_name.changes # signs both the `dsc` and the `changes` files
```

In order to make cowbuilder closer to Launchpad, run:

```sh
printf $'\nEXTRAPACKAGES="dwz pkgbinarymangler"\n' | sudo tee -a /etc/pbuilderrc
```

### Packaging with a Version Control System

There are tools supporting VCSs, eg. [`gbp`](https://honk.sigxcpu.org/projects/git-buildpackage/manual-html/gbp.html).

- `upstream` branch to track upstream with `upstream/<version>` tags
- `master` branch tracks the Debian package
- `debian/<version>` tags for each upload
- `pristine-tar` branch to be able to rebuild the upstream tarball

`Vcs-*` fields in `debian/control` to locate the repository:

- `Vcs-Browser:http://anonscm.debian.org/gitweb/?p=collab-maint/devscripts.git`
- `Vcs-Git:git://anonscm.debian.org/collab-maint/devscripts.git`

## Building source packages

### Extra steps not mentioned in the Ruby packaging article

Using `dch`:

```sh
# Run this after `dh_make`, then use `dch` on subsequent iterations in order ot add entries.
#
perl -i -pe "s/unstable/$(lsb_release -cs)/" debian/changelog

# Using `dch`
# Requires `libdistro-info-perl` package.
#
# -v: specify the version
# -i: increment the debian release number
# -U: increment the upstream release number
#
# If `-D` is not specified, the current entry distro will be `UNRELEASED`, and change are applied
# to it (without creating a new version)
#
dch -D xenial -v 0.0.20 $'Add a certain functionality\nFix a certain bug'
```

Other infos:

```sh
# Update the copyright file, when a standard license is specified.
#
perl -i -0777 -pe 's/Copyright: \K.+\n +.+/2018 Barry Foo <barry@foo.com>/' debian/copyright

# Use when Lintian doesn't recognize the source of the built binary:
#
echo "[linux-any]: source-is-missing src/github.com/goby-lang/goby/goby" > debian/source/lintian-overrides

# Copy extra files not installed by the makefile. Relative to `src/`.
# If the files are binary, the source will need to be provided as well, otherwise lintian will warn
# `source: source-is-missing`.
#
echo 'goby_extra_script.sh /usr/lib/goby-0.1.9/' >> debian/install

# Extra rules: set new variable (add to 6th line).
#
sed -i -E '6iexport PATH := /usr/lib/go-1.10/bin:$(PATH)' debian/rules
```

### Issues

In the following cases, the PPA won't show anything, but there will be no errors:

- no source packages are uploaded;
- the GPG key is not present in the launchpad configuration.

On signature error `Error checking signature from $KEY_ID`, run:

```sh
# Verify the error
gpg --verify $changefile

# Trust the key
printf $'trust\n5\ny\n' | gpg --command-fd 0 --edit-key $key_id
```

"Published" status in the PPA packages page is misleading (packages may not be yet published). One must either click on the package, or go to the per-package "Build Status" page.

## References

Useful links:

- https://www.debian.org/doc/devel-manuals#packaging-tutorial
- https://www.debian.org/doc/developers-reference
- http://developer.ubuntu.com/resources/tools/packaging/
- https://www.debian.org/doc/debian-policy/ch-maintainerscripts
- https://www.debian.org/doc/developers-reference/best-pkging-practices.html
- https://people.debian.org/~srivasta/MaintainerScripts.html
- http://packaging.ubuntu.com/html/packaging-new-software.html
- https://wiki.debian.org/cowbuilder#Building_your_package_for_many_distributions_at_once
