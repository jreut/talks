% ALL YOUR REBASE ARE BELONG TO US
% or: the internals of "the stupid content tracker"
% 24 July 2017

# Git internals

## empty repo

```sh
% cd $TMPDIR
% git init my-little-repo
% cd my-little-repo
% tree .git

.git
├── HEAD
├── config
└── objects/
    └── 65/
        └── 66899ce31c049799f8f79c2ec72092ddc31fc8
```

## `.git/objects`

- single "table" database

- content-addressable

- indexed by content hash

## writing objects

### we can generate our own hashes:

```sh
% echo 'Hello, git!' | git hash-object --stdin

6566899ce31c049799f8f79c2ec72092ddc31fc8
```

### we can write our own objects:

```sh
% echo 'Hello, git!' | git hash-object --stdin -w
% tree .git/objects

.git/objects
└── 65/
    └── 66899ce31c049799f8f79c2ec72092ddc31fc8
```

## reading objects

### this doesn't work

```sh
% cat .git/objects/65/66899ce31c049799f8f79c2ec72092ddc31fc8
```

### this does

```sh
% git cat-file -p 6566899ce31c049799f8f79c2ec72092ddc31fc8

Hello, git!
```

## why?

- header: "\<object type\> \<size in bytes\>\<null byte\>\<contents\>"

- zlib compressed

- `git cat-file` looks up the object, decompresses it, validates the file, and prints the contents

## blobs

### "files"

```sh
% git cat-file -t 6566899ce31c049799f8f79c2ec72092ddc31fc8

blob

% git cat-file -p 6566899ce31c049799f8f79c2ec72092ddc31fc8

Hello, git!
```

## trees

### "directories"

```sh
% echo 'Hello, git!' > a
% git add a
% git ls-files --stage

100644 6566899ce31c049799f8f79c2ec72092ddc31fc8 0	a
```

| mode | object id | | filename |
|------|-----------|-|----------|
|`100644`|`6566899...`|`0`|`a`|

Table: the format of a tree^[kinda. this is a special tree for the "stage".]

## commits

```sh
% git status

...
	new file:   a
...

% git commit -m 'Add file "a"'

[master (root-commit) a2459f4] Add file "a"
 1 file changed, 1 insertion(+)
 create mode 100644 a
```

## commits

```sh
% git cat-file -p HEAD

tree 3dafa3ae74c689274be2cc26b53d7d850f7a79e1
author Jordan Ryan Reuter <me@jreut.com> 1499427764 -0400
committer Jordan Ryan Reuter <me@jreut.com> 1499427764 -0400

Add file "a"
```

- one tree
- zero or more parents
- one author with timestamp
- one committer with timestamp
- one message

## trees again

```sh
# these are equivalent
% git ls-tree HEAD
% git cat-file -p HEAD^{tree}

100644 blob 6566899ce31c049799f8f79c2ec72092ddc31fc8	a
```

|mode|type|object id|filename|
|----|----|---------|--------|
|`100644`|`blob`|`6566899...`|`a`|

Table: the real format of a tree

## so far

- objects stored in a database
- blobs, trees, commits

```sh
% tree .git/objects

.git/objects
├── 3d/ # the root commit's tree
│   └── afa3ae74c689274be2cc26b53d7d850f7a79e1
├── 65/ # our blob
│   └── 66899ce31c049799f8f79c2ec72092ddc31fc8
└── a2/ # the root commit
    └── 459f444c58cd71cd1e47367faeecae8acfd15d
```

# fun with trees

## copying a file

```sh
% cp a b
% git add b
% git commit -m 'Copy file "a" to "b"'

[master 84d0a32] Copy file "a" to "b"
 1 file changed, 1 insertion(+)
 create mode 100644 b

% git ls-tree HEAD

100644 blob 6566899ce31c049799f8f79c2ec72092ddc31fc8	a
100644 blob 6566899ce31c049799f8f79c2ec72092ddc31fc8	b
```

look, we reused the blob!

## copying a file

```sh
% tree .git/objects

.git/objects
├── 3d/ # the root commit's tree
│   └── afa3ae74c689274be2cc26b53d7d850f7a79e1
├── 65/ # our lonely blob
│   └── 66899ce31c049799f8f79c2ec72092ddc31fc8
├── 6e/ # the second commit's tree
│   └── 69a27840b4463b0cbf90e69f6b9f9426c552e2
├── 84/ # the second commit
│   └── d0a3247cb13a4aebdcf4e11b4b2b4cc33c2205
└── a2/ # the root commit
    └── 459f444c58cd71cd1e47367faeecae8acfd15d
```

## making subdirectories

```sh
% mkdir folder
% cp a folder/c
% git add folder/c
% git commit -m 'Add a subdirectory'

[master cbf2bd2] Add a subdirectory
 1 file changed, 1 insertion(+)
 create mode 100644 folder/c
```

## making subdirectories

```sh
% git ls-tree HEAD
```

. . .

```sh
100644 blob 6566899ce31c049799f8f79c2ec72092ddc31fc8	a
100644 blob 6566899ce31c049799f8f79c2ec72092ddc31fc8	b
040000 tree e64c2de4724a32cf98e1aa3d16aff43d07d828f2	folder

% git ls-tree HEAD:folder

100644 blob 6566899ce31c049799f8f79c2ec72092ddc31fc8	c
```

# interlude

## what's up with `.gitkeep`?

- tree objects _must_ be nonempty

- `.gitkeep` is a convention for making a nonempty directory

## the graph

```sh
% tree .git/objects

.git/objects
├── 3d/ # the root commit's tree
│   └── afa3ae74c689274be2cc26b53d7d850f7a79e1
├── 65/ # our lonely blob
│   └── 66899ce31c049799f8f79c2ec72092ddc31fc8
├── 6e/ # the second commit's tree
│   └── 69a27840b4463b0cbf90e69f6b9f9426c552e2
├── 84/ # the second commit
│   └── d0a3247cb13a4aebdcf4e11b4b2b4cc33c2205
├── a2/ # the root commit
│   └── 459f444c58cd71cd1e47367faeecae8acfd15d
├── cb/ # the third commit (our current HEAD)
│   └── f2bd279d18b6cec6cff122ff1379cf23655e9f
├── dd/ # the third commit's toplevel tree
│   └── 9127813ea3c413faa7204454829b9cffdb8dd8
└── e6/ # the third commit's subtree
    └── 4c2de4724a32cf98e1aa3d16aff43d07d828f2
```

## the graph

![all the objects in our repository](img/git-graph.pdf)

# refs

## references to commits

### In a new repo...

```sh
% tree .git

.git
├── HEAD
├── config
├── objects/
│   ├── info/
│   └── pack/
└── refs/
    ├── heads/
    └── tags/
```


## refs are just files

```sh
% git commit --allow-empty --allow-empty-message

[master (root-commit) 8495043]

% tree .git/refs

.git/refs
├── heads/
│   └── master
└── tags/

% cat .git/refs/heads/master

8495043ef1734f7d23a6d116e596bd97e404e48c
```


## make our own ref

```sh
% echo 8495043... > .git/refs/my-great-ref
% git log my-great-ref

commit 8495043ef1734f7d23a6d116e596bd97e404e48c
Author: Jordan Ryan Reuter <me@jreut.com>
Date:   Fri Jul 14 17:27:23 2017 -0400
```

## checkout our ref

```sh
% git checkout my-great-ref

You are in 'detached HEAD' state.

% cat .git/HEAD

8495043ef1734f7d23a6d116e596bd97e404e48c
```

### what does that mean?

```sh
% mv .git/refs/my-great-ref .git/refs/heads/my-great-ref
% git checkout my-great-ref
% cat .git/HEAD

ref: refs/heads/my-great-ref
```

## refs and branches

### branch = file in `refs/heads`

```sh
% git branch

  master
* my-great-ref
```

# interlude

## how it's made: `git checkout -b my-branch`

1. read current `HEAD`
1. write `refs/heads/my-branch`
1. write new `HEAD` with `ref: refs/heads/my-branch`

## how it's made: `git commit ...`

1. read current `HEAD` (say, `ref: refs/heads/my-branch`)
1. write commit with "stage" tree, current head as parent, etc.
1. write new commit id to `refs/heads/my-branch`

## how it's made: `git reset some-ref`

1. read current `HEAD` (say, `refs: refs/heads/my-branch`)
1. read commit referenced by `some-ref`
1. write commit id to `refs/heads/my-branch`

# rewriting history

## not so scary!

- committed on the wrong branch
- squash many commits into one
- bring an old feature branch up to date

## committed on the wrong branch

### our normal workflow

1. pull latest `develop`
1. checkout a feature branch
1. do some work
1. push feature branch and open merge request

### what I sometimes do

1. pull latest develop
1. forget to checkout a feature branch
1. do some work
1. realize I just committed to develop
1. fix it really easily!

## committed on the wrong branch

![before](img/reset-develop-1.pdf)

## committed on the wrong branch

![accidentally committing to develop](img/reset-develop-2.pdf)

## committed on the wrong branch

![checkout a new branch](img/reset-develop-3.pdf)

## committed on the wrong branch

![reset develop](img/reset-develop-4.pdf)

## committed on the wrong branch

```sh
% git checkout -b my-feature    # make a new feature branch
% git checkout develop          # go back to develop
% git reset --hard HEAD^        # point develop to previous commit
% git checkout my-feature       # go back to feature branch
% git push -u origin my-feature # push and enjoy
```

## squash many commits into one

### why?

- group related changes together
- clean up mistakes

## squash many commits into one

![before](img/squash-1.pdf)

## squash many commits into one

![make a new commit](img/squash-2.pdf)

## squash many commits into one

![squash into one](img/squash-3.pdf)

## squash many commits into one

```sh
% git commit -m 'some feature'
% git push -u origin my-feature
% git commit --fixup=HEAD
% git rebase --autosquash HEAD^
% git push -uf origin my-feature
```

## bring an old feature branch up to date

### our normal workflow

1. do work
1. wait too long
1. merge conflicts!
1. "could you merge develop?"

### what I usually do

1. do work
1. wait too long
1. merge conflicts!
1. rebase onto develop

## bring an old feature branch up to date

![before](img/rebase-1.pdf)

## bring an old feature branch up to date

![wait too long](img/rebase-2.pdf)

## bring an old feature branch up to date

![catch up](img/rebase-3.pdf)

## bring an old feature branch up to date

```sh
% git commit -m 'some feature'
% git push -u origin my-feature
# ...several days pass...
% git remote update origin
% git rebase origin/develop
% git push -uf origin my-feature
```

# let's do it live

## let's do it live

- interactive rebase
- using reflogs
- working with remotes
- anything else?

# appendix

## references

- [Pro Git (free!)](https://git-scm.com/book/en/v2)
    - [§3.2: Basic Branching & Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging#_basic_merging)
    - [§7.1: Revision Selection](https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection)
    - [§10: Git Internals](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain)

- manpages (they're great!)
    - _gitrevisions(7)_ and _git-rev-parse(1)_
    - _git-rebase(1)_
