
A closer look at HTML's definitions related to the outline algorithm:

* [outliner chapter 4.3.9.](./outliner-4.3.9.md)

## heading content

[Heading content](https://w3c.github.io/html/dom.html#heading-content) -
Heading content defines the header of a section (whether explicitly marked up
using sectioning content elements, or implied by the heading content itself) --
=> h1, h2, h3, h4, h5, h6.

* defines the header of a section
* can be explicitly marked up using sectioning content elements
* may be the header of implied sections

**TODO** - heading content elements may also be marked up using sectioning root
elements - in general *sectioning elements*

[Rank of a heading](https://w3c.github.io/html/sections.html#rank) -
These elements have a rank given by the number in their name. The h1 element
has the highest rank, the h6 element has the lowest rank, and two elements with
the same name have equal rank. -- Use the rank of heading elements to create the
document outline.

* no absolute values - only in relationship with each other - h1 has highest rank

## sectioning content

[Sectioning content](https://w3c.github.io/html/dom.html#sectioning-content) -
Sectioning content is content that defines the scope of headings (=> heading
content) and footers (=> footer element). Each sectioning content element
potentially has a heading and an outline. See the section on headings and
sections for further details.
=> article, aside, nav, section.

* first - the "sectioning content" content model "defines the scope of" - which
  can be understood to refer to any element that has that characteristic -
  then - limited to only a couple of those elements
* potentially has a heading and an outline

**TODO** - no definition of what that heading is, or what it is used for

[Header of](https://w3c.github.io/html/sections.html#headings-and-sections) -
The first element of heading content in an element of sectioning content represents
the heading for that explicit section. Subsequent headings of equal or higher rank
start new implied subsections that are part of the previous section’s parent section.
Subsequent headings of lower rank start new implied subsections that are part of
the previous one. In both cases, the element represents the heading of the implied
section.

* the first sentence must also include to sectioning roots - i.e. in general,
  sectioning elements - because they are treated the exact same way
* the first section in a sectioning element is considered to be an explicit
  section - all other sections are implicit/implied ones
* "equal or higher rank ... new implied *subsections*" - could also be a top-level
  section if the heading's rank is eoh than the rank of the last top-level section
* "parent section" - more general an ancestor section

*heading of a sectioning content element*

* the intention behind this paragraph is to explain how section *objects* are
  created and what their headings are
* it can *not* be seen as a definition of the heading of a sectioning *element*

[Subsections of](https://w3c.github.io/html/sections.html#headings-and-sections) -
Sectioning content elements are always considered subsections of their nearest
ancestor sectioning root or their nearest ancestor element of sectioning content,
whichever is nearest, regardless of what implied sections other headings may have
created.

* translates to "some element is a subsection of its first outer sectioning element"
* regardless of any implied sections - ???

## sectioning roots

[Sectioning roots](https://w3c.github.io/html/sections.html#sectioning-roots) -
Certain elements are said to be sectioning roots, including blockquote and td
elements. These elements can have their own outlines, but the sections and
headings inside these elements do not contribute to the outlines of their
ancestors.
=> blockquote, body, details, dialog, fieldset, figure, td.

* like sectioning content element, they have an outline
* do not contribute to an ancestor outline

**TODO** - no definition of a heading of a sectioning root element because their
outline must not contribute to those of their ancestors?

## outline, section

[outline](https://w3c.github.io/html/sections.html#creating-an-outline) -
The outline for a sectioning content element or a sectioning root element consists
of a list of one or more potentially nested sections. The element for which an
outline is created is said to be the outline’s owner.

* one or more (at least one) sections which themselves may be nested

[section](https://w3c.github.io/html/sections.html#creating-an-outline) -
A section is a container that corresponds to some nodes in the original DOM tree.
Each section can have one heading associated with it, and can contain any number
of further nested subsections. The algorithm for the outline also associates each
node in the DOM tree with a particular section and potentially a heading. (The
sections in the outline aren’t section elements, though some may correspond to
such elements — they are merely conceptual sections.)

* may correspond to no nodes at all
* may have a heading associated with it - may have none
* may have further nested subsections
* the algorithm associates nodes with sections and headings (section.heading)
* sections in an outline are objects/concept - not elements

[tree of sections](https://w3c.github.io/html/sections.html#creating-an-outline) -
The tree of sections created by the algorithm above, or a proper subset thereof,
must be used when generating document outlines, for example when generating tables
of contents.

* the outline creates a tree of sections - does not say a single tree
* table of contents may be built from a subtree

## footer element

[footer element](https://w3c.github.io/html/sections.html#the-footer-element) -
A footer element can also contain entire sections representing appendices,
indexes, long colophons, verbose license agreements, and other such content.

**TODO** - Doesn't that turn those elements into sectioning content/root elements?
