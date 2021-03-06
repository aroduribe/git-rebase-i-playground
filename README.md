
# Learn "git rebase -i" with this lab/playground

Want to learn how to **properly** use `git rebase -i` (also known as an interactive rebase) without putting your production
repo at risk?  You're in the right place!  The script included in this repo will create
a git repo (with 2 cloned copies) that has synthetic checkins and branches.  It makes
for an ideal testing ground to experiment with `git rebase -i` without going insane.


## Getting Started

- Clone this repo: `git clone git@github.com:dmuth/git-rebase-i-playground.git`
- `cd git-rebase-i-playground`
- `./init.sh`

This will create the following repo and directories, each with about a dozen commits:
- `dev-alice` - A clone of the repo made by "Alice", with two branches: `branch1` and `branch2`.  `branch1` is a branch from master while `branch2` is based on `branch1`.  That is by design (and is based on a True Story, heh).
- `dev-bob`- A clone of the repo made by Alice's co-worker "Bob", only containing `master`.
- `repo.git` - A "bare" clone of the repo.  Note that if you `cd` to this directory, commands like `git log --pretty=oneline` will work just fine. That is useful for debugging.

NOTE: Running `init.sh` will remove those directories if they already exist.  This is so that you can have a "clean slate" every time you run the script.

The tree looks like this:

<img src="./img/00-START.png" width="600" />



## Why would you want to use `git rebase -i`?

Running `git rebase -i commit_id` will bring up recent commits in the system editor and allow
you to reorder the commits, edit the commit message, or even delete commits.

Why would you want to delete a commit?  Probably the most common reason is that you made many commits
to a feature branch that you would like to squash into a single commit. 
Or perhaps you made a feature branch based on another branch, and want to deploy the changes 
from that feature branch but not the branch it's based on. (Happened to me once!)


### Any caveats?

Anytime you rebase, you are *rewriting history*.  If you changes have already been pushed, you will
need to push them again, and history will be overwritten.  If other users have already checked out
the branch in question, this will cause a merge on their end, which will (eventually) wind up on 
the server's repo, and in your repo.  It will make the commit history just slightly uglier.


## Exercises

Once that you have the repos set up, here are some sample exercises to try (answers below):

- Switch the order of commits `04-fourth` and `05-fifth`
- Merge the changes of `branch2` into `master` but NOT the changes of `branch1`
- Squash the two commits in `branch1` then push to `origin`
- Switch the order of commits `04-fourth and 05-fifth`, push to `origin`
- Switch the order of commits `01-first-will-conflict` and `02-second-will-conflict`, THEN push to `origin`
- Run `git rebase -i` and delete commit `04-fourth`.  Then recover it.
- Switch the order of commits `01-first-will-conflict` and `02-second-will-conflict`


## Hints

Here are some hints to lead you in the right direction but without fully giving away the solution

- Switch the order of commits `04-fourth` and `05-fifth`
   - *Make sure you are going far enough back in the revision history...*
- Merge the changes of `branch2` into `master` but NOT the changes of `branch1`
   - *You'll need to remove some commits...*
- Squash the two commits in `branch1` then push to `origin`
   - *You'll need to overwrite what's already there...*
- Switch the order of commits `04-fourth and 05-fifth`, push to `origin`
   - *There's more than one way to do this*
- Switch the order of commits `01-first-will-conflict` and `02-second-will-conflict`, THEN push to `origin`
   - *You'll need to handle a merge conflict AND overwrite history in the origin...*
- Run `git rebase -i` and delete commit `04-fourth`.  Then recover it.
   - *The commit wasn't technically deleted...*
- Switch the order of commits `01-first-will-conflict` and `02-second-will-conflict`
   - *You're going to have to resolve that merge conflict...*


## Troubleshooting

If things go wrong, here are some suggestions:

- First, the nuclear option: `./init.sh` - This will reset your Git repo to the initial state.  Useful for if you've gone down a hole and you just want to start over.
- Running `git status` at any time will not harm you, and will provide you with some useful info and what you need to do next.
- `git rebase --abort` will back out of the current rebase.
- `git log --pretty=oneline --graph` - Will print a listing of commits in the current branch.
- `git log --pretty=oneline --graph --all` - Will print a listing of commits across all branches, and show which branches are based on what commits.
- The `tig` tool available at https://github.com/jonas/tig is super useful for browsing the commit history of a branch


## Solutions

Solutions to the above exercises, all done in the `dev-alice/` directory:

- Switch the order of commits `04-fourth` and `05-fifth`
   - Start with `git rebase -i HEAD~5`, switch the lines with those two commits, then save the file. You're done!
   - Graphs:
      - <a href="img/01-Switch 04 and 05.png">After switching 04 and 05</a>
      - <a href="img/02-Switch 04 and 05 Merged.png">After merging in branch2</a>

- Merge the changes of `branch2` into `master` but NOT the changes of `branch1`
   - Start in `branch2` with `git checkout branch2`
   - Now do `git rebase -i HEAD~8`, remove the two commits from `branch1`, save the file
   - `git checkout master`
   - `git merge branch2`
   - That's it, you're done!  Verify with `git log --pretty=oneline`.
   - Graphs:
      - <a href="img/05-Merge just branch2.png">During the middle of rebasing</a>
      - <a href="img/06-Merge just branch2 Merged.png">After merging just branch2</a>

- Squash the two commits in `branch1` then push to `origin`
   - Check out the branch: `git checkout branch1`
   - Rebase the branch and squash the final commit into the previous one, so `branch1` only has one commit.
   - Push with `git push --force-with-lease`
   - Verify by changing into `../repo.git` and running `git log --pretty=oneline branch1`
   - Graphs:
      - <a href="img/07-Squash Branch1 Commits.png">Branch1 after both commits have been squashed</a>

- Switch the order of commits `04-fourth and 05-fifth`, push to `origin`
   - Start with `git rebase -i HEAD~5`, switch the lines with those two commits, then save the file.
   - There are two ways to do this:
      - `git pull && git push`
         - This is the preferred way, and will cause your (re-written) history to be merged in with what's in `origin`.
      - `git push --force-with-lease`
         - This will overwrite what's in `origin` and is only recommended if you are in a branch that only you work on.
         - The difference between `--force-with-lease` versus `--force` is that the latter will overwrite the remote with your changes.  `--force-with-lease` requires that you have the most recent commit (the HEAD) from the remote, to ensure that you don't overwrite changes which were checked in since your last `pull`.  This makes it safer, or at least less unsafe.
   - Verify by changing into `../repo.git` and running `git log --pretty=oneline`
   - Graphs:
      - <a href="img/08-Delete 04.png">After 04 has been deleted (removed from master)</a>
      - <a href="img/09-Delete 04 and Merged It Back.png">After 04 has been merged back in to master</a>

- Switch the order of commits `01-first-will-conflict` and `02-second-will-conflict`, THEN push to `origin`
   - Start with `git rebase -i HEAD~5`, switch the lines with those two commits, then save the file.
   - Edit `file.txt` and resolve the conflicts
   - `git add file.txt`
      - Note that `git log` will show a VERY incomplete history at this point. That's fine--you traveled back in time.
   - `git commit`
   - Running `git status` tells you that you're not done yet, and tells you what to do next:
   - `git rebase --continue`
   - Uh oh, we have another conflict because the next commit also changes `file.txt`.
   - Edit `file.txt` and resolve the conflicts
   - `git add file.txt`
   - `git commit`
   - `git rebase --continue`
   - At this point, you've now merged your changes into master, let's push them with `git push origin`.
   - That's it, you're done!
   - Verify by changing into `../repo.git` and running `git log --pretty=oneline`

- Run `git rebase -i` and delete commit `04-fourth`.  Then recover it.
   - Use `git reflog` to find the commit you removed from `master`.
   - `git show COMMIT_ID` can be used to confirm it's the commit you want.
   - `git merge COMMIT_ID` will merge that commit back into `master`
   - *Extra Credit*: Use `git rebase -i HEAD~5` to move the commit back to its original location
   - There are a few other ways as well.  Feel free to play around in Git, that's what this repo is for!
   
- Switch the order of commits `01-first-will-conflict` and `02-second-will-conflict`
   - Start with `git rebase -i HEAD~5`, switch the lines with those two commits, then save the file.
   - Edit `file.txt` and resolve the conflicts
   - `git add file.txt`
      - Note that `git log` will show a VERY incomplete history at this point. That's fine--you traveled back in time.
   - `git commit`
   - Running `git status` tells you that you're not done yet, and tells you what to do next:
   - `git rebase --continue`
   - Uh oh, we have another conflict because the next commit also changes `file.txt`.
   - Edit `file.txt` and resolve the conflicts
   - `git add file.txt`
   - `git commit`
   - `git rebase --continue`
   - No more conflicts, so that's it, you're done! Verify with `git log`.
   - Graphs:
      - <a href="img/10-Switch 01 and 02.png">Switched 01 and 02</a>
      - <a href="img/11-Switch 01 and 02 Merged.png">After merging in branch2</a>


If the commit history for any of the above look "weird" when you're done, that's because you're rewriting history, so yeah.  The tool <a href="https://github.com/jonas/tig">tig</a> can make the history a little
clearer to read.


## FAQ

### Q: What's the story with Alice and Bob?

A: Alice and Bob are used as placeholder names in cryptology, science, and engineering literature: https://en.wikipedia.org/wiki/Alice_and_Bob  I find using the names to be useful because then I don't have to focus on the underlying details _quite_ as much.


## Additional Resources

- https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase
- https://thoughtbot.com/blog/git-interactive-rebase-squash-amend-rewriting-history
- https://git-scm.com/docs/git-rebase
- https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History
- https://www.dmuth.org/howto-safely-use-git-rebase-i/ - My blog post on this topic
- http://gitready.com/advanced/2009/01/17/restoring-lost-commits.html
- http://effectif.com/git/recovering-lost-git-commits
- http://gitready.com/intermediate/2009/02/09/reflog-your-safety-net.html


## TODO

- I want to make some simple/slides graphs that show the state of the Git tree for some of the exercises


## Contact

My email is doug.muth@gmail.com.  I am also <a href="http://twitter.com/dmuth">@dmuth on Twitter</a> 
and <a href="http://facebook.com/dmuth">Facebook</a>!  Additional ways to find me <a href="http://www.dmuth.org/contact">are on my website</a>.

