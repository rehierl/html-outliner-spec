
This document provides an overview of the objects the algorithm creates and the
object properties it will use.

* [Association notes](./issue-associations-notes.md)

Notes

* The current state of this document assumes that sectioning content and
  sectioning root elements can be treated the same. The basis for this is that
  both groups of elements are defined to have an outline (with one or more inner
  sections).
* For sectioning content elements (`section` elements in particular), it could
  turn out to make more sense to always have a single root section (i.e. a
  `section` element corresponds with a single section object).
* If sectioning root elements could also have a single root section, then a
  distinction is superfluous.
* Instead of a `Outline Section.parentOutline` property you would get a
  `SectioningElement Section.outlineOwner` property and thus could get rid of the
  `Outline class`.
* The other way would then have to be implemented by an
  `Section SectioningElement.innerSection` property.
* The heading of the body's root section would then become the heading of the
  document (a.k.a. the document's title).

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
never hold a null reference.

A sectioning element is not considered to be an inner node of one of its inner
sections. Doing so would associate such an element with a subordinate entity
(e.g. the first inner section) and make it impossible to clearly define the
`Node.parentSection` property.

In contrary to that, an element of heading content is always considered to be an
inner node of the section to which it is assigned as heading. A heading element
is always an inner node of an explicit, and the first inner node of an implied
section.

**TODO** - is there a reason to redefine heading elements similar to sectioning
elements? - shift towards a container element with an open end?

As a result, the expression `section.startingElement.parentSection` is not
guaranteed to yield the same section reference. The property's name is therefore
not entirely accurate because **a section does not have to begin with its
`startingElement`**. This property should be understood to state that the section
starts with, or immediately after the referenced element's starting tag.

This property needs to be used when jumping to the beginning of a section if the
corresponding table of contents entry is selected. It is required because the
first inner node of a section may be a non-element text node:

Firefox defines a `Element.scrollIntoView()` method that can be used to jump to
an element, but there does not seem to be a similar method that allows to jump
to a non-element node. As a consequence, there is no guarantee that the expression
`section.firstInnerNode.scrollIntoView()` will work.

As a result, this property is necessary because it is currently not possible to
jump to a non-element node.

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
have a single **root section** because of the definition of an
[outline](https://w3c.github.io/html/sections.html#outline)
(these may have more than one top-level section). -
a sectioning element could still be considered to correspond with a root section. -
affects how one has to look at sectioning elements
(element-to-section relationship, a 1:1 or a 1:N) -
if a root section would exist, all top-level sections would have to be subsections
of that root section. -
could this point of view be fleshed out? -
if a text node is the first node of a sectioning element, it should be possible
to associate it with the root section. -
such a root section can't have a heading -
at least not a heading that is defined 'the old way' ...

<h3 id="section-parent-section">
Section.parentSection, Section.subSections</h3>

The `parentSection` property will be set if a section is added as subsection to
another section. It must hold a null reference if a section is not a subsection.

The `parentSection` property corresponds with the `subSections` property such that
the expression `section.subSections[anyIndex].parentSection` yields the same section
reference. These lists are therefore limited to direct subsections only.

Note that a section can be added as subsection to a section of a different outline.
This will be the case for sections of an inner sectioning content element.

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
inner sectioning elements not included).

The `heading` property corresponds with the `Node.parentSection` property such
that the expression `section.heading.parentSection` yields the same section
reference. This just clarifies that a heading is an inner node of its section.

Note that a heading element, if one exists, does not always have to be the first
inner node of a section. The first inner section of a sectioning element (explicit
section) may begin with a non-element text node. Implied/implicit sections on the
other hand will always begin with a heading element.

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

DOM's `Node` type, and not the `Element` type, needs to define the `parentSection`
property because sectioning elements themselves are considered to be inner nodes
of an outer parent section. If a text node is a direct child of a sectioning
element, and if DOM's `Element` type would define the `parentSection` property,
then the expression `textNode.parentElement.parentSection` would have to be used
in order to try to get the node's parent section. But as `parentElement` is a
sectioning element, the `parentElement.parentSection` reference will refer to a
section that is located outside of the parent sectioning element. This would
contradict that a text node that is located inside a sectioning element must be
associated with an inner section of that element.

<h3 id="section-inner-nodes">
Node.parentSection, Section.innerNodes</h3>

The `parentSection` property corresponds with the `Section.innerNodes` property
such that the expression `section.innerNodes[anyIndex].parentSection` yields the
same section reference. The `innerNodes` property could be used to modify a
document (e.g. for normalization purposes).

The `innerNodes` list by itself does not require that it contains all the nodes
associated with a section. This list could be limited to the top-level nodes of
a section.

A node is considered to be a **top-level node of a section** if it has no parent
node within that section. A node is considered to be an **inner node of a
section** if it is a top-level node, or if it is a descendant of such a node
(with no top-level node of a different section in between).

**TODO** - Defining a method to meaningfully fill this list is an open issue: It
has little to no use to add all the nodes (element and non-element nodes) of a
section because the node hierarchy will be lost (flat list). It should however be
possible to use the `currentOutlineOwner` reference to limit which nodes are
added - e.g. if a nodes is associated with a section, add it to the list if, and
only if, the `node.parentNode` property matches the `currentOutlineOwner` reference.
This would essentially limit this list to the top-level nodes of a section. This,
on the other hand, would not add any inner sectioning or heading elements, if they
are hidden inside some other element - e.g. a `div` container.

<h3 id="node-heading">
Node.heading</h3>

The `Node.heading` property can be seen as an optional shortcut for
`node.parentSection.heading`, which must yield the same heading reference.

The `heading` property will be a self-reference if `node` is a reference of an
heading element - i.e. `node.heading == node`.

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
