Decentralized
- By storing whole chain and distributed, each replica is a backup (as opposed to centralized source of truth if down then everyone screwed)
What is git
- Git isn’t storing changes, it stores snapshot of state. If no changes made it references prev
- Most operations add-only to the Git database, which means free to experiment and screw up b/c can undo
Git setup
- `git config` stored in 3 places: `--system` level (all users), `--global` level (all of my repos), `--local` (only this repo)
	- each level overrides the boarder one before it
	- ex. name, email, editor, default branch name
Git repo
- `git init` creates subfolder called .git
- `git clone` clones everything (every version of every file) 
Repo changes
- Tracked files are files that were in the last snapshot + newly staged files
	- `git status` tells you untracked files
- `git add` says add precisely this content at this moment to the next commit (therefore used for start tracking, staging, and marking resolved merge conflicts)
- `git diff` shows changes that aren't staged, `git diff --staged` compares staged changes to last commit
Commit history
- `git log --patch --3` shows difference introduced in each commit, limited to 3 entries
- `--pretty` to format
Undoing things
- 