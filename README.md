# Git, GitHub, Zotero and Overleaf in one research project

**What this guide assumes.** You use git from a terminal (clone, commit, push, pull) and
have a GitHub account. You write papers in LaTeX on Overleaf, and you keep your
references in Zotero. If you need a refresher on the terminal, git or GitHub, start with
[codingCS](https://github.com/alopezespino/codingCS) and come back here.

> **Made and tested for macOS.** Git stores the Overleaf token in the macOS Keychain.
> Everything else works the same on any system.

This guide shows how I connect the four tools in each of my projects. The project is
one git repository on GitHub. The paper is an Overleaf project, which appears inside
that repository as the folder `draft/`. The references are a Zotero collection, which
Zotero writes into `draft/` as a `.bib` file and keeps up to date. You can then write on
overleaf.com or locally, and the code, the notes, the text and the references stay in
one place.

The guide has two parts. **Part 1** is the one-time setup. **Part 2** is the routine for
every working session. Not everyone follows every step: a project has three kinds of
collaborators, and [Who does what](#who-does-what) lists the steps for each.

## Contents

- [How the pieces fit](#how-the-pieces-fit)
- [Who does what](#who-does-what)
- [Part 1. One-time setup](#part-1-one-time-setup)
  - [Step 1. Overleaf token and Keychain (owner, local)](#step-1-overleaf-token-and-keychain-owner-local)
  - [Step 2. The Overleaf project as a submodule (owner)](#step-2-the-overleaf-project-as-a-submodule-owner)
  - [Step 3. A Zotero collection that exports itself (owner)](#step-3-a-zotero-collection-that-exports-itself-owner)
  - [Step 4. Join a project that is set up (local)](#step-4-join-a-project-that-is-set-up-local)
- [Part 2. Operation](#part-2-operation)
- [When something goes wrong](#when-something-goes-wrong)
- [License](#license)

## How the pieces fit

```text
Zotero collection "myproject"
   |  Better BibTeX rewrites the .bib on every change
   v
myproject/              git repository  --push-->  GitHub (code, notes, issues)
  draft/                git submodule   --push-->  Overleaf (the paper)
    main.tex
    myproject.bib
```

- **GitHub** holds the project repository: code, notes, literature summaries, and the
  issues that track tasks.
- **Overleaf** holds the paper. Its own git server is the remote of `draft/`, so a push
  from `draft/` appears in the Overleaf editor, and a pull brings in the edits made
  there.
- **The project repository** records, for each of its commits, which Overleaf version
  of the paper goes with it. GitHub stores this pointer, not the text of the paper.
- **Zotero** holds one collection per project. The Better BibTeX plugin exports it to
  `draft/myproject.bib` and rewrites the file whenever an item in the collection
  changes. One person owns the collection; the others only read the file.

Throughout, `myproject` is the project folder, `<owner>/myproject` its GitHub
repository, and `<project-id>` the Overleaf project's ID.

## Who does what

A project has three kinds of collaborators:

- **The owner** sets the project up, keeps the Zotero collection and works with git.
  There is one owner per project.
- **Local collaborators** edit the draft on their own computer (in VS Code or another
  editor) and send it to Overleaf with git. They need access to the GitHub repository,
  edit rights on the Overleaf project, and their own Overleaf token.
- **Overleaf-only collaborators** write only on overleaf.com. They need edit rights on
  the Overleaf project and nothing else: no git, no token, no Zotero. Access to the
  GitHub repository is optional, for following the issues.

| Section | Owner | Local | Overleaf-only |
|---|---|---|---|
| Step 1. Overleaf token and Keychain | Once per computer | Once per computer | Skip |
| Step 2. The Overleaf project as a submodule | Once per project | Skip | Skip |
| Step 3. A Zotero collection that exports itself | Once per project | Skip | Skip |
| Step 4. Join a project that is set up | On each further computer | Once per computer | Skip |
| Part 2. Before you write; Write locally | Yes | Yes | Skip |
| Part 2. Write on Overleaf | Yes | Yes | Yes |
| Part 2. Record the draft version | Yes | Yes | Skip |
| Part 2. Add or fix a reference | Yes | Send the DOI to the owner | Send the DOI to the owner |
| When something goes wrong | Yes | Yes | Skip |

A collaborator can change kind at any time: an Overleaf-only collaborator who wants to
work locally does Steps 1 and 4.

## Part 1. One-time setup

Overleaf-only collaborators skip Part 1: the owner's invitation to the Overleaf project
is all they need.

### Step 1. Overleaf token and Keychain (owner, local)

Overleaf's Git integration is a paid feature, available when the project owner has a
paid plan. Each person who pushes or pulls needs their own token.

1. On Overleaf, open Account Settings, find **Git Integration** and generate a token.
   Copy it: Overleaf shows it once, and it expires after one year.
2. Make sure Git saves passwords in the Keychain. This prints `osxkeychain` on most
   Macs:

   ```bash
   git config --global credential.helper
   ```

   If it prints nothing, run `git config --global credential.helper osxkeychain`.

The first time Git contacts Overleaf it asks for a password: paste the token. The
Keychain remembers it until the token expires.

### Step 2. The Overleaf project as a submodule (owner)

On Overleaf, create the project (or open an existing one), open **Integrations** in the
left panel, choose **Git**, and copy the URL, `https://git.overleaf.com/<project-id>`.
Then, in the project repository:

```bash
cd ~/path/to/myproject
git submodule add https://git@git.overleaf.com/<project-id> draft
git -C draft config pull.rebase false
git config push.recurseSubmodules on-demand
git commit -m "draft: add the Overleaf project as a submodule"
git push
```

- The `git@` in the URL is the user name Overleaf expects, so Git asks only for the
  token.
- `pull.rebase false` makes a pull merge the two sides when you and a coauthor have
  both edited, instead of stopping with an error.
- `push.recurseSubmodules on-demand` makes every `git push` of the project repository
  push the `draft/` commits that Overleaf does not have yet. GitHub then never records a
  draft version that Overleaf lacks.
- New Overleaf projects use the branch `main`, older ones `master`.
  `git -C draft branch --show-current` tells you which; this guide says `main`.

**Why Overleaf's Git integration, not its GitHub integration.** Overleaf offers both.
The GitHub integration ties the Overleaf project to a GitHub repository of its own,
which you sync by pressing buttons in the Overleaf menu. The paper then lives in a
second repository, apart from the code and notes. The Git integration gives the
Overleaf project a git address. Added as a submodule, the paper becomes a folder of the
project repository whose remote is Overleaf itself: a push from `draft/` goes straight
to the editor, there is no second GitHub repository, and nobody presses a sync button.

Last, keep LaTeX build files off Overleaf. A local compile writes them into `draft/`,
and `git add -A` would push them. Create `draft/.gitignore` with these lines, then
commit and push it from `draft/`:

```text
*.aux
*.bbl
*.bcf
*.blg
*.fdb_latexmk
*.fls
*.log
*.out
*.run.xml
*.synctex.gz
*.toc
.DS_Store
/main.pdf
```

### Step 3. A Zotero collection that exports itself (owner)

1. Install [Better BibTeX for Zotero](https://retorque.re/zotero-better-bibtex/)
   (download the `.xpi`; in Zotero, Tools > Plugins, the gear menu, Install Plugin From
   File).
2. Choose the citation key formula once, in Zotero Settings > Better BibTeX, before the
   first citation. Do not change it during the project: a new formula renames keys the
   paper already cites. My keys look like `Author2024`, `AuthorAuthor2024` and
   `Author-etal2024`.
3. In My Library, create a collection named after the project folder (`myproject`), and
   add to it every paper the project cites.
4. Right-click the collection, Export Collection. Choose the format **Better BibLaTeX**
   (or **Better BibTeX** if the paper uses BibTeX and natbib), tick **Keep updated**,
   and save the file as `myproject.bib` inside `draft/`. Zotero Settings > Better
   BibTeX > Automatic export now lists it.
5. Point the paper to the file. With biblatex, in `main.tex`:

   ```latex
   \usepackage[backend=biber,style=authoryear]{biblatex}
   \addbibresource{myproject.bib}
   % ... the paper ...
   \printbibliography
   ```

   Other LaTeX files in the repository (notes, literature summaries) use a relative
   path to the same file, for example `\addbibresource{../draft/myproject.bib}`.
6. Commit and push the `.bib` from `draft/` (Part 2, "Write locally").

The file lives in `draft/` because Overleaf can only read files inside its own project.
Only the owner keeps an automatic export to it: two exports writing one file would
overwrite each other. Collaborators do not need Zotero; the `.bib` reaches them with the
paper.

### Step 4. Join a project that is set up (local)

For local collaborators, and for the owner on any computer other than the one used in
Steps 2 and 3. You need access to the GitHub repository, an invitation to the Overleaf
project with edit rights, and your own token (Step 1). Then:

```bash
git clone --recurse-submodules \
  https://github.com/<owner>/myproject.git
cd myproject
git -C draft switch main
git -C draft config pull.rebase false
git config push.recurseSubmodules on-demand
```

The clone leaves `draft/` at the version the repository recorded, on no branch, and
commits made there are easy to lose. `git -C draft switch main` puts it on Overleaf's
branch. If you cloned without `--recurse-submodules`, run
`git submodule update --init draft` first.

## Part 2. Operation

Each paragraph names who follows it, as in [Who does what](#who-does-what).

**Before you write** (owner, local), bring in both sides:

```bash
git pull                 # the project repository, from GitHub
git -C draft pull        # the paper, from Overleaf
```

Use `git -C draft pull`, not `git submodule update`, to get the newest text: the second
checks out the recorded version on no branch.

**Write locally** (owner, local), then send the changes to Overleaf and record the
version in the project repository:

```bash
git -C draft add -A
git -C draft commit -m "Introduction: state the contribution"
git -C draft push        # now visible in the Overleaf editor
git add draft
git commit -m "draft: sync with Overleaf"
git push                 # GitHub records which draft goes with the code
```

**Write on Overleaf** (everyone): write as usual. Edits pushed from a computer appear
in the editor at once, and Overleaf merges them with yours. Do not edit the `.bib` file
on Overleaf (see "Add or fix a reference"). The owner and local collaborators who also
write on Overleaf pull before their next local edit.

**Record the draft version** (owner, local): edits made on Overleaf reach the project
repository only when someone with git records them. Do it whenever the code or notes
should point to the current text, for example before a submission or after the text
changes in a way the code depends on:

```bash
git -C draft pull
git add draft
git commit -m "draft: sync with Overleaf"
git push
```

**Add or fix a reference** (owner; the others send the DOI):

1. Add the paper to the project's collection, with the Zotero Connector in the browser
   or with Add Item by Identifier (the magic wand) and its DOI. Better BibTeX rewrites
   `draft/myproject.bib` within seconds.
2. Cite it with the key in its Citation Key field: `\textcite{Author2024}`.
3. Commit and push the `.bib` from `draft/`, as in "Write locally". Overleaf sees the new
   entry only after the push.

Fix a wrong record in Zotero, never in the `.bib` or on Overleaf: the next export
overwrites any edit made to the file. Local and Overleaf-only collaborators send the
DOI to the owner, or open an issue.

**Rules that prevent most problems:**

- Owner and local: pull from Overleaf before you edit locally.
- Owner and local: Overleaf has one branch. Do not create other branches in `draft/`.
- Everyone: a commit that finishes a task closes its GitHub issue (write `closes #N` in
  the commit message); without git, close the issue on GitHub by hand.

## When something goes wrong

For the owner and local collaborators. Overleaf-only collaborators do not meet these
problems.

| Symptom | Fix |
|---|---|
| `git -C draft push` is rejected ("fetch first") | Overleaf has edits you do not have. `git -C draft pull`, resolve any conflict, push again. |
| A pull reports a conflict | Edit the file between the `<<<<<<<` and `>>>>>>>` markers, then `git -C draft add <file>`, `git -C draft commit`, push. |
| `HEAD detached` in `draft/` | Run `git -C draft switch main` before editing. If you already committed there: `git -C draft branch keep`, `git -C draft switch main`, `git -C draft merge keep`. |
| "Authentication failed" on Overleaf | The token expired or was deleted. Generate a new one, delete the `git.overleaf.com` entry in Keychain Access, and paste the new token at the next pull. |
| Citation undefined on Overleaf, fine locally | The new `.bib` was not pushed. Commit and push from `draft/`. |
| Citation undefined everywhere | The paper is not in the collection, or its key changed after an edit to the author or year. Copy the key from Zotero. |
| `git status` shows `draft (new commits)` | The recorded version is behind. `git add draft`, commit, push. |
| "Need to specify how to reconcile divergent branches" | Run `git -C draft config pull.rebase false` and pull again. |

## License

MIT. See [LICENSE](LICENSE).
