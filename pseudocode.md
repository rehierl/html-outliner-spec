
These are the results when trying to write down the [outliner-steps](./outliner-steps.md)
using pseudocode as accurately as possible (i.e. almost word-by-word).

* `outliner-steps > pseudocode`
* The algorithm's main entry point is the [createOutline()](#create-outline-func) function.
* The most important part of this document is the [visit()](#visit-func) function
  and the [listing of issues](#visit-issues) beneath it.

## skipped

* (4.11) In addition, whenever the walk exits a node, after doing the steps above,
  if the node is not associated with a section yet, associate the node with current
  section.
* (5) Associate all non-element nodes (REM e.g. text nodes?) that are in the
  subtree for which an outline is being created, with the section with which
  their parent element is associated.
* (6) Associate all nodes in the subtree with the heading of the section with
  which they are associated, if any. (REM not necessary: node.section.heading)

## cancelled

* unfortunately, this transformation does not result in an easy-to-follow pseudocode.
* the only way to clear things up is to try a hands-on approach ...

<hr/>
## <span id="globals-class">Globals Class</span>

'Globals' will be used to provide access to the shared variables mentioned in
(1), (2) and (3).

### variables

* *currentOutlineOwner* : the current Node element whose outline is being
  created (1); also known as the current **outline's owner**.
* *currentSection* : a reference to a section object, so that elements in the
  DOM tree can be associated with it (2).
* *stack* : used to skip nodes and handle nesting (3).

### pseudocode

```
begin class Globals
  currentOutlineOwner = null
  currentSection = null
  stack = new Stack()
end
```

<hr/>
## <span id="stack-class">Stack Class</span>

'Stack' is a class that supports the commonly known stack operations:

### interface

* The topmost Node entry of the stack, also known as the top-of-the-stack
  entry, which will simply be referred to as **tos** entry.
* *push(Node node)* : will add the given node to the stack as the stack's new tos
  entry. For the sake of simplicity, assume that this function won't throw any error.
* *Node pop()* : will, if the stack is not empty, remove and return the current
  tos entry. If the stack is empty, calling this function must throw an error.
* *Node tos()* : will, if the stack is not empty, simply return the current tos
  entry. If the stack is empty, calling this function must throw an error.
* *bool isEmpty()* : will return true, if the stack has no tos entry, and false
  otherwise. This function must not throw any error.

### pseudocode

```
begin class Stack
  void push(Node node)
  Node pop()
  Node tos()
  bool isEmpty()
end
```

<hr/>
## <span id="create-outline-func">createOutline()</span>

The algorithm's overall steps can be looked at as being executed by a global
'createOutline()' function:

### signature

* *Outline createOutline(Node node)*
* *node parameter* : the node for which to create the outline. 'node' must refer
  to a sectioning content or a sectioning root element. *node* must also not
  represent a hidden element.
* *returns* : the outline created for the input node object.

### pseudocode

```
Outline createOutline(Node node) begin
01: assert(node.isSectioningContent || node.isSectioningRoot)
02: vars = new Globals()

03: walk(vars, node)

04: assert(vars.stack.isEmpty)
05: return node.outline
end
```

<hr/>
## <span id="walk-func">walk()</span>

For the sake of simplicity, let 'walk()' recursively traverse the node subtree:

### signature

* *void walk(Globals vars, Node node)*
* *vars parameter* : a Globals object that provides access to shared variables.
* *node parameter* : the current node to traverse.

### pseudocode

```
void walk(Globals vars, Node node) begin
01: visit(vars, node, true, false)

02: child = node.firstChild
03: while(child != null) begin
04:   walk(vars, child)
05:   child = child.nextSibling
06: end while

07: visit(vars, node, false, true)
end
```

<hr/>
## <span id="visit-func">visit()</span>

'visit()' implements the steps as defined in (4).

Introduction to (4) : Walk over the DOM in tree order, starting with the
sectioning content or sectioning root element at the root of the subtree for
which an outline is to be created, and trigger the first relevant step below for
each element as the walk enters and exits it.

### signature

* *void visit(Globals vars, Node node, bool entering, bool exiting)*
* *vars parameter* : A Globals object that provides access to shared variables.
* *node parameter* : The current Node object to visit.
* *entering parameter* : true if node is being entered, false otherwise.
* *exiting parameter* : true if the node is being exited, false otherwise.

### pseudocode

* As *Introduction to (4)* does not state what to do with steps that follow the
  first relevant step, lets assume that these need to be implemented as a
  sequence of *if* clauses.
* For the sake of readability, almost all of the usually required parentheses,
  have been dropped; e.g. *tos vs. tos()*

```
void visit(Globals vars, Node node, bool entering, bool exiting) begin
00: assert((entering && !exiting) || (!entering && exiting))

01: if(exiting && (node == vars.stack.tos)) then
02:   vars.stack.pop()
03: end if

04: if(vars.stack.tos.isHeadingContent
05: || vars.stack.tos.isHidden) then
06:   //- do nothing
07: end if

08: if(entering && node.isHidden) then
09:   vars.stack.push(node)
10: end if

11: if(entering && node.isSectioningContent) then
12:   if(vars.currentOutlineOwner != null) then
13:     if(!vars.currentSection.hasHeading) then
14:       vars.currentSection.setImpliedHeading
15:     end if
16:     vars.stack.push(vars.currentOutlineOwner)
17:   end if
18:   vars.currentOutlineOwner = node
xx:   //- currentSection, then currentOutlineOwner.section - see line 36
19:   vars.currentSection = new Section(vars.currentOutlineOwner)
xx:   //- 'section' same meaning as 'parentSection'? see line 36
20:   vars.currentOutlineOwner.section = vars.currentSection
21:   vars.currentOutlineOwner.outline = new Outline(vars.currentSection)
22: end if

23: if(exiting && node.isSectioningContent && !vars.stack.isEmpty) then
24:   if(!vars.currentSection.hasHeading) then
25:     vars.currentSection.setImpliedHeading
26:   end if
27:   vars.currentOutlineOwner = vars.stack.pop()
xx:   //- does not make the construct of implied headings a necessity
28:   vars.currentSection = vars.currentOutlineOwner.outline.lastSection
29:   vars.currentSection.appendOutline(node.outline)
30: end if

31: if(entering && node.isSectioningRoot) then
32:   if(vars.currentOutlineOwner != null) then
xx:     //- no set-implied-heading here - see line 13
33:     vars.stack.push(vars.currentOutlineOwner)
34:   end if
35:   vars.currentOutlineOwner = node
xx:   //- currentOutlineOwner.parentSection, then currentSection - see line 19
xx:   //- save the current section - see line 44
36:   vars.currentOutlineOwner.parentSection = vars.currentSection
37:   vars.currentSection = new Section(vars.currentOutlineOwner)
38:   vars.currentOutlineOwner.outline = new Outline(vars.currentSection)
39: end if

40: if(exiting && node.isSectioningRoot && !vars.stack.isEmpty) then
41:   if(!vars.currentSection.hasHeading) then
42:     vars.currentSection.setImpliedHeading
43:   end if
xx:   //- restore the current section - see line 36
44:   vars.currentSection = vars.currentOutlineOwner.parentSection
45:   vars.currentOutlineOwner = vars.stack.pop()
46: end if

47: if(exiting && vars.stack.isEmpty
48: && (node.isSectioningContent || node.isSectioningRoot)) then
49:   if(!vars.currentSection.hasHeading) then
50:     vars.currentSection.setImpliedHeading
51:   end if
52:   //- the walk is over
53: end if

54: if(entering && node.isHeadingContent) then
55:   if(!vars.currentSection.hasHeading) then
56:     vars.currentSection.heading = node
57:   else if(vars.currentOutlineOwner.outline.lastSection.hasImpliedHeading
58:   || (node.rank >= vars.currentOutlineOwner.outline.lastSection.heading.rank)) then
59:     vars.currentOutlineOwner.outline.appendSection(new Section())
60:     vars.currentSection = vars.currentOutlineOwner.outline.lastSection
61:     vars.currentSection.heading = node
62:   else
63:     candidateSection = vars.currentSection
64:     while(true) begin //- "Heading loop"
65:       if(node.rank < candidateSection.heading.rank) begin
66:         candidateSection.appendSection(new Section())
67:         vars.currentSection = candidateSection.lastSection
68:         vars.currentSection.heading = node
69:         break //- "Abort these substeps"
70:       end if
71:       candidateSection = candidateSection.parentSection
72:     end while
73:   end if
74:   vars.stack.push(node)
75: end if

76: if(true) then
77:   //- do nothing
78: end if
end
```

### <span id="visit-notes">notes</span>

These notes are statements/notes taken from the algorithm's definition that do
not translate into code.

* line 02: The element being exited is a heading content element, or an element
  with a 'hidden' attribute.
* line 09: This will cause the algorithm to skip that element and any of its
  descendants.
* line 29: This does not change which section is the last section in the outline.
* line 48: currentOutlineOwner is the element being exited and is the
  sectioning content or sectioning root element at the root of the subtree for
  which the outline is being generated.
* line 51: Skip to the next step in the overall set of steps. (The walk is over)
* lines 58, 65: Recall that h1 has highest rank, and h6 has the lowest rank.
* line 59: This new section must become the last section of that outline.
* line 66: This does not change which section is the last section in the outline.
* line 74: This causes the algorithm to skip any descendants of the element.
* line 77: Otherwise, Do nothing.

### <span id="visit-issues">issues</span>

1. Introduction to (4) : does not state what to do if succeeding steps would
   also be applicable. it merely states "trigger the first relevant step".
1. line 01: stack.tos could fail due to stack being empty;
   think of creating the outline for an empty SR or SC.
1. line 04: better change to 'if((entering || exiting) && (A or B))' in order to
   clarify that this step applies to both (entering and exiting) events.
1. line 04: will fail if visit() is entered for the very first time (empty stack).
1. line 04: unclear what tos refers to when executing visit() during an "exit"
   event and when stack.pop() (see line 02) was executed. strictly implemented,
   this would be the next tos entry after the one that was popped.
1. line 20: "Associate ... with" is too unspecific; compare with line 36.
1. line 29: *Section.appendOutline()*: unclear how to execute this operation -
   see [issue with inner SCEs](./issue-inner-sce.md)
1. line 38: note that it does not state to associate currentOutlineOwner with
   the newly created section; compare with line 20.
1. line 59: it should say "create a new section for the element that is being
   entered", not just "create a new section and".
1. line 65: possible problem - *rank* is not defined for implied headings; an
   implied heading represents a heading that does not exits.
1. line 66: same as 59
1. line 71: 'Let candidateSection be the section that contains candidateSection
   in the outline of currentOutlineOwner' is quite unclear. should state the
   parent-child relationship between sections established when appending a
   section to another one (line 66).
1. line 65 after executing 71: implies that a section always has a parent
   section, which is not the case once you reach a SR or SC.
1. lines 02, 27, 45: push/pop stack operations are rather decoupled from each other.
1. lines 36, 44: the *Node.parentSection* property is only used to *save and
   restore* the current section.

### <span id="faq-like-notes">faq-like notes</span>

#### line 13

* could currentSection still not be set (i.e. null)?
* the algorithm requires to begin with a SC or a SR element
* entering a SC element will always set currentOutlineOwner and currentSection
* entering a SR element will always set currentOutlineOwner and currentSection
* result : line 13 cannot fail

#### lines 13, 24, 32, 41, 48, 55:

* why all the create-and-set-implied-heading statements? - compare lines 12 and 32
* child SRs must not have any effect (don't contribute to) parent outlines
* a create-and-set in line 32 would essentially let SRs have such an effect
* once the outliner has finished, the distinction (exists || implied) is no
  longer relevant; i.e. "implied" will completely count as "doesn't exits"

#### line 23

* is '&& !vars.stack.isEmpty' necessary?
* run the algorithm with an empty SC element
* if(entering SC) won't push(), the stack remains empty
* yes, it is necessary
* similar for line 40

#### line 29

* without 'currentSection.appendOutline(node.outline)',
* SC elements would act like a SR element
* i.e. create separate outlines

<hr/>
## <span id="not-relevant">Not relevant</span>

* for the time being, ignore the following pseudocode:

### <span id="section-list-class">SectionList Class</span>

```
begin class SectionList
  //- add the section to the end of this list
  void append(Section section)
  
  //- return the last section of this list
  //- throw an error if empty
  Section lastSection()
end
```

### <span id="section-class">Section Class</span>

```
begin class Section
  //- the node that caused the creation
  //  of this section
  Node associatedNode = null
  
  //- the heading associated with this section
  Node heading = null

  //- this section is subsection to parentSection
  Section parentSection = null

  //- list of sub-sections contained
  //  within this section
  SectionList subsections = null
  
  //- create a new section and associate it
  //  with the given node
  Section(Node node) begin
    associatedNode = node
    subsections = new SectionList()
  end

  //- let sec become a subsection
  void appendSection(Section sec) begin
    sec.parentSection = this
    sections.append(sec)
  end

  //- is that the meaning of line 29?
  void appendOutline(Outline outline) begin
    for each(Section sec in outline) begin
      appendSection(sec)
    end
  end
end
```

### <span id="outline-class">Outline Class</span>

```
begin class Outline
  //- list of top-level sections contained
  //  within this outline
  SectionList sections = null

  //- create a new outline and use firstSection
  //  as the only section in this outline
  Outline(Section firstSection) begin
    sections = new SectionList()
    appendSection(firstSection)
  end

  //- iterate over objects in sections
  //  from first to last
  //- see Sections.appendOutline()
  //support for for-each statements

  void appendSection(Section sec) begin
    sections.append(sec)
  end

  //- return the last section of this outline
  //- throw an error if empty
  Section lastSection() begin
    return sections.lastSection
  end
end
```

### <span id="implied-heading-class">ImpliedHeading Class</span>

```
begin class ImpliedHeading
  //- used like an instanceof operator
  const bool isImpliedHeading = true
end
```

### <span id="node-class">Node Class</span>

'Node' serves as a wrapper class for a DOM node objects. It supports those
properties and operations specific to DOM nodes and in addition to that, those
required by the algorithm.

Some of the latter ones can be considered temporarily (.firstChild, .isHidden);
i.e. they are merely required while executing the algorithm. Others can be
considered more permanent (.section, .outline, .heading); i.e. they link to
different parts of the created outline.

```
begin class Node
dom-specific:

  //- return first child or null
  Node firstChild()

  //- return next sibling or null
  Node nextSibling()

required:

  //- has the 'hidden' attribute set?
  bool isHidden()
  
  //- corresponds to a /h[1-6]/ element?
  bool isHeadingContent()

  //- used like an instanceof operator
  const bool isImpliedHeading = false

  //- corresponds to an X element?
  //  X in [article,aside,nav,section] element?
  bool isSectioningContent()

  //- corresponds to an X element?
  //  X in [blockquote,body,details,dialog,fieldset,figure,td]
  bool isSectioningRoot()

  //- calculate the rank of the current heading
  int rank()

  //- this sectioning root is child to this section
  Section parentSection = null

  //- node is located in this section
  //- what about (node != node.section.associatedNode)?
  Section section = null

  //- node has this outline
  Outline outline = null
end
```
