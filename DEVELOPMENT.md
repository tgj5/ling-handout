# Development notes

Maintainer (tgj5) notes for ling-handout. This file is listed in `exclude` in `typst.toml`, so it stays in the GitHub repo but isn't published to Universe. Don't reference it from anything inside the package.

## Project layout

```
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

```
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
