
A restructured summary of Chapter 4.3.9

* [W3C, HTML 5.2, Editor's Draft, 3 May 2017](https://w3c.github.io/html)

*acronyms*

* sectioning element (SE) - can be an SRE or an SCE
* sectioning content (SC) - SC element (SCE)
* sectioning root (SR) - SR element (SRE)
* heading content (HC) - HC element (HCE)

*associate a node with a section*

* (step 4.11) on exit with the current section - issue is 'on exit'

## [4.3.9 Headings and sections](https://w3c.github.io/html/sections.html#headings-and-sections)

*this section should only contain definitions*

### 'first inner/outer X' element

*notes*

1. the first inner X element of Y - refers to the first child of node Y found
   that matches X when searching Y's sub-tree in a pre-order tree-order traversal.
1. the first outer X element of Y - refers to the first ancestor node of Y found
   that matches X when going up from one parent/ancestor node to the next parent
   node - i.e. start with checking Y's parent node, then, if that doesn't match,
   continue with the parent node of that node, etc.

### sections

1. a section is a container that corresponds to some nodes in the dom tree
1. each section can have a heading associated with it
1. each section can contain any number of further nested (sub)sections
1. these sections aren't section elements, although some may correspond to
   such elements - they are merely conceptual sections

*notes*

1. does allow for sections to be empty - may contain no nodes/headings at all
1. the first inner HCE of a section, that is not part of an inner SE, represents
   the heading for that section
1. implies a hierarchy (parent-child relationship) of sections

### outline

1. the outline of an SCE consists of one or more potentially nested sections
1. the element for which an outline is created is said to be the **outline's owner**
1. the outline created for the body element of a document is the outline of the
   entire document - the body element is the owner of the document's outline

*notes*

1. does not allow outlines to be empty - must have at least one section
1. an outline is merely a sequence of one or more sections

### sectioning element (SE)

*notes*

1. define the term "sectioning element" as a general reference for a SCE or SRE
1. each SE has an outline - combines sections into a sequence of sections
1. the term **sections of an SE** refers to all sections in its outline,
   excluding any subsections they might have.

### sectioning content (SC, SCE)

1. elements - **article, aside, nav, section**
1. sectioning content is content that defines the scope of headings and footers
1. each SCE potentially has a heading and an outline
1. SCEs are always considered subsections of their nearest ancestor SR/SC,
   whichever is nearest, regardless of what implied sections other headings
   may have created - see [inner sce](./issue-inner-sce.md)

*notes*

1. what exactly is the heading of a SCE supposed to be?
   the first heading of the first section? what if that section has no heading,
   but the 2nd one does?
1. if there was a clear definition for a SCE's heading, what if there are
   multiple headings that have the same level/rank? wouldn't that nullify any
   potential use?
1. what exactly is the outline of an SCE? -
   needs an explicit inner/outer reference

### sectioning roots (SR, SRE)

1. elements - **blockquote, body, details, dialog, fieldset, figure, td**
1. SRs are distinct from SCs
1. can have their own outlines
1. sections and headings inside SRs don't contribute to the outlines of their
   ancestors; i.e. SCs and/or SRs

*notes*

1. should say that they also potentially have a heading?
1. implies an hierarchy of outlines
1. implies some cut/break of the parent-child relationship between sections

### heading content (HC, HCE)

1. elements - **h1, h2, h3, h4, h5, h6**
1. defines the header of a section - whether explicitly marked up using SCEs,
   or implied by the HC itself
1. each SCE has a rank given by the number of their name. h1 has highest rank,
   h6 has lowest rank. two elements with the same name have equal rank

*notes*

1. distinction between HC and HE? concept vs. element?
1. the algorithm will associate an implied heading with each section that does
   not have a heading associated with it

### HCEs inside SEs

1. the first HCE of a section, that is not part of an inner SE, represents the
   heading for that section
1. subsequent headings of lower rank start implied sections that are part of
   the previous one.
1. subsequent headings of equal or higher rank start new (implied) sections
1. in both cases, the HCE represents the heading of the implied section.

*notes, if an SE contains no other inner SE*

1. when entering an HCE, if the current section *S* has no heading, the algorithm
   will associate that heading with section *S*
1. in general, each subsequent HCE will start a new implied section with that
   HCE, representing the implied section's heading

*notes, if an SE contains an inner SE*

### outline depth

1. the outline depth of a HCE associated with a section *S* is the number of
   sections that are ancestors of *S* in the outermost outline that *S* finds
   itself in when the outlines of its document's element are created, plus 1
1. the outline depth of a HCE, not associated with a section, is 1

*in general*

1. what is this definition used for?

*notes on part 1*

1. defined for HCEs, not for SEs
1. what is the outermost outline? - the outline of the first outer SRE or
   the document/body's outline?
1. HCEs that have the body element as parent must have depth value 1 -
   because they have 0 parent sections.
1. what is the depth value of an HCE inside a nested SR? - distinguis between
   relative depth (first outer SRE) and absolute depth (body) values?

*notes on part 2*

1. note that you do not have to start at the body element to create an outline
1. only those HCEs will not be associated with a section, that aren't part of
   the starting node's subtree - i.e. don't have the root as parent/ancestor -
   i.e. can't be reached by the root's outline.
1. why give them a depth value that matches the value of a top-level heading?
1. why not 0, which would allow to determine if a HCE is even part of the outline?

## [4.3.9.1 Examples]()

### example 24

1. body, h1-A, h2-B, blockquote, h3-D, /blockquote, p-E, h2-F, section, h3-G
   /section, p-H, /body
1. explicit body section
1. h1-A contains p-H
1. h2-B contains blockquote and p-E
1. a subsection (h2-B) contains a SR - i.e. a SR is itself still associated
   with their parent section - their 'inner' outliner will still not contribute
   to its parent outline (body)
1. h2-F contains only itself
1. explicit section 'section'
1. section ends the earlier implicit section (h2-F) so that the later paragraph
   (p-H) is back at the top level

### example 25

1. semantically identical documents when using section elements
1. avoid relying on implicitly generated sections

### example 26

1. body, h1-A, p-B, h2-C, p-D, h2-E, p-F, /body
1. body section associated with h1-A and p-B
1. h2-C section associated with h2-C and p-D
1. h2-E section associated with h2-E and p-F

### example 27

1. user agents should provide default/implied headings
1. not specified by this specification
1. might vary with the user's language, etc.

## [4.3.9.2 Clarifications]()

*this section should only contain clarifications*

### clarification

1. SCEs are always considered subsections of their nearest ancestor SE,
   regardless of what implied sections other headings may have created

*notes*

1. see example 24
1. does not require a direct parent-child relationship - feels more like a
   'contains' statement
1. does not seem to imply that there is a cut/break of the parent-child
   relationship between sections of different nesting level

### clarification

1. when creating an interactive table of contents, entries should forward the
   user to the relevant SE if the entry's section was created for a real element,
   or to the relevant HCE, if the entry's section was generated for a heading
1. selecting the first section of the document always takes the user to the top
   of the document, regardless of where the first heading in the body is located

*notes*

1. sections are always created because of real/exisiting elements
1. forward to the node that is the first node associated with that section

### clarification

1. user agents should provide default headings for sections that do not have
   explicit section headings.
1. e.g. "Untitled document" for body, "Navigation" for nav, "Sidebar" for aside
1. default headings not specified by HTML's spec
1. might vary depending on the user's language, etc.

*notes*

1. see example 27

### clarification

1. authors are also encouraged to explicitly wrap sections in elements of SC,
   instead of relying on the implicit sections generated by having multiple
   headings in one SCE

*notes*

1. see example 25

### clarification

1. sections may contain headings of a rank equal to their section nesting level
1. authors should use headings of the appropriate rank for the section's nesting
   level

*notes*

1. by itself unclear as what that is supposed to mean ...
1. HTML 5.1 no longer promotes the use of only h1 elements for
   [compatibility reasons](http://html5doctor.com/interview-steve-faulkner-html5-editor-new-doctor/#comment-32325)

### clarification

1. HCE must not be used to markup subheadings, subtitles, etc.
1. instead use the markup patterns in the 4.13 section of this spec
1. e.g. header, p, span, ul/li, etc.

*notes*

1. same clarification as in 4.3.6. The h1, ..., h6 elements

## [4.3.9.3 Creating an outline](https://w3c.github.io/html/sections.html#creating-an-outline)

*this section should only contain the algorithm's steps*

1. algorithm for creating an outline for a SE
1. defined in terms of a walk over the nodes of a dom tree in tree-order, whith
   each node being visited when it is entered or exited during the walk.
1. associates each node with a section and potentially a heading

*notes*

1. [outliner-steps](./outliner-steps.md)
1. [outliner-steps](./outliner-steps-mod.md) (modified)

## [4.3.9.4 Code fragments]()

*this section should only contain sample or pseudocode fragments*

1. JavaScript tree-order traversal
