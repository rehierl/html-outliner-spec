
html-outliner-spec
===============

```
html-specification > outliner-steps > pseudocode > implementation
```

It might seem straight forward to take the HTML specification, extract the
outline algorithm's definition, translate it into pseudocode and then just
implement it.

Unfortunately, there are quite a few details that still need to be clarified ...

## Overview

Documents

* [chapter 4.3.9](./outliner-4.3.9.md) - The unmodified content of chapter
  [4.3.9. Headings and sections](https://w3c.github.io/html/sections.html#headings-and-sections)
* [notes on chapter 4.3.9.](./outliner-notes.md) - A restructured summary of
  chapter 4.3.9.
* [outline algorithm](./outliner-steps.md) - The outline algorithm (as is) split
  up into separate expressions which allows to reference each one of them.
* [notes on html](./html-notes.md) - Notes on different aspects of HTML related
  to the outline algorithm - i.e. notes on certain elements and boolean attributes.

Associations

* [notes](./issue-associations-notes.md) - The notes I took when I tried to
  understand what the algorithm's result is supposed to be.
* [overview](./implementation-overview.md) - My take on a detailed description
  of the algorithm's result.
* [summary](./issue-associations-summary.md) - A summary of the issues and bugs
  mentioned in the notes - including hints on how to fix these.

TODOs

* [issue-headings](./issue-headings.md)
* [issue-inner-sce](./issue-inner-sce.md)

Implementations

* [pseudocode](./pseudocode.md) - the algorithm's steps translated into pseudocode.
* [implementations](./implementations.md) - contains a short list of available
  open source implementations. (TODO - add link to nu-checker source)

## Related Links

[W3C, HTML 5.2, Editor's Draft, 2017-05-03](https://w3c.github.io/html/)

* [4.3.9. Headings and sections](https://w3c.github.io/html/sections.html#headings-and-sections)
* [4.3.9.1. Creating an outline](https://w3c.github.io/html/sections.html#creating-an-outline)

[W3C, HTML 5.2, Working Draft, 2017-05-09](https://www.w3.org/TR/html52/)

* [4.3.9. Headings and sections](https://www.w3.org/TR/html52/sections.html#headings-and-sections)
* [4.3.9.1. Creating an outline](https://www.w3.org/TR/html52/sections.html#creating-an-outline)

[W3C, HTML 5.1, Recommendation, 2016-11-01](https://www.w3.org/TR/html51/)

* [4.3.10. Headings and sections](https://www.w3.org/TR/html51/sections.html#headings-and-sections)
* [4.3.10.1. Creating an outline](https://www.w3.org/TR/html51/sections.html#creating-an-outline)

[W3C, HTML 5, Recommendation, 2014-10-28](https://www.w3.org/TR/html5/sections.html)

* [4.3.10. Headings and sections](https://www.w3.org/TR/html5/sections.html#headings-and-sections)
* [4.3.10.1 Creating an outline](https://www.w3.org/TR/html5/sections.html#outlines)

[W3C, HTML 5, Working Draft, 2008-01-22](https://www.w3.org/TR/2008/WD-html5-20080122)

* [3.8.11. Headings and sections](https://www.w3.org/TR/2008/WD-html5-20080122#headings)
* [3.8.11.1 Creating an outline](https://www.w3.org/TR/2008/WD-html5-20080122#outlines)

W3C, World Wide Web Consortium

* [W3C, homepage](https://www.w3.org)
* [W3C, github account](https://github.com/w3c)
* [W3C, Markup Validation Service](https://validator.w3.org)
* [W3C, Nu Html Checker](https://validator.w3.org/nu)

HTML Specification

* [W3C, HTML on GitHub](https://github.com/w3c/html)
* [W3C, HTML Editor's Draft](https://w3c.github.io/html) (2017-05-03)
* [W3C, HTML Recommendation](https://www.w3.org/TR/html5) (2014-10-28)

Outline Algorithm

* [W3C, HTML Editor's Draft, Outline Algorithm](https://w3c.github.io/html/sections.html#creating-an-outline)
* [W3C, HTML Recommendation, Outline Algorithm](https://www.w3.org/TR/html5/sections.html#outlines)

## License

This repository contains content derived from

* [HTML, Editor's Draft, 2017-05-03](https://w3c.github.io/html)
* [Permissive Document License](https://www.w3.org/Consortium/Legal/2015/copyright-software-and-document)
* Copyright © 2017 W3C® (MIT, ERCIM, Keio, Beihang)
