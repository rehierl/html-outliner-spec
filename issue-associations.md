
The purpose of this document is to isolate the different relationships needed
between nodes and sections and to clearly define those. It is based upon:

* [Outliner steps](./outliner-steps.md)
* [Outliner pseudocode](./pseudocode.md)

One of the problems the outline algorithm currently has is that it does not
clearly define what you will get as a result if you decide to run it. The general
idea should be that it will create outline and section objects, connect them with
each other and attach them to the nodes of the dom subtree for which the algorithm
was started.

When it comes to attaching outline objects to nodes, the algorithm is rather
straight forward. Unfortunately, the associations/connections between nodes and
sections are rather vague. To make this more obvious, it helps to split the
algorithm's steps into [separate expressions](./outliner-steps.md) and to turn
those into [pseudocode](./pseudocode.md/#visit-func).

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
because the majority of elements can't associate a meaning with such a property
as they don't have a definition for an inner outline.

NOTE - An outline is only defined for elements of sectioning content and
sectioning root elements.

If a term would exist (e.g. **sectioning element**) that allowed to refer to the
group/class of elements for which an inner outline is defined, this term could
then be used for the name of a class that inherits from the `Element` type and
which it extends by an `innerOutline` property. This would then allow to refine
the above two properties into: `Outline SectioningElement.innerOutline` and
`SectioningElement Outline.outlineOwner`.

<!-- ####################################################################### -->
<h2 id="node-section">Associate node X with section Y</h2>

The following sections identify the steps that establish a connection between
nodes and section objects.

<!-- ======================================================================= -->
<h3 id="type-1">Type 1) When creating a new section object</h3>

The first time a section object will be created is when executing
[step 4.4.3](./outliner-steps.md/#4-4-3) (i.e. "Let current section be a newly
created section for the current outline owner element"). I would translate this
statement into `currentSection = new Section(currentOutlineOwner)`.
The exact same kind of connection will be established when executing
[step 4.6.4.](./outliner-steps.md/#4-6-4).

For the moment, I read this statement as defining a `Element Section.startingNode`
property. I don't read it to state that `startingNode` must always be an inner
element of said section - this node can be an inner element of this section, but
it does not always have to be one - i.e. the name `startingNode` isn't overly
accurate.

NOTE - `startingNode` will always be either a sectioning root element, an element
of sectioning content or a heading element (see [type-5](#type-5)).

NOTE - Here I assume that a sectioning element will never be an inner element of
one of its inner sections.

The main characteristic of that connection seems to be that the section object
is created when reaching the given node. Therefore, this connection could be
described as: "This element triggered the creation of that section container".

To understand why this connection has its use, it might help to recall the
paragraph related to creating an interactive table of contents (TOC). In order
to support interactive TOCs, it must be possible to forward the user to the top
of the section the user selected by clicking on a heading - i.e.
`heading.section.startingNode`. Here, `heading.section` means to get a
reference of the section in which the heading is located.

**TODO** - is it possible to jump to a non-element node?

NOTE - I don't require this connection to define a two-way relationship; i.e. I
don't implement an `Element` property that implements the other direction (from
a node object towards a section object) but still refers to the same relationship.

<!-- ======================================================================= -->
<h3 id="type-2">Type 2) When entering a sectioning content element</h3>

In [step 4.4.4.](./outliner-steps.md/#4-4-4), the definition states to
associate `current outline owner` with `current section`. At that point,
`current outline owner` is the sectioning content element being entered and 
`current section` is a new section object created for that element. This new
section is the first section in the outline of the sectioning content element
being entered.

This type of connection **points inwards** from the sectioning content element
towards its first inner section.

NOTE - The algorithm's definition does not state the purpose of this type of
connection, hence it is unclear if this step is meant to represent the counter
part of "create section for". Therefore this could refer to an unspecified, but
completely different type of relationship.

**TODO** - This must be a **bug**, as that step is probably supposed to directly
follow step 4.4.2 - i.e. it should appear before creating the first inner section.

<!-- ======================================================================= -->
<h3 id="type-3">Type 3) When entering a sectioning root element</h3>

Before creating a new section object, [step 4.6.3.](./outliner-steps.md/#4-6-3)
states to "Let current outline owner's parent section be current section". At
that point, `current outline owner` is the sectioning root element being entered
and `current section` is the section object outside and in front of that element.

This connection **points outwards** from the sectioning root element towards its
first outer section, which clearly distinguishes it from the previous connection
type. **(type-3 != type-2)**

The nature of this connection and its naming (i.e. `parent section`) can be
understood to state that the sectioning root element itself is an inner element
of that outer section. So it should be intended to implement this connection
via a `Section Node.parentSection` property.

NOTE - It still applies that the outline of a sectioning root element must not
contribute to the outlines of its ancestor sectioning element.

NOTE - After creating a new section, i.e. the first inner section of the element
being entered, the algorithm does not have a statement that is similar to
[step 4.4.4.](./outliner-steps.md/#4-4-4).

NOTE - In this particular case, the `Node.parentSection` property is also used
as a means to save and restore a reference to the `current section` before
overwriting this variable with a reference to the first inner section (see
[step 4.7.2](./outliner-steps.md/#4-7-2)). The negative side effect of this
save-and-restore method is that it turns write-access to the nodes in the dom
subtree into a mandatory requirement for the outline algorithm.

<!-- ======================================================================= -->
<h3 id="type-4">Type 4) When entering a heading content element</h3>

Although the algorithm's definition does not (yet) use the exact same wording as
in [step 4.4.3](./outliner-steps.md/#4-4-3) (i.e. "create a section *for*") when
entering a heading content element, [step 4.9.2.3](./outliner-steps.md/#4-9-2-3) and
[step 4.9.3.2.1.1](./outliner-steps.md/#4-9-3-2-1-1) can be seen to establish the
same type of connection. **(type-4 == type-1)**

NOTE - In contrary to type-1, the heading content element being entered clearly
is an inner element of the created section because this heading element represents
the heading of that new section.

<!-- ======================================================================= -->
<h3 id="type-5">Type 5) Associate node X with 'current section'</h3>

[step 4.11.](./outliner-steps.md/#4-11) states:
Whenever the walk exits a node, if the node is not already associated with a
section, then associate the node with `current section`.

NOTE - This step uses the wording "a node", it applies to **any nodes** (element
nodes and non-element nodes) that the previous steps have not already been processed.

NOTE - The previous steps apply to any elements that extend the outlines of their
ancestors. When entering or exiting any other element node, the references to
`current section`, `currentOutlineOwner` and the current outline (i.e.
`currentOutlineOwner.innerOutline`) stay the same throughout the visit (enter and
exit) of these remaining nodes. And because of that, this very step could also be
executed when **entering** these nodes.

**TODO** - exit is a bug - order when associating section with node

<!-- ======================================================================= -->
<h3 id="type-6">Type 6) Associate non-element node X with parent.section</h3>

[step 5.](./outliner-steps.md/#5) states:
Associate all non-element nodes that are in the subtree for which an outline is
being created with the section with which their parent element is associated.

NOTE - The DOM Node type defines a `Node.parentElement` property which makes it
unnecessary to clarify if a non-element node can have a non-element node as its
parent - that is if the `Node.parentElement` property is used instead of the
`Node.parentNode` property.

**TODO** - bug - body, text, /body - body has no parent section

<!-- ####################################################################### -->
<h2 id="node-heading">Associate node X with heading Y</h2>

[step 6.](./outliner-steps.md/#6) states:
Associate all nodes in the subtree with the heading of the section with which
they are associated, if any.
