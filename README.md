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

  # Index/Staging Area<br/>
   Let's consider what happens when we use the git <br/>
add command to add a file, but did not yet create a new commit adding it.<br/>
A version control system needs to store such information somewhere. Git uses something <br/>
called the index for this; it is the staging area that stores information that will go into <br/>
the next commit. The git add <file> command stages the current contents (current version) of the file, <br/>
adding it to the index.<br/>
  
git add takes the content of the file from the working directory and puts it into the staging area.<br/>
  
git commit -a command (which is git commit --all), which will take all the changed tracked files,<br/>
add them to the staging area (as if with git add -u, at least in modern Git), and create a new <br/>
commit.The new files still need to be explicitly git add to be tracked, and to be included in the new commit.<br/>
   
  You create a new revision with the git commit command, which takes the <br/>
files as they are in the staging area and stores that snapshot permanently to <br/>
your local repository.<br/>
  
  To see what you've changed but not yet staged, type git diff with no other <br/>
arguments.<br/>
  
  To see what you've staged that will go into your next commit, use git diff <br/>
--staged (or git diff --cached). This command compares what is in your <br/>
staging area to the content of your last commit.<br/>
  
  You can use git diff HEAD to compare what is in your working directory with the <br/>
last commit (or arbitrary commit with git diff <commit>). <br/>

  # Push modes and their use<br/>
The behavior of git push, in the absence of the parameters specifying what to push, <br/>
and in the absence of the configured push refspec, is specified by the push mode. <br/>
  
  The simple push mode – the default<br/>
The default push mode in Git 2.0 and later is the so-called simple mode.<br/>
  With this mode, you always push the current local branch into the same named <br/>
branch in the remote repository. If you push into the same repository you fetch from <br/>
(the centralized workflow), it requires the upstream to be set for the current branch. <br/>
The upstream is named the same as the branch.<br/>
  This is the safest option; it is well-suited for beginners, which is why it is the default <br/>
mode. You can turn it on explicitly with git config push.default simple.<br/>
  
  The matching push mode for maintainers<br/>
  Before version 2.0 of Git, the default push mode was matching. This mode is most <br/>
useful for the maintainer (also known as the integration manager) in a blessed <br/>
repository workflow.<br/>
  With the matching mode, Git will push all the local branches that have their <br/>
equivalent with the same name in the remote repository. This means that only the <br/>
branches that are already published will be pushed to the remote repository. To <br/>
make a new branch public you need to push it explicitly the first time, for example:<br/>
git push origin maint-1.4<br/>
  To turn on the matching mode globally, you can run:<br/>
git config push.default matching<br/>
If you want to turn it on for a specific repository, you need to use a special refspec <br/>
composed of a sole colon. Assuming that the said repository is named origin and <br/>
that we want a not forced push, it can be done with:<br/>
git config remote.origin push :<br/>
You can,push matching branches using this refspec on the command line:<br/>
git push origin :<br/>
  
  The upstream push mode for the centralized workflow<br/>
  That is what the upstream push mode was created for:<br/>
git config push.default upstream<br/>
This mode makes Git push the current branch to the specific branch in the remote <br/>
repository—the branch whose changes are usually integrated into the current <br/>
branch. This branch in the remote repository is the upstream branch<br/>
(and can be referenced as @{upstream}). Turning this mode on makes it possible to simplify the <br/>
last command in both examples to the following:<br/>
git push<br/>
The information about the upstream is created either automatically <br/>
(while forking off the remote-tracking branch), or explicitly with the --track option. It is stored <br/>
in the configuration file and it can be edited with ordinary configuration tools. <br/>
Alternatively, it can be changed later with the following:<br/>
git branch --set-upstream-to=<branchname><br/>
  
  The current push mode for the blessed repository workflow<br/>
  In the blessed repository workflow, each developer has his or her own private and <br/>
public repository. In this model, one fetches from the blessed repository and pushes <br/>
to his or her own public repository.<br/>
In this workflow, you start working on a feature by creating a new topic branch for it:<br/>
git checkout -b fix-tty-bug origin/master<br/>
When the features are ready, you push it into your public repository, perhaps <br/>
rebasing it first to make it easier for the maintainer to merge it:<br/>
git push origin fix-tty-bug<br/>
  To configure Git so when on fix-tty-bug branch it is enough to just run git push, <br/>
you need to set up Git to use the current push mode, which can be done with the following:<br/>
git config push.default current<br/>
This mode will push the current branch to the branch with the same name at the receiving end.<br/>

  # Using git stash
 git stash command<br/>
 Stashing takes the dirty state of your working area—that is, your modified <br/>
tracked files in your worktree (though you can also stash untracked files with the <br/>
--include-untracked option), and the state of the staging area, then saves this <br/>
state, and resets both the working directory and the index to the last committed <br/>
version (to match the HEAD commit), effectively running git reset --hard HEAD. <br/>
You can then reapply the stashed changes at any time.<br/>
Stashes are saved on a stack: by default you apply the last stashed changes (stash@{0}),<br/>
though you can list stashed changes (with git stash list), and explicitly select any of the stashes.<br/>
 By default git stash pop will apply the last stashed changes, and delete the  <br/>
stash if applied successfully.<br/>
 You can use any of the older stashes by specifying the stash name as an argument. <br/>
For example, you can run git stash apply stash@{1} to apply it, and you can <br/>
drop it (remove it from the list of stashes) with git stash drop stash@{1}; the  <br/>
git stash pop command is just a shortcut for apply + drop.<br/>
 You can make git stash keep the state of the index, and reset the <br/>
working area to the staged state, with the --keep-index option.<br/>
 To stash away your changes, Git creates two automatic commits: one for the index <br/>
(staging area), and one for the working directory. With git stash --include-untracked,<br/>
 Git creates an additional third automatic commit for untracked files.<br/>
 The commit containing the work in progress in the working directory (the state of <br/>
files tracked from there) is the stash, and has the commit with the contents of the <br/>
staging area as its second parent. This commit is stored in a special ref: refs/stash. <br/>
Both WIP (stash) and index commits have the revision you were on when saving <br/>
changes as its first (and only for the index commit) parent.<br/>

  # Command aliases for Git<br/>
  It is very easy in theory to create an <br/>
alias. You simply need to create an alias.<command-name> configuration variable; <br/>
its value is the expansion of alias.<br/>
  One of the uses for aliases is defining short abbreviations for commonly used <br/>
commands and their arguments. Another is creating new commands. Here are a <br/>
couple of examples you might want to set up:<br/>
 git config --global alias.co checkout<br/>
 git config --global alias.ci commit<br/>
 git config --global alias.lg log --graph --oneline --decorate<br/>
  The preceding setup means that typing, for example, git ci would be the same as <br/>
typing git commit. Aliases take arguments just as the regular Git commands do. <br/>
Git does not provide any default aliases that are defining shortcuts for the common <br/>
operations.Arguments are split by space, the usual shell quoting and escaping is supported; in <br/>
particular, you can use a quote pair ("a b") or a backslash (a\ b) to include space in a single argument.<br/>
  You cannot have the alias with the same name as a Git command;<br/>
  You cannot use aliases to change the behavior of commands.<br/>
  The reasoning behind this restriction is that it could make existing scripts and hooks fail unexpectedly.<br/>
  Aliases that hide existing Git commands (with the same name as Git commands) are simply ignored.<br/>
  
  # Git Administration<br/>
  You can run auto gc manually with git gc --auto, or force garbage collection <br/>
with git gc. The git count-objects command (perhaps with the help of the -v parameter) <br/>
can be used to check whether there are signs that repack is needed. <br/>
You can even run individual steps of the garbage collection individually with <br/>
git repack, git pack-refs, git prune, and git prune-packed.<br/>
  By default, Git would try to reuse the results of the earlier packing to reduce CPU <br/>
time spent on the repacking, while still providing good disk space utilization. In <br/>
some cases, you would want to more aggressively optimize the size of repository <br/>
at the cost of it taking more time: this is possible with git gc --aggressive (or <br/>
with repacking the repository by hand with git repack, run with appropriate <br/>
parameters). It is recommended to do this after import from other version control <br/>
systems; the mechanism that Git uses for importing (fast-import stream) is optimized <br/>
for the speed of the operation, not for the final repository size.<br/>
  There are issues of maintenance not covered by git gc, because of their nature. One <br/>
of them is pruning (deleting) remote-tracking branches that got deleted in the remote <br/>
repository. This can be done with git fetch --prune or git remote prune, or <br/>
on a per-branch basis with git branch --delete --remotes <remote-tracking branch>. 
This action is left to the user, and not run by git gc, because Git simply <br/>
cannot know whether you have based your own work on the remote-tracking branch that is to be pruned.<br/>
  Often, the simplest way to find and recover lost commits is to use the git reflog tool.<br/>
For each branch, and separately for HEAD, Git silently records (logs) where the <br/>
tip of the branch was in your local repository, at what time it was there, and how it <br/>
got there.This record is called the reflog. Each time you commit or rewind a branch, <br/>
the reflog for the branch and for the HEAD is updated. Each time you change the <br/>
branches, the HEAD reflog is updated, and so on.<br/>
  You can see where the tip of branch has been at any time by running git reflog or <br/>
git reflog <branch>. You can run git log -g instead, where -g is a short way <br/>
of saying --walk-reflog; this gives you a normal configurable log output. There is <br/>
also --grep-reflog=<pattern> to search the reflog.<br/>
  git reflog<br/>
  The main purpose of git fsck is to check for repository corruption. <br/>
  
  Server-side hooks<br/>
Hooks that are invoked on the server can be used for server administration; among <br/>
others, these hooks can control the access to the remote repository by performing <br/>
the authorization step, and can ensure that the commits entering the repository meet <br/>
certain minimal criteria.<br/>
  
  Reducing the size taken by repositories<br/>
  If you are hosting many forks (clones) of the same repository, you might want to <br/>
reduce disk usage by somehow sharing common objects. One solution is to use <br/>
alternates (for example, with git clone --reference) while creating a fork. In this <br/>
case, the derived repository would look to its parent object storage if the object is not <br/>
found on its own.<br/>
  There are, two problems with this approach. First is that you need to <br/>
ensure that the object the borrowing repository relies on does not vanish from the <br/>
repository set as the alternate object storage (the repository you borrow from). This <br/>
can be done, for example, by linking the borrowing repository refs in the repository <br/>
lending the objects, for example, in the refs/borrowed/ namespace. Second is that <br/>
the objects entering the borrowing repository are not automatically de-duplicated: <br/>
you need to run git repack -a -d -l, which internally passes the --local option to git pack-objects.<br/>
An alternate solution would be to keep every fork together in a single repository, <br/>
and use gitnamespaces to manage separate views into the DAG of revisions, one <br/>
for each fork. With plain Git, this solution means that the repository is addressed by <br/>
the URL of the common object storage and the namespace to select a particular fork.<br/> 
Usually, this is managed by a server configuration or by a repository management <br/>
tool; such mechanism translates the address of the repository into a common <br/>
repository and the namespace. The git-http-backend manpage includes an <br/>
example configuration to serve multiple repositories from different namespaces in <br/>
a single repository. Gitolite also has some support for namespaces in the form of <br/>
logical and backing repositories and option namespace.pattern, though not every <br/>
feature works for logical repositories.<br/>
Storing multiple repositories as the namespace of a single repository avoids storing <br/>
duplicated copies of the same objects. It automatically prevents duplication between <br/>
new objects without the need for ongoing maintenance, as opposed to the alternates <br/>
solution. On the other hand, the security is weaker; you need to treat anyone with <br/>
access to the single namespace, which is within the repository as if he or she had an <br/>
access to all the other namespaces, though this might not be a problem for your case.<br/>
  
  Solving the large nonresumable initial clone problem<br/>
  One solution is to create, with the git bundle command, a static file that can be <br/>
used for the initial clone, or as reference repository for the initial clone (the latter can <br/>
be done with the git clone --reference=<bundle> --dissociate command if <br/>
you have Git 2.3 or a newer looks unnecessary). This bundle file can be distributed <br/>
using any transport; in particular, one that can be resumed if interrupted, be it HTTP, <br/>
FTP, rsync, or BitTorrent. The conventions people use, besides explaining how to <br/>
get such a bundle in the developer documentation, is to use the same URL as for the <br/>
repository, but with the .bundle extension (instead of an empty extension or a .git suffix).<br/>
There are also more esoteric approaches like a step by step deepening of a shallow <br/>
clone (or perhaps, just using a shallow clone with git clone --depth is all that's <br/>
needed), or using approaches such as GitTorrent.<br/>
