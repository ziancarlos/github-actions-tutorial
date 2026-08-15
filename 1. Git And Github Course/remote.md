## 1. Add a Remote Connection

Add a remote repository (e.g. GitHub) to your local repository:

git remote add <remote_name> <github_link>

Example:

git remote add origin https://github.com/username/repository.git

- `<remote_name>` → name for the remote connection, commonly `origin`
- `<github_link>` → URL of the remote GitHub repository

---

## 2. Push a Branch to a Remote

Push a local branch to the remote repository:

git push <remote_name> <branch_name>

Example:

git push origin master

- If the branch doesn't exist on the remote → Git creates it.
- If the branch already exists → Git updates it with your new commits.

---

## 3. Clone a Public Repository

Clone a remote repository to your local machine:

git clone <repo_url> <dir_name_we_want>

Example:

git clone https://github.com/username/repository.git my-project

- `<repo_url>` → URL of the repository
- `<dir_name_we_want>` → optional name for the local directory
- If you omit `<dir_name_we_want>`, Git uses the repository name as the directory name.