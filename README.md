
## Branching Strategy & Workflow

1. **Branches**:

- `dev`: Active development (merged features are deployed to a development environment).
- `stg`: Staging/Testing (features ready for QA/UAT).
- `main`: Production/Live (only tested/approved features are deployed to a production environment).

2. **Key Principles**:

- **Environment Isolation**: Each branch maps to a deployment environment.
- **Pull Requests (PRs)**: Code changes are reviewed and merged via PRs.
- **Promotion Workflow**: Code moves sequentially: `feature → dev → stg → main`.

## Step-by-Step Workflow

**1. Initialize the Repository**

```
mkdir gh-branching-strategy
cd gh-branching-strategy
git init
git checkout -b dev
git add .gitignore
git commit -m "Initial Commit"
git remote add origin git@github.com:sarubhai/gh-branching-strategy.git
git push -u origin dev

git checkout -b stg 
git push -u origin stg

git checkout -b main 
git push -u origin main
```

**2. Creating Feature Branches**

**2.1.** Create feature branches from `dev` (e.g., `feature/login`, `feature/payment`, `feature/search`).

**Feature Login**

```
git checkout dev
git pull origin dev
git checkout -b feature/login
git add login.html
git commit -m "Add login.html"
git push origin feature/login
```

- Create a PR in GitHub from `feature/login → dev`

```
gh pr create --head feature/login --base dev --title "Login Feature" --body "Login Page"
```

**Feature Payment**

```
git checkout dev
git pull origin dev
git checkout -b feature/payment
git add payment.html
git commit -m "Add payment.html"
git push origin feature/payment
```

- Create a PR in GitHub from `feature/payment → dev`

```
gh pr create --head feature/payment --base dev --title "Payment Feature" --body "Payment Page"
```

**Feature Search**

```
git checkout dev
git pull origin dev
git checkout -b feature/search
git add search.html
git commit -m "Add search.html"
git push origin feature/search
```

- Create a PR in GitHub from `feature/search → dev`

```
gh pr create --head feature/search --base dev --title "Search Feature" --body "Search Page"
```

**2.2.** Merge features into dev via PRs after code review
- Merge all 3 features into dev via PRs:
- Approve and merge all three PRs with merge commits

**Feature Login Merge Commit**

```
git checkout dev
git pull origin dev
git merge --no-ff feature/login -m "Merge feature/login into dev"
git push origin dev
git branch -d feature/login
git push origin --delete feature/login
```

**Note:** For messy commit histories in a feature branch, use squash merges to collapse all commits into one.

```
git merge --squash feature/login
git commit -m "Add login feature"
```

**Feature Payment Merge Commit**

```
git checkout dev
git pull origin dev
git merge --no-ff feature/payment -m "Merge feature/payment into dev"
git push origin dev
git branch -d feature/payment
git push origin --delete feature/payment
```

**Feature Search Merge Commit**

```
git checkout dev
git pull origin dev
git merge --no-ff feature/search -m "Merge feature/search into dev"
git push origin dev
git branch -d feature/search
git push origin --delete feature/search
```

- Now dev branch has:
```
login.html
payment.html
search.html
```

**3. Promote Approved Features to** `stg`

After merging all 3 features into `dev`, **selectively promote 2 features** to `stg`:

**3.1.** Create a Release Branch from `dev` and Exclude `feature/search`

```
git checkout dev
git pull origin dev
git checkout -b release/v1.0
```

- Find the commit hash for "Add search.html"

```
git log --graph --oneline --all
```

- Remove the unwanted feature (e.g., search.html)
- Look for the Commit Hash of "Merge feature/search into dev"
- Revert the MERGE COMMIT (b240064):

```
GIT_EDITOR=vim git revert -m 1 b240064
# git revert -m 1 b240064 -F "Revert search feature merge" # -m 1 specifies the "mainline parent" (dev branch).
git push origin release/v1.0
```

**3.2.** Create a PR in GitHub from `release/v1.0 → stg`

```
gh pr create --head release/v1.0 --base stg --title "Release v1.0 to stg" --body "Login & Payment Page Release"
```

**3.2.** Merge the Release Branch into stg (Staging)

```
git checkout stg
git pull origin stg
git merge --no-ff release/v1.0 -m "Merge release/v1.0 into stg"
git push origin stg
```

**3.3.** Test in stg

- Deploy the stg branch to your staging environment and validate login.html and payment.html.
- Now release/v1.0 has:

```
login.html
payment.html
# search.html is removed via revert
```

**4. Deploy to Production** `main`

- Merge the same release branch into main:

**4.1.** Create a PR in GitHub from `release/v1.0 → main`

```
gh pr create --head release/v1.0 --base main --title "Release v1.0 to main" --body "Login & Payment Page Release"
```

**4.2.** Merge the Release Branch into main (Production)

```
git checkout main
git pull origin main
git merge --no-ff release/v1.0 -m "Merge release/v1.0 into main"
git push origin main
```

**4.3** Delete the Release Branch

```
git branch -d release/v1.0
git push origin --delete release/v1.0
```

prod now has login.html and payment.html (no search.html).

Later, feature/search can be added in a future release.


### Final Branch States

| Branch | Files	                             | Notes                                  |
| -------|---------------------------------------|----------------------------------------|
| dev	 | login.html, payment.html, search.html | All features (including reverted ones).|
| stg	 | login.html, payment.html	             | Tested features ready for main.        |
| main   | login.html, payment.html	             | Live code.                             |



**5. Fix the Reverted Feature**

Later, fix search.html in a new release:

**5.1.** Create feature/search-fix from dev, fix the bug

```
git checkout dev
git pull origin dev
git checkout -b feature/search-fix
git add search.html
git add README.md
git commit -m "Add search.html, README.md"
git push origin feature/search-fix
```

**5.2.** Create a PR in GitHub from `feature/search-fix → dev`

```
gh pr create --head feature/search-fix --base dev --title "Search Feature Fix" --body "Search Page & ReadMe"
```

**5.3.** Merge feature into dev

```
git checkout dev
git pull origin dev
git merge --no-ff feature/search-fix -m "Merge feature/search-fix into dev"
git push origin dev
git branch -d feature/search-fix
git push origin --delete feature/search-fix
```

**5.4.** Create a Release Branch from `dev`

```
git checkout dev
git pull origin dev
git checkout -b release/v1.1
```

**5.5.** Create a PR in GitHub from `release/v1.1 → stg`

```
gh pr create --head release/v1.1 --base stg --title "Release v1.1 to stg" --body "Search Page & ReadMe Release"
```

**5.6.** Merge the Release Branch into stg (Staging)

```
git checkout stg
git pull origin stg
git merge --no-ff release/v1.1 -m "Merge release/v1.1 into stg"
git push origin stg
```

**5.7.** Create a PR in GitHub from `release/v1.1 → main`

```
gh pr create --head release/v1.1 --base main --title "Release v1.1 to main" --body "Search Page & ReadMe Release"
```

**5.8.** Merge the Release Branch into main (Production)

```
git checkout main
git pull origin main
git merge --no-ff release/v1.1 -m "Merge release/v1.1 into main"
git push origin main
```

**5.9.** Delete the Release Branch

```
git branch -d release/v1.1
git push origin --delete release/v1.1
```


### Why Avoid Merging Directly Between Persistent Branches?
Problem: Merging dev → stg → main directly risks deploying untested code.

Solution: Use short-lived release branches to batch-test features before promoting to stg/main.

By adopting this workflow, you ensure stability in main while maintaining agility in dev and stg.

### Key Takeaways

`--no-ff` **Merges**: Always create merge commits for better auditability and traceability.

**Release Branches**: Temporary branches used to bundle features, test changes, and remove unwanted updates before merging into `stg` and `main`. They are deleted after merging.

**Environment Sync**: Both `stg` and `main` are updated using the same release branch, ensuring consistency across environments.

**Atomic Releases**: Features are grouped into a release branch for structured and predictable deployments.

**Flexibility**: Need a quick hotfix? Cherry-pick specific commits from the release branch into main without deploying everything.

**Stable Production**: This workflow guarantees that only thoroughly tested and approved features make it to production, while dev remains open for active development.

🚀 Ensuring a smooth and reliable deployment process!
