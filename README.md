factor-packages
===============

From the email discussion "[Factor-talk] A stub of a package manager"

Hi, following this [small discussion](https://news.ycombinator.com/item?id=8750720) I thought I
might give a try at designing a simple package manager.

It is inspired by [Metacello](https://code.google.com/p/metacello/)
but delegates to existing source control management software such as
git the actual handling of repositories, similar to
[Bower](http://bower.io/).

You can find the stub of a package manager
[here](https://github.com/andreaferretti/factor-packages) (keep in
mind: this is just the work of a day, and I am still a beginner).

Outline
-------

The central concept is that of a *project*. A project has a name and a
version, and contains some vocabularies. It can also depend on other
projects. There are syntax words to define the structure of a project,
which currently look like this (for a hypothetical project):


USING: packages.syntax
IN: example-project

PROJECT: example

VERSION: 0.2.0-alpha

VOCAB: example.thrift

VOCAB: example.main

DEPENDENCY: thrift
GITHUB{ "factor" "thrift" "0.2.5" }

One can then do the following in the listener

IN: scratchpad
USE: packages.projects
"example-project" activate

This clones the repository for each of the dependencies in a subfolder
of "resource:cache" and switches to the given version. It is expected
that each dependency contains a toplevel file `project.factor` as
above. Then, transitive dependencies are recursively fetched.

After fetching all dependencies, the vocabulary roots are adjusted,
and finally all vocabularies mentioned in the project or in a
dependency are `require`d.

If you import the package manager in your work folder, you can see a
[simple example](https://github.com/andreaferretti/factor-packages/blob/master/packages/example/example.factor)
in action. Just do

IN: scratchpad
USE: packages.projects
"packages.example" activate

You will then be able to use, say, the `monoid` dependency, like this

IN: scratchpad
3 5 |+|

I would like to get feedback on the above design, so as to understand
if it is worth the effort to develop this into something actually
usable. While very simplistic, I think this approach has a couple of
advantages:

* it does not require any change to the Factor vocabulary system
* by exploiting existing SCM (git and hg are currently supported, but
it would be easy to add more) one can avoid the need to setup a
repository and to develop a custom protocol.

Issues
------

If the above looks any good, there are a number of things one can
improve. All of those seem easy enough, they just requires some time.

* support other SCM (svn, darcs, bazaar, ...)
* improve the DSL (for instance, should we have VOCABS: ?)
* add support for publishing, as well as consuming packages. This
would allow to do something like `"mypackage" publish` and
- copy the vocabularies in the cache under the appropriate subfolder
- copy the definition file as project.factor
- commit the result
- tag with the declared version
- possibly sync it with online repositories
* add error reporting
* add standard stuff: tests, docs...
* general cleanup and refactoring
* GUI tools to support the workflows

There are also some issues one may want to consider:

* Are there any performance problems in supporting a possibly large
set of vocab-roots? (I think not, I see there is a cache for this)
* What about the files `project.factor`? Currently they do not follow
the standard conventions about file placement. Should I change the
directory structure to put them in valid places?
* In every package manager there is the issue of transitive
dependencies that are included multiple times with different versions.
As of now, the result of the above algorithm is that the last version
mentioned wins. What could be a better policy? One issue here is that
other package managers - Maven, for instance - fetch the specification
(in the form of pom.xml) separately from the artifact itself. This
allows them to have the complete picture *before* starting to download
dependencies, hence apply any policy they choose. The way I have
handled it, project.factor files coexist in the same repository as the
vocabularies themselves, which complicates things a bit (I have to
checkout the repositories to even look at dependencies, but later I
may change idea about which versions to use).
* If a package manager was used, what would be the impact on the
Factor ecosystem? After all, the current model of putting everything
in the main repository has some advantages:
- it is easier to discover libraries
- we avoid redundancy, which at this scale is important (no need for
anyone to make another json library, as it is already there)
Both points could be partially mitigated having a set of officially
endorsed packages, which are one click away in the listener. On the
other hand, it will not be feasible forever to have everything in the
main repository, and in particular this presents issues when
developing libraries which are tied to external projects whose
versions (and possibly protocols) do change (MySQL, Redis, Kafka,
ZeroMQ and so on).
