% ALL YOUR REBASE ARE BELONG TO US
% or: the internals of "the stupid content tracker"

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

# merging

# the rebase
