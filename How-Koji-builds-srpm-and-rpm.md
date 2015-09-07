## How Koji builds srpm and rpm
This article is meant for rpm packagers using ExclusiveArch tag and everyone interested in 3-phase process from building srpm to building rpm from it. Its aim is to explaing process of building rpm(s) from a spec file and use of ExclusiveArch. Basic knowledge of rpm packaging (spec file, tags, sections, %if* macros) and fedpkg command is required.

### Simple workflow
What is a usual workflow of building rpm(s) from a spec file?
```vim
$ rpmbuild -ba package-name.spec
```
Based on the spec file, result is one or more rpms. For fedora repository?
```vim
$ fedpkg local
```
This time, building procedure creates noarch or arch-specific directories (e.g. x86_64) which contains rpms. What about scratch-build?
```vim
$ fedpkg srpm
Wrote: /home/.../package-name-V-R.fc24.src.rpm
$ fedpkg scratch-build --srpm=/home/.../package-name-V-R.fc24.src.rpm
```
Building of rpm(s) is run in Koji as a testing (scratch) build which is not publicly available (unless url of the build is specified) and is removed in a few days. And finally, raw build?
```vim
$ fedpkg build
```
Again, building of rpm(s) is run in Koji. This time, the build is publicaly available [1] and does not get removed after few days.

[1] http://koji.fedoraproject.org/koji/

### Architectures and ExclusiveArch macro
Sometimes rpm(s) is not meant to be built on every available architecture. That is where ExclusiveArch tag comes [2]. A package builder, in this case Koji, uses the tag to specify that a given rpm gets built only on the specified architectures. The tag has a global scope (thus it is used in a top part of a spec file).

[2] http://www.rpm.org/max-rpm/s1-rpm-inside-tags.html

### Three-phase process of building rpm in Koji
Building of raw rpm in Koji (no scratch build) takes three steps:
1) build srpm from git repository on a random machine and choose architectures to build on
2) for each chosen architecture rebuilt srpm
3) build rpm from rebuilt srpm

In step 1) and 2), Koji (re)builds srpm with 'rpmbuild -bs' command with '--nodeps' option. Thus, all BuildRequires tags in a spec file are ignored. At the same time to be able to build rpms, minimal set of packages has to be installed in a buildroot (called minimal buildroot). So as long as ExclusiveArch tag is not defined by a specific macro (i.e. %{go_arches}) or is defined by macros provided by packages in the minimal buildroot, building is fine.

### ExclusiveArch tag and macros
For combination of ExclusiveArch and specific macros provided by a macro-package not in the minimal buildroot, i.e.
```vim
ExclusiveArch: %{go_arches}
```
packager has to make sure the package providing such a macro has to be included in the minimal buildroot. For example, making the macro package runtime dependency of redhat-rpm-config package. However, all such packages has to be justified before added to the minimal buildroot as all of them get installed into a buildroot of each Koji build.

### Exclusive arch of srpm for raw build and scratch build
When a srpm is built, it can end up with different value of ExclusiveArch. For an example above:
- if the srpm is built locally, value of %{go_arches} macro is evaluated based on a local definition of the macro.
- if the srpm is build in Koji, value of %{go_arches} macro is evaluated based on a definition in a package in the minimal buildroot.

When a srpm is built, list of exclusive architectures is listed in srpm's metainformation. The metainformation is created in a time of building srpm. So when a value of %{go_arches} differs on a local machine (macro package is not up-to-date or has been modified), ExclusiveArch tag can be evaluated differently. Thus, scratch build and raw build can choose different architectures and building can result in unexpected or misleading behaviour.

### How can I make subpackages or part of a spec file arch specific?
As ExclusiveArch is used by a package builder, it has no effect when used in a definition of a subpackage. To make a definition of a package arch specific, it can be wrapped with %ifarch macro [3]. The same holds for any other part of a spec file. E.g.
```vim
%ifarch %{gccgo_arches}
%package NAME
Summary:       Lorem ipsum dolor sit amet, consectetur adipiscing elit.
...
%description NAME
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
%endif
```

[3] http://www.rpm.org/max-rpm/s1-rpm-inside-conditionals.html
