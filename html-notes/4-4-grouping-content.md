
[4.4. Grouping content](https://w3c.github.io/html/grouping-content.html#grouping-content)

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
