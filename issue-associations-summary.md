
This document provides a summary of the issues mentioned in:

* [Association notes](./issue-associations-notes.md)

I have tried to condense my point of view with regards to associations into
[this overview]().

Initially I intended to propose to add such an overview because a clear definition
of the algorithm's result is much needed. It would help to reduce the amount of
guesswork one has to do when trying to implement the algorithm. Unfortunately,
the overview still has some open TODOs and, most importantly, is in conflict with
the current algorithm.

The conflicting part is that I consider any node (no exceptions) to be an inner
node of some outer parent section. This point of view disallows to associate any
sectioning element with one of its inner sections.

With that in mind, I understand the algorithm's statement "associate node X with
section Y" to define and update the following two properties:

```
Section Node.parentSection
Node[]  Section.innerNodes
```

The `parentSection` property can be used to display a node's "current location"
(e.g. breadcrumbs) and the `innerNodes` property could be used to add missing
`<section>` elements (e.g. for normalization purposes).

With that point of view in mind, it should be possible to understand the
following issues:

### (1) When entering a sectioning content element

""Associate current outline owner with current section.""

At that point, `currentOutlineOwner` is the element being entered and
`currentSection` represents its first inner section. The algorithm therefore
**associates each sectioning content element with its first inner section
(inwards)**.

Note that the actual meaning of the `parentSection` property can only be understood
to express that the associated node is located inside the corresponding section.

In contrary to this step, any other node (sectioning root elements included) will
be associated with a section that is located outside of it - i.e. **outwards**,
in the direction of the `<body>` element.

The side effect of this step is that the `Node.parentSection` property has,
depending on the property's node, two different meanings (i.e. inwards or
outwards). The result is, that **this property can not be clearly defined**.

But, as it turns out, this step is required as is ... see below

### (2) Final paragraph in section 4

""In addition, whenever the walk exits a node, after doing the steps above, if
the node is not associated with a section yet, associate the node with the section
current section.""

Note that, as this paragraph uses the description "a node", it can be seen to be
applicable to any node. Because of (3), it should be made clear that this step
applies to **any node (element and non-element nodes alike)**.

Note that there are no explicit "associate" statements when entering heading
elements. This step must therefore also be used to associate these elements.

If the relationship merely consists of a `Node.parentSection` property, then it
does not matter if nodes are associated with sections **when exiting** them. If
however, a `Node.innerNodes` list property is defined, things turn out to become
unnecessarily complicated ...

One aspect is that this adds inconsistency to the algorithm as some nodes
(sectioning elements) are associated when entering them and others (any other
node) when exiting them.

Note also, that child nodes will be exited before their parent nodes are exited.
If, for example, a heading element is the first node of a section, then its
inner text node will be the first node in the section's `innerNodes` list. In
short, the very first node associated with a section has no guaranteed
characteristic with regards to its section.

As a result, the first node's `nextSibling` property can not be used to get to
the section's next inner top-level node (understand "top-level" to express "has
no parent node within the same section"). If the `nextSibling` property could be
used, then the `innerNodes` list could even be reduced to a single
`Node Section.firstInnerNode` property and thus could be used to reduce the
result's memory usage.

In addition to that, and if browsers would support jumping to a non-element node,
that same property could be used to forward a user to the beginning of a section.

Currently, the very last node associated with a section (`innerNodes[N-1]`) is
guaranteed to be a top-level node. This forces one to traverse those kind of nodes
in reversed order (`previousSibling`), which adds unnecessary requirements to an
implementation (i.e. the list type used must support to add nodes to its beginning).

If there is no good reason to associate the remaining nodes when exiting them,
then the algorithm should associate those **when entering** them. As a matter of
clarity, explicit "associate" statements should be added to the algorithm's "when
entering a heading content element" section.

### (3) Step 5

""Associate all non-element nodes that are in the subtree for which an outline is
being created with the section with which their parent element is associated.""

```
<body>
  Hello world!
<body>
```

In this example, the non-element text node `Hello world!` has the body element
as its parent element. This sectioning root element will always be the top-most
element of a document and, as such, can itself never be associated with any outer
parent section. And because of that, this step will never associate this text node
with the body's first inner section, which it is actually supposed to do.

This is clearly bugged and I don't see any way to "fix" it. **This step must be
removed**. Note that the paragraph referenced in (2) also applies to non-element
nodes. 

### (4) Step 6

""Associate all nodes in the subtree with the heading of the section with which
they are associated, if any.""

This step can be understood to define an `Element Node.heading` property. In
addition to that, the algorithm will associate a heading (when entered) with the
current section and therefore defines a `Element Section.heading` property. Note
that (2) also applies to heading elements (i.e. `Node.parentSection`).

I therefore consider this step to be an optional one which **could be removed**
if there is no need for an explicit `heading` property. It will always be possible
to get the heading of a node using the expression `node.parentSection.heading`.
This expression can also be used if `node` itself represents a heading element (i.e. 
`headingElement == headingElement.parentSection.heading == headingElement.heading`).

**The bigger problem with this step is** this: Because of No (1), sectioning
content elements will be associated with their first inner heading (if one exists).
The result is that, like the `Node.parentSection` property, the `Node.heading`
property has no clear meaning (inwards vs. outwards) and can therefore not be
clearly defined.

### (5) First paragraph of chapter 4.3.9.

""The first element of heading content in an element of sectioning content
represents the heading for that explicit section ...""

The actual intention behind the first paragraph is to try to explain what the
algorithm will do when entering a **sectioning element**.

Note that the sentence is not entirely accurate because it must also apply to
sectioning root elements.

The first heading element within such a sectioning element will be used as
heading for the first inner **section object**. Subsequent headings will cause
the algorithm to create new implied section objects.

Note that such a new section will be added to the sectioning element's outline as
a top-level section (and **not as a sub-section**), if a subsequent heading has a
rank that is greater or equal than the rank of the last top-level section.

With that in mind: **The first sentence must not be misunderstood to define what
the heading of a sectioning content *element* is**. (I misunderstood that myself!).
So we are back at square one: There is no definition for the heading of a sectioning
content *element* other than the one mentioned in (6).

### (6) Definition of "sectioning content"

""Each sectioning content element potentially has a heading and an outline.""

It took me a lot of guesswork before I realized that the actual reason for the
issues mentioned in (1) and (4) is to make the algorithm comply with the
"potentially has a heading" part.

--

Outline-to-Section, a 1:1 or a 1:N relationship?


<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
