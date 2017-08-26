
This document provides an overview of the objects the algorithm creates and the
object properties it will use.

* [Association notes](./issue-associations-notes.md)

<!-- ####################################################################### -->
<!-- ####################################################################### -->
<hr /><h2 id="overview">Overview</h2>

In general, the algorithm creates outline and section objects, connects them
with each other and attaches those to the DOM subtree for which the algorithm
was started. These objects and their properties can be described as follows.

Depending on the goal of an implementation, **certain properties can be
considered to be optional**. The object and property definitions below describe
what an implementation will need if it intends to support all the features of
the algorithm.

The intention of array types like `Section[]` is to represent general **lists of
objects of a specific type**. These kind of properties could also be implemented
as linked lists instead of using actual array types.

<!-- ####################################################################### -->
<!-- ####################################################################### -->
<hr /><h2 id="outline-class">Outline class</h2>

```
Begin Class Outline
  SectioningElement outlineOwner
  Section[] innerSections
End Class
```

<h3 id="outline-owner">
Outline.outlineOwner</h3>

The `outlineOwner` property will be set if a new outline object is created
for a **sectioning element**, which is either an element of sectioning content
or a sectioning root element. It will never hold a null reference.

**TODO** - Define `Outline.parentOutline` and `Outline.innerOutlines` properties?

<!-- ####################################################################### -->
<!-- ####################################################################### -->
<hr /><h2 id="section-class">Section class</h2>

```
Begin Class Section
  Element startingElement
  Outline parentOutline
  Section parentSection
  Section[] subSections
  Element heading
  Node[] innerNodes
End Class
```

<h3 id="section-starting-element">
Section.startingElement</h3>

The `startingElement` property will be set if a new section object is created
when entering a sectioning element, or an element of heading content. It will
never hold a null reference. This property is needed in order to allow to jump
to the beginning of a section if its corresponding table of contents entry is
selected.

The property's name is not entirely accurate because **a section does not have to
begin with its `startingElement`**. This property should be understood to state that
the section starts with, or immediately after the referenced element's starting tag.

A sectioning element is not considered to be an inner node of one of its inner
sections. In contrary to that, an element of heading content is always considered
to be an inner node of the section to which it is assigned as heading.

This property is necessary because the first inner node of a section could be a
non-element text node. - Firefox defines a `Element.scrollIntoView()` method that
can be used to jump to an element, but there does not seem to be a similar method
definition for the Node type.

<h3 id="section-parent-outline">
Section.parentOutline, Outline.innerSections</h3>

The `parentOutline` property will be set if a section is added to an outline. It
must hold a null reference if a section is not a top-level section of an outline.
The `parentOutline` property corresponds with the `Outline.innerSections` property
such that the expression `outline.innerSections[anyIndex].parentOutline` yields
the same outline reference. These lists are therefore limited to all top-level
sections of an outline.

A section is considered to be a **top-level section of an outline** if it has no
parent section within that outline. A section is considered to be an **inner
section of an outline** if it is a top-level section, or if it is a descendant of
such a section (with no top-level section of a different outline in between).

**TODO** - An outline object, or a sectioning element for that matter, can not
have a single **root section** because it may have more than one top-level
sections. A sectioning element could still be considered to correspond with such
a root section. -
affects how one has to look at sectioning elements (element/outline-to-section
relationship, a 1:1 or a 1:N) -
if a root section would exist, all top-level sections would have to be subsections
of that root section. -
could this point of view be fleshed out? -
if a text node is the first node of a sectioning element, it should be possible
to associate it with the root section. -
such a root section can't have a heading ...

<h3 id="section-parent-section">
Section.parentSection, Section.subSections</h3>

The `parentSection` property will be set if a section is added as subsection to
another section. It must hold a null reference if a section is not a subsection
of another section. The `parentSection` property corresponds with the `subSections`
property such that the expression `section.subSections[anyIndex].parentSection`
yields the same section reference. These lists are therefore limited to direct
subsections only.

<h3 id="section-traversal">
Section.parentOutline, Section.parentSection</h3>

When traversing a structure created by the algorithm, it must be possible to
determine if continuing to traverse it outwards, in the direction of the body
element, would result in leaving the outline of the current sectioning element,
or if a root sectioning element was reached. The latter can be the case for the
starting sectioning element, which was used to start the algorithm, or an inner
sectioning root element. The following conditions allow to distinguish these cases:

- If `(parentOutline == null) && (parentSection == null)` is true, the section
has just been initialized. This condition indicates an error if it is still true
after the algorithm has finished.
- If `(parentOutline == null) && (parentSection != null)` is true, the section
is an inner subsection. To continue traversing the structure without leaving the
current outline, the `parentSection` property can be used.
- If `(parentOutline != null) && (parentSection != null)` is true, the section
is a top-level section of an inner sectioning content element. If the `parentSection`
property is used, the current (inner) outline will be exited and the parent (outer)
outline will be entered.
- If `(parentOutline != null) && (parentSection == null)` is true, the section
is a top-level section of a root sectioning element. This element can be the
starting sectioning element, or an inner sectioning root element. In order to
traverse into the parent (outer) outline, the expression
`currentSection.parentOutline.outlineOwner.parentSection` can be used. If, in that
expression, the `parentSection` property holds a null reference, then the
`outlineOwner` is the starting sectioning element.

**TODO** - Consider subsections to be inner sections (a parent section contains
all its subsections), or do sections have to be considered to be separate entities
(comes after) that have some definition of "order"? Could a parent section
"continue" after one of its subsections (Tower of Hanoi, a wood stack game)?

<h3 id="section-heading">
Section.heading, Node.parentSection</h3>

The `heading` property holds a reference of the heading element that represents
the section's heading. It may hold a null reference, which will be the case when
a sectioning element does not contain a single heading element (headings within
inner sectioning elements not included). The `heading` property corresponds with
the `Node.parentSection` property such that the expression
`section.heading.parentSection` yields the same section reference.

The algorithm needs to distinguish the following three states:
(1) The algorithm has created a section but did not yet encounter its heading -
(2) The element reference, the `heading` property holds, represents the section's
heading - (3) The algorithm has finished processing the section and has
determined that it has no heading. - In order to distinguish state 1 from state
3, the algorithm will associate an **implied heading** (a pseudo heading) with
each section that has no heading. Once the algorithm has finished, each `heading`
property will either hold a valid element reference, or it will represent an
implied heading.

<!-- ####################################################################### -->
<!-- ####################################################################### -->
<hr /><h2 id="node-type">Node type</h2>

```
Begin Class Node
  Section parentSection
  // Element heading
End Class Node
```

<h3 id="node-parentSection">
Node.parentSection</h3>

The `parentSection` property connects a DOM node with its parent section. It must
be set whenever the algorithm associates a node with a section. In such a case,
the referencing node is considered to be an inner node of the referenced section.
This property can be used to provide a "current path" (e.g. breadcrumbs) depending
on which nodes are visible inside a browser's window.

**The starting sectioning element has no parent section**. In order to associate
it with one, a `startingElement` would be required that would have to be located
outside of the current subtree. The starting element, and any nodes located outside
of the current subtree, are therefore the only nodes whose `parentSection` property
will hold a null reference. Because of that, these nodes can not be considered to
be inner nodes of any section.

DOM's `Node` type, and not the `Element` type, needs a `parentSection` property
because, in general, sectioning elements themselves are considered to be inner
nodes of an outer parent section. If a text node is a direct child of a sectioning
element, and if only DOM's `Element` type had a `parentSection` property, then
`textNode.parentNode.parentSection` will be a section located outside of its
parent sectioning element. This would contradict that a text node inside a
sectioning element must be associated with an inner section of that element.

<h3 id="section-inner-nodes">
Node.parentSection, Section.innerNodes</h3>

The `parentSection` property corresponds with the `Section.innerNodes` property
such that the expression `section.innerNodes[anyIndex].parentSection` yields the
same section reference. This property could be used to automatically inject
`<section>` elements into documents if such elements are needed (e.g. for
normalization purposes).

The `innerNodes` list by itself does not require that it contains all the nodes
associated with a section. This list could therefore be limited to the top-level
nodes of a section.

A node is considered to be a **top-level node of a section** if it has no parent
node within that section. A node is considered to be an **inner node of a
section** if it is a top-level node, or if it is a descendant of such a node
(with no top-level node of a different section in between).

**TODO** - Defining a method to meaningfully fill this list is an open issue: It
has little to no use to add all the nodes (element and non-element ones) of a
section because the node hierarchy will be lost (flat list). It should however be
possible to use the `currentOutlineOwner` reference to limit which nodes are
added - e.g. add a node to the list if, and only if, the `node.parentNode`
property matches the `currentOutlineOwner` reference. This would essentially limit
this list to the top-level nodes of a section. This, on the other hand, would not
add any inner sectioning or heading elements, if they are hidden inside some other
element - e.g. a `div` container.

<h3 id="node-heading">
Node.heading</h3>

The `heading` property (`Node.heading`) can be seen as an optional shortcut for
`node.parentSection.heading`, which must yield the same heading reference. Note
that the `heading` property will be a self-reference if `node` is a reference to
a heading element - i.e. `node.heading == node`.

<!-- ####################################################################### -->
<!-- ####################################################################### -->
<hr /><h2 id="sectioning-element-type">SectioningElement type</h2>

```
Begin Class SectioningElement
  Outline innerOutline
End Class SectioningElement
```

The `SectioningElement` object type can be seen to inherit DOM's `Element` type.
It extends the `Element` type by the `innerOutline` property and could be used to
implement the algorithm itself. This allows to not associate invalid references
with element nodes for which there is no definition for an inner outline. As
long as there is no such DOM `SectioningElement` type, the existing `Element`
type must be extended.

<h3 id="element-inner-outline">
SectioningElement.innerOutline</h3>

The `innerOutline` property corresponds with the `Outline.outlineOwner` property
such that the expression `sectioningElement.innerOutline.outlineOwner` yields
the same element reference.

Like the `Node.parentSection` property, the `innerOutline` property allows to
enter the outline/section tree created by the algorithm.
