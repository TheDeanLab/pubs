# Projects (git submodules)

Each writing project (grant, manuscript, response-to-reviewers, etc.) lives in its own private repository
and is included in the hub as a **git submodule** under `projects/`. If you do not have access to a project repo, you will not be able to fetch that submodule.

## Creating a new repository.

Our canonical workflow: create child repo from template, then register it as a submodule in pubs

1) Create the child repo on GitHub (web UI)
	•	Use the template repo → “Use this template” → create new repo (e.g., 2026-high-na-ctaslm)
	•	Ensure the default branch is what you expect (main typically)

2) In pubs, add the child as a submodule at the final path.
   From the pubs root:
   ```bash
   cd /Users/Dean/Documents/GitHub/pubs
  
   # pick your final destination
   SUB=projects/2026-high-na-ctaslm
   URL=git@github.com:TheDeanLab/2026-high-na-ctaslm.git
  
   git submodule add "$URL" "$SUB"
   git submodule update --init --recursive "$SUB"
   ```

  Key point: git submodule add <url> <path> is the “install” step.
  Do not git clone and then mv as a substitute. Moving a submodule directory does not move the parent’s gitlink.

3) Commit the submodule registration in pubs (this is what “records it”)
   ```bash
   git add .gitmodules "$SUB"
   git commit -m "Add 2026-high-na-ctaslm submodule"
   git push
   ```

   This records:
	•	.gitmodules (URL + path)
	•	the gitlink SHA entry at projects/2026-high-na-ctaslm

## Day-to-day workflow once it is  a submodule.

1) Make changes in the child repo
   ```bash
    cd projects/2026-high-na-ctaslm

    # attach to a branch before committing
    git switch main
    git pull
    
    # edit, then
    git add -A
    git commit -m "Describe change"
    git push
    ```

2) “Bump” the parent to point at the new child commit
   ```bash
   cd ../..  # back to pubs root

   git add projects/2026-high-na-ctaslm
   git commit -m "Bump 2026-high-na-ctaslm submodule"
   git push
   ```
## How to clone the parent repository properly

In one line...
```bash
git clone --recurse-submodules git@github.com:TheDeanLab/pubs.git
```

If the repository is already cloned...
```bash
git submodule update --init --recursive
```
