---
layout: post
title: "fatal: The current branch master has multiple upstream branches, refusing to push."
description: ""
category:
tags: ['git', 'github']
---

Sometimes when you have a github repository, you get the above error message.

The resolution is documented on stackoverflow [here](http://stackoverflow.com/questions/13030714/git-fatal-the-current-branch-master-has-multiple-upstream-branches-refusing-t).

Tell your git repository to push HEAD to the remote origin.

```
git config remote.origin.push HEAD
```

For convenience, I wrap this in a bash utility script:

```
#!/bin/bash

echo "setting 'git config remote.origin.push HEAD'... ✅️"
git config remote.origin.push HEAD

```
