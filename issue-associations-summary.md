
Title: Outline Algorithm - Association issues/bugs

*My apologies for this lengthy post. Its size is necessary to explain in detail
which issues I see and how they need to be fixed. - If you spot an error in my
logic, please do let me know.*

- Related to W3C`s HTML outline algorithm
- A detailed look at the algorithm's association issues/bugs

After a lot of guesswork, I have compiled my point of view with regards to
associations into [this overview]().

Initially, I intended to propose to add such an overview because a clear definition
of the algorithm's result is needed in order to reduce the amount of guesswork one
has to do in order to implement the algorithm. Unfortunately, the overview still
has some open TODOs and is in conflict with the current algorithm.

The conflicting part is that I consider any node (no exceptions) to be an inner
node of some outer parent section. This point of view disallows to associate a
sectioning element with one of its inner sections.

With that in mind, I understand the algorithm's statement "associate node X with
section Y" to define and update the following two properties:

```
Section Node.parentSection
Node[]  Section.innerNodes
```

The `parentSection` property can be used to display a node's "current location"
(e.g. breadcrumbs) and the `innerNodes` property could be used to normalize a
document (e.g. inject `<section>` elements if those are missing).

And with that point of view in mind, you should be able to understand the
following issues:

**(1) - When entering a sectioning content element:**
""Associate current outline owner with current section.""

At that point, `currentOutlineOwner` is the element being entered and
`currentSection` represents its first inner section. In short: The algorithm
associates sectioning content elements with their first inner sections (inwards),
turning the meaning the word `parent` inside the `parentSection` property
upside-down. Note that this also defines sectioning content elements to be inner
nodes of their first inner section. In contrary to that, any other node (sectioning
root elements included) will always be associated with a section that is located
outside (outwards, in the direction of the `<body>` element).

Disregarding the mentioned side effects, this kind of association is needed ...
see below

**(2) - Final paragraph in section 4:**
""In addition, whenever the walk exits a node, after doing the steps above, if
the node is not associated with a section yet, associate the node with the section
current section.""

Note that, as this paragraph uses the description "a node", it can be seen to be
applicable to any node. Because of (3), it should be made clear that this step
applies to **any node (element and non-element nodes alike)**.

Note also that there are no explicit "associate" statements when entering heading
elements. Therefore, this step must also be understood to associate heading elements.

If the relationship merely consists of a `Node.parentSection` property, then it
does not matter if nodes are associated with sections when *exiting* them. It
starts to get weird if you also define and update a `Section.innerNodes` list...

One aspect is that it is inconsistent as some nodes (sectioning elements) are
associated when entering them and others (any other node) when exiting them.

Note also, that inner nodes will be exited before their parent nodes are exited.
If, for example, a heading element is the first node of a section, then its
inner text node will be the first node in the section's `innerNodes` list.

As a result, the first node's `nextSibling` property can not always be used to
get to the section's next inner top-level node (understand "top-level" to express
"has no parent node within the same section"). If it were possible, the `innerNodes`
list could be reduced to a mere `Node Section.firstInnerNode` property.

Currently, only the very last node associated with a section (`innerNodes[N-1]`)
is guaranteed to be a top-level node. This forces one to traverse those kind of
nodes in reversed order (`previousSibling`), which adds unnecessary requirements
to an implementation (i.e. the list type used must allow to add nodes to the
beginning).

If there is no good reason to associate the remaining nodes when exiting them,
then the algorithm should associate those when **entering** them. This change
will of course make it necessary to add explicit "associate" statements to the
"when entering a heading content element" section.

**(3) Associate all non-element nodes that are in the subtree for which an outline
is being created with the section with which their parent element is associated.**

```
<body>
  "Hello world!"
<body>
```

In this example, the non-element text node `Hello world!` has the body element
as its parent element. This sectioning root element will always be the top-most
element of a document and, as such, can itself never be associated with any outer
parent section. And because of that, this step will never associate this text node
with the body's first inner section, which it is actually supposed to do.

This is clearly bugged and I don't see any way to "fix" this it. Note that the
paragraph referenced in (2) also applies to non-element nodes. So the "fix" here
must be to **remove this paragraph**.

----------------------------------

The definition a heading for an element of sectioning content is in direct
conflict with the overview's `DomNode.heading` property. The first property one
refers to a heading element located inside the sectioning content element, the
second property to a heading element located outside of it.
