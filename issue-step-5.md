
[Outline algorithm, Step 5, Association bug and fix](https://github.com/w3c/html/issues/1001)

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
                   <h2>G</h1>
                   H
                 </X>
```

**The big issue** with this step is that the non-element text node A has the body
element as its parent element. Because of the way sectioning root elements are
associated (i.e. with some outer section), the body's `parentSection` property can
never be set. This step will therefore never associate such a text node with any
section.

For text nodes C, E and G, it does not matter what kind of element the `<X>`
element is. Their parent element will always be the corresponding heading element.

If the `<X>` element is a **sectioning root element**, then this step will fail
to properly associate text nodes B, D, F and H because their parent node is always
the sectioning root element and not one of the heading elements. It also does not
matter where the sectioning root element is located within a document.

Because sectioning root elements will always be associated with some outer section,
this step will, in general, never associate any non-element node, that is a direct
child of a sectioning root element, with one of the element's inner sections (which
is what this step is actually supposed to do).

If the `<X>` element is a **sectioning content element**, then this step will
"only" fail for text nodes F and H. Text nodes B and D will be properly associated,
but only because sectioning content elements are themselves associated with their
first inner section.

NOTE - This actually is another problem: The only reason I can see, why sectioning
content elements are associated with their first inner section (i.e. associate with
a subordinate entity, i.e. violate the definition of a `parent` property in general),
is to comply with the "heading of a sectioning content element" (see in combination
with step 6).

The last paragraph of the algorithm's tree traversal step 4, just before step 5,
provides **a possible fix**:

""In addition, whenever the walk exits a node, after doing the steps above, if
the node is not associated with a section yet, associate the node with the section
current section.""

This step/statement does not distinguish between element and non-element nodes.
The only difference is, that this step will also associate any of the
remaining/unassociated element nodes (which includes any heading element).

Unfortunately, this step seems to have issues of its own:

For example when heading elements are located inside container elements and if
implied sections need to be created. The possible issues I currently see inside
the tag sequence `(section, h1, A, /h1, B, div, C, h2, D, /h2, E, /div, F, /section)`
is that `div` will be associated with `h2` (because of "when exiting", which appears
to be intentional), but `C` with `h1` (unintentional?).

Still, I don't see any way to fix step 5. I therefore suggest to:

- completely remove step 5,
- add explicit "associate" statements to the algorithm's
  "when entering a heading content element" section,
- and to clearly state that the last paragraph of step 4 applies to
  element and non-element nodes.
