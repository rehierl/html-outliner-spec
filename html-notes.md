
These are notes on different aspects of HTML that are related to the outline algorithm.

* `html-specification`
* [W3C, HTML 5.2, Editor's Draft, 3 May 2017](https://w3c.github.io/html)
* [HTML's Elements, a tabular overview](https://w3c.github.io/elements-of-html/)

<hr/>

[**hgroup**]()

* [html5 Doctor, The hgroup element](http://html5doctor.com/the-hgroup-element)
* was a new element defined in html5
* intended to group titles with subtitles
* possibly inside a header element
* the doc outline would have masked all but the highest level heading
* [Steve Faulkner's request for feedback to drop](https://lists.w3.org/Archives/Public/public-html/2013Mar/0026.html), 2013-03-06
* it promotes an anti-pattern, is a mere styling hook, etc.
* [W3C's decision on request to drop hgroup](https://lists.w3.org/Archives/Public/public-html-admin/2013Apr/0003.html), 2013-04-02
* Plan 2014: request to be dropped due to missing complete implementations of the semantics
* got removed on 2013-04-16

[**Categories**](https://w3c.github.io/html/dom.html#categories)

* i.e. the 'categories' property of an element's definition
* a list of categories to which an element belongs
* used when defining the content models of each element

[**Content model**](https://w3c.github.io/html/dom.html#content-model)

* i.e. the 'content model' property of an element's definition
* a normative description of what content must be included as children and
  descendants of an element.

<hr/>
### [2.4. Common microsyntaxes](https://w3c.github.io/html/infrastructure.html#common-microsyntaxes)

[2.4.2. **Boolean attributes**](https://w3c.github.io/html/infrastructure.html#sec-boolean-attributes)

* the presence of a boolean attribute on an element represents the true value,
  and the absence of the attribute represents the false value
* value must either be the empty string, or a value that is an ascii case-insensitive
  match for the attribute's canonical name with no leaing or trailing white space.
* note: checked seems to be allowed and equivalent to checked=""
* note: "true" and "false" as values are not allowed
* to represent a false value, the attribute has to be omitted altogether!
* e.g. &lt;input checked disabled=""&gt; - mixed styles are allowed
* e.g. checked, checked="", checked=checked, checked=CHECKED, ...

<hr/>
### [3.2.4. Content models](https://w3c.github.io/html/dom.html#content-models)

[3.2.4.2.2. **Flow content** (FC)](https://w3c.github.io/html/dom.html#flow-content)

* elements - **dialog, h1-6, section, text, ...**

```
a, abbr, address, area (if descendant of map), article, aside
audio, b, bdi, bdo, blockquote, br, button, canvas, cite, code, data
datalist, del, details, dfn, dialog, div, dl, em, embed, fieldset, figure
footer, form, h1, h2, h3, h4, h5, h6, header, hr, i, iframe, img
input, ins, kbd, label, link (if allowed inside body), main, map, mark
MathML, math, menu, meter, nav, noscript, object, ol, output, p, picutre
pre, progress, q, ruby, s, samp, script, section, select, small, span
strong, style, sub, sup, svg, table, template, textarea, time, u, ul
var, video, wbr, text
```

*notes*

* a mere listing of elements to classify them as 'flow content'
* won't say what the actual meaning of flow content is

[3.2.4.2.3. **Sectioning content** (SC)](https://w3c.github.io/html/dom.html#sectioning-content)

* elements - **article, aside, nav, section**
* content that defines the scope of headings (3.2.4.2.4) and footers (4.3.8)
* each SC element **can have a heading and an outline**
* a sectioning root can also have an outline

*notes*

* content models - **FC** - nav and aside must not contain a main element

[3.2.4.2.4. **Heading content** (HC)](https://w3c.github.io/html/dom.html#heading-content)

* see elements - **h1, h2, h3, h4, h5, h6** (4.3.6)
* defines the header of a section (whether explicitly marked up using
  SCs, or implied by the HC itself)

*notes*

* distinction between HC and HE? concept vs. element?

[3.2.4.2.5. **Phrasing content** (PC)](https://w3c.github.io/html/dom.html#phrasing-content-2)

* elements - **i, s, span, strong, sub, sup, text, ...**

```
a, abbr, area (if descendant of map), audio, b, bdi, bdo,
br, button, canvas, cite, code, data, datalist, del, dfn, em embed, i,
iframe, img, input, ins, kbd, label, link (if allowed in body), map,
mark, math, meter, noscript, object, output, picuture, progress, q,
ruby, s, samp, script, select, small, span, strong, sub, sup, svg,
template, textarea, time, u, var, video, wbr, text
```

* phrasing content is the text (i.e. text nodes or nothing) of the document
* including the elements that mark up that text at the intra-paragraph level
* may contain other PCE, but no FCE

*notes*

* **FCEs that are no PCEs** - address, article, aside, blockquote, details, dialog,
  div, dl, fieldset, figure, footer, form, h1-6, header, hr, main, menu, nav, ol,
  p, pre, section, style, table, ul
* **FCEs, that are no PCEs** - HCEs, SCEs, SREs
  (body is no FC/PC; td excluded via table)

[3.2.4.2.8. **Palpable content** (PalpC)](https://w3c.github.io/html/dom.html#palpable-content-2)

* elements - **h1-6, i, s, section, strong, text, ...**

```
a, abbr, address, article, aside, audio (if controls present), b,
bdi, bdo, blockquote, button, canvas, cite, code, data, details, dfn, div,
dl (if children include one/more name-value group), em, embed,
fieldset, figure, footer, form, h1, h2, h3, h4, h5, h6, header, i,
iframe, img, input (if type-attrib not hidden), ins, kbd, label,
main, map, mark, math, meter, nav, object, ol (if children include one/more li),
output, p, pre, progress, q, ruby, s, samp,
section, select, small, span, strong, sub, sup, svg, table, textarea,
time, u, ul (if chilren include one/more li), var, video,
text (that is not inter-element white space)
```

* in general, elements whose content model allows any FC or PC should have at
  least one node in its contents that is palpable content and that does not have
  the hidden attribute specified.
* makes an element non-empty (i.e. text, audio, video, image, controls, canvas)
* not a hard requirement, e.g. may be temporarily empty

[4.3.9. **sectioning roots** (SR)](https://w3c.github.io/html/sections.html#sectioning-roots)

* elements **blockquote, body, details, dialog, fieldset, figure, td**
* inner sections and headings do not contribute to the outlines of their ancestors
* can have their own outlines

*notes*

* does not say, like for SCEs that they also can have a heading
* content models - **FC** - additional requirements for details, fieldset and
  figure elements

<hr/>
### [4.3. Sections](https://w3c.github.io/html/sections.html#sections)

[4.3.1. The **body** element](https://w3c.github.io/html/sections.html#the-body-element)

* categories - **SR** - content model - **FC**
* represents the content of a document
* only one per document

[4.3.2. The **article** element](https://w3c.github.io/html/sections.html#the-article-element)

* categories - **FC (no main), SC, PalpC** - content model - **FC**
* represents a complete, or self-contained, composition in a document - e.g. a forum post
* in general only appropriate if the contents would be listed explicitly in the outline
* nested inner articles are in principle related to the outer article

[4.3.3. The **section** element](https://w3c.github.io/html/sections.html#the-section-element)

* categories - **FC, SC, PalpC** - content model - **FC**
* represents a generic section - e.g. chapters
* a section, in this context, is a thematic grouping of content
* each section should be identified by including a heading (h1-6) element as a child

[4.3.4. The **nav** element](https://w3c.github.io/html/sections.html#the-nav-element)

* categories - **FC, SC, PalpC** - content model - **FC (no main)**
* represents a section of a page that links to other pages/parts
* so basically a section with navigational links

[4.3.5. The **aside** element](https://w3c.github.io/html/sections.html#the-aside-element)

* categories - **FC, SC, PalpC** - content model - **FC (no main)**
* represents a section of a page that consists of content that is tangentially
  related to the content around the aside element - e.g. remarks, external links
* often represented as sidebars when printed

[4.3.6. The **h1, h2, h3, h4, h5, h6** elements (HE)](https://w3c.github.io/html/sections.html#the-h1-h2-h3-h4-h5-and-h6-elements)

* categories - **FC, HC, PalpC** - content model - **PC** (i.e. no HC/SC/SR)
* represent headings for their sections
* semantics and meaning defined in "Headings and sections"
* have a **rank** - given by the number of their name.
  h1 has highest rank, h6 has lowest rank and
  two elements with the same name have equal rank.
* use the rank of HEs to create the document outline

*notes*

* does not say 'rank(h1)==1', just 'rank(h1)==MAX_RANK'

[4.3.7. The **header** element](https://w3c.github.io/html/sections.html#the-header-element)

* categories - **FC, PalpC** - content model - **FC (no main, header, ...)**
* represents introductory content for its nearest ancestor (i.e. main, SC, SR)
* typically contains introductory or navigational aids
* applies to the whole page if the ancestor is the body element
* intended, but not required, to usually contain the section's heading

[4.3.8. The **footer** element](https://w3c.github.io/html/sections.html#the-footer-element)

* categories - **FC, PalpC** - content model - **FC (no main, header, ...)**
* a footer for its nearest ancestor (main element, sectioning content, sectioning root)
* contains information about its section - e.g. links to related documents
* doesn't have to appear at the end of a section

<hr/>
### [4.4. Grouping content](https://w3c.github.io/html/grouping-content.html#grouping-content)

[4.4.2. The **address** element](https://w3c.github.io/html/grouping-content.html#the-address-element)

* categories - **FC, PalpC** - content model - **FC (no HC, SC, ...)**
* represents contact information for a person, people or organization
* must not be used for arbitrary addresses
* typically included in a footer element
* got moved from 4.3.9

[4.4.5. The **blockquote** element](https://w3c.github.io/html/grouping-content.html#the-blockquote-element)

* categories - **FC, SR, PalpC** - content model - **FC**
* represents content that is quoted from another source
* optionally with a citation, which must be within a footer or cite element
* optionally with in-line changes such as annotations an abbreviations
* if a cite attribute is present, it must be a valid URL;
  otherwise use a cite element

[4.4.12. The **figure** element](https://w3c.github.io/html/grouping-content.html#the-figure-element)

* categories - **FC, SR, PalpC** - content model - **FC (figcaption?)**
* represents some flow content that is self-contained
* typically referenced as a single unit from the main flow of the document

[4.4.14. The **main** element](https://w3c.github.io/html/grouping-content.html#elementdef-main)

* categories - **FC, PalpC** - content model - **FC**
* represents the main content of the body of a document
* has no effect on the document outline
* there must not be more than one visible main element in a document
* must not be used as a descendant of article, aside, footer, header, nav

<hr/>
### [4.9. Tabular data](https://w3c.github.io/html/tabular-data.html#tabular-data)

[4.9.9. The **td** element](https://w3c.github.io/html/tabular-data.html#the-td-element)

* categories - **SR** - content model - **FC**
* represents a data cell in a table
* with its 'colspan, rowspan, headers' attributes takes part in the
  [table model](https://w3c.github.io/html/tabular-data.html#tabular-data-processing-model)

<hr/>
### [4.10. Forms](https://w3c.github.io/html/sec-forms.html#sec-forms)

[4.10.15. The **fieldset** element](https://w3c.github.io/html/sec-forms.html#the-fieldset-element)

* categories - **FC, SR, PalpC, ...** - content model - **(legend?, FC*)**
* represents a set of form controls under an optional common name
* the name of the group is given by the first legend child element
* the disabled attribute allows to disable all controls, except for those
  inside the first legend child element

<hr/>
### [4.11. Interactive elements](https://w3c.github.io/html/interactive-elements.html#interactive-elements)

[4.11.1. The **details** element](https://w3c.github.io/html/interactive-elements.html#the-details-element)

* new since html5.1?
* categoires - **FC, SR, IC, PalpC** - content model - **(summary, FC*)**
* represents a disclosure widget from which the user can obtain additional
  information or controls
* not appropriate for footnotes
* may have a summary child element, which will represent the detail's summary or legend
* may have an open attribute to dynamically show/hide the additional information
  in addition to the summary

[4.11.7. The **dialog** element](https://w3c.github.io/html/interactive-elements.html#the-dialog-element)

* new since html5.2?
* categories - **FC, SR** - content model - **FC**
* represents a part of an application the user interacts with; e.g. dialog box
* the open attribute, if specified, indicates that the dialog element is active

<hr/>
### [5. User interaction](https://w3c.github.io/html/editing.html#editing)

[5.1. The **hidden** attribute](https://w3c.github.io/html/editing.html#the-hidden-attribute)

* all html elements may have the hidden content attribute set
* it is a boolean attribute (see 2.4.2)
* specifies, that an element is not yet, or no longer directly relevant to the
  page's current state, or it is used to declare content to be reused by other
  parts of the page; as opposed to being directly accessed by the user
* user agents should not render such elements
* typically implemented using style sheets (CSS)
* if something is marked hidden, it is hidden from all presentations
  including, for instance, screen readers
* non-hidden elements must not link to hidden ones
