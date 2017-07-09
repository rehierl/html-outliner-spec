
Issue - What to do with the sections of an inner SCE?

* [W3C, HTML 5.2, Editor's Draft, 3 May 2017](https://w3c.github.io/html)
* [W3C, HTML 5.2, Working Draft, 9 May 2017](https://www.w3.org/TR/html52/)

*acronyms*

1. sectioning element (SE) - can be an SRE or an SCE
1. sectioning content (SC) - SC element (SCE)
1. sectioning root (SR) - SR element (SRE)
1. heading content (HC) - HC element (HCE)

*definitions*

1. see [outliner-chapter](./outliner-chapter.md) for a definition of the expression
   "first inner/outer X element of element Y"

## [Steps 4.5.4 and 4.5.5](./outliner-steps.md)

1. step 4.5.4. - Let current section be the last section in the outline of the
   current outline target element.
1. step 4.5.5. - Append the outline of the sectioning content element being
   exited to the current section.
1. note - 4.5.5. does not change which section is the last section in the outline
1. note - 4.5.5. does not change the reference to the current section - it keeps
   referencing the last section

*more formal*

1. append the outline of the SCE being exited to the last section of the first
   outer SE
1. **unclear operation** - append an outline to a section
1. note - a section can have inner subsections, but it itself isn't defined to
   have an inner outline

**Version-1** - *one could still read this as*

1. add each section of the inner SCE being exited **as new <u>subsections</u> to the
   last section** of the first outer SE
1. recall that 'section of an SE' only refers to the topmost sections and only
   implicitly includes inner subsections, if any

## [4.3.9 Headings and sections](https://w3c.github.io/html/sections.html#headings-and-sections)

1. From the (5th) paragraph just before example 24:
1. SCEs are always considered subsections of their nearest ancestor sectioning
   root or their nearest ancestor element of sectioning content, whichever is
   nearest, regardless of what implied sections other headings may have created

*more formal*

1. SCEs are always considered subsections of their first outer SE, regardless of
   what implied sections other headings may have created.
1. **unclear meaning** - SEs aren't sections themselves - they may have an outline
   and therefore may contain a list of inner sections - in essence, they act more
   like a container for inner sections.
1. note - an SCE itself isn't a section and therefore can't become a subsection
   of an SE
1. note - an SE isn't a section and itself can't have subsections

**Version-2** - *one could still read this as*

1. add each section of the SCE being exited to the outline of the first
   outer SE - or - add each section of the SCE being exited
   **as new <u>siblings</u> to the last section** of the first outer SE
1. note - the above use of the word 'subsections' does not refer to any specific
   inner section of the first outer SCE - i.e. it does not imply to associate an
   inner SCE with an inner section of the first outer SE, it implies to directly
   associate an inner SCE with the SE itself

**Version-1 vs. Version-2**

1. i.e. (1) add as subsections -vs- (2) add as siblings to the last section
1. **conclusion - both statements contradict each other**

## Version-1: as subsections

*requires that each SE must always have at least one section object*

1. otherwise, the to-the-last-section-of part would not make sense
1. this must-exist section may have no heading or even be completely empty
1. see also - an outline for a SE consists of a list of one or more potentially
   nested sections - i.e. an outline is a *non-empty* sequence of sections
1. see also - each SCE potentially has a heading and an outline
1. see also - SREs can have their own outlines

*does not make the construct of implied headings a necessity*

1. if a section has ended and has no heading, the algorithm will associate a
   pseudo heading (implied heading) with it
1. these implied headings represent two statements - (1) the section has been
   processed/exited and (2) the section has no heading element
1. the main reason for this is to keep the steps, that assign headings to
   sections, from assigning headings to a section of an inner SCE, which have
   already been processed/exited
1. the sections of the preceeding inner SCE will be hidden as subsections inside the
   current/last section of the first outer SE
1. the above steps start at the current section and, if necessary, work their
   way up the hierarchy of sections
1. therefore, they can never come into contact with the merged inner sections
   from an inner SCE
1. as a consequence, the construct of implied headings is not needed and
   should therefore be removed

*one must use HCEs outside of `<section>` elements*

1. to get headings of the same topmost level, HCEs, isolated from `<section>`
   elements, must be used - see the below example
1. with that in mind - the `<section>` element can't be used to its full
   potential - `<subsection>` might have been a better choice

## Version-2: as siblings

1. **to be continued ...........**

## Example fragments

[Nu Html Checker](https://validator.w3.org/nu/) can be used to create outlines
for HTML documents.

**example-1**

the following HTML fragment

```html
<body>
  <section></section>
  <section></section>
  <h1>A</h1>
</body>
```

will produce the following outline:

```
1. body element with no heading
   1.1. section element with no heading
   1.2. section element with no heading
2. heading 'A'
```

(i.e. 'Nu Html Checker' implements Version-1)

**example-2**

```html
<body>
<p>A</p>
<h1>B</h1>
<p>C</p>
</body>
```

will produce the following outline:

```
1. heading 'B'
```

More importantly, both paragraphs *A* and *C* will be associated with heading
*B*. In case of *C*, that is almost always intended by the author. Automatically
associating paragraph *A* with heading *B* on the other hand could turn out to
be not intended.

The problem is, with that implementation, this behavior cannot be avoided!
