---
title: Git Annex
date: 17-08-22
---

# Using git annex

I am finding that git annex allows me to have the best of both wolrds: on the
one hand I can track and manage my files as if they were in a local git
repository, on the other hand I can push the files I don't currently need onto
a remote storage location. What's more, the storage is actually my own
server, so I can circumvent inherent restrictions of some cloud providers
(e.g. I can have random rw and streaming capabilities via sshfs)

## How does it work?

### The basic idea

The actual file contents are moved into .git directory, and the any occurences of a file are substituted for its hash.

## Example workflow

First we need to clone the repository: 

```
git clone ssh://root@109.195.115.125:/mnt/8tbdisk/annex ~/annex
```

Git annex assumes that you can log in into the remote server; (and if you are
Misha, I presume you already have those)

Now you have a directory with a bunch of symlinks that lead nowhere. 

The most basic thing you might want to do is adding files. Do it like so:

1. Move the file into repository and commnd `git annex add` 

   This will add everything.  
   You can also specify files and folders to add

2. To update central repository use `git annex sync`

   This only syncs the information on file locations

3. Now suppose that you are pressed for space? You can move the files to the
   remote computer

   `git annex move --to origin`

   Here again you can specify what to move
  
4. After a white you might need the file again.
   
   Use `git annex get ./your-desired-file`

### Streaming 

But perhaps that doesn't work with video streaming? True. Git annex is mostly a
file tracker. It is useful in cases where you want the same conistent tree-ish
structure for big files on several computers. I now realize that I had wanted
exactly this use case for a while.

Sshfs allows to stream files:

```
sshfs root@109.195.115.125:/mnt/8tbdisk ~/mount/mn2
```

### File management

You can move links and directories around at will (simply move the link to
another place) and commit. This works because symlinks point to hashes. 

### Unlocked files

Sometimes it's not convenient to work with symlinks. You can unlock the files
in the tree then. Read the docs for more.
