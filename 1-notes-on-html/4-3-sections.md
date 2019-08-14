
[4.3. Sections](https://w3c.github.io/html/sections.html#sections)

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
* note - does not say 'rank(h1)==1', just 'rank(h1)==MAX_RANK'

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
