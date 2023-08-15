title: git

# **git**


------------------------------------------------------------------------------------------------------------------------
## **git pull force** --- [[1](https://stackoverflow.com/questions/67054889/force-git-pull-to-resolve-divergent-by-discard-all-local-commits)]

This will:

1. get rid of all local uncommited changes
2. get rid of all local commits
3. and bring your local branch in sync with remote

```bash
git fetch && git reset --hard @{upstream}
```



------------------------------------------------------------------------------------------------------------------------
## **Sign & Verify commits**

Official [git docs](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work)
and [github docs](https://docs.github.com/en/authentication/managing-commit-signature-verification/displaying-verification-statuses-for-all-of-your-commits)

### Signing commits

* Tell git to sign commits: `#!bash git config commit.gpgsign true # --global `
* Specify what key to use by default:
    * Choose the fingerprint of an `S` key: `gpg --list-secret-keys --keyid-format=long`
    * `git config user.signingkey 0A46826A! # --global`
* When committing add `-S`: `git commit -S -m "Test signing"`
* You might want to add gpg key to github, export it via: `gpg --export --armour  --fingerprint ...`


### Verification

* During pull: `git pull --verify-signatures`
    * Make it the default: `#!bash git config merge.verifySignatures true # --global`
* In log: `git log --show-signature`
    * Make it the default: `#!bash git config log.showSignature true # --global`



------------------------------------------------------------------------------------------------------------------------
## **grep history**
Look for the given regex: `#!bash git log -G "#include <sys/ustat.h>"`

Look at the number of occurrences: `#!bash git log -S "#include <sys/ustat.h>"`



------------------------------------------------------------------------------------------------------------------------
## **Multiple branches simultaneously** (in a single repo)

[`git worktree`](https://git-scm.com/docs/git-worktree) makes is possible to materialise multiple branches simultaneously
and work with them as usual in a single git repo:

* [Clone repo with master going to a nested directory](https://stackoverflow.com/a/72289797/758986):
  `mkdir ~/devel/ybd && cd ~/devel/ybd && git clone git@bitbucket.org:yellowbrickdata/ybd.git master  --separate-git-dir=.git`
* Checkout worktree that you need (repeat this as many times as you need): `git worktree add cloud_native`

Operations:

* **add** worktree: `git worktree add <path> <commit-ish>`

    If commit-ish is a branch name is has not yet been checked out locally, then git will create a tracking branch for you.

* **add** worktree: `git worktree add --track -b DS_CN_YBD-22481-GCP-1 objst origin/DS_CN_YBD-22481-GCP-1`

    Use this if you want to create a directory `objst` that has a branch which tracks remote branch.

* **list** worktrees: `git worktree list`
* **move** worktree from one directory to another: `git worktree move`
* **remove** worktree: `git worktree remove`
* **repair**



------------------------------------------------------------------------------------------------------------------------
## **Common git config**

* Prune deleted branches automatically: `git config --global fetch.prune true`
* Merge strategy: `git config pull.rebase true`



------------------------------------------------------------------------------------------------------------------------
## **Git Content Filter Driver**

There is a way to comply with official company-wide code-style **and** work (locally) with the code formatted **as you want**.

The idea is based upon the fact that it is only important to push my code in central repo in the official format, while
nobody prevents me from having and working with code that is formatted in my own way.

Luckily, git has native support for this - [Git Content Filter Driver](https://stackoverflow.com/questions/2316677/can-git-automatically-switch-between-spaces-and-tabs/2316728#2316728).

##### In `.git/config` add

```ini
[filter "clangformat"]
	smudge = .git/clang-formatter.sh .clang-format-my
	clean = .git/clang-formatter.sh .clang-format-company
```

* `smudge` --- the command that is called when you **pull** something (more precisely when git checks sources out onto your disk).
* `clean` --- the command that is called when you **push** something (more precisely when you add files in the git's staging area).
* `.clang-format-my` and `.clang-format-company` --- self-explanatory `clang-format` config files.
* `.git/clang-formatter.sh` is a small script that invokes `clang-format`. The main purpose of the script is to "prepare" value for
  `-style` option (via awk):
    ```bash linenums="1"
    #!/bin/bash
                              # (1)!
    clang-format-8 -style "{$(awk '{if ($0 !~ /#/) {if (NR>1) {printf ", "} printf $0; }}' $1)}" -assume-filename=main.cpp
    ```

    1.  the awk script below converts contents of `.clang-format-my` (or `.clang-format-company`) into inline format: `-style "{Option1: Value, ...}"`,
        because there is no way to specify filename with config to `clang-format`

* do not forget to make the script executable: `chmod +x .git/clang-formatter.sh`


##### In `.git/info/` add `attributes`:
```ini
*.c filter=clangformat
*.h filter=clangformat
*.cpp filter=clangformat
*.hpp filter=clangformat
```

##### More info and explanation...
...can be found [here](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes)




------------------------------------------------------------------------------------------------------------------------
## **Merge-related tricks**


### Ignore whitespaces

`-Xignore-all-space` or `-Xignore-space-change`.


### Checkout their/our/base versions of a conflicted file

```bash linenums="1"
$ git show :1:hello.rb > hello.common.rb
$ git show :2:hello.rb > hello.ours.rb
$ git show :3:hello.rb > hello.theirs.rb
```

Then you can change them and finally merge: `#!bash git merge-file -p hello.ours.rb hello.common.rb hello.theirs.rb > hello.rb`.

##### Same as above but in-place:

```bash linenums="1"
git checkout --ours result/ErrorRepMedium.h
```

NOTE:

* During rebase (`git rebase master`), `--ours` correspond to `master`.


### Diff of the pending merge

* To compare your result to what you had in your branch before the merge, in other words,
  to see what changes the merge is going to introduce: `git diff --ours`.
* If we want to see how the result of the merge differed from what was on their side, you can run `git diff --theirs`.


### Show our/base/their in conflict markers

```bash linenums="1"
$ git checkout --conflict=diff3 hello.rb

def hello
<<<<<<< ours
  puts 'hola world'
||||||| base
  puts 'hello world'
=======
  puts 'hello mundo'
>>>>>>> theirs
end

hello()
```


### Compare two diffs (range-diff)

After a cherry-pick or merge you can check difference of two diffs (1st diff corresponds to a commit
on the `master`, 2nd diff corresponds to the merge commit or cherry-picked commit):

```bash linenums="1"
git range-diff af05d1~1..af05d1 \  # Range1: denotes range (consisting of 1 commit) in our branch
               HEAD~1..HEAD        # Range2: commit on the master

# or (the same as above, but shorter):
git range-diff af05d1^! HEAD^!
```

`^!` notation represents single commit, but unlike just `HEAD` (which also represents a single
commit), it denotes a range.

Diff output is also interesting (read more about it [here](https://git-scm.com/book/en/v2/Git-Tools-Advanced-Merging)):

```diff
-   puts 'hola world'
 -  puts 'hello mundo'
++  puts 'hola mundo'
```

* This shows us that `hola world` was in our side but not in the working copy,
* that `hello mundo` was in their side but not in the working copy
* and finally that `hola mundo` was not in either side but is now in the working copy.

This can be useful to review before committing the resolution.

In other words:

* `' -'` or `' +'` represents what happened in respect to (from the point of view of) the 2nd range (removed / added lines in our branch).
* `'- '` or `'+ '` represents what happened in respect to (from the point of view of) the 1st range (removed / added lines in master branch - where we cherry-pick commit from).


In this mode, the diff of diffs will retain the original diff colors, and prefix the lines with `-/+` markers
that have their background red or green, to make it more obvious that they describe how the diff itself changed.

```diff
// The second '-' shows us that both lines below were removed (the first line was removed from master's commit, the second line was removed from the branch's commit).
// However, the first line was removed from **diff** (first '-' in the first line), and replaced with the second line (first '+' in the second line).
--    MediumHash hashTable = {distinctRows, minThreadSafeBuckets()}; // in master we removed this
+-    MediumHash& hashTable = *new MediumHash(distinctRows, minThreadSafeBuckets()); // but in the branch we removed this


// The second '+' shows us that both lines were added (the first line was added in the master's commit, the second line was added in the branch's commit).
// However, the first line was removed from **diff** (first '-' in the first line), and replaced with the second line (first '+' in the second line).
-+    MediumHash hashTable = {distinctRows, BuildPartitioner::minThreadSafeBuckets()}; // in master we added this
++    MediumHash& hashTable = *new MediumHash(distinctRows, BuildPartitioner::minThreadSafeBuckets()); // but in the branch we added this
```


Another example:
```
There was diff (second '-' and '+') in the branch's commit, but there is no diff between (1) master's
diff and (2) branch's diff (because we have space). Meaning that master has the same change.
 -    for (Row* row : packet) {
 +    for(Row* row: packet) {
```


The second `+/-` tells us what happened in one of the diffs (master's commit or branch's commit).

The first `+/-` is from diff of the two diffs. If there is space, it means there is no diff (between the two diffs).



------------------------------------------------------------------------------------------------------------------------
## **Submodules cheat-sheet**

[Official docs.](https://www.git-scm.com/book/en/v2/Git-Tools-Submodules)

Remember is that there are two states:

1. the state associated with git repo that is a submodule
    * and therefore you need to do all usual things with it, such as adding files, committing, pushing and so on...
2. and the state stored in the parent repo, that describes submodules (this is in addition to usual git repo state
   and operations, the parent repo also includes list of submodules, urls from which they have been clones,
   branches they track, SHA "pointers" to commits of the branches, all of which you have to commit/pull/push).

Another non-obvious thing is that there are two places where repo URLs are kept: `.gitmodules` and `<repo>/.git/config`.
If someone updates and URL of a repo, you get the changes in `.gitmodules` via git pull. But `<repo>/.git/config`
is not updated. This is why you need to use `git submodule sync --recursive` (after `git pull`).


* **Show statuses of submodules**: `git config status.submodulesummary 1`


* **Show diff of submodules**:
    * `git diff --submodule=diff`
    * `git config          diff.submodule diff`
    * `git config --global diff.submodule diff`


* **Checkout submodules** too (if you issue git checkout in the root repo, it will checkout submodules as well):
    * `git checkout --recurse-submodules`
    * `git config submodule.recurse true` (this will add `--recurse-submodules` to all git commands)
    * `git config --global submodule.recurse true`


* **Verify that submodules were pushed before super-repo push**:
    * `git config push.recurseSubmodules check`


* **Checkout all submodules to their correct versions**

    ```bash
    git submodule foreach --recursive git reset --hard
    git submodule update --init --recursive
    ```


* **Clone a repo with all submodules**: `git clone --recurse-submodules git@github.com:DimanNe/scripts.git`
    * or only selected submodules: (1)
      {.annotate}

        1. here:

            ```bash
            git clone git@github.com:DimanNe/scripts.git
            cd scripts
            git submodule init path/to/your/submodule
            git submodule update
            ```


* **Add a submodule in a parent repo**

    ```bash
    git submodule add -b branch_you_need git@github.com:DimanNe/result.git contrib/result/result
    # Below is not needed if you used -b above:
    # cd contrib/result/result && git checkout branch_you_need && cd ../../../
    # Tell git which branch to pull/track
    # git config -f .gitmodules submodule.contrib/result/result.branch branch_you_need
    # git add .gitmodules
    git commit -m "Add result as a submodule"
    ```


* **Remove a submodule from a repo** [Src](https://stackoverflow.com/questions/1260748/how-do-i-remove-a-submodule)

    ```bash
    git rm <path-to-submodule>
    ```

    This removes the filetree at `<path-to-submodule>`, and the submodule's entry in the `.gitmodules` file.

    Additionally, you might want to remove `.git` directory of the submodule from `modules/`:

    ```bash linenums="1"
    rm -rf .git/modules/<path-to-submodule>
    git config --remove-section submodule.<path-to-submodule>.
    ```


* **Pull submodules**:

    * **Update submodule from upstream**: `#!bash git submodule update --remote --merge contrib/result/result/`
      (Omit the last arg if you want to pull all submodules)

    * **Get the state expected/committed in super-repo**: `#!bash git pull && git submodule sync --recursive && git submodule update --init --recursive`


* **Push a submodule**: work with submodule as usual: make changes in submodule's files, add the changed files,
  commit and push them (note: you do not need to push explicitly each submodules if you set `git config submodule.recurse true`).

    Finally, fix/set the new submodule hash in the root repo:
    `git add contrib/result/result && git commit -m "Update contrib/result/result submodule" && git push`





------------------------------------------------------------------------------------------------------------------------
## **Default branch name**

```bash
git symbolic-ref --short refs/remotes/origin/HEAD
```



------------------------------------------------------------------------------------------------------------------------
## **git-svn: skip revisions**

Omit 1395620-1395673 revisions:
```bash linenums="1"
git svn fetch -r1395673
git prune
git svn rebase
```



------------------------------------------------------------------------------------------------------------------------
## **Add remote upstream**

```bash
git remote add upstream https://github.com/UPSTREAM-USER/ORIGINAL-PROJECT.git
```
