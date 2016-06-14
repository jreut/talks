% Refactoring is Easy--er
% Jordan Reuter
% 14 June 2016

# A story

## Bug report

A user complains they cannot download a file.

. . .

### Some background

- I work for a company that ships a VM with a Web server on it.
- I'm a developer.
- I'm also the manager of the tech support department (__read:__ I answer the
phone most of the time).

<div class="notes">
- Who here talks directly with customers?
- Who here is on customer support?
- If you had to estimate, approximately what percentage of your support calls are the customer telling you how great your product is?
</div>

## So I get a phone call

### User can't download a file.

. . .

### I now know two things.

. . .

- We have a feature where users can download files.
- It is broken (sometimes).

## What's wrong?

### Initial troubleshooting

> - Trace the HTTP requests. I get a `500 Server Error` to `download.cgi`.
> - Look at the server logs:

`Error compressing /path/to/file/-^-_download: cat: invalid option -- '^': Try 'cat --help' for more information.: zip error: Invalid command arguments (short option '^' not supported)`

. . .

### First impressions

- Looks like I'm calling `cat` and `zip`.
- `zip` is a utility for compressing files.
- The `^` (carat) is in the file name and `cat`'s error message.
- Stinks of unsanitized input.
- Wait---why am I calling `cat` in the first place?

## Root cause

- Source code search for `zip`.
- The apparent source is `download.pl`, line 481:

. . .

```perl
$compress_command =<<CMD
  cd '$download_directory' && \
  cat '${archive_folder_name}.$a.list' | \
  zip -@ '${compressed_filename}';
CMD;
system $compress_command;
```

### Oh, I didn't tell you this is a Perl application?

## But what does it _do_?

The perl `system` function essentially executes its argument as a shell script.

. . .

```sh
cd '$download_directory'
```

Navigate to the temporary working directory where we gather some files.

. . .

```sh
cat '${archive_folder_name}.$a.list'
```

Read some file and...

. . .

```sh
zip -@ '${compressed_filename}'
```

...send its contents to the `zip` utility.

<div class="notes">
`-@` instructs the utility to use stdin as its file list rather than the command line.
</div>


## The problem {.t}

This is fundamentally an issue with improper (absent) escaping of a filename.
There are three ways to fix it, with three accompanying attitudes.

. . .

> - __This is easy:__ Hack together some sort of "sanitization" function for
`$archive_folder_name`.
> - __I don't understand this:__ Refactor the download code to make sense, and
then fix it sensibly.
> - __This code sucks:__ Rewrite the entire thing.

## The solution {.t}

This is fundamentally an issue with improper (absent) escaping of a filename.
There are three ways to fix it, with three accompanying attitudes.

> - ~~__This is easy:__ Hack together some sort of "sanitization" function for `$archive_folder_name`.~~
> - __I don't understand this:__ Refactor the download code to make sense, and then fix it sensibly.
> - __This code sucks:__ Rewrite the entire thing.

_This is a 1588 line procedural program. I'm not taking my chances._

## The solution {.t}

This is fundamentally an issue with improper (absent) escaping of a filename.
There are three ways to fix it, with three accompanying attitudes.

> - ~~__This is easy:__ Hack together some sort of "sanitization" function for `$archive_folder_name`.~~
> - __I don't understand this:__ Refactor the download code to make sense, and then fix it sensibly.
> - ~~__This code sucks:__ Rewrite the entire thing.~~

_It would be great to be able to say `DownloadableFile->new(my_domain_object)`,
but I'd have to throw away a ton of existing code. I'm not too confident I can
do that without breaking anything._

. . .

Another thing. __I don't have tests__.

<div class="notes">
So I'm left with one option.
</div>

# This is going to suck

## Understand the intent

- Use your source control.
- Reverse-engineer the code by looking for patterns. What sorts of design
decisions would cause you to write code like this?
- Define inputs and outputs. Find your edge cases.
- If you don't have tests, write a short program to drive the feature.
- Don't rely on comments. They might be wrong.

. . .

```perl
# Store each archive file in hash with filesize (KB)
my $archivefile =
  "${download_directory}/${compressed_filename}";
push @archivefiles, $compressed_filename;
```

<div class="notes">
That first line is an unused variable assignment. That second line is an array manipulation. No hashes or file sizes in sight.
</div>

## Respect prior art

### Your predecessors are not evil

> - They had time constraints.
> - They didn't know what you know now.
> - You don't know things they knew then.

### This code has value

> - It (perhaps badly) encodes the original intent.
> - You're going to have to figure out the intent anyway even if you rewrite.

## Add value

Provide a context for future maintainers (like your future self).

I committed the shell script I used to drive my tests. I rebased it into the
same commit as the bugfix itself to make it easier to search.

```
commit 1a22f13178f8bb234470f265aa0308caa3a0e59e
Author: Jordan Ryan Reuter <jordan.reuter@heartit.com>
Date:   Fri Jun 10 13:59:13 2016 -0400

    Refactor download archive generation

    * (fix #54) Use `run3` to protect against wild
      filenames by avoiding string interpolation.

    * Add tool to reprocess downloads.

    ...
```

## Submit a patch

- Your changes are going to be reviewed by someone else.
- The reviewer in general _wants_ to accept your patch.
- The reviewer wants to do as little work as possible.
- Diffs to existing code are probably easier to understand than wholesale rewrites.

# Everyone wins with humility

## When it comes to it, you still have to decide

### Sometimes a rewrite is just necessary.

- The code could be _that bad_.
- It could be so old it doesn't run.
- There could be nobody left who knows anything about it.

. . .

and even so...

> A rewrite is just a refactoring plus scope creep

## It's all about feelings

- Respect prior art
- Understand intent
- Add value
- Submit better patches

<div class="notes">
- look for patterns
- narrow the scope
- you're not always going to have tests

1. bug report
1. offending code
1. identify dependencies
1. use, write or hack up a test

in my case, the offender was a 
</div>
