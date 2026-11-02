# Development notes

Maintainer (tgj5) notes for ling-handout. This file is listed in `exclude` in `typst.toml`, so it stays in the GitHub repo but isn't published to Universe. Don't reference it from anything inside the package.

## Project layout

```text
ling-handout/
├── typst.toml          # manifest: name, version, entrypoint, template config
├── lib.typ             # the template library: ex, ling-handout, to-plain (flatten content to plain text for PDF metadata)
├── README.md
├── LICENSE             # MIT-0 text, must match `license` in typst.toml
├── thumbnail.png       # generated; for typst universe; don't reference it anywhere
├── DEVELOPMENT.md      # this file (excluded from the package)
└── template/
    └── main.typ        # starter document copied by `typst init`
```

## Todo
- configure optional bibliography in template
- include the universal-style-sheet-for-linguistics.csl in the package, set it as default for bib.
- add support for IPA transcription.
- add support for syntax trees.
- add support for phonological rules.
- adapt or accommodate hw assignment template.
- adapt or accommodate essay template.
- make roadmap from iffiness handout into an outline element and include as optional element in template
- allow user to choose their own number patterning for example list and sub-lists. ideally user can mix and match the sub-list choice (e.g. use (i.) pattern sometimes, (a.) pattern other times in the same document)
- tagging an enum item directly isn't possible, but the user should be able to append a sub-list identifier onto the example when referencing it in text (so e.g. they can make (3.i) appear in text and still link to (3))
- abstract the example list into a separate element, so you could have two (or more) co-occurring running lists (e.g. you could have one list for example sentences, numbered as (1), (2),... and one list for definitions numbered as I, II,... (and possibly allow a pre-number label. so it would render as Definition I, Definition II,...))
- add support for Leipzig glossing.
- have header render the optional args on a second line, rather than after a comma
- allow user to specify datetime display pattern parameter for heading render
- make header fully customizable (include/leave out whatever you want, choose what is placed and where, toggle on/off on certain pages or completely)

## One-time setup

### Symlink the project into the local packages directory

The link name must equal `version` in `typst.toml`, and the project directory can be named anything. Use an absolute target path.

macOS:
```bash
PKG="$HOME/Library/Application Support/typst/packages/local/ling-handout"
mkdir -p "$PKG"
ln -s /full/path/to/ling-handout "$PKG/0.1.0"
```

Check: `ls "$PKG/0.1.0"` should list `typst.toml`, `lib.typ`, etc. Remove a link with `rm "$PKG/0.1.0"` (no trailing slash, no `-r`). For a new version, add another link named after it (e.g. `0.2.0`).

### Special tools

- `typst` (duh! the manifest's `compiler` field is the minimum version, currently 0.15.1)
- `oxipng` (optional, compresses the thumbnail)

## Development

Edits to `lib.typ` show up immediately through the symlink; just recompile.

### Test locally

This can be run in any local directory. Start by removing any old test copy if its present.

```bash
rm -rf test-handout
```

Initialize a new test Typst project based on the template.
```bash
typst init @local/ling-handout:0.1.0 test-handout
```

Change the import statement in the new project's ``main.typ`` file to the ``@local`` environment path.

```typst
#import "@local/ling-handout:0.1.0": ling-handout, ex
```

```bash
typst compile test-handout/main.typ
```

Delete `test-handout/` before re-running (typst init won't overwrite it). It's listed in `.gitignore`.

### PDF checks

- header: group + location on the left, author + date on the right
- title renders under the header
- examples show as (1), (2), ...; sub-lists as (i), (ii), ...
- `@ex-first` and `@ex-second` render as (1) and (2) and are clickable

### Customization checks

- `group: [CUNY *GC* Semantics]` (content)
- `handout-date: datetime.today()` (datetime)
- `handout-date` and `location` omitted, and set to `none`
- `handout-author: [Turner _Jordan_]`, then check the PDF's document properties: the author should be plain text (this exercises `to-plain`)
- only `location` set (no stray comma before it)
- no header parameters at all (empty header, no title, no errors)

## Thumbnail

Regenerate the thumbnail whenever the starter's appearance changes, and before each release.

```bash
typst compile -f png --pages 1 --ppi 250 template/main.typ thumbnail.png
oxipng -o 4 --strip safe thumbnail.png
```

The starter imports `@preview/...`, which only resolves once the package is published or while the `preview` symlink to your fork exists (release step 6). Generate the thumbnail then, or temporarily switch the starter's import to `@local/...`. The import line isn't rendered, so the image is the same.

## Tag a release

```bash
git tag v0.1.0
git push --tags
```

Don't tag a commit with a version until the PR is approved and merged. If you need to move a tag: `git tag -f v0.1.0 && git push --force origin v0.1.0`.

## Manifest notes (typst.toml)

- `exclude = ["DEVELOPMENT.md", ".gitignore"]` keeps dev files out of the published package. Nothing in the package may reference excluded files.

## Releasing to Typst Universe

Packages are submitted as pull requests to `typst/packages`. Re-read that repo's README right before submitting, since the rules change.

1. Finalize the version in `typst.toml`; update the import in `template/main.typ`.
2. Regenerate and optimize the thumbnail (see above).
3. In `template/main.typ`, change the import to the published form: `#import "@preview/ling-handout:0.1.0": ling-handout, ex` (use `@local/...` while testing from your machine).
4. Fork `typst/packages` and clone your fork.
5. Create a branch, then copy only the package files into `packages/preview/ling-handout/<version>/`: `typst.toml`, `lib.typ`, `README.md`, `LICENSE`, `thumbnail.png`, `template/`. Don't copy `.git`, PDFs, test output, or dev notes.
6. Test from the fork: symlink `~/Library/Application Support/typst/packages/preview` to the `packages/preview` folder of your fork, then:
   ```bash
   typst init @preview/ling-handout:<version> test-preview
   typst compile test-preview/main.typ
   ```
   The version must be given explicitly, since the package isn't indexed yet. Remove the link afterwards (`rm` on the link, no trailing slash), so that `@preview` imports resolve normally again rather than from your fork.
7. Commit, push the branch, and open a PR titled `ling-handout:<version>`, ticking "new package" (or "update" for later versions).
8. The `typst-package-check` bot comments with errors and warnings; fix them by pushing more commits. Reviewers may also request changes. After the PR is merged, the package appears on Universe.

### Releasing an update

- Bump `version` in `typst.toml`, update the import in `template/main.typ`, regenerate the thumbnail. Tag the release only after the PR is merged.
- Add a new folder `packages/preview/ling-handout/<new-version>/` in the fork (leave older version folders alone) and open a PR with the "update" box ticked.
- Add a new `@local` symlink for the new version to keep testing locally.

## Releasing a new version

Set the version once at the start; the commands below use these variables.
They only last for one terminal session, so set them again in a new window.

```bash
OLD=0.1.0
NEW=0.2.0
SCRIPTS=/Users/turner/Documents/linguistics/scripts
PROJ=$SCRIPTS/ling-handout
FORK=$SCRIPTS/typst-packages
DEST=$FORK/packages/preview/ling-handout/$NEW
PREVIEW="$HOME/Library/Application Support/typst/packages/preview"
LOCAL="$HOME/Library/Application Support/typst/packages/local/ling-handout"
```

Choosing the number (SemVer): patch (0.1.1) for bug fixes; minor (0.2.0) for
new features or anything that changes how existing documents look or behave. typst-packages doesn't seem to like you to update the major version, so keep at 0.

### 1. Prepare the project

1. Make and commit the changes in `$PROJ`.
2. Bump the version everywhere it appears:
   ```bash
   cd "$PROJ"
   grep -rn "$OLD" --exclude-dir=.git .
   ```
   Expected hits: `version` in `typst.toml`, the import in `template/main.typ`,
   and the imports and `typst init` line in `README.md`. Update each to `$NEW`.
   Leave `$OLD` in this file's examples if they're meant as history.
3. Update the README for any new or changed features and parameters. Every
   code block needs a language: `typst` for Typst code, `text` for rendered
   output, `bash` for commands. Unlabeled blocks are parsed as Typst, and the
   package-check bot flags syntax errors in them (e.g. a lone `*`).
4. Add a local symlink for the new version (keep the old one so old documents
   still compile):
   ```bash
   ln -s "$PROJ" "$LOCAL/$NEW"
   ```
5. Test locally (see "Test locally" above), using `@local/ling-handout:$NEW`,
   and run the PDF and customization checks. Also compile an existing handout
   that imports `$OLD` to make sure nothing about the old setup broke.
6. Commit and push to your GitHub repo. Don't tag yet.
   ```bash
   cd "$PROJ"
   git add -A
   git commit -m "Prepare $NEW"
   git push
   ```

### 2. Refresh the fork

The fork falls behind `typst/packages` between releases, so sync it first.

1. On GitHub, open `github.com/tgj5/packages` and click **Sync fork → Update
   branch** (or run `gh repo sync tgj5/packages`).
2. Make a fresh sparse clone (simplest; avoids shallow-clone pull problems):
   ```bash
   rm -rf "$FORK"
   cd "$SCRIPTS"
   git clone --depth 1 --filter=blob:none --sparse git@github.com:tgj5/packages.git typst-packages
   cd "$FORK"
   git sparse-checkout set packages/preview/ling-handout
   git checkout -b ling-handout-$NEW
   ```
   If SSH fails, see "SSH troubleshooting" below.
3. Copy the package files into a new version folder. Never edit an existing
   version folder.
   ```bash
   mkdir -p "$DEST"
   cd "$PROJ"
   cp typst.toml lib.typ README.md LICENSE "$DEST"/
   cp -R template "$DEST"/
   ls -R "$DEST"
   ```
   Only those files plus `template/main.typ` (no PDFs, no `.DS_Store`).

### 3. Thumbnail and fork test

1. Link Typst's `preview` folder to the fork (check nothing is there first):
   ```bash
   ls "$HOME/Library/Application Support/typst/packages/"   # expect only: local
   ln -s "$FORK/packages/preview" "$PREVIEW"
   ls "$PREVIEW/ling-handout/$NEW"                          # should list your files
   ```
2. Generate and optimize the thumbnail in the fork's package folder:
   ```bash
   cd "$DEST"
   typst compile -f png --pages 1 --ppi 250 template/main.typ thumbnail.png
   oxipng -o 4 --strip safe thumbnail.png
   file thumbnail.png && ls -lh thumbnail.png   # longer edge >= 1080px, < 3MB
   cp thumbnail.png "$PROJ"/
   ```
3. Test the published form, outside the package folder:
   ```bash
   cd /tmp && rm -rf test-preview
   typst init @preview/ling-handout:$NEW test-preview
   typst compile test-preview/main.typ && open test-preview/main.pdf
   ```
4. Remove the link and the test copy (no trailing slash on the link):
   ```bash
   rm "$PREVIEW"
   rm -rf /tmp/test-preview
   ```

### 4. Submit the PR

1. Check, commit, and push:
   ```bash
   cd "$FORK"
   git status            # only packages/preview/ling-handout/$NEW should be new
   git add packages/preview/ling-handout/$NEW
   git commit -m "ling-handout:$NEW"
   git push -u origin ling-handout-$NEW
   ```
2. On `github.com/tgj5/packages`, click **Compare & pull request**. Confirm the
   base is `typst/packages` / `main` and the diff contains only
   `packages/preview/ling-handout/$NEW/`.
3. Title: `ling-handout:$NEW`. In the PR template, tick "an update for a
   package" and complete the checklist. Briefly list what changed.
4. Fix anything the `typst-package-check` bot or reviewers raise by editing
   files in `$DEST`, then `git add`, `git commit`, `git push` on the same branch.
   Mirror every fix in `$PROJ`.

### 5. After merge

```bash
cd "$PROJ"
git add thumbnail.png
git commit -m "Thumbnail for $NEW"
git tag v$NEW
git push && git push --tags
```

Final check with no symlink in place:

```bash
cd /tmp && rm -rf final-check
typst init @preview/ling-handout:$NEW final-check
typst compile final-check/main.typ
rm -rf final-check
```

Then update this file: bump the `compiler` note if the minimum Typst version
changed, and remove finished items from the Todo list.
