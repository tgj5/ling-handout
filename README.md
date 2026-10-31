# ling-handout

A Typst template for linguistics handouts. Examples are rendered as (1), (2),... and numbered continuously across the whole document. Sub-lists inside an example render as (i), (ii), ...

## Usage

```typst
#import "@preview/ling-handout:0.1.0": ling-handout, ex

#show: ling-handout.with(
  handout-title: [Title],
  handout-author: "Your Name",
  group: "Course",
)

#ex[
  + First.
  + Second.
] <ex-1>

See @ex-1.
```

Or start from the template: `typst init @preview/ling-handout:0.1.0`

## Parameters

| Parameter | Default | Notes |
|---|---|---|
| `handout-title` | `"Untitled"` | `string` or `content`; also sets `document.title` |
| `handout-author` | `"FirstName LastName"` | `string` or `content`; also sets `document.author` |
| `handout-date` | `none` | `string`, `content`, or `datetime`; optional, use `none` to omit; does NOT set `document.date`, only renders in header |
| `group` | `"Course"` | `string` or `content` |
| `location` | `none` | `string` or `content`; optional, use `none` to omit |
| `paper` | `"us-letter"` | passes value to `page.paper` |
| `margin` | `1in` | passes value to `page.margin` |
| `font` | `"New Computer Modern"` | passes value to `text.font` |
| `font-size` | `12pt` | passes value to `text.size` |
| `leading` | `0.52em` | passes value to `par.leading`|
| `spacing` | `0.9em` | passes value to `par.spacing` |

## Roadmap
- set this up to be used as a package, upload to typst universe.
- at least publish repo to github.
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

## CLI commands
1. Generate new thumbnail image (use before new version releases): 
    ```
    typst compile template/main.typ thumbnail.png --pages 1 --ppi 250
    ```
2. Initialize new project with template:
    ```
    typst init @preview/ling-handout:0.1.0
    ```
