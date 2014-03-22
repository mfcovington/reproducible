# Log::Reproducible (0.1.1)

Increase your reproducibility with the Perl module Log::Reproducible. Set it and forget it... *until you need it!*

## Usage

### Creating Archives

Just add these two lines near the top of your Perl script before accessing `@ARGV` or processing command line options with a module like [Getopt::Long](http://perldoc.perl.org/Getopt/Long.html):

```perl
use Log::Reproducible;
reproduce();
```

That's all!

Now, every time you run your script, the command line options and other arguments passed to it will be archived in a simple log file whose name reflects the script and the date/time it began running.

#### Other Archive Contents

Also included in the archive are (in order):

- custom notes, if provided (see [Adding Archive Notes](#adding-archive-notes), below)
- the date/time
- the working directory
- the directory containing the script
- git repository info, if applicable (see [Git Repo Info](#git-repo-info), below)

For example, running the script `use-reproducible.pl` would result in an archive file named `rlog-use-reproducible.pl-YYYYMMDD.HHMMSS`. If it was run as `perl bin/use-reproducible.pl -a 1 -b 2 -c 3 OTHER ARGUMENTS`, the contents of the archive file would be:

    use-reproducible.pl -a 1 -b 2 -c 3 OTHER ARGUMENTS
    #WHEN: YYYYMMDD.HHMMSS
    #WORKDIR: /path/to/working/dir
    #SCRIPTDIR: bin (/path/to/working/dir/bin)

### Reproducing an Archived Analysis

<!-- In order to reproduce an archived run, you can look at the archive contents and re-run the contents; however, that is a waste of time (and has the potential for typos or copy/paste errors).
 -->

To reproduce an archived run, all you need to do is run the script followed by `--reproduce` and the path to the archive file. For example:

```sh
perl use-reproducible.pl --reproduce rlog-use-reproducible.pl-YYYYMMDD.HHMMSS
```

This results in:

1. The script being executed with the command line options and arguments used in the original archived run
2. The creation of a new archive file identical to the older one (except with an updated date/time in the archive filename)

### Adding Archive Notes

Notes can be added to an archive using `--repronote`:

```sh
perl use-reproducible.pl --repronote 'This is a note'
```

If the note contains spaces, it must be surrounded by quotes.

Notes can span multiple lines:

```sh
perl use-reproducible.pl --repronote "This is a multi-line note:
The moon had
a cat's mustache
For a second
  — from Book of Haikus by Jack Kerouac"
```

### Where are the Archives Stored?

When creating or reproducing an archive, a status message gets printed to STDERR indicating where the archive's location. For example:

    Reproducing archive: /path/to/repro-archive/rlog-use-reproducible.pl-20140321.144307
    Created new archive: /path/to/repro-archive/rlog-use-reproducible.pl-20140321.144335

#### Default

By default, runs are archived in a directory called `repro-archive` that is created in the current working directory (i.e., whichever directory you were in when you executed your script).

#### Global

You can set a global archive directory with the environmental variable `REPRO_DIR`. Just add the following line to `~/.bash_profile`:

```sh
export REPRO_DIR=/path/to/archive
```

#### Script

You can set a script-level archive directory by passing the desired directory to the `reproduce()` function in your script:

```perl
reproduce("/path/to/archive");
```

This approach overrides the global archive directory settings.

#### Via Command Line

You can override all other archive directory settings by passing the desired directory to the on the command line when you run your script:

```sh
perl use-reproducible.pl --reprodir /path/to/archive
```

### Git Repo Info

*PSA: If you are writing, editing, or even just using Perl scripts and are at all concerned about reproducibility, you should be using [git](http://git-scm.com/) (or another version control system)!*

If git is installed on your system and your script resides within a git repository, a useful collection of info about the current state of the git repository will be included in the archive:

- Current branch
- Truncated SHA1 hash of most recent commit 
- Commit message of most recent commit 
- List of modified, added, removed, and unstaged files
- A summary of changes to previously committed files (both staged and unstaged)

An example of the git info from an archive:

    #GITCOMMIT: develop f483a06 Awesome commit message
    #GITSTATUS: M  staged-modified-file
    #GITSTATUS:  M unstaged-modified-file
    #GITSTATUS: A  newly-added-file
    #GITSTATUS: ?? untracked-file
    #GITDIFFSTAGED: diff --git a/staged-modified-file b/staged-modified-file
    #GITDIFFSTAGED: index ce2f709..a04c0f6 100644
    #GITDIFFSTAGED: --- a/staged-modified-file
    #GITDIFFSTAGED: +++ b/staged-modified-file
    #GITDIFFSTAGED: @@ -1,3 +1,3 @@
    #GITDIFFSTAGED:  An unmodified line
    #GITDIFFSTAGED: -A deleted line
    #GITDIFFSTAGED: +An added line
    #GITDIFFSTAGED:  Another unmodified line
    #GITDIFF: diff --git a/unstaged-modified-file b/unstaged-modified-file
    #GITDIFF: index ce2f709..a04c0f6 100644
    #GITDIFF: --- a/unstaged-modified-file
    #GITDIFF: +++ b/unstaged-modified-file
    #GITDIFF: @@ -1,3 +1,3 @@
    #GITDIFF:  An unmodified line
    #GITDIFF: -A deleted line
    #GITDIFF: +An added line
    #GITDIFF:  Another unmodified line

If you are familiar with git, you will be able to figure out that, in addition to a newly added file and an untracked file, there are two previously-committed modified files. One modified file has subsequently been staged (`staged-modified-file`) and the other is unstaged (`unstaged-modified-file`). Both modified files have had `A deleted line` replaced with `An added line`.

For most purposes, you might not need this information; however, if you need to determine the conditions that existed when you ran a script six months ago, these details could be critical!

## Future Features

Some features I may add are:

- Verbose mode
    - Print to screen the parameters used when reproducing an archived run
- Standalone script that can be used upstream of any command line functions
