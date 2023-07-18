- ### Successfully merged old code into tgrecojs/daemon-docs branch
- ```zsh
  $ git checkout tgrecojs/daemon-docs
  Switched to branch 'tgrecojs/daemon-docs'
  $ git rebase kris-daemon-host-guest
  warning: skipped previously applied commit 2dec975b1
  warning: skipped previously applied commit 4635aa369
  warning: skipped previously applied commit 40ca8884b
  warning: skipped previously applied commit 25a673fd7
  warning: skipped previously applied commit 1c5b22695
  warning: skipped previously applied commit 6f6ec803e
  warning: skipped previously applied commit d3b7ea230
  warning: skipped previously applied commit 82b244caf
  warning: skipped previously applied commit 9bb196ee6
  warning: skipped previously applied commit af88db4bd
  warning: skipped previously applied commit 0f89f5edb
  warning: skipped previously applied commit 1acb6ab83
  warning: skipped previously applied commit ae21af37d
  warning: skipped previously applied commit a3d48f943
  warning: skipped previously applied commit 819246ed1
  warning: skipped previously applied commit 97dd2e697
  warning: skipped previously applied commit 18a56b478
  warning: skipped previously applied commit 100f6b3ff
  warning: skipped previously applied commit c215c9d17
  warning: skipped previously applied commit c53e63abc
  warning: skipped previously applied commit f855ce40b
  warning: skipped previously applied commit 72d000a26
  warning: skipped previously applied commit 4710f16c1
  warning: skipped previously applied commit 4bcaf746c
  warning: skipped previously applied commit 13dd0fe12
  hint: use --reapply-cherry-picks to include skipped commits
  hint: Disable this message with "git config advice.skippedCherryPicks false"
  Successfully rebased and updated refs/heads/tgrecojs/daemon-docs.
  $ git status
  On branch tgrecojs/daemon-docs
  nothing to commit, working tree clean
  ```
- https://learngitbranching.js.org/
- To complete this level, do the following steps:
	- Make a new branch called `bugFix`
	- Checkout the `bugFix` branch with `git checkout bugFix`
	- Commit once
	- Go back to `main` with `git checkout`
	- Commit another time
	- Merge the branch `bugFix` into `main` with `git merge`
tags:: #[[Git]]
