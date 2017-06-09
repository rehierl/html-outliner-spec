
Issue - What to do with the sections of an inner SCE?

* [W3C, HTML 5.2, Editor's Draft, 3 May 2017](https://w3c.github.io/html)

*acronyms*

1. sectioning element (SE) - can be an SRE or an SCE
1. sectioning content (SC) - SC element (SCE)
1. sectioning root (SR) - SR element (SRE)
1. heading content (HC) - HC element (HCE)

## [Steps 4.5.4 and 4.5.5 of the Outline Algorithm](./outliner-steps.md)

1. 4.5.4. - Let current section be the last section in the outline of the current
   outline target element.
1. 4.5.5. - Append the outline of the sectioning content element being exited to
   the current section.
1. note - 4.5.5. does not change which section is the last section in the outline
1. note - 4.5.5. does not change the reference to the current section

*more formal*

1. append the outline of the SCE being exited to the last section of the first
   outer SE
1. note - a section can have inner (sub)sections, but it itself isn't defined to
   have an inner outline

**Version-1** - *one could still read this as*

1. add each section of the inner SCE being exited **as new subsections to the
   last section** of the first outer SE
1. recall that 'section of an SE' only refers to the topmost sections and only
   implicitly includes the inner subsections, if any

## [4.3.9 Headings and sections](https://w3c.github.io/html/sections.html#headings-and-sections)

1. SCEs are always considered subsections of their nearest ancestor sectioning
   root or their nearest ancestor element of sectioning content, whichever is
   nearest, regardless of what implied sections other headings may have created

*more formal*

1. SCEs are always considered subsections of their first outer SE, regardless of
   what implied sections other headings may have created.
1. note - an SCE itself isn't a section and therefore can't become a subsection
   of an SE - they represent sequences of sections
1. note - an SE isn't a section and itself can't have subsections

**Version-2** - *one could still read this as*

1. add each section of the SCE being exited to the outline of the first
   outer SE - or, put differently - add each section of the SCE being exited
   **as new siblings to the last section** of the first outer SE
1. note - the above use of the word 'subsections' does not refer to any inner
   section of the first outer SCE - i.e. it does not imply to associate an inner
   SCE with an inner section of the first outer SE, it implies to associate
   these with the SE itself

**Version-1 vs. Version-2**

1. i.e. (1) add as subsections -vs- (2) add as siblings to the last section
1. **conclusion** - both statements contradict each other

## Version-2 (as siblings)

1. **to be continued ...........**

## Version-1 (as subsections)

*each SE must always have at least one section object*

1. otherwise, the to-the-last-section-of part would not make sense
1. this must-exist section may have no heading or even be completely empty
1. see also - an outline for a SE consists of a list of one or more potentially
   nested sections - i.e. an outline is a *non-empty* sequence of sections
1. see also - each SCE potentially has a heading and an outline
1. see also - SREs can have their own outlines

*does not make the construct of implied headings a necessity*

1. if a section has ended and has no heading, the algorithm will associate a
   pseudo heading (implied heading) with it
1. these implied headings represent two statements: (1) the section has ended
   and (2) the section has no heading element
1. the main reason for this is to keep the steps, that assign headings to sections,
   from assigning headings to a section of an inner SCE, which have ended
1. the sections of the preceeding inner SCE will be hidden inside the
   current/last section of the first outer SE
1. the steps above steps, that assign headings to sections, start at the current
   section and, if necessary, work their way up the hierarchy of sections
1. so these steps can never come into contact with the merged inner sections
   from an inner SCE
1. the construct of implied headings is therefore not needed and should be removed

*must use HCEs outside of `<section>` elements*

1. to get headings of the same topmost level, separate HCEs must be used
1. with that in mind, the `<section>` element's name `section` is somewhat
   violated - `<subsection>` would have been more appropriate.

**example-1**

the following HTML fragment

```html
<body>
  <section></section>
  <section></section>
  <h1>A</h1>
</body>
```

will produce the following outline
([Nu Html Checker](https://validator.w3.org/nu/)):

```
1. body element with no heading
   1.1. section element with no heading
   1.2. section element with no heading
2. heading 'A'
```

(i.e. 'Nu Html Checker' implements Version-1)
