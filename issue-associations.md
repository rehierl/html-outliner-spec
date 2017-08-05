
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

*acronyms*

1. heading content element (HC)
1. sectioning content element (SC)
1. sectioning root element (SR)
1. sectioning element (SE) - a SC or SR element

<!-- ####################################################################### -->
<h2 id="algorithm">From the algorithm's steps</h2>

<!-- ####################################################################### -->
<h3 id="node-outline">node.innerOutline -
Associate node X with outline Y</h3>

As stated before, that part of the algorithm is rather straight forward. The
definition states to create "a new outline for the new current outline owner"
element when entering an element of sectioning content
([step 4.4.5.](./outliner-steps.md/#4-4-5)) or when entering a sectioning root
element ([step 4.6.5.](./outliner-steps.md/#4-6-5)).

For the sake of simplicity, I assume that this two-way 1:N relationship can be
implemented by the following two properties:

- `Outline Element.innerOutline`
- `Element Outline.outlineOwner`
- `(element == element.innerOutline.outlineOwner)`

NOTE - An outline is only defined for sectioning elements.

One aspect worth mentioning is that, if there was only an Element type, the DOM
node tree would hold many elements with an undefined `innerOutline` property
because the majority of elements does not have a definition for such a property.

If an official term like 'sectioning element' would exist that allowed to refer
to the group/class of elements for which an inner outline is defined, it could
be used for the name of a class that inherits from the `Element` type and which
would define this property:

- `Outline SectioningElement.innerOutline`
- `SectioningElement Outline.outlineOwner`
- `(sectioningElement == sectioningElement.innerOutline.outlineOwner)`

<!-- ####################################################################### -->
<h3 id="section-starting-node">section.startingNode -
Create section X for node Y</h3>

[step 4.4.3](./outliner-steps.md/#4-4-3) states: Let current section be a newly
created section for the current outline owner element.

This can be translated into the following pseudocode:

- `currentSection = new Section(currentOutlineOwner)`

This can be understood as initializing the following property:

- `Element Section.startingNode`

NOTE - The exact same kind of connection will be established when executing
[step 4.6.4.](./outliner-steps.md/#4-6-4),
[step 4.9.2.3](./outliner-steps.md/#4-9-2-3) and
[step 4.9.3.2.1.1](./outliner-steps.md/#4-9-3-2-1-1).
That is `startingNode` will always be either a sectioning element, or an
element of heading content.

I do not read this property to state that `startingNode` must always be an inner
element of the property's section object. The referenced node can be an inner
element, but it does not always have to be one - i.e. the name `startingNode` is
not entirely accurate.

**TODO** - In addition to that, I assume that a sectioning element will never be
considered an inner element of any of its inner sections.

The main characteristic of these kind of connections seems to be that the section
object is created when reaching the given node. Therefore, this connection could
be described as: "This element triggered the creation of that section object"
or "This section starts with, or just after the starting tag of the given element".

NOTE - I do not consider this connection to define a two-way relationship; i.e. I
do not implement an `Element` property that implements the other direction (from
a node object towards a section object) and that represents the exact same meaning.

To understand why this property has its use, it will help to recall the paragraph
related to interactive tables of contents (TOCs). In order to support those it
must be possible to forward the user to the top of the section the user selected
by clicking the heading that is associated with it - i.e.
`heading.section.startingNode`.

**TODO** - is this connection absolutely necessary? -
is there any other way to jump to the beginning of a section -
like `section.firstInnerNode`? -
`firstInnerNode` could be a non-element node. -
is it possible to jump to a non-element node?

<!-- ####################################################################### -->
<h3 id="section-parent-outline">section.parentOutline -
Associate outline X with section Y</h3>

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

NOTE - This does not say anything about what value the `parentOutline` properties
will have for sections that are subsections of the section objects inside
`innerSections`. Currently, I prefer to leave `parentOutline` initialized to a
`null` reference as this allows to determine if a section object represents a
top-level section within an outline.

<!-- ####################################################################### -->
<h3 id="section-parent-section">section.parentSection -
Associate section X with section Y</h3>

[step 4.5.4.](./outliner-steps.md/#4-5-4) states: Append the outline of the
sectioning content element being exited to the current section.

Strictly implemented, this statement would throw an `InvalidCall` Error because
there is no definition for adding an outline object to a section object. This can
only be understood as adding the top-level inner sections of said outline to the
current section.

[step 4.9.3.2.1.1](./outliner-steps.md/#4-9-3-2-1-1) states: create a new section,
and append it to candidate section.

This can be translated into the following pseudocode:

- `section = new Section(node)`
- `section.addSubSection(section)`

The last expression can be understood to define the following two properties:

- `Section Section.parentSection`
- `Section[] Section.subSections`
- `(section == section.subSections[anyIndex].parentSection`

NOTE - The `parentSection` property will remain initialized to a `null` reference
for section objects that are added as inner sections of an outline (see
[innerSections](#section-parent-outline)). The only exception to that rule is
when the inner sections of an outline are added to the outline of an ancestor
sectioning element (step 4.5.4).

<!-- ####################################################################### -->
<h3 id="node-parent-section">node.parentSection -
Associate node X with section Y</h3>

<!-- ======================================================================= -->
<h4 id="type-2">Type 2 - When entering a sectioning content (SC) element</h4>

[step 4.4.4.](./outliner-steps.md/#4-4-4) states: Associate current outline owner
with current section.

At that point, `current outline owner` is the sectioning content element being
entered and `current section` is a new section object created for that element.

This new section represents the first section in the outline of this sectioning
content element. In short, this connection **points inwards** from the sectioning
content element towards its first inner section.

The algorithm's definition does not state the purpose of this type of association,
it is therefore unclear if this step is intended to represent the counter part of
"create section for". Because of that, this step might as well refer to an
unspecified, but completely different type of relationship.

I can only assume that the intended association is meant to be the same as in
[type 3](#type-3) and [type 6](#type-6). If that is the case, then this statement
is **bugged** as it points into the wrong direction.

**TODO** - how to fix it

<h5 id="type-3-current-section">current section variable</h5>

When entering a SC element, [step 4.4.3](./outliner-steps.md/#4-4-3) changes
the `current section` variable to reference the first inner section of the SC
element.

When exiting a SC element, [step 4.5.3](./outliner-steps.md/#4-5-3) changes
the `current section` variable to reference the last section in the outline of
`current outline owner`. This will be the section to which the top-level inner
sections of the SC element being exited will be added.

NOTE - With regards to [type 5](#type-5), both cases work as intended.

**TODO** - what is the meaning of `current section` after exiting a SC?

<!-- ======================================================================= -->
<h4 id="type-3">Type 3 - When entering a sectioning root (SR) element</h4>

[step 4.6.3.](./outliner-steps.md/#4-6-3) states: Let current outline owner's
parent section be current section.

At that point, `current outline owner` is the sectioning root element being entered
and `current section` is the section object located outside and in front of that
element.

This connection **points outwards** from the sectioning root element being entered
towards one of the inner (sub)section of the node's first ancestor sectioning
element. This direction clearly distinguishes it from [type 2](#type-2) - that is
unless type-2 is not bugged, the following is true: **(type-3 != type-2) ???**

NOTE - After creating a new section, i.e. the first inner section, the algorithm
does not have a statement that is similar to [step 4.4.4.](./outliner-steps.md/#4-4-4).

The direction of this connection and its name (i.e. `parent section`) can be
understood to state that the sectioning root element itself is an inner element
of that outer section and that it defines the following two properties:

- `Section Node.parentSection`
- `Node[] Section.innerNodes`
- `(section == section.innerNodes[anyIndex].parentSection)`

NOTE - It still applies that the outline of a sectioning root element must not
contribute to the outline of its first outer sectioning element.

NOTE - In this particular case, the `Node.parentSection` property is also used
as a means to save and restore a reference to the `current section` before
overwriting this variable with a reference to the first inner section (see
[step 4.7.2](./outliner-steps.md/#4-7-2)). The side effect of this method is
that it turns **write-access** to the nodes in the dom subtree into a mandatory
requirement for the outline algorithm. It is possible to completely avoid any
write-access by including the `current section` when saving the current context
(stack operations).

<h5 id="type-3-current-section">current section variable</h5>

When entering a SR element, the `current section` variable will be changed to
reference the first inner section of the SR element.

When exiting a SR element, the `current section` variable will be reset to the
same reference that it had before entering the SR element - i.e. the last section
that is located in front of the SR element.

NOTE - With regards to [type 5](#type-5), both cases work as intended. That is,
a SR element does not contribute to the outline of its ancestor SE.

<!-- ======================================================================= -->
<h4 id="type-4">Type 4 - When entering a heading element</h4>

[step 4.9.2.3](./outliner-steps.md/#4-9-2-3) and
[step 4.9.3.2.1.1](./outliner-steps.md/#4-9-3-2-1-1) state: "create a new section"

Although the algorithm's definition does not (yet) use the exact same words as
in [step 4.4.3](./outliner-steps.md/#4-4-3) (i.e. "create a section *for*") when
entering a heading content element, both steps can be understood to establish
a [type 1](#type-1) connection - i.e. **(type-4 == type-1)**

**TODO** - Heading elements are inner nodes of the implied sections they create,
or of the first inner section within a sectioning element. This is intended ...

<h5 id="type-4-current-section">current section variable</h5>

When entering the first heading element inside some sectioning element, this
heading will be used as the heading of the first inner section of the sectioning
element. The `current section` variable will remain unchanged - i.e. it keeps
referencing that first inner section.

When entering additional heading elements, new implied sections will be created
and the `current section` variable will be changed to reference these new implied
sections.

When exiting a heading element, the `current section` variable will remain
unchanged - i.e. it will keep referencing the section of the heading element
being exited.

NOTE - With regards to [type 5](#type-5), all cases work as intended. That is,
other nodes will be associated with the section of the last heading element.

<!-- ======================================================================= -->
<h4 id="type-5">Type 5) Associate node X with 'current section'</h4>

[step 4.11.](./outliner-steps.md/#4-11) states:
Whenever the walk exits a node, if the node is not already associated with a
section, associate the node with `current section`.

NOTE - Because this step uses the description "a node", it applies to **any nodes**
(element nodes and non-element nodes alike) for which the previous steps didn't
create an association.

NOTE - The previous steps apply only to element nodes that extend the outlines of
their ancestors (sectioning content elements or heading element) or that create
new outlines and sections (sectioning roots). When entering or exiting any other
element node, the references to `current section`, `currentOutlineOwner` and the
current outline (i.e. `currentOutlineOwner.innerOutline`) stay the same throughout
the visit (enter and exit) of those remaining nodes.

Recall that "associate node X with section Y" can be read as defining a two-way
relationship defined by the following two properties: `Section Node.parentSection`
and `Node[] Section.innerNodes`.

Because `parentSection` represents only a single reference, this step could be
executed either when entering, or when exiting a node. With regards to `innerNodes`,
this step turns out to be **bugged**, if the association is established when
exiting the remaining nodes.

The reason for this is, that adding a node to the `innerNodes` array will append
it to the end of that array. The sequence of nodes that this array represents
will not correspond to the document's structure, if nodes are added when exiting
them (reversed order). So with regards to an `innerNodes` list/array, this step
must be executed when entering the remaining nodes.

<h5 id="type-5-current-section">What does 'current section' represent?</h5>

In order to understand the meaning of this step, it is necessary to know what the
value of the `current section` variable represents when this step is supposed to
be executed.

NOTE - Recall, that each node in the DOM tree will be visited in tree order. This
means that a node's parent element will be entered before its inner child nodes.
Likewise, parent elements will be exited after their child nodes have been
processed. In addition to that, preceeding siblings will have been visited before
a node is entered.

The relevant parts that change the 'current section' variable are:

1. When entering a sectioning content (SC) element -
   changed to the SC's first inner section
2. When exiting a SC element -
   changed to the last section of the outline of the SC's
   parent sectioning element (SE)

2 - **TODO** - is that another bug ?!?

<!-- ======================================================================= -->
<h4 id="type-6">Type 6) Associate non-element node X with parent.section</h4>

[step 5.](./outliner-steps.md/#5) states:
Associate all non-element nodes that are in the subtree for which an outline is
being created with the section with which their parent element is associated.

```
case 1:                  case 2:
=======                  =======

<body>                   <body>
  hello world!             <section>
<body>                       hello world!
                           </section>
                         </body>
```

**case 1**
The `<body>` element is the topmost element of a document and, as such, can not
have an outer section object associated with it. Because of that, this step can
not associate the non-element `#text` node "hello world!" with any section,
although it must be associated with `<body>`'s first inner section.

**case 2**
The parent element of `#text` node is the `<section>` element. Because of that,
this step will associate the `#text` node with the `<section>` element's parent
section, i.e. a section object that is located outside of the `<section>` element.
Same issue here: It must be instead associated with `<section>`'s first inner
section.

NOTE - If all sectioning elements are associated with some outer section object,
then this step is **bugged** with regards to both cases.

NOTE - With the current association of sectioning content elements (inwards) and
sectioning root elements (outwards), case-1 will produce a **bug** and case-2
turns out to be a non-issue.

NOTE - If [step 4.11.](./outliner-steps.md/#4-11) is intended to not be limited
to element nodes only, then a separate step, i.e. this step, is not needed.

NOTE - The DOM Node type defines a `Node.parentElement` property which makes it
unnecessary to clarify if a non-element node can have a non-element node as its
parent - that is as far as the `Node.parentElement` property is used instead of
`Node.parentNode`.

<!-- ####################################################################### -->
<h3 id="type-5">section.heading -
Associate section X with heading Y</h3>

NOTE - if node's section has a heading, the following must be true:
`(node.parentSection === node.parentSection.heading.parentSection)`

NOTE - after executing this step, the following must be true:
`(node.parentSection === node.heading.parentSection)`

<!-- ####################################################################### -->
<h3 id="node-heading">node.heading -
Associate node X with heading Y</h3>

[step 6.](./outliner-steps.md/#6) states: Associate all nodes in the subtree with
the heading of the section with which they are associated, if any.

Having this step as an explicit step inside the algorithm's specification turns
it into a mandatory step unless it is explicitly declared to be optional.

NOTE - I don't see a real need for this step as you could always use the
following expression: `node.parentSection.heading`
