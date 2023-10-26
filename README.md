# git-update-faithful

`git-update-faithful` is a collection of shells commands you can
use to maintain so-called *boilerplate dependencies*.

Boilerplate dependencies are basically common files shared across
similar projects, but that are separately committed to each project
repository.

These are not dependencies you can just manage with a package
manager (like `pip`, or `node`), but rather they're integral to
each project and are considered first-class (file) citizens.

Consider, for example, starting a project from one of the 12,000
and counting
[boilerplate projects on GitHub](https://github.com/topics/boilerplate).
What happens when the boilerplate author adds a fancy new feature
after you've used their boilerplate to start a new project? Do you
manually incorporate the changes into your project (or *projects*)?
No, you automate the process, or at least that's what `update-faithful`
does.

## Usage

First, setup the update operation. Tell `update-faithful` the path
to the *canon* repo. This is the boilerplate project with which
you'd like to keep a derived (downstream) project current.

You can either set a global environ variable, or you can call the
begin operation:

  ```
  # Tell update-faithful canon path, option 1:
  export UPDEPS_CANON_BASE_ABSOLUTE="/full/path/to/boilerplate"

  # Tell update-faithful canon path, option 2:
  update-faithful-begin "/full/path/to/boilerplate"
  ```

Next, pull changes from those files you want to keep current with.
This is generally equivalent to a `cp` operation, albeit with
elaborate state checking to ensure that any changes you might
have made will not be clobbered. However, if you use `git-put-wise`
(see below), this will not copy any "PRIVATE" changes you've made
to the canon sources.

E.g., here are some Python files that are shared verbatim between
my Python projects:

  ```
  update-faithful-file "codecov.yml"
  update-faithful-file "CODE-OF-CONDUCT.rst"
  update-faithful-file ".editorconfig"
  update-faithful-file "Makefile"
  update-faithful-file ".readthedocs.yml"
  update-faithful-file "tox.ini"
  update-faithful-file ".yamllint"
  ```

You can also *render* files when the differences are trivial, e.g.,
if a derived project file just needs to change the project name
and URL, you can easily templatize the difference, e.g.,:

  ```
  render-faithful-file "CONTRIBUTING.rst"
  render-faithful-file "docs/index.rst"
  ```

Finally, finish the operation and commit the changes:

   ```
   update-faithful-finish
   ```

Take a look at the generated commit to see how `update-faithful`
prepares for the next update operation. It records the SHA from
the canon project that it copied or generated files from. That
way, the next time you run the update, it'll be smart enough to
fail if you accidentally edited a boilerplate file and it now
deviates from canon. Otherwise, `update-faithful` verifies that
the derived file is unchanged from the previous update (using
the SHA reference), and then it only updates the file if it's
confident that it won't clobber any changes you might have made.

- Here's an example `update-faithful-finish` commit from this project:

  https://github.com/thegittinsgood/git-update-faithful/commit/453b7e9ab142560125cbac6b40ca7f6514182faf

### Actual Examples

The author uses `git-update-faithful` to keep a set of *Shoilerplate*
dependencies synced. Any by "shoilerplate", I mean Shell code boilerplate,
because there's not a built-in, obvious way to manage inter-project
shell dependencies. So I devised my own strategy: I copy shell
dependencies (like a Python-esque logger) under a `deps/` directory
in each shell project; I use a hard link so that, at least locally,
all projects use the same sources; and then I call a custom update
function that uses `git-update-faithful` to keep all my dependencies
current. (This also means I don't have to manually, tediously git-add
and git-commit to every project after I edit an upstream source file.)

You can see how this is implemented in the
[DepoXy](https://github.com/DepoXy/depoxy#🍯)
project [ohmyrepos](https://github.com/landonb/ohmyrepos#😤) (OMR)
config:

- https://github.com/DepoXy/depoxy/blob/release/home/.config/ohmyrepos/my-deps-manage-shoilerplate.sh

(Note that DepoXy is essentially a development environment orchestrator.
I install it to all new dev machines, and it wires all my config and
inter-project dependencies. And *[OMR](https://github.com/landonb/ohmyrepos#😤)* (Oh! My Repos) is a library of
[myrepos](https://myrepos.branchable.com/) extensions that I use to
manage the hundreds of Git repos I install to every dev machine.)

Another example is from
[easy-as-pypi](https://github.com/doblabs/easy-as-pypi#🥧) (EAPP).
EAPP is boilerplate for a handful of core Python projects I use in many 
end-user Python apps that I maintain. (One of these projects manages user
config values; another managers terminal I/O; another simply wraps `appdirs`;
etc.). Each of the EAPP-derived projects share a common set of files, like
`Makefile`, `.codecov.yml`, etc., and even GitHub Action scripts, under
`.github/workflows/`. So all I need to do is to edit the source EAPP file,
and then I `cd` to each derived project and run the `update-deps` script:

- https://github.com/doblabs/easy-as-pypi/blob/release/bin/update-deps

### Git-Put-Wise Compatible

The `update-faithful` commands are git-put-wise-aware:

  https://github.com/DepoXy/git-put-wise#🥨

So if you use "PRIVATE" commits in the canon project, these commits
will not be copied to the downstream projects. The `update-faithful`
operation will only copy or render the latest publicly-scoped commit.

Note that if you don't use `git-put-wise`, then `update-faithful` will
simply copy or render from canon `HEAD`.

## Setup

None. The project is self-contained.

But note you cannot run copies of the scripts; they must be run from
within the project directory.

- I.e., you cannot `cp update-faithful-file ~/.local/bin` and expect it to work.

  But you can `ln -s "$(realpath "update-deps")" ~/.local/bin` and it'll work.

## Compares to

Nothing that I know about! But please PR this README if you know of
other tools that manage shell and boilerplate dependencies similarly.

