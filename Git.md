---
title: Git
lastmod: 2024-03-04T17:41:26-06:00
---
# Git
## Submodules
Pull submodules after clone
```bash
git submodule update --init
```
## Command Tricks
If you need to trigger a new build on your CI/CD, you can create an empty commit.
```bash
git commit --allow-empty
```
If you screw up your local copy
```bash
git reset --hard <remote branch name>
git reset --hard origin/develop
```
### Interactive Rebase
Don't waste time mucking with `git ammend`; it's messy and complicated.
TODO: more info here
If you forgot something and you already pushed up your changes, you can still fix your commit. You need to instead run your rebase on the remote branch and do a force push. That means first **make your fix and push it**. If you try to rebase with local commits, it will screw everything up because you're not in sync and you'll create weird merge commits. Then after your fix, just run rebase normally but on the remote branch.
```bash
git commit -m "fix I forgot to add before pushing"
git push
git rebase -i origin/branch_name~2 branch_name
git push origin +branch_name
```
## Resolving Issues
### Fetch Crashing
If on `git fetch` you encounter an error related to "early EOF fatal," you can increase the amount of RAM git has access to by adding this to your `$HOME/.gitconfig`.
```
[core] 
	packedGitLimit = 512m 
	packedGitWindowSize = 512m 
[pack] 
	deltaCacheSize = 2047m 
	packSizeLimit = 2047m 
	windowMemory = 2047m
```
Another possibility is the "unable to update local ref" error. A simple fix is to try to prune.
```bash
git gc --prune=now
```
### Trashing Broken Branch
In the event of a branch that can't easily be recoverable, you can simply rename the bad branch, create a new one, then apply all changes to the new branch as unstaged changes. Then you can clean up the code and make a new commit. This unfortunately destroys the git history.
```bash
# rename locally
git branch -m feature_name feature_name_old

# delete original on remote
git push origin --delete feature_name

# unset upstream
git branch --unset-upstream feature_name_old

# push new branch name
git push origin feature_name_old

# set new upstream
git push origin -u feature_name_old

# checkout branch you want to branch from
git checkout develop

# might as well grab latest
git fetch
git pull

# recreate original branch name
git checkout -b feature_name

# copy over all changes as unstaged
git merge --no-commit --no-ff feature_name_old
```
## Obsidian Setup
```bash
git config --global --add safe.directory C:/Users/rickd/Source/Repos/Obsidian
```
Windows-style backslashes will not work. If you run the above command by pasting the windows directory, you will need to edit the global `.gitconfig` file. It is in the user's home directory. You can quickly open it with `git config --global -e` to fix the slashes.
## Force GPG Login
```bash
echo "test" | gpg --clearsign
```
## Gitflow
To review:
https://jeffkreeftmeijer.com/git-flow/
https://nvie.com/posts/a-successful-git-branching-model/
## Common Commands
| Alias                | Command                                                                                                                                                                                  |
| :------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| g                    | git                                                                                                                                                                                      |
| ga                   | git add                                                                                                                                                                                  |
| gaa                  | git add --all                                                                                                                                                                            |
| gapa                 | git add --patch                                                                                                                                                                          |
| gau                  | git add --update                                                                                                                                                                         |
| gav                  | git add --verbose                                                                                                                                                                        |
| gap                  | git apply                                                                                                                                                                                |
| gapt                 | git apply --3way                                                                                                                                                                         |
| gb                   | git branch                                                                                                                                                                               |
| gba                  | git branch --all                                                                                                                                                                         |
| gbd                  | git branch --delete                                                                                                                                                                      |
| gbda                 | git branch --no-color --merged \| grep -vE "^([+*]\|\s*(<span>$</span>(git_main_branch)\|<span>$</span>(git_develop_branch))\s*<span>$</span>)" \| xargs git branch --delete 2>/dev/null |
| gbD                  | git branch --delete --force                                                                                                                                                              |
| gbg                  | git branch -vv | grep ": gone\]"                                                                                                                                                         |
| gbgd                 | local res=$(git branch -vv | grep ": gone\]" | awk '{print $1}') && [[ $res ]] && echo $res | xargs git branch -d                                                                        |
| gbgD                 | local res=$(git branch -vv | grep ": gone\]" | awk '{print $1}') && [[ $res ]] && echo $res | xargs git branch -D                                                                        |
| gbl                  | git blame -b -w                                                                                                                                                                          |
| gbnm                 | git branch --no-merged                                                                                                                                                                   |
| gbr                  | git branch --remote                                                                                                                                                                      |
| gbs                  | git bisect                                                                                                                                                                               |
| gbsb                 | git bisect bad                                                                                                                                                                           |
| gbsg                 | git bisect good                                                                                                                                                                          |
| gbsr                 | git bisect reset                                                                                                                                                                         |
| gbss                 | git bisect start                                                                                                                                                                         |
| gc                   | git commit --verbose                                                                                                                                                                     |
| gc!                  | git commit --verbose --amend                                                                                                                                                             |
| gcn!                 | git commit --verbose --no-edit --amend                                                                                                                                                   |
| gca                  | git commit --verbose --all                                                                                                                                                               |
| gca!                 | git commit --verbose --all --amend                                                                                                                                                       |
| gcan!                | git commit --verbose --all --no-edit --amend                                                                                                                                             |
| gcans!               | git commit --verbose --all --signoff --no-edit --amend                                                                                                                                   |
| gcam                 | git commit --all --message                                                                                                                                                               |
| gcas                 | git commit --all --signoff                                                                                                                                                               |
| gcasm                | git commit --all --signoff --message                                                                                                                                                     |
| gcsm                 | git commit --signoff --message                                                                                                                                                           |
| gcb                  | git checkout -b                                                                                                                                                                          |
| gcf                  | git config --list                                                                                                                                                                        |
| gcl                  | git clone --recurse-submodules                                                                                                                                                           |
| gccd                 | git clone --recurse-submodules "\$@" && cd "\$(basename \$\_ .git)"                                                                                                                      |
| gclean               | git clean --interactive -d                                                                                                                                                               |
| gpristine            | git reset --hard && git clean -dffx                                                                                                                                                      |
| gcm                  | git checkout $(git_main_branch)                                                                                                                                                          |
| gcd                  | git checkout $(git_develop_branch)                                                                                                                                                       |
| gcmsg                | git commit --message                                                                                                                                                                     |
| gco                  | git checkout                                                                                                                                                                             |
| gcor                 | git checkout --recurse-submodules                                                                                                                                                        |
| gcount               | git shortlog --summary -n                                                                                                                                                                |
| gcp                  | git cherry-pick                                                                                                                                                                          |
| gcpa                 | git cherry-pick --abort                                                                                                                                                                  |
| gcpc                 | git cherry-pick --continue                                                                                                                                                               |
| gcs                  | git commit -S                                                                                                                                                                            |
| gcss                 | git commit -S -s                                                                                                                                                                         |
| gcssm                | git commit -S -s -m                                                                                                                                                                      |
| gd                   | git diff                                                                                                                                                                                 |
| gdca                 | git diff --cached                                                                                                                                                                        |
| gdcw                 | git diff --cached --word-diff                                                                                                                                                            |
| gdct                 | git describe --tags $(git rev-list --tags --max-count=1)                                                                                                                                 |
| gds                  | git diff --staged                                                                                                                                                                        |
| gdt                  | git diff-tree --no-commit-id --name-only -r                                                                                                                                              |
| gdnolock             | git diff $@ ":(exclude)package-lock.json" ":(exclude)\*.lock"                                                                                                                            |
| gdup                 | git diff @{upstream}                                                                                                                                                                     |
| gdv                  | git diff -w $@ \| view -                                                                                                                                                                 |
| gdw                  | git diff --word-diff                                                                                                                                                                     |
| gf                   | git fetch                                                                                                                                                                                |
| gfa                  | git fetch --all --prune                                                                                                                                                                  |
| gfg                  | git ls-files \| grep                                                                                                                                                                     |
| gfo                  | git fetch origin                                                                                                                                                                         |
| gg                   | git gui citool                                                                                                                                                                           |
| gga                  | git gui citool --amend                                                                                                                                                                   |
| ggf                  | git push --force origin $(current_branch)                                                                                                                                                |
| ggfl                 | git push --force-with-lease origin $(current_branch)                                                                                                                                     |
| ggl                  | git pull origin $(current_branch)                                                                                                                                                        |
| ggp                  | git push origin $(current_branch)                                                                                                                                                        |
| ggpnp                | ggl && ggp                                                                                                                                                                               |
| ggpull               | git pull origin "$(git_current_branch)"                                                                                                                                                  |
| ggpur                | ggu                                                                                                                                                                                      |
| ggpush               | git push origin "$(git_current_branch)"                                                                                                                                                  |
| ggsup                | git branch --set-upstream-to=origin/$(git_current_branch)                                                                                                                                |
| ggu                  | git pull --rebase origin $(current_branch)                                                                                                                                               |
| gpsup                | git push --set-upstream origin $(git_current_branch)                                                                                                                                     |
| gpsupf               | git push --set-upstream origin $(git_current_branch) --force-with-lease --force-if-includes (git version >= 2.30)                                                                        |
| gpsupf               | git push --set-upstream origin $(git_current_branch) --force-with-lease (git version < 2.30)                                                                                             |
| ghh                  | git help                                                                                                                                                                                 |
| gignore              | git update-index --assume-unchanged                                                                                                                                                      |
| gignored             | git ls-files -v \| grep "^\[[:lower:]\]"                                                                                                                                                 |
| git-svn-dcommit-push | git svn dcommit && git push github $(git_main_branch):svntrunk                                                                                                                           |
| gk                   | gitk --all --branches &!                                                                                                                                                                 |
| gke                  | gitk --all $(git log --walk-reflogs --pretty=%h) &!                                                                                                                                      |
| gl                   | git pull                                                                                                                                                                                 |
| glg                  | git log --stat                                                                                                                                                                           |
| glgp                 | git log --stat --patch                                                                                                                                                                   |
| glgg                 | git log --graph                                                                                                                                                                          |
| glgga                | git log --graph --decorate --all                                                                                                                                                         |
| glgm                 | git log --graph --max-count=10                                                                                                                                                           |
| glo                  | git log --oneline --decorate                                                                                                                                                             |
| glol                 | git log --graph --pretty='%Cred%h%Creset -%C(auto)%d%Creset %s %Cgreen(%ar) %C(bold blue)<%an>%Creset'                                                                                   |
| glols                | git log --graph --pretty='%Cred%h%Creset -%C(auto)%d%Creset %s %Cgreen(%ar) %C(bold blue)<%an>%Creset' --stat                                                                            |
| glod                 | git log --graph --pretty='%Cred%h%Creset -%C(auto)%d%Creset %s %Cgreen(%ad) %C(bold blue)<%an>%Creset'                                                                                   |
| glods                | git log --graph --pretty='%Cred%h%Creset -%C(auto)%d%Creset %s %Cgreen(%ad) %C(bold blue)<%an>%Creset' --date=short                                                                      |
| glola                | git log --graph --pretty='%Cred%h%Creset -%C(auto)%d%Creset %s %Cgreen(%ar) %C(bold blue)<%an>%Creset' --all                                                                             |
| glog                 | git log --oneline --decorate --graph                                                                                                                                                     |
| gloga                | git log --oneline --decorate --graph --all                                                                                                                                               |
| glp                  | git log --pretty=\<format\>                                                                                                                                                              |
| gm                   | git merge                                                                                                                                                                                |
| gms                  | git merge --squash                                                                                                                                                                       |
| gmom                 | git merge origin/$(git_main_branch)                                                                                                                                                      |
| gmtl                 | git mergetool --no-prompt                                                                                                                                                                |
| gmtlvim              | git mergetool --no-prompt --tool=vimdiff                                                                                                                                                 |
| gmum                 | git merge upstream/$(git_main_branch)                                                                                                                                                    |
| gma                  | git merge --abort                                                                                                                                                                        |
| gp                   | git push                                                                                                                                                                                 |
| gpd                  | git push --dry-run                                                                                                                                                                       |
| gpf                  | git push --force-with-lease --force-if-includes (git version >= 2.30)                                                                                                                    |
| gpf                  | git push --force-with-lease (git version < 2.30)                                                                                                                                         |
| gpf!                 | git push --force                                                                                                                                                                         |
| gpoat                | git push origin --all && git push origin --tags                                                                                                                                          |
| gpod                 | git push origin --delete                                                                                                                                                                 |
| gpr                  | git pull --rebase                                                                                                                                                                        |
| gpu                  | git push upstream                                                                                                                                                                        |
| gpv                  | git push --verbose                                                                                                                                                                       |
| gr                   | git remote                                                                                                                                                                               |
| gra                  | git remote add                                                                                                                                                                           |
| grb                  | git rebase                                                                                                                                                                               |
| grba                 | git rebase --abort                                                                                                                                                                       |
| grbc                 | git rebase --continue                                                                                                                                                                    |
| grbd                 | git rebase $(git_develop_branch)                                                                                                                                                         |
| grbi                 | git rebase --interactive                                                                                                                                                                 |
| grbm                 | git rebase $(git_main_branch)                                                                                                                                                            |
| grbom                | git rebase origin/$(git_main_branch)                                                                                                                                                     |
| grbo                 | git rebase --onto                                                                                                                                                                        |
| grbs                 | git rebase --skip                                                                                                                                                                        |
| grev                 | git revert                                                                                                                                                                               |
| grh                  | git reset                                                                                                                                                                                |
| grhh                 | git reset --hard                                                                                                                                                                         |
| groh                 | git reset origin/$(git_current_branch) --hard                                                                                                                                            |
| grm                  | git rm                                                                                                                                                                                   |
| grmc                 | git rm --cached                                                                                                                                                                          |
| grmv                 | git remote rename                                                                                                                                                                        |
| grrm                 | git remote remove                                                                                                                                                                        |
| grs                  | git restore                                                                                                                                                                              |
| grset                | git remote set-url                                                                                                                                                                       |
| grss                 | git restore --source                                                                                                                                                                     |
| grst                 | git restore --staged                                                                                                                                                                     |
| grt                  | cd "$(git rev-parse --show-toplevel \|\| echo .)"                                                                                                                                        |
| gru                  | git reset --                                                                                                                                                                             |
| grup                 | git remote update                                                                                                                                                                        |
| grv                  | git remote --verbose                                                                                                                                                                     |
| gsb                  | git status --short -b                                                                                                                                                                    |
| gsd                  | git svn dcommit                                                                                                                                                                          |
| gsh                  | git show                                                                                                                                                                                 |
| gsi                  | git submodule init                                                                                                                                                                       |
| gsps                 | git show --pretty=short --show-signature                                                                                                                                                 |
| gsr                  | git svn rebase                                                                                                                                                                           |
| gss                  | git status --short                                                                                                                                                                       |
| gst                  | git status                                                                                                                                                                               |
| gsta                 | git stash push (git version >= 2.13)                                                                                                                                                     |
| gsta                 | git stash save (git version < 2.13)                                                                                                                                                      |
| gstaa                | git stash apply                                                                                                                                                                          |
| gstc                 | git stash clear                                                                                                                                                                          |
| gstd                 | git stash drop                                                                                                                                                                           |
| gstl                 | git stash list                                                                                                                                                                           |
| gstp                 | git stash pop                                                                                                                                                                            |
| gsts                 | git stash show --text                                                                                                                                                                    |
| gstu                 | git stash --include-untracked                                                                                                                                                            |
| gstall               | git stash --all                                                                                                                                                                          |
| gsu                  | git submodule update                                                                                                                                                                     |
| gsw                  | git switch                                                                                                                                                                               |
| gswc                 | git switch -c                                                                                                                                                                            |
| gswm                 | git switch $(git_main_branch)                                                                                                                                                            |
| gswd                 | git switch $(git_develop_branch)                                                                                                                                                         |
| gts                  | git tag -s                                                                                                                                                                               |
| gtv                  | git tag \| sort -V                                                                                                                                                                       |
| gtl                  | gtl(){ git tag --sort=-v:refname -n --list ${1}\* }; noglob gtl                                                                                                                          |
| gunignore            | git update-index --no-assume-unchanged                                                                                                                                                   |
| gunwip               | git log --max-count=1 \| grep -q -c "\-\-wip\-\-" && git reset HEAD~1                                                                                                                    |
| gup                  | git pull --rebase                                                                                                                                                                        |
| gupv                 | git pull --rebase --verbose                                                                                                                                                              |
| gupa                 | git pull --rebase --autostash                                                                                                                                                            |
| gupav                | git pull --rebase --autostash --verbose                                                                                                                                                  |
| gupom                | git pull --rebase origin $(git_main_branch)                                                                                                                                              |
| gupomi               | git pull --rebase=interactive origin $(git_main_branch)                                                                                                                                  |
| glum                 | git pull upstream $(git_main_branch)                                                                                                                                                     |
| gluc                 | git pull upstream $(git_current_branch)                                                                                                                                                  |
| gwch                 | git whatchanged -p --abbrev-commit --pretty=medium                                                                                                                                       |
| gwip                 | git add -A; git rm $(git ls-files --deleted) 2> /dev/null; git commit --no-verify --no-gpg-sign --message "--wip-- [skip ci]"                                                            |
| gam                  | git am                                                                                                                                                                                   |
| gamc                 | git am --continue                                                                                                                                                                        |
| gams                 | git am --skip                                                                                                                                                                            |
| gama                 | git am --abort                                                                                                                                                                           |
| gamscp               | git am --show-current-patch                                                                                                                                                              |
| gwt                  | git worktree                                                                                                                                                                             |
| gwtls                | git worktree list                                                                                                                                                                        |
| gwtmv                | git worktree move                                                                                                                                                                        |
| gwtrm                | git worktree remove                                                                                                                                                                      |
