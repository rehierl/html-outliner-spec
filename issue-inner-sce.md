
Issue - What to do with the sections of an inner SCE?

* [W3C, HTML 5.2, Editor's Draft, 3 May 2017](https://w3c.github.io/html)

*acronyms*

* sectioning element (SE) - can be an SRE or an SCE
* sectioning content (SC) - SC element (SCE)
* sectioning root (SR) - SR element (SRE)
* heading content (HC) - HC element (HCE)

## [Step 4.5.5 of the Outline Algorithm](./outliner-steps.md)

append the outline of the SCE being exited to the last section of the current
outline (i.e. the outline of the first outer SE).

*notes*

* from a programmer's perspective, the concept of a section can have inner
  sections (i.e. subsections), but no inner outline -
* the operation 'append an outline to a section' makes no sense

**Version-1** - *one could read this as*

* add each section of the SCE being exited to the last **section** of the first
  outer SE (the current outline)
* put differently - add each section of the SCE being exited as the next child
  section / **subsection** of the last section of the first outer SE

## [4.3.9 Headings and sections](https://w3c.github.io/html/sections.html#headings-and-sections)

SCEs are always considered subsections of their first outer SE,
regardless of what implied sections other headings may have created.

*notes*

* an SCE itself isn't a section and therefore can't become a subsection of an SE -
  they represent sequences of sections - i.e. the word 'subsection' is confusing -
  should state: each section of an SCE (excluding any subsections) counts as
  subsection of the first outer SE
* in addition to that, you most probably read the word 'subsections' as to
  directly associate each section of an SCE with its first outer SE;
  i.e. not associate them with one of that SE's sections.

**Version-2** - *one could read this as*

* add each section of the SCE being exited to the **outline** of the first
  outer SE
* put differently - add each section of the SCE being exited as the next
  **sibling** of the last section of the first outer SE

## Version-1 vs. Version-2

* the difference is: add as subsection, or as a sibling of the last section
* both can be seen as referring to the current outline's last section
  in a different way
* put differently - both statements contradict each other

## Version-2

* **outline / sibling**

*to be continued ...........*

## Version-1

* **section / subsection**

*each SE must always have at least one section*

* otherwise, the to-the-last-section-of part would not make sense
* this must-exist section may even be completely empty - i.e. no heading
* see also - an outline for a SE consists of a list of one or more potentially
  nested sections -
  i.e. an outline is a *non-empty* sequence of sections
* see also - each SCE potentially has a heading and an outline
* see also - SREs can have their own outlines
* what exactly is the heading of a SCE?
* potentially/can - drop those words - they always have a non-empty outline,
  even if the single section they must at least contain is completely empty

*must associate some rank value with implied headings*

* empty sections will get an implied heading associated with them once a new
  section starts, or an SE is exited
* if an inner SCE is exited, XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

*must use HCEs outside of `<section>` elements*

* to get headings of the same topmost level.
* with that in mind, the `<section>` element's name `section` is somewhat
  violated: `subsection` would have been more appropriate.

**example-1**

```html
<body>
  <section></section>
  <section></section>
  <h1>A</h1>
</body>
```

[Nu Html Checker](https://validator.w3.org/nu/)
will produce the following outline:

```
1. body element with no heading
   1.1. section element with no heading
   1.2. section element with no heading
2. heading 'A'
```

i.e. 'Nu Html Checker' implements Version-1
