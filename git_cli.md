## Basic commond git commands(Linux)
```bash
git init                                      # Init the git
git clone <url>                               # Clone project from URL

git status                                    # Chech repository statu/changes
git add "file to stage"                              # Make the modified selected files ready for staging(commit). git add . for all changes
git commit -m "message"                     # Commit the changes localy
git push                                      # Push the commited changes to git repository 


git log                                       # Commits log
git show                                      # Last commited commit changes
git diff                                      # Show current unstagged changes in terminal

git pull                                      # Update current branch
git pull "remote" "desired_branch"            # Pull only a specific branch
git checkout "verbose" "branch name"          # Switch to new branch. use -b verbose to create a new one

git reset "verbose" "remote" "branch"         # Reset current branch to be in same with remote branch, verbose --hard will reset the code,while --soft only commit
git merge "desired branch"                    # Merge changes in desired branch to current branch

git rev-list --count "branch"                 # Count all commits of a branch,replace "branch" with --all for all branches

ssh-keygen -t ed25519 -C "email"              # Generate ssh key for ssh clone/deploy...etc
```
