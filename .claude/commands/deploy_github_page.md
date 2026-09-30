---
description: Build the game and deploy it to GitHub Pages
argument-hint: "[repo-name]"
---

# Deploy to GitHub Pages

This command deploys the treasure hunt game to GitHub Pages. It sets up the repository and handles authentication along the way.

- Repository name: `$ARGUMENTS`. If no name is given, use `claude_code_treasure_game`. Below, `REPO_NAME` means this value.
- Live site: `https://USERNAME.github.io/REPO_NAME/`
- Source code: `https://github.com/USERNAME/REPO_NAME`

## Automated Deployment Process

Claude handles these steps in order and stops at the first one it can't finish:
1. Prerequisite tools: git, Node.js/npm, and GitHub CLI
2. Authentication check and GitHub CLI login
3. Local git repository and first commit
4. Repository creation or validation, and remote URL configuration
5. Vite base path configuration
6. Build and deployment to the `gh-pages` branch
7. GitHub Pages activation and a check that the live URL responds

Some steps need the user to act, such as installing tools or an interactive login. When that happens, stop and tell the user exactly what to run. Resume from that step after they confirm. Never ask the user to paste a token or password into the chat.

## Command Logic

```bash
# Step 1: Check prerequisite tools
git --version
node --version
npm --version
gh --version
# If a tool is missing, stop and ask the user to install it, then restart the terminal and VS Code:
#   Node.js:    winget install OpenJS.NodeJS.LTS   (macOS: brew install node)
#   GitHub CLI: winget install GitHub.cli          (macOS: brew install gh)
#   git:        winget install Git.Git             (macOS: xcode-select --install)

# Step 2: Check authentication status
gh auth status
# If not logged in: `gh auth login` is interactive, so Claude must not run it.
# Ask the user to run this in their own terminal and choose GitHub.com, HTTPS, and log in with a web browser:
#   gh auth login
# After they log in, let git push with the gh credentials:
gh auth setup-git
USERNAME=$(gh api user --jq .login)

# Step 3: Make sure this is a git repository with at least one commit
git rev-parse --is-inside-work-tree || git init -b main
git config user.name && git config user.email   # if either is empty, ask the user for their values; don't guess
git add -A
git commit -m "Initial commit"                  # only if there is something to commit

# Step 4: Check the current git remote and permissions
git remote -v
gh repo view "$USERNAME/REPO_NAME" --json name,visibility,viewerPermission
# - The repo doesn't exist: confirm the name with the user and tell them it will be PUBLIC
#   (GitHub Pages on a free account needs a public repo). Then create it and push:
#     gh repo create REPO_NAME --public --source=. --remote=origin --push
# - The repo exists and viewerPermission is ADMIN or WRITE: point origin at it:
#     git remote add origin https://github.com/$USERNAME/REPO_NAME.git
#     (or `git remote set-url origin ...` if origin already exists)
# - origin points to a repo the user can't push to (for example the tutorial's upstream repo):
#   do the same as "the repo doesn't exist" and repoint origin to the user's own repo

# Step 5: Verify or update the Vite configuration
# vite.config.ts must contain base: '/REPO_NAME/' inside defineConfig({ ... }).
# Without it, the page loads blank on GitHub Pages because the asset paths start at /.

# Step 6: Build and deploy
npm install
npm install --save-dev gh-pages      # only if it isn't in devDependencies yet
npm run build                        # Vite writes to build/, not dist/
git add -A && git commit -m "Configure GitHub Pages deployment"   # only if there are changes
git push -u origin HEAD              # push source code first
npx gh-pages -d build -b gh-pages    # deploy build/ to the gh-pages branch

# Step 7: Enable Pages if needed and check the site
gh api "repos/$USERNAME/REPO_NAME/pages" \
  || gh api -X POST "repos/$USERNAME/REPO_NAME/pages" -f "source[branch]=gh-pages" -f "source[path]=/"
gh api "repos/$USERNAME/REPO_NAME/pages/builds/latest" --jq .status   # repeat until "built"
curl -sI "https://$USERNAME.github.io/REPO_NAME/" | head -1            # expect HTTP 200

echo "🚀 Deployment complete! Your game is available at:"
echo "https://$USERNAME.github.io/REPO_NAME/"
echo "Source code: https://github.com/$USERNAME/REPO_NAME"
echo "Note: It may take a few minutes for the site to be live."
```

## What This Does

1. Checks that git, Node.js/npm, and GitHub CLI are installed, and gives install commands for any that are missing
2. Checks GitHub authentication and walks the user through `gh auth login` if needed
3. Initializes the local git repository and makes the first commit if there isn't one
4. Checks repository access and creates a new public repo if needed
5. Updates the git remote URL to match the authenticated user
6. Sets the Vite base path for GitHub Pages
7. Builds the React app for production (the output goes to `build/`)
8. Installs the gh-pages utility, pushes the source code, and deploys `build/` to the `gh-pages` branch
9. Turns on GitHub Pages for the `gh-pages` branch and checks that the live URL returns HTTP 200

## Common Scenarios Handled

### Authentication Issues
- **Not logged in**: Stops and has the user run `gh auth login` in their own terminal
- **Git push asks for a password**: Runs `gh auth setup-git` so git uses the gh credentials
- **Permission denied**: Creates a repository under the user's account and updates the remote URL

### Repository Issues
- **Not a git repository**: Runs `git init -b main` and makes the first commit
- **No remote**: Creates the repository and sets it as `origin`
- **Wrong remote**: Updates the remote URL to the authenticated user's repository
- **Repository doesn't exist**: Creates a new public repository after the user confirms the name

### Configuration Issues
- **Missing base path**: Adds `base: '/REPO_NAME/'` to vite.config.ts
- **Wrong base path**: Updates it to match the repository name

## Troubleshooting

- **404 errors**: Check that Pages is set to the `gh-pages` branch (Settings → Pages), then wait for the build to finish
- **Site not updating**: GitHub Pages can take 5–10 minutes to show changes
- **Assets not loading (blank page)**: The `base` in vite.config.ts doesn't match the repository name
- **Browser shows old version**: Hard refresh (Cmd+Shift+R / Ctrl+F5) or try an incognito window
- **Permission errors**: The command creates a new repository under the user's account
- **Build fails**: Run `npm install` to make sure all dependencies are installed
- **`npm`/`gh` not found right after installing**: Restart the terminal and VS Code so they pick up the new PATH

## Manual Override Options

To use a specific repository or account:
```bash
# Use an existing repository (you need push access)
git remote set-url origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
# Or pass a repository name to the command
/deploy_github_page my-treasure-game
```

## Clean up (Optional)

Run these only when the user asks for them:
```bash
gh auth logout
git branch --unset-upstream
```
