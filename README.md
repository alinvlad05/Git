# Git
# Merging Changes Together<br/>
  <br/>
  Merging branches<br/>
The merge operation joins two (or more) separate branches together, including all the changes since<br/>
the point of divergence into the current branch. You do this with the git merge command:<br/>
git checkout master<br/>
git merge bugfix123<br/>
We first switched to a branch we want to merge into (master in this example), <br/>
and then provided the branch to be merged (here, bugfix123).<br/>
  
  1.No divergence – fast-forward and up-to-date cases<br/>
Git would, by default, simply move the branch pointer of the current branch forward.<br/>
We can force creating a merge commit even in such cases with the git merge --no-ff command.<br/>
The default is --ff; to fail instead of creating a merge commit you can use --ff-only (ensuring fast-forward only).<br/>
  <br/>
  2.Creating a merge commit<br/>
  Because the top commit on the branch you are on (and are merging into) is not a <br/>
direct ancestor or a direct descendant of the branch you are merging in, Git has to <br/>
do more work than just moving the branch pointer. In this case, Git does a merge of <br/>
changes since the divergence, and stores it as a merge commit on the current branch. <br/>
This commit has two parents denoting that it was created based on more than one <br/>
commit (more than one branch): the first parent is the previous tip of the current <br/>
branch and the second parent is the tip of branch you are merging in.<br/>
  You can either ask Git to not autocommit a merge with git merge --no-commit <br/>
to examine it first, or you can examine the merge commit and then use the <br/>
git commit --amend command if it is incorrect.<br/>
  <br/>
Cherry-pick – creating a copy of a changeset  <br/>
  You can create a copy of a commit (or a series of commits) with the cherry-pick <br/>
command. Given a series of commits (usually, just a single commit), it applies the <br/>
changes each one introduces, recording a new commit for each.<br/>
  By default, Git does not save information about where the cherry-picked <br/>
commit came from. You can append this information to an original commit message, <br/>
as a (cherry-picked from the commit <sha-1>) line with git cherry-pick -x <commit>. <br/>
This is only done for cherry-picks without conflicts. <br/>
  
  Revert – undoing an effect of a commit<br/>
  This "undoing of a commit" can be done by creating a commit with a reversal of changes, <br/>
something like cherry-pick but applying the reverse of changes. This is done with the revert command.<br/>
  If you want to revert all the changes made to the whole working area, <br/>
you can use git reset (in particular, the --hard option).<br/>
  If you want to revert changes made to a single file, use git checkout <file>.<br/>
  The git revert command records a new commit to reverse the effect of the earlier commit (often, a faulty one).<br/>
  <br/>
  What happens if you want to cherry-pick or revert a merge commit? <br/>
  Such commits have more than one parent, thus they have more than one change associated with them.<br/>
  You have to tell Git which change you want to pick up (in the case of cherry-pick), <br/>
 or back out (in the case of revert) with the -m <parent number> option.<br/>
<br/>
  Rebasing a branch<br/>
  Besides merging, Git supports additional way to integrate changes from one branch <br/>
into another: namely the rebase operation.<br/>
Like a merge, it deals with the changes since the point of divergence (at least, by default). <br/>
  But while a merge creates a new commit by joining two branches, rebase <br/>
takes the new commits from one branch (takes the commits since the divergence) <br/>
and reapplies them on top of the other branch.<br/>
  With merge, you first switched to the branch to be merged and then used the merge <br/>
command to select a branch to merge in. With rebase, it is a bit different. First you <br/>
select a branch to rebase (changes to reapply) and then use the rebase command to select where to put it.<br/>
git checkout i18n<br/>
git rebase master<br/>
  Or, you can use git rebase master i18n as a shortcut. The rebase operation takes the master..i18n range of revisions, <br/>
replays it on top of master, and finally points i18n to the replayed commits.<br/>
  Old versions of commits doesn't vanish, at least not immediately. They would be accessible via reflog (and ORIG_HEAD) for a period.<br/>
  
  Advanced rebasing techniques<br/>
You can also have your rebase operation replay on something other than the target branch of the rebase with --onto <newbase>, <br/>
separating selected range of revisions to replay from the new base to replay onto.<br/>
 Or perhaps, you want to rebase only a part of the branch. You can do this with git rebase --interactive, <br/>
 but you can also use git rebase --onto <new base> <starting point> <branch>.<br/>
  You can even choose to rebase the whole branch (usually, an orphan branch) with the --root option. <br/>
  In this case, you would replay the whole branch and not just a selected subset of it.<br/>
  <br/>
  The three-way merge<br/>
  Git uses the three-way merge algorithm to come up with the result of the merge, <br/>
comparing the common ancestors (base), side merged in (theirs), and side merged into (ours).<br/>
  <br/>
  
| ancestor (base)  | HEAD (ours) | branch (theirs)  | result |  
| ------------- | ------------- | ------------- | ------------- | 
| A  | A  | A  | A  |
| A  | A  | B  | B  |
| A  | B  | A  | B  |
| A  | B  | B  | B  |
| A  | B  | C  | merge conflict  |
  <br/>
  The rules for the trivial tree-level three-way merges are:<br/>
• If only one side changes a file, take the changed version<br/>
• If both the sides have the same changes, take the changed version<br/>
• If one side has a different change from the other, there is merge conflict at the contents level<br/>
  <br/>
  Conflict markers<br/>
<br/>  <<<<<<< HEAD:src/rand.c<br/>
<br/>fprintf(stderr, "Usage: %s <number> [<count>]\n", argv[0]);<br/>
<br/>=======<br/>
<br/>fprintf(stderr, _("Usage: %s <number> [<count>\n"), argv[0]);<br/>
<br/>>>>>>>> i18n:src/rand.c<br/>
  This means that the ours version on the current branch (HEAD) in the src/rand.c file <br/>
is there at the top of this block between the <<<<<<< and ======= markers, while the <br/>
theirs version on the i18n branch being merged (also from src/rand.c) is there at <br/>
the bottom part between the ======= and >>>>>>> markers.<br/>
  You need to replace this whole block by the resolution of the merge.<br/>
  <br/>
  Three stages in the index<br/>
  It turns out that it is another use for the staging area of the commit (a merge commit in this case), <br/>
which is also known as the index. In the case of conflicts, Git stores all of conflicted files versions <br/>
in the index under stages; each stage has a number associated with it. <br/>
Stage 1 is the common ancestor (base), stage 2 is the merged into version from HEAD, that is, the current branch (ours), <br/>
and stage 3 is from MERGE_HEAD, the version you're merging in (theirs).<br/>
You can see these stages for the unmerged files with the low level (plumbing) <br/>
command git ls-files --unmerged (or for all the files with git ls-files --stage).<br/>
  You can refer to each version with the :<stage number>:<pathname> specifier. If you want to view a common ancestor version of src/rand.c,<br/>
 you can use the following:<br/>
git show :1:src/rand.c<br/>
If there is no conflict, the file is in stage 0 of the index.<br/>

 # An interactive rebase<br/>
 Sometimes, you might want to edit commit deeper in history, or reorganize commits <br/>
into a logical sequence of steps. One of the built-in tools that you can use in Git for <br/>
this purpose is git rebase --interactive.<br/>
   Commands:<br/>
  p, pick = use commit<br/>
  r, reword = use commit, but edit the commit message<br/>
  e, edit = use commit, but stop for amending<br/>
  s, squash = use commit, but meld into previous commit<br/>
  f, fixup = like "squash", but discard this commit's log message<br/>
  x, exec = run command (the rest of the line) using shell<br/>
  d, drop = remove commit<br/>
  You can, make the interactive rebase test each commit with the --exec option, for example:<br/>
git rebase --interactive --exec "make test"<br/>
