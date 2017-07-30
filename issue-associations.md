
This document is based upon:

* [Outliner steps](./outliner-steps.md)
* [Outliner pseudocode](./pseudocode.md)

One of the problems the outline algorithm currently has is that it does not
clearly define what you will get as a result if you decide to run it. The general
idea should be that it will create outline and section objects, connect them with
each other and attach them to the nodes of the dom subtree for which the algorithm
was started.

When it comes to attaching outline objects to nodes, the algorithm is rather
straight forward. Unfortunately, the relationships established between nodes
and sections are rather vague and contradictory.

<!-- ####################################################################### -->
<h2 id="node-outline">Associate node X with outline Y</h2>

As stated before that part of the algorithm is rather straight forward. The
definition states to create "a new outline for the new current outline owner"
element when entering an element of sectioning content
([step 4.4.5.](./outliner-steps.md/#4-4-5)) or a sectioning root element
([step 4.6.5.](./outliner-steps.md/#4-6-5)).

For the sake of simplicity, I assume that this two-way relationship can be
implemented by the following two properties: `Outline Element.innerOutline` and
`Element Outline.outlineOwner`.

One aspect worth mentioning is that, if there was only an Element type, the
node tree would hold many elements with an undefined `innerOutline` property
because the majority of elements does not have a definition for such a property.

NOTE - An outline is only defined for elements of sectioning content and
sectioning root elements.

If a term would exist (e.g. **sectioning element**) that allowed to refer to the
group/class of elements for which an inner outline is defined, it could be used
for the name of a class that inherits from the `Element` type and which would
define an `innerOutline` property. This new class would allow to refine the above
two properties into: `Outline SectioningElement.innerOutline` and
`SectioningElement Outline.outlineOwner`.

<!-- ####################################################################### -->
<h2 id="node-section">Associate node X with section Y</h2>

The following sections are related to steps that establish a connection between
nodes and section objects.

<!-- ======================================================================= -->
<h3 id="type-1">Type 1) When creating a new section object</h3>

The first time a section object will be created is when executing
[step 4.4.3](./outliner-steps.md/#4-4-3). I would translate this statement into
`currentSection = new Section(currentOutlineOwner)`.

The exact same kind of connection will be established when executing
[step 4.6.4.](./outliner-steps.md/#4-6-4) and the steps mentioned in
[type 4](#type-4).

NOTE - `startingNode` will always be either a sectioning root element, an element
of sectioning content or a heading element.

For the moment, I read those statements as defining a `Element Section.startingNode`
property. I do not read this property to state that `startingNode` must always
be an inner element of the property's section object - this node can be an inner
element, but it does not always have to be one - i.e. the name `startingNode`
isn't overly accurate. In addition to that, I assume that a sectioning element
will never be an inner element of any of its inner sections.

The main characteristic of these kind of connections seems to be that the section
object is created when reaching the given node. Therefore, this connection could
be described as: "This element triggered the creation of that section container"
or "This section starts with or right after the starting tag of the given element".

To understand why this connection has its use, it will help to recall the
paragraph related to creating an interactive table of contents (TOC). In order
to support interactive TOCs, it must be possible to forward the user to the top
of the section the user selected by clicking the heading that is associated with
it - i.e. `heading.section.startingNode`. Here, `heading.section` means to get a
reference of the section in which the heading is located.

**TODO** - is it possible to jump to a non-element node?

NOTE - I don't require this connection to define a two-way relationship; i.e. I
don't implement an `Element` property that implements the other direction (from
a node object towards a section object) but still has the exact same meaning.

<!-- ======================================================================= -->
<h3 id="type-2">Type 2) When entering a sectioning content element</h3>

In [step 4.4.4.](./outliner-steps.md/#4-4-4), the definition states to
associate `current outline owner` with `current section`.

At that point, `current outline owner` is the sectioning content element being
entered and  `current section` is a new section object created for that element.
This new section will be used as the first section in the outline of the
sectioning content element being entered. In short, this connection
**points inwards** from the sectioning content element towards its first inner
section.

The algorithm's definition does not state the purpose of this type of association,
hence it is unclear if this step is meant to represent the counter part of
"create section for". Because of that, this step might as well refer to an
unspecified, but completely different type of relationship.

I can only assume that the actual intention behind this association is meant to
be the same as in [type 3](#type-3) and [type 6](#type-6). If that is the case,
then this statement is **bugged** as it points into the wrong direction.

<!-- ======================================================================= -->
<h3 id="type-3">Type 3) When entering a sectioning root element</h3>

Before creating a new section object, [step 4.6.3.](./outliner-steps.md/#4-6-3)
states to "Let current outline owner's parent section be current section".

At that point, `current outline owner` is the sectioning root element being entered
and `current section` is the section object outside and in front of that element.

This connection **points outwards** from the sectioning root element being entered
towards an inner section of the node's its first outer sectioning element. This
direction clearly distinguishes it from [type 2](#type-2) - that is unless type-2
isn't bugged, the following is true: **(type-3 != type-2)**

NOTE - After creating a new section, i.e. the first inner section of the element
being entered, the algorithm does not have a statement that is similar to
[step 4.4.4.](./outliner-steps.md/#4-4-4).

The direction of this connection and its name (i.e. `parent section`) can be
understood to state that the sectioning root element itself is an inner element
of that outer section and that it defines a `Section Node.parentSection` property.
The meaning of this property allows to define a `Node[] Section.innerNodes`
property that represents the other direction and thus turns this connection into
defining a two-way relationship.

NOTE - It still applies that the outline of a sectioning root element must not
contribute to the outlines of its ancestor sectioning element.

NOTE - In this particular case, the `Node.parentSection` property is also used
as a means to save and restore a reference to the `current section` before
overwriting this variable with a reference to the first inner section (see
[step 4.7.2](./outliner-steps.md/#4-7-2)). The side effect of this save-and-restore
method is that it turns write-access to the nodes in the dom subtree into a
mandatory requirement for the outline algorithm.

<!-- ======================================================================= -->
<h3 id="type-4">Type 4) When entering a heading content element</h3>

Although the algorithm's definition does not (yet) use the exact same wording as
in [step 4.4.3](./outliner-steps.md/#4-4-3) (i.e. "create a section *for*") when
entering a heading content element, [step 4.9.2.3](./outliner-steps.md/#4-9-2-3)
and [step 4.9.3.2.1.1](./outliner-steps.md/#4-9-3-2-1-1) can be seen to establish
the same type of connection; i.e. **(type-4 == type-1)**

NOTE - In contrary to type-1, additional heading elements must also be inner nodes
to the implied sections they create because these heading element represent the
headings of their sections.

<!-- ======================================================================= -->
<h3 id="type-5">Type 5) Associate node X with 'current section'</h3>

[step 4.11.](./outliner-steps.md/#4-11) states:
Whenever the walk exits a node, if the node is not already associated with a
section, associate the node with `current section`.

NOTE - Because this step uses the description "a node", it applies to **any nodes**
(element nodes and non-element nodes) for which the previous steps didn't create
an association.

NOTE - The previous steps apply only to element nodes that extend the outlines of
their ancestors (sectioning content elements or heading element) or that create
new outlines and sections (sectioning roots). When entering or exiting any other
element node, the references to `current section`, `currentOutlineOwner` and the
current outline (i.e. `currentOutlineOwner.innerOutline`) stay the same throughout
the visit (enter and exit) of the remaining nodes.

NOTE - Recall that "associate node X with section Y" can be read as defining
a two-way relationship defined by the following two properties:
`Section Node.parentSection` and `Node[] Section.innerNodes`. Because
`parentSection` represents only a single reference, this step could be executed
either when entering, or when exiting a node. With regards to `innerNodes`, this
step turns out to be **bugged**, if the association is established when exiting
those nodes. The reason for this is that adding a node to the `innerNodes` array
will append it to the end of that array. The node sequence that this array
represents will not correspond to the document's structure if nodes are added
when exiting them (reversed order). So with regards to a `innerNodes` list/array,
this step must be executed while entering these remaining nodes.

<h4 id="type-5-sections">What is 'current section'?</h4>

<!-- ======================================================================= -->
<h3 id="type-6">Type 6) Associate non-element node X with parent.section</h3>

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
<h2 id="node-heading">Associate node X with heading Y</h2>

[step 6.](./outliner-steps.md/#6) states:
Associate all nodes in the subtree with the heading of the section with which
they are associated, if any.

Having this step as an explicit step inside the algorithm's specification
turns it into a mandatory step unless it is explicitly marked as "otpional".

NOTE - I don't see a real need for this step as you could always use the
following expression: `node.parentSection.heading`

NOTE - if node's section has a heading, the following must be true:
`(node.parentSection === node.parentSection.heading.parentSection)`

NOTE - after executing this step, the following must be true:
`(node.parentSection === node.heading.parentSection)`
