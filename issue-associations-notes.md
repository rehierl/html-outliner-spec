
These are the notes I took when I tried to understand what result the algorithm
is supposed to produce.

* `outliner steps > objects and properties`
* [Outliner steps](./outliner-steps.md)
* [Outliner pseudocode](./pseudocode.md)

Warning - These notes were taken from an inconsistent and even bugged spec. As
such, these notes can not solve the issues the spec has, they can merely make
these issues more apparent.

*acronyms*

- heading content element (HC)
- sectioning content element (SC)
- sectioning root element (SR)
- sectioning element (SE) - a SC or SR element

**TODO** - properties, the next level -
traversability of outline and section objects -
it is possible to cross the boundaries -
`section.parentOutline.outlineOwner.parentSection`

<!-- ####################################################################### -->
<!-- ####################################################################### -->
<!-- ####################################################################### -->

<hr />
<h2 id="overview">overview</h2>

One of the problems the outline algorithm currently has is that it does not
clearly define what you will get as a result. The general idea should be that it
will create outline and section objects, connect them with each other and attach
them to the nodes of the DOM subtree.

With regards to attaching outline objects to nodes, the algorithm is rather
straight forward. Unfortunately, the relationships established between nodes
and sections are vague and contradictory.

The goal of this document is to allow to identify which relationships the
algorithm establishes and to clearly define those.

<!-- ####################################################################### -->
<!-- ####################################################################### -->
<!-- ####################################################################### -->

<hr />
<h2 id="definitions">definitions</h2>

[chapter 4.3.9.1](./outliner-steps.md) states:
A **section** is a container that corresponds to some nodes in the original DOM
tree. Each section can have one heading associated with it, and can contain any
number of further nested subsections. The algorithm for the outline also associates
each node in the DOM tree with a particular section and potentially a heading.
(The sections in the outline aren’t section elements, though some may correspond
to such elements — they are merely conceptual sections.)

* `section.nodes`, `node.section`
* `section.heading` - may have no heading
* `section.subsections[]` - may have no nested subsections
* `node.section`, `node.heading` - may have no heading
* `sectionObject != sectionElement`

[chapter 4.3.9.1](./outliner-steps.md) states:
The **outline** for a sectioning content element or a sectioning root element
consists of a list of one or more potentially nested sections. The element for
which an outline is created is said to be the outline’s owner.

* `outline.sections[]` - one or more inner sections
* `outline.outlineOwner` - will always have an owner

By looking at this HTML fragment ... `<body></body>` ... you can easily tell that
an outline can be "empty" (i.e. has no actual inner section). The current algorithm
will, in such a case, create an outline that contains a single "empty" inner
section (i.e. has no nodes associated with it).

In addition to that, taking this HTML fragment ... `<body> </body>` ... will make
the algorithm create a an outline that contains a single inner section, which
itself holds a single non-element text node that represents the inner whitespace.

It would be easier to determine if an outline is empty, if (in such a case) its
`sections[]` list would not contain a single section object (i.e. an empty list).
I therefore do *not* read "consists of a list of one or more" to define that an
outline must (by its definition) always have at least one inner section.

**TODO** - The spec should state that an outline could have no section object
associated with it. It should also state that the algorithm will (currently)
always add a section object which itself could end up having no node associated
with it.

<!-- ####################################################################### -->
<!-- ####################################################################### -->
<!-- ####################################################################### -->

<hr />
<h2 id="node-outline">node.innerOutline</h2>
"Associate node X with outline Y"

[step 4.4.5.](./outliner-steps.md/#4-4-5) states: Let there be a new outline for
the new current outline owner, ...

This can be translated into the following pseudocode:

- `outline = new Outline(element)`

Which can be understood to define the following two properties:

- `Outline Element.innerOutline`
- `Element Outline.outlineOwner`
- `(element.innerOutline.outlineOwner == element)`

NOTE - The exact same kind of connection will be established when executing
[step 4.6.5.](./outliner-steps.md/#4-6-5).

NOTE - `outlineOwner` will always be a sectioning element because new outline
objects will only be created when entering such an element.

The disadvantage of these definitions is that any element will have an
`innerOutline` property. In general, this property will be undefined as most of
the elements are no sectioning elements and, as such, have no definition for an
inner outline.

If an official term, e.g. 'sectioning element', would exist that allowed to refer
to the group of elements for which an inner outline is defined, it could be used
for the name of a class that inherits from the `Element` type. These properties
could then be defined as follows:

- `Outline SectioningElement.innerOutline`
- `SectioningElement Outline.outlineOwner`
- `(sectioningElement.innerOutline.outlineOwner == sectioningElement)`

The difference is that only those nodes, for which a definition of an inner
outline exists, have an `innerOutline` property.

<!-- ####################################################################### -->
<!-- ####################################################################### -->
<!-- ####################################################################### -->

<hr />
<h2 id="section-starting-node">section.startingElement</h2>
"Create section X for node Y"

<!-- ======================================================================= -->
<h3 id="create-section-for">"create a new section for"</h3>

When entering a sectioning content element,
[step 4.4.3](./outliner-steps.md/#4-4-3) states: Let current section be a newly
created section for the current outline owner element.

This can be translated into the following pseudocode:

- `currentSection = new Section(currentOutlineOwner)`

Which can be understood to define the following property:

- `Element Section.startingElement`

NOTE - The exact same kind of connection will be established when entering a
sectioning root element ([step 4.6.4.](./outliner-steps.md/#4-6-4)).

The `startingElement` property will always hold a reference to a sectioning element,
or an element of heading content because new section objects will only be created
when entering such an element.

I do not read this property to state that `startingElement` must be an inner
element of the section object created. The referenced element can be an inner
element, but it does not have to be one - i.e. the name `startingElement` is not
entirely accurate as it implies that the given element is the first inner node of
that section.

In addition to that, I assume that a sectioning element is never considered to
be an inner element of any of its inner sections. The specification does not have
a clear statement that supports this point of view, but a `'parent section'`
property (`Node.parentSection` - see below), as it is mentioned when entering a
sectioning root element, implies that these elements are inner elements of their
parent section.

**TODO** - There needs to be a clear statement.

The main characteristic of these kind of connections seems to be that the section
object is created when reaching the given element. This connection could therefore
be described as: "This element triggered the creation of that section object" or
"This section begins with, or just after the starting tag of the referenced
element".

I do not consider this connection to define a two-way relationship - i.e. I do
not implement an `Element` property that implements the other direction (from an
element towards a section object) which refers to the same relationship.

To understand why this property has its use, recall the paragraph related to
interactive tables of contents (TOCs). In order to support those, it must be
possible to scroll a page to the beginning of a section, if the corresponding
entry is selected - e.g. `heading.section.startingElement.scrollIntoView()`.

NOTE - This property is necessary because the first inner node of a section could
be a non-element text node. Firefox does define a `Element.scrollIntoView()`
method which can be used to jump to the beginning of a section. There does not
seem to be a similar method for the `Node` type.

<!-- ======================================================================= -->
<h3 id="create-section">"create a new section"</h3>

When entering a heading content element,
[step 4.9.2.3](./outliner-steps.md/#4-9-2-3) states: "create a new section"

This can be translated into the following pseudocode:

- `section = new Section(heading)`

Although the algorithm's definition does not (yet) use the exact same words as
in [step 4.4.3](./outliner-steps.md/#4-4-3) (i.e. "create a section *for*"),
this step can be understood to define the exact same property.

NOTE - The same applies when executing
[step 4.9.3.2.1.1](./outliner-steps.md/#4-9-3-2-1-1).

Heading elements are considered to be inner nodes of the implied sections they
create, or of the first inner section within a sectioning element.

(see also - [section.heading](#section-heading))

<!-- ####################################################################### -->
<!-- ####################################################################### -->
<!-- ####################################################################### -->

<hr />
<h2 id="section-parent-outline">section.parentOutline</h2>
"Associate outline X with section Y"

[step 4.4.5.](./outliner-steps.md/#4-4-5) states: Let there be a new outline for
the new current outline owner, initialized with just the new current section as
the only section in the outline.

This can be translated into the following pseudocode:

- `owner.innerOutline = new Outline(owner)`
- `owner.innerOutline.addInnerSection(currentSection)`

The last expression can be understood to define the following two properties:

- `Outline Section.parentOutline`
- `Section[] Outline.innerSections`
- `(outline.innerSections[anyIndex].parentOutline == outline)`

NOTE - The exact same kind of connection will be established when executing
[step 4.6.5](./outliner-steps.md/#4-6-5) and
[step 4.9.2.3](./outliner-steps.md/#4-9-2-3).

This does not say anything about what value the `parentOutline` property will
have for sections that are not added to an outline's `innerSections` list. I
prefer to leave `parentOutline` initialized to a `null` reference as this allows
to determine if a section object represents a top-level section of an outline.
With that in mind, the `innerSections` list of an outline will only contain the
top-level sections of an outline object.

A section is said to be **a top-level section of an outline**, if it has no parent
section within that outline. A section is said to be **an inner section of an
outline (or a sectioning element)**, if it is a top-level section of that outline,
or if it is a descendant of such a section (with no top-level section of another
outline in between).

A method could be defined which would only return a `null` reference, if a
section is not an inner section of any outline:

```
function Section.getParentOutline() {
  if(this.parentOutline != null) {
    return this.parentOutline;
  }
  if(this.parentSection != null) {
    return this.parentSection.getParentOutline();
  }
  return null;
}
```

<!-- ####################################################################### -->
<!-- ####################################################################### -->
<!-- ####################################################################### -->

<hr />
<h2 id="section-parent-section">section.parentSection</h2>
"Associate section X with section Y"

<!-- ======================================================================= -->
<h3 id="section-parent-section-hce">
entering a heading content element</h3>

[step 4.9.3.2.1.1](./outliner-steps.md/#4-9-3-2-1-1) states:
create a new section, and append it to candidate section.

This can be translated into the following pseudocode:

- `section = new Section(node)`
- `candidateSection.addSubSection(section)`

The last expression can be understood to define the following two properties:

- `Section Section.parentSection`
- `Section[] Section.subSections`
- `(section.subSections[anyIndex].parentSection == section)`

The `parentSection` property will remain initialized to a `null` reference for
section objects that are added as inner sections of an outline. The only exception
to that statement is (in step 4.5.4) when the top-level sections of an outline
are connected with the outline of an ancestor sectioning element.

<!-- ======================================================================= -->
<h3 id="section-parent-section-sre">
exiting an inner sectioning root element</h3>

The outline of a sectioning root (SR) element must not contribute to the outline
an ancestor sectioning element:

- `(sr.innerOutline.innerSections[anyIndex].parentOutline == sr)`
- `(sr.innerOutline.innerSections[anyIndex].parentSection == null)`

The `parentSection` property of all top-level sections of an outline of a
sectioning root element must not be connected (null reference) with an inner
section of any ancestor sectioning element.

<!-- ======================================================================= -->
<h3 id="section-parent-section-sce">
exiting an inner sectioning content element</h3>

[step 4.5.3](./outliner-steps.md/#4-5-3) states: Let current section be the last
section in the outline of the current outline owner element.

I can only assume that one has to understand "last section" to be the *last
top-level section* added to the outline's `innerSections` list. If it weren't
for the "in the outline" part, one could also understand this to refer the
*last active section* in that outline, which could even be an inner subsection.

**TODO** - Why "last top-level" and not "last active" section? -
A matter of definition? - This needs to be clearly stated.

[step 4.5.4.](./outliner-steps.md/#4-5-4) states: Append the outline of the
sectioning content element being exited to the current section.

Strictly implemented, this statement will throw a `TypeError` because it is not
possible to add an outline object to the `Section.subSections` list. This can only
be understood to add the top-level sections of said outline as subsections to the
current section. With that point of view, this step can be understood to refer to
the same properties as defined above.

**TODO** - This step needs to be rephrased.

[chapter 4.3.9](./outliner-4.3.9.md) states: Sectioning content elements are always
considered subsections of their nearest ancestor sectioning root or their nearest
ancestor element of sectioning content, whichever is nearest, regardless of what
implied sections other headings may have created.

* A sectioning content element is not a section object and, as such, can not be
  a subsection - i.e. a `TypeError`.
* An ancestor sectioning element is not a section object and, as such, can itself
  not have any subsections - i.e. another `TypeError`.
* This paragraph does *not* refer to any section inside the ancestor sectioning
  element, it refers to the element itself - i.e. add element 1 to element 2.

**TODO** - This paragraph needs to be rephrased.

**TODO** - Why "last top-level" and not "add the inner top-level sections to the
outline object" itself? - Issue: A section object can only have a single
`parentOutline` object associated with it.

**Cancelled** - How (and why) exactly the outline of an inner sectioning content
element is "merged into" its first outer sectioning element needs to be revisited
at some later time (see also [inner sce](./issue-inner-sce.md)). Just assume for
the moment that some section exists to which the top-level sections of an inner
sectioning content element are added ...

The outline of a sectioning content (SC) element must (by definition) contribute
to the outline of an ancestor sectioning element:

- `(sc.innerOutline.innerSections[anyIndex].parentOutline == sc)`
- `(sc.innerOutline.innerSections[anyIndex].parentSection != null)`

The `parentSection` property of all top-level sections of an outline of a
sectioning content element must be connected (non-null reference) with an inner
section of its first outer sectioning element.

<!-- ####################################################################### -->
<!-- ####################################################################### -->
<!-- ####################################################################### -->

<hr />
<h2 id="node-parent-section">node.parentSection</h2>
"Associate node X with section Y"

<!-- ======================================================================= -->
<h3 id="type1">Type 1 - When entering a sectioning content element</h3>

[step 4.4.4.](./outliner-steps.md/#4-4-4) states: Associate current outline owner
with current section.

At that point, `currentOutlineOwner` is the sectioning content element being
entered and `currentSection` is a new section object created for that element.

This new section represents the first inner section in the outline of this
sectioning content element. In short, this connection **points inwards**, from
the sectioning content element towards its first inner section. This defines the
sectioning content element to be an inner node of that section.

The algorithm's definition does not state the purpose of this type of association.
It is therefore unclear if this step is meant to represent the counter part of
"create a section for" or not. I can only assume that the intended association
is meant to be the same as in [type-2](#type2). If that is the case, then this
statement is **bugged** as it points into the **wrong direction**.

NOTE - I consider this to be the "wrong direction" because the name `parentSection`
implies that the section object is a superordinate and the sectioning content
element a subordinate entity. Obviously, that is not the case.

From the [definition of sectioning content](https://w3c.github.io/html/dom.html#sectioning-content-2): Each sectioning content element potentially has a heading and an outline.

**TODO** - The only reason I can think of, why the algorithm is supposed to establish
that kind of connection, is to comply with the "the heading of a sectioning content
element is..." definition - See the `node.heading` section below - The association
that needs to be established should match the way the top-level sections of a
sectioning content element will be connected with the sections of the element's
parent outline. - *explain why*

<!-- ======================================================================= -->
<h3 id="type2">Type 2 - When entering a sectioning root element</h3>

[step 4.6.3.](./outliner-steps.md/#4-6-3) states:
Let current outline owner's parent section be current section.

At that point, `currentOutlineOwner` is the sectioning root element being entered
and `currentSection` represents the preceeding section located outside of that
element. Unlike when entering an element of sectioning content, `currentSection`
does *not* refer to the sectioning root's first inner section.

This connection **points outwards** from the sectioning root element being entered
towards one of the inner sections of the element's first ancestor sectioning
element. Unless [type-1](#type1) is not bugged, this direction clearly
distinguishes it from a type-1 connection.

NOTE - After creating a new section, i.e. the first inner section, the algorithm
does not have an "associate with" statement that is similar to
[step 4.4.4.](./outliner-steps.md/#4-4-4). I can therefore only assume that the
meaning is supposed to be the same.

The direction of this connection and its name (i.e. `parent section`) can be
understood to state that the sectioning root element itself is an inner element
of the preceeding outer section (The sectioning root element is a subordinate
and the outer section a superordinate entity).

This can be translated into the following pseudocode:

- `currentOutlineOwner.parentSection = currentSection`

Which can be understood to define the following two properties:

- `Section Node.parentSection`
- `Node[] Section.innerNodes`
- `(section == section.innerNodes[anyIndex].parentSection)`

The `parentSection` property must be defined for the `Node` type, and not the
`Element` type, because the first inner node of a sectioning element could be a
non-element text node. Therefore, the expression `node.parentElement.parentSection`
can not be used to get a reference of the text node's parent section (In case of
a parent sectioning root element, this would be an outer section).

NOTE - The `parentSection` property allows to implement interactive table-of-contents
(TOCs) - You scroll down within a large document and parts of the document's TOC get
folded and unfolded depending on which nodes are visible within the browser's window.

NOTE - The `innerNodes` property could allow to inject `<section>` elements if
they are missing. That way, each document could be changed such that each section
element corresponds with exactly one section object - i.e. **normalized**.

The `innerNodes` list property itself does not state that any node within a section
must be added to that list. The number of nodes associated with a certain section
(i.e. `Node.parentSection`) could be larger than the number of nodes within the
section's `innerNodes` list. Still, the `parentSection` property should be set for
all the nodes within a section.

**TODO** - Is there any plausible reason to add all the nodes and not just the
immediate descendant element and non-element nodes (aka. top-level nodes) of a
sectioning element?

**TODO** - Could use a `numberOfInnerNodes` property?

NOTE - In this particular step, the `Node.parentSection` property is also used
as a means to save and restore a reference to `currentSection` before overwriting
this variable with a reference to the first inner section (compare with
[step 4.7.2](./outliner-steps.md/#4-7-2)). The side effect of this method is
that it turns **write-access** to the nodes in the DOM subtree into a mandatory
requirement for the outline algorithm. **read-only-access** is possible by
including the `currentSection` variable when saving the algorithm's current
context (i.e. stack operations).

<!-- ======================================================================= -->
<h3 id="type3">Type 3 - When entering a heading content element</h3>

None of the steps inside [section 4.9.](./outliner-steps.md/#4-9) does explicitly
associate heading elements with sections with regards to [type-2](#type2). I can
therefore only assume that [type-4](#type4) applies.

**TODO** - There need to be explicit "associate" statements - The reason is that
type-4's "when exiting" should, in the far future, be changed into "when entering".

<!-- ======================================================================= -->
<h3 id="type4">Type 4 - Associate node X with 'current section'</h3>

[step 4.11.](./outliner-steps.md/#4-11) states: Whenever the walk exits a node,
if the node is not already associated with a section, associate the node with
`current section`.

Recall that "associate node X with section Y" can be understood to define a
two-way relationship:

- `Section Node.parentSection`
- `Node[] Section.innerNodes`.

NOTE - Because this step uses the unspecific description "a node", it can be
understood to apply to **any node** (element nodes and non-element nodes alike).

NOTE - It seems that, with regards to heading content elements, the substeps in
[step 4.9.](./outliner-steps.md/#4-9) need to be executed before executing this
step. To make things more clear, the specification **should explicitly associate
headings** ([type-2](#type2)) with sections because this step must be executed
when *entering* the remaining nodes (see below).

<!-- ----------------------------------------------------------------------- -->
<h4 id="type4-current-section">
What does the 'current section' variable stand for?</h4>

In order to understand this step it is necessary to know what the value of the
`currentSection` variable will represent in a given context. The relevant parts
that change this variable are:

(1) <span id="type4-current-section-sc">
enter/exit an element of sectioning content (SC)</span>

When entering a SC element, the `currentSection` variable will be changed to
reference the first inner section of the SC element.

When exiting a SC element, the `currentSection` variable will be changed to
reference the last section in the outline of the `currentOutlineOwner`. This is
the section to which the top-level inner sections of the SC element being exited
will be added (see [step 4.5.4.](./outliner-steps.md/#4-5-4)).

**TODO** - What is the meaning of `currentSection` after exiting a SC? -
Why add the inner top-level sections to that outer section and not to any other
section? (see [section.parentSection](#section-parent-section-sce))

(2) <span id="type4-current-section-sr">
enter/exit an element of sectioning root (SR)</span>

When entering a SR element, the `currentSection` variable will be changed to
reference the first inner section of the SR element.

When exiting a SR element, the `currentSection` variable will be reset to the
same reference that it had just before entering the SR element - i.e. the last
section that is located in front of the SR element.

(3) <span id="type4-current-section-hc">
enter/exit an element of heading content (HC)</span>

When entering the first heading element inside some sectioning element, this
heading will be used as the heading of the sectioning element's first inner
section. The `currentSection` variable will remain unchanged - i.e. it keeps
referencing that first inner section.

When entering any additional heading element, a new implied section will be
created and the `currentSection` variable will be changed to reference this new
implied section.

NOTE - Both of these cases need to taken into consideration because heading
elements are themselves container elements that have inner nodes - e.g. they
need to have at least one inner non-element text node that defines the heading's
title.

When exiting a heading element, the `currentSection` variable will remain
unchanged - i.e. it keeps referencing the section of the heading element that
was exited.

<!-- ----------------------------------------------------------------------- -->
<h4 id="type4-enter-or-exit">"when entering", or "when exiting"? -
warning: extremely messy ...</h4>

(1) <span id="type4-eoe-node-parent-section">
"when exiting" or "when entering" does make a difference</span>

Example: `<body> <h1/> <div> <h2/> </div> </body>`

The goal of "when exiting" seems to be to ensure that the `<div>` container is
associated with `<h2/>`. The **issue** with that is:
A seemingly minor modification will make it fail:

Example: `<body> <h1/> <div> A <h2/> </div> </body>`

If associated "when exiting", the non-element text node `A` will be associated
with `<h1/>` and the `<div>` container with `<h2/>` - this will tear the `div`
container aparat.

For that to consistently work, `<h2/>`'s section needs to end when the `<div>`
container is exited - `<h1/>`'s section would have to be reestablished just
before, or while, the `<div>` container is being exited. **TODO**

The same applies when the `<h2/>` element is replaced with a `<section>` element
because exiting such an element will change the `currentSection` variable.

(2) <span id="type4-eoe-section-inner-nodes">
The 'Node[] Section.innerNodes' property</span>

Ignore for a moment the issues mentioned in (1). What effect does it have when
nodes are associated "when entering" or "when exiting" them? Assume that each
time a node is associated with a section, it is added to the `innerNodes` list.

Example: `<body> <h1> A </h1> B </body>`

Text nodes A and B will all be associated with `<body>`'s first and only inner
section - no issue here. But, you will get the following `innerNodes` list for
that section:

`Section.innerNodes`: `[A, <h1>, B]`

Note that text node B will be associated before its parent heading element
(reversed order). This means that there is no guaranteed characterisitc, with
regards to the section, that the first inner node associated with a section has.

Recall that each node in the DOM tree will be visited in tree order. This means
that a node's parent element will be entered before the node itself. Likewise,
parent elements will be exited after their child nodes have been exited. In
addition to that, preceeding siblings and all their inner child nodes will have
been (completely) visited before a node is even entered.

The way implied sections are created (only when reaching a new heading element)
and when they end (when another heading element is reached, or when the ancestor
sectioning element ends) does not even guarantee that the very last node associated
with a section has such a characteristic. **TODO**

Example: `<body> <h1/> <div> <h2/> </div> A <h3/> </body>`

With that in mind, it seems as if a `Section.innerNodes` property was not taken
into consideration at the time the algorithm was written.

(3) <span id="type4-eoe-when-entering">
"when entering" could be more useful</span>

**TODO** - limit `innerNodes` to the top-level nodes of a section - excludes
elements (could be sectioning element, headings, ...) burried/hidden inside
inner container elements - add iif (node.parentNode == currentOutlineOwner)?

<!-- ======================================================================= -->
<h3 id="type5">Type 5 - Associate non-element node X with parent.section</h3>

[step 5.](./outliner-steps.md/#5) states: Associate all non-element nodes that
are in the subtree for which an outline is being created with the section with
which their parent element is associated.

Assuming that all sectioning elements are already associated with an outer
section, consider the following two cases:

```
case-1:     case-2:        case-3:
=======     =======        =======
<body>      <body>         <body>
  A           <section>      <section>
<body>          B              C
              </section>       <h1>D</h1>
            </body>            E
                               <h1>F</h1>
                               G
                             </section>
                           </body>
```

**case 1** -
The `<body>` element is the topmost element of a document and, as such, can not
have an outer section object associated with it. Because of that, this step can
not associate the non-element text node 'A' with any section. This is bugged
because that text node must be associated with `<body>`'s first inner section.

**case 2** -
The parent element of the text node 'B' is the `<section>` element. Because of
that, this step will associate the text node with the `<section>` element's
parent section, i.e. a section object that is located outside of the `<section>`
element. Same issue here: Text node 'B' must be associated with `<section>`'s
first inner section.

**case 3** -
The parent elements of text nodes 'D' and 'F' are the corresponding heading
elements - no issue here. However, text nodes 'C', 'E' and 'G' will be associated
with some outer section because the `<section>` element is their parent element.

NOTE - If all sectioning elements are associated with some outer section object,
then this step is **bugged** with regards to all three cases.

NOTE - With the current association of sectioning content elements (inwards) and
sectioning root elements (outwards), case-1 is bugged and case-2 works as intended.
However, case-3 is again bugged because text node 'G' will not be associated with
the implied section defined by text node's 'F' parent heading element.

NOTE - If [step 4.11.](./outliner-steps.md/#4-11) is intended to *not* be limited
to element nodes only, then a separate step, i.e. this step, is uncalled-for -
which means that **this step must be removed**.

NOTE - The DOM `Node` type defines a `Node.parentElement` property which makes
it unnecessary to clarify if a non-element node can have a non-element node as
its parent - that is as far as the `Node.parentElement` property is used instead
of `Node.parentNode` to implement the "parent element" part of this step.

<!-- ####################################################################### -->
<!-- ####################################################################### -->
<!-- ####################################################################### -->

<hr />
<h2 id="section-heading">section.heading</h2>
"Associate section X with heading Y"

When entering a heading content element,
[step 4.9.1](./outliner-steps.md/#4-9-1) states:
Let the element being entered be the heading for the current section.

This can be translated into the following pseudocode:

- `currentSection.heading = currentElement`

Which can be understood to define the following property:

- `Element Section.heading`
- `(section.heading.parentSection == section)`

NOTE - The exact same kind of connection will be established when executing
[step 4.9.2.5](./outliner-steps.md/#4-9-2-5) and
[step 4.9.3.2.1.3](./outliner-steps.md/#4.9.3.2.1.3).

Heading elements must be considered to be inner nodes of the sections they are
assigned to as "heading of that section". That is because the first heading
inside a sectioning element will be used as the heading of the sectioning element's
first inner section, regardless of any preceeding content (sectioning content
elements and heading elements excluded).

NOTE - I consider this to also apply to heading elements that create new implied
sections - Either all in, or all out. - Sectioning elements are outer and heading
elements are inner nodes.

NOTE - A section may end up with no heading associated with it. This will be the
case if a sectioning element contains not even a single heading element -
although it might still contain heading elements inside inner sectioning elements.

**TODO** - Why not start a new section with each heading element (no exception)? -
How to avoid auto-creating a new section when entering a sectioning element? -
The current way to implement the algorithm allows for section objects to be
completely empty (have no nodes)!

NOTE - The answer for this TODO is the shift away from HTML4 heading elements.
The first heading element is (for backwards compatibility reasons - and there
is no new heading element) only to define the heading of a section. It is not
intended to mark the beginning of a new subsection!

**TODO** -
Why not treat heading elements as another category of sectioning elements? -
This would make it necessary to start a new section with each heading -
They would have to be considered as "sectioning content elements with an open end" -
This would actually reflect the way they are used -
Heading elements would then no longer be inner nodes of the sections they create -
In such a case, how to associate a heading with a section? -
Heading elements are themselves containers that have inner #text nodes!

<!-- ####################################################################### -->
<!-- ####################################################################### -->
<!-- ####################################################################### -->

<hr />
<h2 id="node-heading">node.heading</h2>
"Associate node X with heading Y"

[step 6.](./outliner-steps.md/#6) states: Associate all nodes in the subtree
with the heading of the section with which they are associated, if any.

This can be translated into the following pseudocode:

- `node.heading = node.parentSection.heading`

Which can be understood to define the following property:

- `Element Node.heading`
- `(node.heading.parentSection == node.parentSection)`

**TODO** - What is the use of a separate `Node.heading` property? -
Couldn't you always use `node.parentSection.heading` instead? -
It probably only makes sense if you don't support a `parentSection` property. -
Which doesn't seem to make sense: If you don't have a `parentSection` property,
then how can you assign a heading to a node? By having a `currentHeading` variable?

**TODO** - Having this step as an explicit step within the specification turns it
into a mandatory one, unless it is explicitly declared to be optional. It should
be marked to be optional.

**This property is in conflict with the definition of a heading for an element
of sectioning content**. In essence, there are two definitions for a `heading`
property that have a different meaning: `Node.heading` refers to an outer/parent
heading element (superordinate) and `SectioningContentElement.heading` (from the
definition of the 'sectioning content element' itself) to an inner heading element
(subordinate).
