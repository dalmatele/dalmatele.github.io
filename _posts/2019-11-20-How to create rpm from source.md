---
layout: post
title: "How to create a Makefile"
date: 2019-11-20 16:35:10 -0000
categories: Centos
---

## Prepare tools
On a RPM-based system, install the following programs:
```
yum install gcc rpm-build rpm-devel rpmlint make python bash coreutils diffutils patch rpmdevtools
```
To init your enviroment, you must run bellow command:
```
rpmdev-setuptree
```
The created directories serve these purpose:

Directory | Purpose
----------|---------
BUILD     | When packages are built, various %buildroot directories are created here. This is useful for investigating a failed build if the logs output do not provide enough information.
RPMS      | Binary RPMs are created here, in subdirectories for different architectures, for example in subdirectories `x86_64` and `noarch`.
SOURCES   | Here, the packager puts compressed source code archives and patches. The `rpmbuild` command looks for them here.
SPECS     | The packager puts SPEC files here.
SRPMS     | When `rpmbuild` is used to build an SRPM instead of a binary RPM, the resulting SRPM is created here.

## Create your program
### Prepare source
You must put your source code directory in SOURCES folder.
### Prepare spec file
A SPEC file can be thought of as the "recipe" that the `rpmbuild` utility uses to actually build an RPM. It tells the build system what to do by defining instructions in a series of sections. The sections are defined in the *Preamble* and the *Body*. The *Preamble* contains a series of metadata items that are used in the Body. The *Body* contains the main part of the instructions.
####Preamable Items
This table lists the items used in the Preamble section of the RPM SPEC file:
SPEC Directive | Definition
---------------|-----------
Name           | The base name of the package, which should match the SPEC file name.
Version        | The upstream version number of the software.
Release        | The number of times this version of the software was released. Normally, set the initial value to 1%{?dist}, and increment it with each new release of the package. Reset to 1 when a new Version of the software is built.
Summary        | A brief, one-line summary of the package.
License        | The license of the software being packaged. ifdef::community[] For packages distributed in community distributions such as Fedora this must be an open source license abiding by the specific distribution’s licensing guidelines. endif::community[]
URL            | The full URL for more information about the program. Most often this is the upstream project website for the software being packaged.
Source0        | Path or URL to the compressed archive of the upstream source code (unpatched, patches are handled elsewhere). This should point to an accessible and reliable storage of the archive, for example, the upstream page and not the packager’s local storage. If needed, more SourceX directives can be added, incrementing the number each time, for example: Source1, Source2, Source3, and so on.
Patch0         | The name of the first patch to apply to the source code if necessary. If needed, more PatchX directives can be added, incrementing the number each time, for example: Patch1, Patch2, Patch3, and so on.
BuildArch      | If the package is not architecture dependent, for example, if written entirely in an interpreted programming language, set this to BuildArch: noarch. If not set, the package automatically inherits the Architecture of the machine on which it is built, for example x86_64.
BuildRequires  | A comma- or whitespace-separated list of packages required for building the program written in a compiled language. There can be multiple entries of BuildRequires, each on its own line in the SPEC file.
Requires       | A comma- or whitespace-separated list of packages required by the software to run once installed. There can be multiple entries of Requires, each on its own line in the SPEC file.
ExcludeArch    | If a piece of software can not operate on a specific processor architecture, you can exclude that architecture here.

The `Name`, `Version`, and `Release` directives comprise the file name of the RPM package. RPM Package Maintainers and Systems Administrators often call these three directives N-V-R or NVR, because RPM package filenames have the NAME-VERSION-RELEASE format.

You can get an example of an NAME-VERSION-RELEASE by querying using `rpm` for a specific package:

```
$ rpm -q python
python-2.7.5-34.el7.x86_64
```

Here, `python` is the Package Name, `2.7.5` is the Version, and `34.el7` is the Release. The final marker is x86_64, which signals the architecture. Unlike the NVR, the architecture marker is not under direct control of the RPM packager, but is defined by the rpmbuild build environment. The exception to this is the architecture-independent noarch package.

#### Body Items

This table lists the items used in the Body section of the RPM SPEC file:

SPEC Directive | Defination
---------------|------------
%description   | A full description of the software packaged in the RPM. This description can span multiple lines and can be broken into paragraphs.
%prep          | Command or series of commands to prepare the software to be built, for example, unpacking the archive in Source0. This directive can contain a shell script
%build         | Command or series of commands for actually building the software into machine code (for compiled languages) or byte code (for some interpreted languages).
%install       | Command or series of commands for copying the desired build artifacts from the %builddir (where the build happens) to the %buildroot directory (which contains the directory structure with the files to be packaged). This usually means copying files from ~/rpmbuild/BUILD to ~/rpmbuild/BUILDROOT and creating the necessary directories in ~/rpmbuild/BUILDROOT. This is only run when creating a package, not when the end-user installs the package. See [Working with SPEC files](https://rpm-packaging-guide.github.io/#working-with-spec-files) for details.
%check         | Command or series of commands to test the software. This normally includes things such as unit tests.
%files         | The list of files that will be installed in the end user’s system.
%changelog     | A record of changes that have happened to the package between different Version or Release builds.

#### Advance items
The SPEC file can also contain advanced items. For example, a SPEC file can have scriptlets and triggers. They take effect at different points during the installation process on the end user’s system (not the build process).

See the [Scriptlets and Triggers](https://rpm-packaging-guide.github.io/#triggers-and-scriptlets) for advanced topics.

#### Working with SPEC files

To package new software, you need to create a new SPEC file. Instead of writing it manually from scratch, use the `rpmdev-newspec` utility. It creates an unpopulated SPEC file, and you fill in the necessary directives and fields.
```
rpmdev-newspec test
```

### Build and test
#### Source RPMs
To create a SRPM:
```
$ cd ~/rpmbuild/SPECS/
$ rpmbuild -bs _SPECFILE_
```
Change `_SPECFILE_` by your spec file.

#### Binary RPMS
There are two methods for building Binary RPMs:

1. Rebuilding it from a SRPM using the `rpmbuild --rebuild` command.

2. Building it from a SPEC file using the `rpmbuild -bb ` command. The -bb option stands for "build binary"
##### Rebuilding from a source RPM
Now you have built RPMs. A few notes:

* The output generated when creating a binary RPM is verbose, which is helpful for debugging. The output varies for different examples and corresponds to their SPEC files.

* The resulting binary RPMs are in ~/rpmbuild/RPMS/YOURARCH where YOURARCH is your architecture or in ~/rpmbuild/RPMS/noarch/, if the package is not architecture-specific.

* Invoking rpmbuild --rebuild involves:

    1. Installing the contents of the SRPM - the SPEC file and the source code - into the ~/rpmbuild/ directory.

    2. Building using the installed contents.

    3. Removing the SPEC file and the source code.

You can retain the SPEC file and the source code after building. For this, you have two options:

* When building, use the --recompile option instead of --rebuild.

* Install the SRPMs using these commands:
```
$ rpm -Uvh ~/rpmbuild/SRPMS/test.src.rpm
Updating / installing...
   1:test                  ################################# [100%]
```

##### Building Binary from the SPEC file

To build `test` from SPEC file, run:
```
$rpmbuild -bb ~/rmpbuild/SPEC/test.spec
```

[Source link](https://rpm-packaging-guide.github.io/#rpm-packaging-tools)
