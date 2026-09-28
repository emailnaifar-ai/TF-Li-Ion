# Uploading this folder to GitHub

This file is for the authors and is excluded from the repository by `.gitignore`.

Target repository: https://github.com/emailnaifar-ai/TF-Li-Ion. Create it on GitHub first as an **empty**
repository, without a README, licence or .gitignore, because this folder already contains them.

Run the commands from this folder (`github/`). Git Bash, PowerShell and Linux/macOS shells all accept them
(one command per line).

```
git init
git config core.longpaths true   # Windows: some result paths are long (harmless elsewhere)
git add .
git status                       # check: no dataset/, data_raw/, data_processed/, scratch/ or UPLOAD.md
git commit -m "TF-Li-Ion v1.0.0: reproducibility package"
git branch -M main
git remote add origin https://github.com/emailnaifar-ai/TF-Li-Ion.git
git push -u origin main
```

Optional, to mark the release:

```
git tag -a v1.0.0 -m "v1.0.0"
git push origin v1.0.0
```

Then, on GitHub, create a release from the tag `v1.0.0` and paste the text of `RELEASE_NOTES.md`.

If the folder is rebuilt with `python scripts/make_release.py --out github` (the `.git` folder is kept), commit the
changes with these three commands, one per line (Windows PowerShell 5.1 has no `&&`):

```
git add -A
git commit -m "..."
git push
```

## After the push

1. **About box** (gear icon next to "About" on the repository page).
   - Description: *Tempered fractional-order state-of-charge observers for Li-ion cells: certified finite memory,
     rate ceiling and a negative offline test on CALCE data (reproducibility package).*
   - Topics: `battery`, `lithium-ion`, `state-of-charge`, `soc-estimation`, `fractional-calculus`,
     `tempered-fractional`, `observer`, `kalman-filter`, `lmi`, `calce`, `python`, `reproducibility`.
2. Check that GitHub shows "Cite this repository" (from `CITATION.cff`) and the MIT licence.
3. Before making the repository public, replace the entity author "The TF-Li-Ion authors" in `CITATION.cff` (and,
   if wanted, in `LICENSE`) with the author names. Add the article reference to `README.md` and `CITATION.cff`
   (`preferred-citation`) once the paper is published.
