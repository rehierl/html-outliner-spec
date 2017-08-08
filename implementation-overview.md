
- Related to W3C`s HTML outline algorithm
- A much needed overview

In general, the algorithm will create outline and section objects, connect them
with each other and attach those to the DOM subtree for which the algorithm was
started. These objects and their properties can be described as follows.

Note that, depending on the goal of an implementation, certain properties can be
considered to be optional. The outline algorithm below describes the steps needed
to implement an outliner intended to support all its features.

Note that array types like `Section[]` are intended to represent a mere list of
certain objects. These kind of list properties could also be implemented as linked
lists and not actual arrays.

```
Begin Class Outline
  SectioningElement outlineOwner
  Section[] innerSections
End Class
```

The `outlineOwner` property will be set if a new outline object is created for
a **sectioning element**, which is either an element of sectioning content or a
sectioning root element. It will always hold a reference of such an element.

```
Begin Class Section
  Element startingElement
  Outline parentOutline
  Section parentSection
  Section[] subSections
  Element heading
  // DomNode[] innerNodes
End Class
```

The `startingElement` property will be set if a new section object is created when
entering a sectioning element, or an element of heading content. It will always
hold a reference of such an element. This property is needed in order to allow to
jump to the beginning of a section if the corresponding table of contents entry is
selected.

Note that the property's name is not entirely accurate: A sectioning element is
not considered to be an inner node of one of its inner sections. In contrary to
that, an element of heading content is always considered to be an inner node of
the section to which it is assigned as heading.

**TODO** - The first node within a section may be a non-element node (a text node
for example). Is it possible to jump to a non-element node? If the answer is yes,
then it would be possible to jump to `section.firstInnerNode` instead.

The `parentOutline` property will be set if a section is added to an outline.
It corresponds with the `Outline.innerSections` property such that the expression
`outline.innerSections[anyIndex].parentOutline` yields the same outline reference.

Note that the `parentOutline` property must hold a null reference if a section
itself is not added to an outline. This allows to determine if a section is a
top-level section of an outline (if non-null), or a subsection of another section
(if null).

The `parentSection` property will be set if a section is added as subsection to
another section. It corresponds with the `subSections` property such that the
expression `section.subSections[anyIndex].parentSection` yields the same section
reference.

During traversal of the structure the algorithm created it must be possible to
determine if continuing to traverse it in the direction of the body element would
result in leaving the outline of a sectioning element, or if the root sectioning
element was reached. The latter can be the case for the starting sectioning
element, which was used to start the algorithm, or an inner sectioning root
element. The following conditions allow to determine the current location:

- If `(parentOutline == null) && (parentSection == null)` is true, the section
has just been initialized. This condition indicates an error if it is still true
after the algorithm has finished.
- If `(parentOutline == null) && (parentSection != null)` is true, the section
is an inner subsection.
- If `(parentOutline != null) && (parentSection == null)` is true, the section
is a top-level section of a root sectioning element. This element can be the
starting sectioning element, or an inner sectioning root element.
- If `(parentOutline != null) && (parentSection != null)` is true, the section
is a top-level section of an inner sectioning content element, which is not the
starting sectioning element.

The `heading` property holds a reference of the heading content element that
represents the section's heading. This heading element is considered to be an
inner node of the property's section; `section.heading.parentSection` must yield
the same section reference.

Note that the algorithm will distinguish the following three states:
(1) The algorithm has entered a section but did not yet encounter its heading -
(2) The element reference the `heading` property holds represents the section's
heading - and (3) The algorithm has finished processing the section and has
determined that it has no heading. - In order to distinguish state 1 from state
3, the algorithm will associate a pseudo heading (implied heading) with each
section. Once the algorithm has finished, each `heading` property's value will
either be a valid element reference, or it represents an implied heading.

```
Begin Class DomNode
  Section parentSection
  // Element heading
End Class DomNode
```

The `parentSection` property connects a DOM node with its parent section. If it
holds non-null value, the node is considered to be an inner node of that section.
The property must be set if the algorithm associates a node with a section and
can be used to provide a "current path" (e.g. breadcrumbs) depending on which
nodes are visible inside a browser's window.

Note that DOM's `Node` type, not the `Element` type, needs a `parentSection`
property because sectioning elements themselves are considered inner nodes of an
outer parent section. If a text node is a direct child of a sectioning element
and if only DOM's `Element` type had a `parentSection` property, then
`textNode.parentElement.parentSection` will be a section located outside of its
parent sectioning element. This would contradict that a text node inside a
sectioning element must be associated with an inner section of that sectioning
element.

The `Section.innerNodes` property corresponds with the `parentSection` property
such that the expression `section.innerNodes[anyIndex].parentSection` yields the
same section reference. This property could be used to allow folding and unfolding
of large documents depending on which table of contents entry was selected.

**TODO** - Note that defining a method to meaningfully fill this list is still
an open issue: It has little use to add all the nodes (element and non-element
ones) of a section to that list as the node hierarchy is lost (flat list). It
should however be possible to use the `currentOutlineOwner` reference to limit
which nodes are added to that list - e.g. add a node to the list if, and only if,
the `node.parentNode` property matches the `currentOutlineOwner` reference. But
that approach would not add inner sectioning elements if they are hidden inside
another element (a `div` container for example). The open question is: Would
that approach be of any use?

The `heading` property can be seen as an optional shortcut for
`node.parentSection.heading`, which must yield the same element reference.

```
Begin Class SectioningElement
  Outline innerOutline
End Class SectioningElement
```

The `SectioningElement` class can be seen to inherit DOM's `Element` type. It
extends the `Element` type by the `innerOutline` property and could be used to
implement the algorithm itself. This allows to not associate null references with
element nodes for which there is no definition for an inner outline. As long as
there is no such DOM `SectioningElement` type, the existing `Element` type must
be used.

The `innerOutline` property corresponds with the `Outline.outlineOwner` property
such that the expression `element.innerOutline.outlineOwner` yields the same
element reference.
