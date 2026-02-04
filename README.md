# Dean Lab Publications (Central Repo)

This repository is the **index / portfolio** for writing projects (grants, manuscripts, responses to reviewers, protocols, etc.) maintained as **separate private repos** and pulled in here as **git submodules**.  
**Access control is repo-level**: if you don’t have permission to a given project repo, you won’t be able to fetch that submodule.

---

### Strategy
The winning pattern: “text in Git, PDFs everywhere”
- Keep **all sources in Git** (`.tex`, `.bib`, `.cls`, `.sty`, figures, scripts).
- **Do not commit build artifacts** (`.aux`, `.log`, `.bbl`, `_minted*`, etc.). Use a solid `.gitignore`.
- Generate PDFs **locally and/or via CI** so anyone can pull and build (and reviewers can download PDFs from CI artifacts).

---

## Repository layout
```
pubs/
  projecs/              # submodule (one repo per grant/paper)
  	project1/              # submodule
  	project2/              # submodule
  templates/             # templates for new grants/paper
  style/                 # optional: shared voice/rules/exemplars (non-sensitive)
```

**Rule:** Do not write the actual grant/manuscript content in this central repo.  
Write in the project submodule repos.

---

## Quickstart

### Clone everything (recommended)
```bash  
git clone --recurse-submodules git@github.com:TheDeanLab/pubs.git
```

### If you already cloned without submodules
```bash
git submodule update --init --recursive
```

### Pull updates (including submodules)
```bash
git pull
git submodule update --recursive
```

### Update submodules to their latest remote (when you want to advance pins)
```bash
git submodule update --remote --merge
git status   # you'll see the submodule pointer changes in this hub repo
git commit -am "Update project submodules"
git push
```

### Adding a new project (as a submodule)
From the hub repo root:
```bash
mkdir -p projects
git submodule add git@github.com:TheDeanLab/<PROJECT_REPO>.git projects/<project-name>
git commit -m "Add <project-name> submodule"
git push
```
A minimal, scalable LaTeX writing structure:
### Baseline project template (for each project repo)
```code
<project>/
  tex/
    main.tex
    macros.tex
    refs.bib
    sections/
      00_abstract.tex
      01_significance.tex
      02_innovation.tex
      03_approach.tex
  figures/
    fig01.pdf
    fig02.png
  build/
    Makefile            # or latexmkrc
  .gitignore
  README.md
```
*Hard rule:* split content into sections/*.tex and \input{...} them from main.tex.
This reduces merge conflicts dramatically and makes review cleaner.

---

## Collaboration conventions

1) One section per file
	•	Two people can edit different sections with minimal conflicts.
	•	Keep main.tex “structural only” (inputs, global macros, packages).

2) One sentence per line

Use semantic line breaks:
```tex
This is sentence one.
This is sentence two.
This is sentence three.
```

Benefits:
- Git diffs become readable.
- Merge conflicts shrink.
- Review becomes surgical.

3) Branch + PR workflow
- Work in short-lived branches: aim1-rewrite, rev-response-round2, fig1-caption.
- Merge via PR so CI builds and the PDF artifact is available for review.

### Build System (latexmk)
Use latexmk for consistent builds:
```bash
latexmk -pdf -interaction=nonstopmode -halt-on-error -file-line-error tex/main.tex
```

Makefile snippit
```make
pdf:
\tlatexmk -pdf -interaction=nonstopmode -halt-on-error -file-line-error tex/main.tex

clean:
\tlatexmk -C
```

### Local reproducibility (optional)
- Consider a pinned TeXLive environment (Docker or a documented TeXLive year).
- Avoid “works on my machine” by standardizing the build command (make pdf).

### CI: compile and attach the PDF on every PR (required)

Goal: every PR produces a downloadable PDF artifact so collaborators don’t need TeX locally.
Typical approach:
- GitHub Actions runs latexmk
- Uploads main.pdf as a build artifact
- Optionally blocks merging if compilation fails

(Keep CI config in each project repo, not in this hub repo.)

### Human readable diffs (latexdiff)

When you need a redline PDF between two versions:
- Tag important milestones (e.g., v0_submitted, v1_resubmission)
- Generate a redline PDF and share it with collaborators who don’t want to read git diffs

Typical workflow (conceptual):
	1.	git tag v0_submitted
	2.	Later: generate a latexdiff between v0_submitted and HEAD
	3.	Build the diff PDF and attach to PR / email / Slack

### Citation management (recommended default)

What I recommend as the baseline
- Maintain references in a plain BibTeX .bib file (refs.bib) with stable citekeys.
- Use biblatex + biber for drafts when you control the build, because it’s more robust and flexible.
- When a journal template requires it, switch to the template’s expected system (often natbib + BibTeX + journal .bst).

Why this approach works across NIH / Cell / Nature / Science / eLife:
- The .bib database stays the same.
- You can adapt the rendering layer to match the venue without rewriting your library.

Citekey policy (do this once; save pain forever)

Use consistent keys (pick one convention and stick with it):
- LastNameYYYYShortTitle (e.g., Fiolka2019ASLM)
- Avoid duplicates; avoid auto-generated garbage keys.

Tooling recommendation
- Zotero + Better BibTeX (or equivalent) to export curated .bib with stable keys.
- Keep the exported .bib in-repo, and treat it as the source of truth for the project.

### Figures and large files (Git LFS)

Use Git LFS for large binaries that change over time (high-res figures, Illustrator files, large PDFs):
	•	Track selectively (don’t LFS everything)
	•	Typical patterns:
	•	*.psd, *.ai, large *.pdf, large *.tif, etc.

Add a .gitattributes in each project repo, e.g.:
```code
*.psd filter=lfs diff=lfs merge=lfs -text
*.ai  filter=lfs diff=lfs merge=lfs -text
*.tif filter=lfs diff=lfs merge=lfs -text
```

### Comments / TODOs inside LaTeX
```tex
% TODO(Kevin): tighten this paragraph for significance.
% NOTE: verify this claim w/ citation.
```

### Editor tooling

PyCharm

Recommended:
- TeXiFy-IDEA plugin for LaTeX editing/build/navigation
- Configure a run configuration:
- make pdf (preferred) or the latexmk command directly
- Use a consistent project-level build command so everyone shares the same workflow

Optional:
- JetBrains AI Assistant / custom model endpoint for in-editor rewriting
- Project rules/style files (e.g., /style/voice.md, /style/nih_rules.md) in each project repo

VS Code (common default)

Recommended:
- LaTeX Workshop extension
- Configure:
  - build tool = latexmk
  - forward/inverse search (PDF ↔ source)
  - format-on-save if you adopt latexindent

 ### Submodule etiquette (important)
	•	The hub repo pins each project submodule to a specific commit.
	•	Updating a project repo does not automatically update the hub pointer.
	•	If you want the hub to reflect new work, run:
```bash
git submodule update --remote --merge
git commit -am "Bump submodules"
```

If someone can’t access a project repo, they’ll see an empty/uninitialized submodule directory. That’s expected and is the access-control mechanism.
