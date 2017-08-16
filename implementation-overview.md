
- Related to W3C`s HTML outline algorithm
- A much needed overview

In general, the algorithm creates outline and section objects, connects them
with each other and attaches those to the DOM subtree for which the algorithm
was started. These objects and their properties can be described as follows.

Note that depending on the goal of an implementation, certain properties are
optional. The object and property definitions below describe what an
implementation will need if it intends to support all the features of the
algorithm.

Note that the intention of array types like `Section[]` is to represent lists of
certain objects. These kind of properties could also be implemented as linked
lists instead of using actual array types.

```
Begin Class Outline
  SectioningElement outlineOwner
  Section[] innerSections
End Class
```

The `outlineOwner` property will be set if a new outline object is created for
a **sectioning element**, which is either an element of sectioning content or a
sectioning root element. It will never hold a null reference.

**TODO** - Define `Outline.parentOutline` and `Outline.innerOutlines` properties?

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

The `startingElement` property will be set if a new section object is created
when entering a sectioning element, or an element of heading content. It will
never hold a null reference. This property is needed in order to allow to jump
to the beginning of a section if its corresponding table of contents entry is
selected.

Note that a sectioning element is not considered to be an inner node of one of
its inner sections. In contrary to that, an element of heading content is always
considered to be an inner node of the section to which it is assigned as heading.

The property's name is not entirely accurate because a section does not have to
begin with its `startingElement`. This property should be understood to state that
the section starts with, or immediately after the referenced element's starting tag.

**TODO** - The first node within a section may be a non-element node, a text node
for example. Is it possible to jump to a non-element node? If the answer is yes,
it would be possible to jump to `section.innerNodes[0]` instead - **issue** -
associate nodes when exiting. Would there still be a need for a `startingElement`
property? Yes, because the `innerNodes` list can be ignored if your only goal is
to produce a table of contents.

The `parentOutline` property will be set if a section is added to an outline. It
must hold a null reference if a section is not a top-level section of an outline.
The `parentOutline` property corresponds with the `Outline.innerSections` property
such that the expression `outline.innerSections[anyIndex].parentOutline` yields
the same outline reference. These lists are therefore limited to all top-level
sections of an outline.

A section is considered to be a **top-level section of an outline** if it has no
parent section, or if its parent section is an inner section of a different
outline. Note that an outline may have multiple top-level sections.

A section is considered to be an **inner section of an outline** if it is a
top-level section, or if it is a descendant of such a section (with no top-level
section of another outline in between).

The `parentSection` property will be set if a section is added as subsection to
another section. It must hold a null reference if a section is not a subsection
of another section. The `parentSection` property corresponds with the `subSections`
property such that the expression `section.subSections[anyIndex].parentSection`
yields the same section reference. These lists are therefore limited to direct
subsections only.

**TODO** - Consider subsections to be inner sections (a parent section contains
all its subsections), or do sections have to be considered to be separate entities
(comes after) that have some definition of "order"? Could a parent section
"continue" after all of its subsections? (wood stack game - Tower of Hanoi)

When traversing a structure created by the algorithm, it must be possible to
determine if continuing to traverse it in the direction of the body element would
result in leaving the outline of a sectioning element, or if a root sectioning
element was reached. The latter can be the case for the starting sectioning
element, which was used to start the algorithm, or an inner sectioning root
element. The following conditions allow to distinguish these cases:

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

The `heading` property holds a reference of the heading element that represents
the section's heading. This property may hold a null reference, which will be the
case when a sectioning element does not contain a single heading element (headings
within inner sectioning elements excluded). The `heading` property corresponds
with `Node.parentSection` property such that the expression
`section.heading.parentSection` yields the same section reference.

Note that the algorithm needs to distinguish the following three states:
(1) The algorithm has created a section but did not yet encounter its heading -
(2) The element reference, the `heading` property holds, represents the section's
heading - (3) The algorithm has finished processing the section and has
determined that it has no heading. - In order to distinguish state 1 from state
3, the algorithm will associate a pseudo heading (implied heading) with each
section that has no heading. Once the algorithm has finished, each `heading`
property will either hold a valid element reference, or it will represent an
implied heading.

```
Begin Class Node
  Section parentSection
  // Element heading
End Class Node
```

The `parentSection` property connects a DOM node with its parent section. If it
holds a non-null value, the node is considered to be an inner node of that section.
This property must be set whenever the algorithm associates a node with a section
and can be used to provide a "current path" (e.g. breadcrumbs) depending on which
nodes are visible inside a browser's window.

Note that the starting sectioning element can not be associated with a parent
section because such a section would require a `startingElement` that is located
outside of the current subtree. Hence, the starting element and any node outside
of the current subtree will be the only nodes whose `parentSection` property will
hold a null reference.

Note that DOM's `Node` type, not the `Element` type, needs a `parentSection`
property because sectioning elements themselves are considered to be inner nodes
of an outer parent section. If a text node is a direct child of a sectioning
element, and if only DOM's `Element` type had a `parentSection` property, then
`textNode.parentNode.parentSection` will be a section located outside of its
parent sectioning element. This would contradict that a text node inside a
sectioning element must be associated with an inner section of that element.

The `Section.innerNodes` property corresponds with the `parentSection` property
such that the expression `section.innerNodes[anyIndex].parentSection` yields the
same section reference. This property could be used to automatically inject
`<section>` elements into documents if such elements are needed (e.g. for
normalization purposes).

Note that the `innerNodes` property itself does not require that it contains all
the nodes associated with a section. This list can be limited to the top-level
nodes of a section.

A node is considered to be a **top-level node of a section** if it has no parent
node within the same section. Note that the starting sectioning element (including
the `<body>` element) can not be associated with a section.

A node is considered to be an **inner node of a section** if it is a top-level
node, or if it is a descendant of such a node (with no top-level node of another
section in between). Note that this does not include any of the inner nodes of
the related section`s subsections.

**TODO** - Note that defining a method to meaningfully fill this list is still
an open issue: It has little use to add all the nodes (element and non-element
ones) of a section as the node hierarchy is lost (flat list). It should however
be possible to use the `currentOutlineOwner` reference to limit which nodes are
added - e.g. add a node to the list if, and only if, the `node.parentNode`
property matches the `currentOutlineOwner` reference, which would essentially
limit the list to the top-level nodes of a section. This, on the other hand,
would not add inner sectioning elements if they are hidden inside some other
element (a `div` container for example).

The `heading` property can be seen as an optional shortcut for
`node.parentSection.heading`, which must yield the same element reference. Note
that, if a section has a heading, then `heading.parentSection.heading` will yield
the same heading reference.

```
Begin Class SectioningElement
  Outline innerOutline
End Class SectioningElement
```

The `SectioningElement` class can be seen to inherit DOM's `Element` type. It
extends the `Element` type by the `innerOutline` property and could be used to
implement the algorithm itself. This allows to not associate invalid references
with element nodes for which there is no definition for an inner outline. As
long as there is no such DOM `SectioningElement` type, the existing `Element`
type must be used.

The `innerOutline` property corresponds with the `Outline.outlineOwner` property
such that the expression `sectioningElement.innerOutline.outlineOwner` yields
the same element reference.
