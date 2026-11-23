# git-branch-cleaner

A password-protected Bash script that deletes git branches in bulk while keeping the ones you choose. It works from inside any local git repository.

## Features

- Runs **only on the `main` branch**. On any other branch it refuses to run.
- **`main` is always protected** and is never deleted.
- You choose which branches to **preserve**, either from a numbered list or with a flag.
- **Delete-all protection:** if you preserve no branches, the master password is required.
- **Delete password:** required before any branch is deleted.
- Three wrong password attempts stop the script and nothing is deleted.
- Dry-run mode to preview what would be deleted.
- Optional deletion of remote branches on `origin`.
- Passwords are stored as SHA-256 hashes inside the script, not as plain text.

## Requirements

- Bash 4 or newer is not required; Bash 3.2 (macOS default) works.
- Git
- `sha256sum` (Linux) or `shasum` (macOS). Both are normally pre-installed.
- On Windows, use **Git Bash** or **WSL**.

## Setup (one time)

1. Put the script in a folder on your PATH, for example `~/bin`:

   ```bash
   mkdir -p ~/bin
   mv ~/Downloads/git-branch-cleaner ~/bin/
   chmod +x ~/bin/git-branch-cleaner
   ```

2. Add that folder to your PATH:

   ```bash
   # bash
   echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
   source ~/.bashrc

   # zsh (macOS default)
   echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
   source ~/.zshrc
   ```

3. Check that it works:

   ```bash
   git-branch-cleaner --help
   ```

Because the file name starts with `git-`, git also lets you run it as a subcommand:

```bash
git branch-cleaner
```

## Usage

Go into any repo and switch to `main` first:

```bash
cd /path/to/your/repo
git checkout main
```

### Interactive mode

```bash
git-branch-cleaner
```

The script lists all branches with numbers. Then:

- Type the **numbers or names** of branches to keep, separated by spaces, for example `1 3 release`.
- Press **Enter with nothing typed** to delete all branches except `main`. This asks for the **master password**.
- The script shows what will be kept and what will be deleted, then asks for the **delete password**.

### Keep branches with a flag

```bash
git-branch-cleaner -k "dev,release"
```

This keeps `main`, `dev` and `release`, and deletes everything else after you enter the delete password.

### Preview only (dry run)

```bash
git-branch-cleaner -n
git-branch-cleaner -n -k "dev"
```

Shows what would be deleted and deletes nothing. If you keep no branches, it still asks for the master password first.

### Also delete remote branches

```bash
git-branch-cleaner -r -k "dev"
```

Deletes the branches on `origin` as well as locally. Without `-r`, only local branches are deleted. Remote deletion cannot be undone from this script, so use `-n -r` first to check the list.

## Options

| Option | Long form | Meaning |
|--------|-----------|---------|
| `-k "a,b"` | `--keep` | Comma-separated branches to preserve |
| `-r` | `--remote` | Also delete branches on `origin` |
| `-n` | `--dry-run` | Show what would be deleted, delete nothing |
| `-h` | `--help` | Show help |

## Password flow

| Situation | Passwords asked |
|-----------|-----------------|
| You preserve at least one branch | Delete password |
| You preserve no branches (delete all) | Master password, then delete password |

The passwords themselves are not written in this README. Ask the script owner for them.

## Changing the passwords

Generate the hash of a new password:

```bash
printf '%s' 'your-new-password' | sha256sum        # Linux
printf '%s' 'your-new-password' | shasum -a 256    # macOS
```

Open the script and replace the value of `MASTER_HASH` (delete-all password) or `DELETE_HASH` (delete password) with the new hash.

## Changing the protected branch

If your default branch is not called `main` (for example `master`), edit this line near the top of the script:

```bash
MAIN_BRANCH="main"
```

## Recovering a deleted branch

Local branches deleted with `git branch -D` can usually be recovered from the reflog:

```bash
git reflog
git branch <branch-name> <commit-hash>
```

## Troubleshooting

| Message | Fix |
|---------|-----|
| `You are on 'xyz'. Run 'git checkout main' first.` | Switch to main with `git checkout main`. |
| `Not inside a git repository.` | `cd` into a repo first. |
| `Remote 'origin' not found.` | Use `git remote -v` to check, or run without `-r`. |
| `failed remote: ... (protected or no permission?)` | The branch is protected on the server, or you lack permission. |
| `command not found` | Check the PATH step and run `source ~/.bashrc` (or `~/.zshrc`). |
| `Permission denied` | Run `chmod +x ~/bin/git-branch-cleaner`. |

## Safety notes

- Branches are deleted with `git branch -D` (force), even if they are not merged.
- Hashing the passwords hides them from casual reading, but this is a safety guard against mistakes, not strong security. Anyone with edit access to the script can change it.
