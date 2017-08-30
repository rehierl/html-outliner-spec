
Outline algorithm, Step 5, Association bug and fix -
[Issue 1001](https://github.com/w3c/html/issues/1001)

Step 5 - ""Associate all non-element nodes that are in the subtree for which an
outline is being created with the section with which their parent element is
associated.""

This step can be seen to define a `Node.parentSection` property that could have
the following use: A browser could detect which node is the first visible node
inside one of its windows. In combination with a `Section.parentSection` property,
this information could be used to visualize the current location (e.g. breadcrumbs).
It could even be used to fold and unfold the table of contents of a document
depending on which content is currently visible. The algorithm therefore needs to
associate all nodes with the proper section:

```
(example 1)      (example 2)
<body>           <X>
  A                B
</body>            <h1>C</h1>
                   D
                   <h1>E</h1>
                   F
                   <h2>G</h2>
                   H
                 </X>
```

**The issue** with this step is that the non-element text node A has the body
element as its parent element. Because of the way sectioning root elements are
associated (i.e. with some outer section), the body's `parentSection` property
can never be set. This step will therefore never associate such a text node with
any section.

For text nodes C, E and G, it does not matter what kind of element the `<X>`
element is. Their parent element will always be the corresponding heading element.

If the `<X>` element is a **sectioning root element**, then this step will fail
to properly associate text nodes B, D, F and H because their parent node is always
the sectioning root element and not one of the heading elements.

Because sectioning root elements will always be associated with some outer section,
this step will, in general, never associate any non-element node, that is a direct
child of a sectioning root element, with one of the element's inner sections (which
is what this step is actually supposed to do).

If the `<X>` element is a **sectioning content element** text nodes B and D will
be properly associated, but only because sectioning content elements themselves
are associated with their first inner section. However, text nodes F and H will
also be associated with the first inner section of their parent sectioning content
element, and not with the implied section in which they are located.

NOTE - The `Node` type, and not the `Element` type, needs to define the
`parentSection` property for the exact same reason: If only a
`Element.parentSection` property would exist, then a non-element node would have
to access the `parentSection` property of its parent element (e.g.
`text.parentElement.parentSection`). If the `parentElement` is a sectioning root
element, or even a sectioning content element, then this property will (most
likely) not return the proper section.

NOTE - One of the reasons I see, why sectioning content elements are associated
with their first inner section (i.e. associate with a subordinate entity, i.e.
violate the definition of a `parent` property in general), is to comply with the
definition of a "heading of a sectioning content element" (see in combination
with step 6). But because the direction (inwards vs. outwards) of the
`parentSection` property depends on the referencing object, `Node.parentSection`
(or even `Node.heading`) can not be clearly defined. Another reason could be the
similarity with the `h1-6` heading elements.

The last paragraph of the algorithm's tree traversal step 4, just before step 5,
provides **a possible fix**:

""In addition, whenever the walk exits a node, after doing the steps above, if
the node is not associated with a section yet, associate the node with the
section current section.""

This step/statement does not distinguish between element and non-element nodes.
The main difference is, that this step will also associate any of the
remaining/unassociated element nodes (which currently also includes any heading
element) while the algorithm is still traversing the subtree.

Unfortunately, this step seems to have issues of its own:

For example when heading elements are located inside container elements and if
implied sections need to be created. The possible issues I currently see inside
the tag sequence `(section, h1, A, /h1, B, div, C, h2, D, /h2, E, /div, F, /section)`
is that `div` will be associated with `h2` (because of "when exiting", which
appears to be intentional), but `C` with `h1` (unintentional?).

Still, the main problem with step 5 is when it is supposed to be executed: once
the algorithm has finished traversing the subtree. Therefore, the `parentSection`
property is used to try to get a reference of the node's section, which obviously
can not work. The only way it could work is if the `currentSection` reference is
used while the algorithm is still traversing the subtree. Hence, I don't see any
way to fix step 5.

I therefore suggest to:

- completely remove step 5,
- add explicit "associate" statements to the algorithm's "when entering a heading
  content element" section,
- and to clearly state that the last paragraph of step 4 applies to element and
  non-element nodes.
