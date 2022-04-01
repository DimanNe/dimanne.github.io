title: git

# **git**

## **grep history**
Look for the given regex: `#!bash git log -G "#include <sys/ustat.h>"`

Look at the number of occurrences: `#!bash git log -S "#include <sys/ustat.h>"`



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



## **Submodules cheat-sheet**

### Intro

[Official docs.](https://www.git-scm.com/book/en/v2/Git-Tools-Submodules)

* Better git status output: `git config status.submodulesummary 1`
* Better git diff: `git diff --submodule=diff`
* Consistent git checkout (if you issue git checkout in the root repo, it will checkout submodules as well):
  `git config submodule.recurse true`. More generally, this will add `--recurse-submodules` to all git commands.
* Do not push root repo without ensuring submodules are pushed. The following setting prevents git from pushing
  a new submodule commit pointer in the root repo, unless the commit exists in the remote of the submodule:
  `git config push.recurseSubmodules check`.

Another useful concept you need to remember is that there are two states:

* state associated with git repo that is a submodule (and therefore you need to do all usual things with it,
  such as adding files, committing, pushing and so on...);
* and state stored in the parent repo, that describes submodules (in addition to usual git repo state and operations,
  the parent repo also includes list of submodules, urls from which they have been clones, branches they track, SHA
  "pointers" to commits of the branches, all of which you have to commit/pull/push).

### Clone a repo with submodules

```bash
git clone --recurse-submodules git@github.com:DimanNe/scripts.git
```

### Add a submodule in a parent repo

```bash linenums="1"
mkdir -p contrib/result
git submodule add git@github.com:DimanNe/result.git contrib/result/result
cd contrib/result/result && git checkout branch_you_need && cd ../../../
git config -f .gitmodules submodule.contrib/result/result.branch branch_you_need # Tell git which branch to pull/track
git add .gitmodules
git commit -m "Add result as a submodule"
```

### Remove a submodule from a repo
[Src](https://stackoverflow.com/questions/1260748/how-do-i-remove-a-submodule)
```
git rm <path-to-submodule>
```
This removes the filetree at `<path-to-submodule>`, and the submodule's entry in the `.gitmodules` file
I.e. all traces of the submodule in your repository proper are removed.

Additionally, you might want to remove `.git` directory of the submodule from `modules/`:

```bash linenums="1"
rm -rf .git/modules/<path-to-submodule>
git config --remove-section submodule.<path-to-submodule>.
```


### Pull a submodule (Update specific submodule from its remote)

```
git submodule update --remote --merge contrib/result/result/ # Omit the last arg if you want to pull all submodules
git pull && git submodule sync --recursive && git submodule update --init --recursive --remote
```

### Push a submodule

Work with submodule as usual: make changes in submodule's files, add the changed files, commit and push them (note:
you do not need to push explicitly each submodules if you set git config submodule.recurse true).

Finally, fix/set the new submodule hash in the root repo:
```bash linenums="1"
git add contrib/result/result && git commit -m "Update contrib/result/result submodule" && git push
```


## **Default branch name**

```bash
git symbolic-ref --short refs/remotes/origin/HEAD
```



## **git-svn: skip revisions**

Omit 1395620-1395673 revisions:
```bash linenums="1"
git svn fetch -r1395673
git prune
git svn rebase
```


## **Add remote upstream**

```bash
git remote add upstream https://github.com/UPSTREAM-USER/ORIGINAL-PROJECT.git
```

