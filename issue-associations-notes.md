
This document is based upon:

* [Outliner steps](./outliner-steps.md)
* [Outliner pseudocode](./pseudocode.md)

One of the problems the outline algorithm currently has is that it does not
clearly define what you will get as a result. The general idea should be that it
will create outline and section objects, connect them with each other and attach
them to the nodes of the dom subtree for which the algorithm was started.

With regards to attaching outline objects to nodes, the algorithm is rather
straight forward. Unfortunately, the relationships established between nodes
and sections are rather vague and contradictory.

The goal of this document is to identify which relationships the algorithm
establishes and to clearly define those.

*acronyms*

- heading content element (HC)
- sectioning content element (SC)
- sectioning root element (SR)
- sectioning element (SE) - a SC or SR element

<!-- ####################################################################### -->
<h2 id="node-outline">node.innerOutline -
Associate node X with outline Y</h2>

[step 4.4.5.](./outliner-steps.md/#4-4-5) states: Let there be a new outline for
the new current outline owner, ...

This can be translated into the following pseudocode:

- `outline = new Outline(element)`

Which can be understood as defining the following two properties:

- `Outline Element.innerOutline`
- `Element Outline.outlineOwner`
- `(element == element.innerOutline.outlineOwner)`

NOTE - The exact same kind of connection will be established when executing
[step 4.6.5.](./outliner-steps.md/#4-6-5).

NOTE - `outlineOwner` will always be a sectioning element because new outline
objects will only be created when entering such an element.

One aspect worth mentioning is that, if there is only an Element type, the DOM
node tree will have many elements with an undefined `innerOutline` property
because the majority of elements does not have a definition for such a property.

If an official term, e.g. 'sectioning element', would exist that allowed to refer
to the group/class of elements for which an inner outline is defined, it could
be used for the name of a class that inherits from the `Element` type and which
would allow to define these properties as follows:

- `Outline SectioningElement.innerOutline`
- `SectioningElement Outline.outlineOwner`
- `(sectioningElement == sectioningElement.innerOutline.outlineOwner)`

The disadvantage of the first definition is that any element will have an
`innerOutline` property, although its value will be a null reference as most
elements are no sectioning elements and, as such, have no definition for an
inner outline.

<!-- ####################################################################### -->
<h2 id="section-starting-node">section.startingElement -
Create section X (for node Y)</h2>

<!-- ======================================================================= -->
<h3 id="create-section-for">"create a new section for"</h3>

When entering a sectioning content element,
[step 4.4.3](./outliner-steps.md/#4-4-3) states: Let current section be a newly
created section for the current outline owner element.

This can be translated into the following pseudocode:

- `currentSection = new Section(currentOutlineOwner)`

Which can be understood as initializing the following property:

- `Element Section.startingElement`

NOTE - The exact same kind of connection will be established when entering a
sectioning root element ([step 4.6.4.](./outliner-steps.md/#4-6-4)).

NOTE - `startingElement` will always be a sectioning element, or an element of
heading content because new section objects will only be created when entering
these kind of elements.

I do not read this property to state that `startingElement` must always be an inner
element of the property's section object. The referenced element can be an inner
element, but it does not always have to be one - i.e. the name `startingElement`
is not entirely accurate as it implies that the given node is the first node of
a section and, as such, an inner node of that section.

In addition to that, I assume that a sectioning element is never considered to
be an inner element of any of its inner sections. The spec does not have a clear
statement that supports this point of view, but a `parentSection` property (see
below) implies that these elements are inner elements of their parent sections.

**TODO** - There needs to be a clear statement.

The main characteristic of these kind of connections seems to be that the section
object is created when reaching the given element. Therefore, this connection could
be described as: "This element triggered the creation of that section object"
or "This section starts with, or just after the starting tag of the given element".

NOTE - I do not consider this connection to define a two-way relationship; i.e.
I do not implement an `Element` property that implements the other direction
(from an element towards a section object) which represents the same relationship.

To understand why this property has its use, it helps to recall the paragraph
related to interactive tables of contents (TOCs). In order to support those, it
must be possible to forward the user to the beginning of a section if a user
selected the corresponding entry - i.e. `heading.section.startingElement`.

**TODO** - is this connection absolutely necessary? -
is there any other way to jump to the beginning of a section -
like `section.firstInnerNode`? -
`firstInnerNode` could be a non-element node. -
is it possible to jump to a non-element node?

<!-- ======================================================================= -->
<h3 id="create-section">"create a new section"</h3>

When entering a heading content element,
[step 4.9.2.3](./outliner-steps.md/#4-9-2-3) states: "create a new section"

This can be translated into the following pseudocode:

- `section = new Section(heading)`

Although the algorithm's definition does not (yet) use the exact same words as
in [step 4.4.3](./outliner-steps.md/#4-4-3) (i.e. "create a section *for*"),
this step can be understood to establish the exact same connection.

NOTE - The same applies when executing
[step 4.9.3.2.1.1](./outliner-steps.md/#4-9-3-2-1-1).

NOTE - Heading elements are inner nodes of the implied sections they create,
or of the first inner section within a sectioning element.

**TODO** - Why not start a new section with each heading element? -
how to avoid auto-creating a new section when entering a sectioning element?

<!-- ####################################################################### -->
<h2 id="section-parent-outline">section.parentOutline -
Associate outline X with section Y</h2>

[step 4.4.5.](./outliner-steps.md/#4-4-5) states: Let there be a new outline for
the new current outline owner, initialized with just the new current section as
the only section in the outline.

This can be translated into the following pseudocode:

- `owner.innerOutline = new Outline(owner)`
- `owner.innerOutline.addInnerSection(currentSection)`

The last expression can be understood to define the following two properties:

- `Outline Section.parentOutline`
- `Section[] Outline.innerSections`
- `(outline == outline.innerSections[anyIndex].parentOutline)`

NOTE - The exact same kind of connection will be established when executing
[step 4.6.5](./outliner-steps.md/#4-6-5) and
[step 4.9.2.3](./outliner-steps.md/#4-9-2-3).

This does not say anything about what value the `parentOutline` property will
have for subsections that are not added to an `innerSections` list.

I prefer to leave `parentOutline` initialized to a `null` reference as this
allows to easily determine if a section object represents a top-level section
within an outline. Top-level sections may still have a non-null `parentSection`
property if `parentOutline.outlineOwner` is an inner sectioning content element.

<!-- ####################################################################### -->
<h2 id="section-parent-section">section.parentSection -
Associate section X with section Y</h2>

[step 4.5.4.](./outliner-steps.md/#4-5-4) states: Append the outline of the
sectioning content element being exited to the current section.

Strictly implemented, this statement would throw a `TypeError` exception because
it is not possible to add an outline object to the `innerSections` list. This
can only be understood to add the top-level sections of said outline as
subsections to the current section.

[step 4.9.3.2.1.1](./outliner-steps.md/#4-9-3-2-1-1) states:
create a new section, and append it to candidate section.

This can be translated into the following pseudocode:

- `section = new Section(node)`
- `parent.addSubSection(section)`

The last expression can be understood to define the following two properties:

- `Section Section.parentSection`
- `Section[] Section.subSections`
- `(section == section.subSections[anyIndex].parentSection`

The `parentSection` property will remain initialized to a `null` reference for
section objects that are added as inner sections of an outline The only exception
to that statement is when the inner sections of an outline are added to the
outline of an ancestor sectioning element in step 4.5.4.

<!-- ####################################################################### -->
<h2 id="node-parent-section">node.parentSection -
Associate node X with section Y</h2>

This chapter only describes associations between sections and inner nodes.

<!-- ======================================================================= -->
<h3 id="type-1">Type 1 - When entering a sectioning content (SC) element</h3>

[step 4.4.4.](./outliner-steps.md/#4-4-4) states: Associate current outline owner
with current section.

At that point, `current outline owner` is the sectioning content element being
entered and `current section` is a new section object created for that element.

This new section represents the first section in the outline of this sectioning
content element. In short, this connection **points inwards** from the sectioning
content element towards its first inner section, which turns the sectioning
content element into an inner node of that section.

The algorithm's definition does not state the purpose of this type-of association,
it is therefore unclear if this step is intended to represent the counter part of
"create section for".

I can only assume that the intended association is meant to be the same as in
[type-2](#type-2). If that is the case, then this statement is **bugged** as it
points into the wrong direction.

**TODO** - how to fix it? - depends on the following definition "the heading of
a sectioning content element is..." 

<!-- ======================================================================= -->
<h3 id="type-2">Type 2 - When entering a sectioning root (SR) element</h3>

[step 4.6.3.](./outliner-steps.md/#4-6-3) states:
Let current outline owner's parent section be current section.

At that point, `current outline owner` is the sectioning root element being entered
and `current section` represents the preceeding section located outside of that
element.

This connection **points outwards** from the sectioning root element being entered
towards one of the inner (sub)sections of the element's first ancestor sectioning
element. Unless [type-1](#type-1) is not bugged, this direction clearly
distinguishes it from a type-1 connection.

NOTE - After creating a new section, i.e. the first inner section, the algorithm
does not have an "associate with" statement that is similar to
[step 4.4.4.](./outliner-steps.md/#4-4-4). I can only assume that the meaning is
the supposed to be the same.

The direction of this connection and its name (i.e. `parent section`) can be
understood to state that the sectioning root element itself is an inner element
of the preceeding outer section.

Which can be understood to define the following two properties:

- `Section Node.parentSection`
- `Node[] Section.innerNodes`
- `(section == section.innerNodes[anyIndex].parentSection)`

**TODO** - "why the `Node` type and not the `Element type`"? -
reason is non-element nodes need to be associated as well -
`node.parentElement.parentSection` will not always work -
e.g. the first inner node of a sectioning element is a #text node.

NOTE - The `parentSection` property allows to implement interactive TOCs -
you scroll down within a large document and parts of the document's TOC get
folded and unfolded depending on which nodes are visible inside the browser's
window.

NOTE - The `innerNodes` property allows to normalize documents by automatically
injecting `<section>` elements if needed. **normalized** would then mean: one
section element corresponds with a single section object.

NOTE - The `innerNodes` list property itself does not state that any node within
a section must be added to that list. The number of nodes associated with a
certain section (`Node.parentSection`) could be larger than the number of nodes
within the section's `innerNodes` list. Still, the `parentSection` property should
be set for all the nodes within a section.

**TODO** - Is there any plausible reason to add all the nodes and not just the
immediate descendant element and non-element nodes of a sectioning element?

NOTE - It still applies that the outline of a sectioning root element must not
contribute to the outline of its first outer sectioning element.

NOTE - In this particular case, the `Node.parentSection` property is also used
as a means to save and restore a reference to the `current section` before
overwriting this variable with a reference to the first inner section (see
[step 4.7.2](./outliner-steps.md/#4-7-2)). The side effect of this method is
that it turns **write-access** to the nodes in the DOM subtree into a mandatory
requirement for the outline algorithm. It is possible to completely avoid any
write-access by including the `current section` when saving the current context
(stack operations).

<!-- ======================================================================= -->
<h3 id="type-3">Type 3 - When entering a heading content (HC) element</h3>

None of the steps inside [section 4.9.](./outliner-steps.md/#4-9) does explicitly
associate heading elements with sections with regards to [type-2](#type-2). I can
therefore only assume that [type-4](#type-4) applies.

**TODO** - there need to be explicit statements - explain why

<!-- ======================================================================= -->
<h3 id="type-4">Type 4 - Associate node X with 'current section'</h3>

[step 4.11.](./outliner-steps.md/#4-11) states: Whenever the walk exits a node,
if the node is not already associated with a section, associate the node with
`current section`.

NOTE - Because this step uses the unspecific description "a node", it can be
understood to apply to **any node** (element nodes and non-element nodes alike).

NOTE - It seems that, with regards to heading content elements, the substeps in
[step 4.9.](./outliner-steps.md/#4-9) need to be executed before executing this
step. To make things more clear, the spec **should** explicitly associate headings
with sections with regards to [type-2](#type-2) because this step must be executed
when entering the remaining nodes (see below).

NOTE - When entering or exiting any of the remaining nodes, the references to
`current section` and `currentOutlineOwner` stay the same throughout the visit
(enter and exit) of those remaining nodes.

<!-- ----------------------------------------------------------------------- -->
<h4 id="type-4-when-exiting">"when exiting" is "problematic"</h4>

Recall that "associate node X with section Y" can be read as defining a two-way
relationship defined by the following two properties: `Section Node.parentSection`
and `Node[] Section.innerNodes`.

Because `parentSection` represents only a single reference, this step could be
executed either when entering, or when exiting a node. With regards to `innerNodes`,
this step turns out to be **problematic**, if the association is established when
exiting the remaining nodes (reversed order).

Example: An inner #text node of a heading element will be added to the
`innerNodes` list before the heading element itself.

Recall that each node in the DOM tree will be visited in tree order. This means
that a node's parent element will be entered before the node itself. Likewise,
parent elements will be exited after their child nodes have been exited. In
addition to that, preceeding siblings and all their inner child nodes will have
been visited before a node is entered.

With that in mind, it seems as if a `Section.innerNodes` property was not taken
into consideration at the time the algorithm was written.

<!-- ----------------------------------------------------------------------- -->

**TODO** - `innerNodes` is a flat list -
limit to the top-level nodes of a section? -
the latter might exclude inner sectioning element and even heading elements if
they are buried inside `div` containers.

................ (not quite there yet)

If a heading's inner #text node is associated with the heading's section, then
that #text node will appear as if it were a sibling node of the heading element.
So with that in mind, this step is **bugged**. In contrary to the other bugs,
this can not be fixed easily.

**TODO** - "associate node with section" will result in a flat list of nodes -
what is the intended use? - would it be better to only associate direct child
nodes?

when adding a node to the list, check if it is a descendant of the last node that
was added to the list - can this checking be simplified (tree traversal)?

NOTE - This will only apply for any nodes that are inner child nodes of a heading
element - i.e. the #text nodes that define the title of a heading element.

<!-- ----------------------------------------------------------------------- -->
<h4 id="type-4-current-section">What does 'current section' represent?</h4>

In order to understand the meaning of this step, it is necessary to understand
what the value of the `current section` variable in a given context represents
when this step is supposed to be executed.

The relevant parts that change the 'current section' variable are:

<!-- ----------------------------------------------------------------------- -->
<h4 id="type-4-current-section-sc">enter/exit sectioning content</h4>

When entering a SC element, [step 4.4.3](./outliner-steps.md/#4-4-3) changes
the `current section` variable to reference the first inner section of the SC
element.

When exiting a SC element, [step 4.5.3](./outliner-steps.md/#4-5-3) changes
the `current section` variable to reference the last section in the outline of
`current outline owner`.

This will be the section to which the top-level inner
sections of the SC element being exited will be added.

NOTE - With regards to [type-5](#type-5), both cases work as intended.

**TODO** - what is the meaning of `current section` after exiting a SC?

<!-- ----------------------------------------------------------------------- -->
<h4 id="type-4-current-section-sr">enter/exit sectioning root</h4>

When entering a SR element, the `current section` variable will be changed to
reference the first inner section of the SR element.

When exiting a SR element, the `current section` variable will be reset to the
same reference that it had just before entering the SR element - i.e. the last
section that is located in front of the SR element.

NOTE - With regards to the current step, both cases work as intended. That is,
a SR element does not contribute to the outline of its ancestor sectioning element.

<!-- ----------------------------------------------------------------------- -->
<h4 id="type-4-current-section-hc">enter/exit heading content</h4>

When entering the first heading element inside some sectioning element, this
heading will be used as the heading of the sectioning element's first inner
section. The `current section` variable will remain unchanged - i.e. it keeps
referencing that first inner section.

When entering additional heading elements, new implied sections will be created
and the `current section` variable will be changed to reference these new implied
sections.

NOTE - Both of these cases need to taken into consideration because heading
elements do hold inner non-element #text nodes that define a heading's title.

When exiting a heading element, the `current section` variable will remain
unchanged - i.e. it will keep referencing the section of the heading element
being exited.

NOTE - With regards to the current step, this last case will work as intended.
That is, other nodes will be associated with the section of the last heading
element.

<!-- ======================================================================= -->
<h3 id="type-5">Type 5 - Associate non-element node X with parent.section</h3>

[step 5.](./outliner-steps.md/#5) states: Associate all non-element nodes that
are in the subtree for which an outline is being created with the section with
which their parent element is associated.

Assuming that all sectioning elements are associated with an outer section,
take a look at the following two cases:

```
case-1:                  case-2:
=======                  =======
<body>                   <body>
  "hello world!"           <section>
<body>                       "hello world!"
                           </section>
                         </body>
```

**case 1**
The `<body>` element is the topmost element of a document and, as such, can not
have an outer section object associated with it. Because of that, this step can
not associate the non-element `#text` node "hello world!" with any section,
although it must be associated with `<body>`'s first inner section.

**case 2**
The parent element of the `#text` node "hello world!" is the `<section>` element.
Because of that, this step will associate the `#text` node with the `<section>`
element's parent section, i.e. a section object that is located outside of the
`<section>` element. Same issue here: The `#text` node must be associated with
the `<section>` element's first inner section.

NOTE - If all sectioning elements are associated with some outer section object,
then this step is **bugged** with regards to both cases.

NOTE - With the current association of sectioning content elements (inwards) and
sectioning root elements (outwards), case-1 will produce a bug and case-2 turns
out to be working as intended.

NOTE - If [step 4.11.](./outliner-steps.md/#4-11) is intended to not be limited
to element nodes only, then a separate step, i.e. this step, is uncalled-for.

**TODO** - is there any reason to keep this step?

NOTE - The DOM `Node` type defines a `Node.parentElement` property which makes
it unnecessary to clarify if a non-element node can have a non-element node as
its parent - that is as far as the `Node.parentElement` property is used instead
of `Node.parentNode` to implement the "parent element" part of this step.

<!-- ####################################################################### -->
<h2 id="section-heading">section.heading -
Associate section X with heading Y</h2>

[step 4.9.1](./outliner-steps.md/#4-9-1) states:
Let the element being entered be the heading for the current section.

Which can be understood to define the following property:

- `Element Section.heading`
- `(node.parentSection == node.parentSection.heading.parentSection)`

NOTE - The exact same kind of connection will be established when executing
[step 4.9.2.5](./outliner-steps.md/#4-9-2-5) and
[step 4.9.3.2.1.3](./outliner-steps.md/#4.9.3.2.1.3).

Heading elements must be considered to be inner nodes of the sections they are
assigned to as "heading for that section". That is because the first heading
inside a sectioning element will be used as the heading for the first section of
its parent sectioning element, regardless of any preceeding content (sectioning
content elements and heading elements excluded). I consider this to also apply
to heading elements that create new implied sections - Either all in, or all out.

NOTE - A section may end up with no heading associated with it. This will be the
case if a sectioning element contains not even a single heading element - it may
still have heading elements inside inner sectioning elements.

**TODO** - why not treat heading elements as another category of sectioning
elements? - this would make it necessary to start a new section with each heading -
they would have to be considered as "sectioning content elements with an open
end" - this would actually reflect the way they are used -
heading elements would then no longer be inner nodes of the sections they create -
problem: heading elements are themselves containers (inner #text nodes)

**TODO** - `<body><h1><div>A<h2>B</div></body>` -
`A` will be associated with `h1` and `B` with `h2` -
this will tear the `div` container apart

<!-- ####################################################################### -->
<h2 id="node-heading">node.heading -
Associate node X with heading Y</h2>

[step 6.](./outliner-steps.md/#6) states: Associate all nodes in the subtree
with the heading of the section with which they are associated, if any.

Which can be understood to define the following property:

- `Element Node.heading`
- `(node.parentSection == node.heading.parentSection)`

**TODO** - What would be the use of a `Node.heading` property?

Having this step as an explicit step inside the algorithm's specification turns
it into a mandatory step unless it is explicitly declared to be optional. This
step should probably be marked to be an optional one.

**TODO** - Couldn't you always use `node.parentSection.heading` instead?

NOTE - This property is in direct conflict with the definition of a heading for
an element of sectioning content. In essence, there are two definitions for a
heading of a sectioning content element: This one refers to an outer heading
element (superordinate) and the other one (definition of sectioning content
element) to an inner heading element (subordinate).

<!-- ####################################################################### -->
<h2 id="conclusion">Conclusion</h2>

**TODO** - properties, the next level -
look into the traversability of outline and section objects -
it is possible to cross the boundaries -
`section.parentOutline.outlineOwner.parentSection`