
These are the outline algorithm's steps as defined in [4.3.9. Headings and sections](./outliner-4.3.9.md) and split up into single steps to be able to refer to each one of them:

* `html-specification > outliner-steps`
* [W3C, HTML Editor's Draft, 18 July 2017](https://w3c.github.io/html)
* [W3C, Working Draft, 18 July 2017](https://www.w3.org/TR/html52)

## [4.3.9.1 Creating an outline](https://w3c.github.io/html/sections.html#creating-an-outline)

The algorithm that must be followed during a walk of a DOM subtree rooted at a
sectioning content element or a sectioning root element to determine that
element's outline is as follows:

<table>
<!-- 1. init outline owner -->
<tr id="1"><td>1.</td><td>Le the <i>current outline owner</i> be null. (It holds the element whose outline is being created.)</td></tr>

<!-- 2. init current section -->
<tr id="2"><td>2.</td><td>Let <i>current section</i> be null. (It holds a pointer to a section, so that elements in the DOM can all be associated with a section.)</td></tr>

<!-- 3. init stack -->
<tr id="3"><td>3.</td><td>Create a stack to hold elements, which is used to handle nesting. Initialize this stack to empty.</td></tr>

<!-- 4. do tree walk -->
<tr id="4"><td>4.</td><td>Walk over the DOM in tree order, starting with the sectioning element or sectioning root element at the root of the subtree for which an outline is to be created, and trigger the first relevant step below for each element as the walk enters and exits it.</td></tr>

<!-- 4.1. exit the tos element -->
<tr><td>.</td><td>.</td></tr>
<tr id="4-1"><td>4.1.</td><td><b>When exiting an element, if that element is the element at the top of the stack</b></td></tr>
<tr id="4-1-1"><td>4.1.1.</td><td>Note - The element being exited is a heading content element or an element with a hidden attribute.</td></tr>
<tr id="4-1-2"><td>4.1.2.</td><td>Pop that element from the stack.</td></tr>

<!-- 4.2. the tos element is a heading or hidden -->
<tr><td>.</td><td>.</td></tr>
<tr id="4-2"><td>4.2.</td><td><b>If the top of the stack is a heading content element or an element with a hidden attribute.</b></td></tr>
<tr id="4-2-1"><td>4.2.1.</td><td>Do nothing</td></tr>

<!-- 4.3. enter a hidden element -->
<tr><td>.</td><td>.</td></tr>
<tr id="4-3"><td>4.3.</td><td><b>When entering an element with a hidden attribute.</b></td></tr>
<tr id="4-3-1"><td>4.3.1.</td><td>Push the element being entered onto the stack. (This causes the algorithm to skip that element and any descendants of the element.)</td></tr>

<!-- 4.4. enter a sectioning content element -->
<tr><td>.</td><td>.</td></tr>
<tr id="4-4"><td>4.4.</td><td><b>When entering a sectioning content element</b> - Run these steps:</td></tr>
<tr id="4-4-1"><td>4.4.1.</td><td>If the <i>current outline owner</i> is not null, run these substeps:</td></tr>
<tr id="4-4-1-1"><td>4.4.1.1.</td><td>If the <i>current section</i> has no heading, create an implied heading and let that be the heading of the current section.</td></tr>
<tr id="4-4-1-2"><td>4.4.1.2</td><td>Push <i>current outline owner</i> onto the stack.</td></tr>
<tr id="4-4-2"><td>4.4.2.</td><td>Let <i>current outline owner</i> be the element that is being entered.</td></tr>
<tr id="4-4-3"><td>4.4.3.</td><td>Let <i>current section</i> be a newly created section for the <i>current outline owner element</i></td></tr>
<tr id="4-4-4"><td>4.4.4.</td><td>Associate <i>current outline owner</i> with <i>current section</i>.</td></tr>
<tr id="4-4-5"><td>4.4.5.</td><td>Let there be a new outline for the new <i>current outline owner</i>, initialized with just the new <i>current section</i> as the only section in the outline.</td></tr>

<!-- 4.5. exit a sectioning content element, stack not empty -->
<tr><td>.</td><td>.</td></tr>
<tr id="4-5"><td>4.5.</td><td><b>When exiting a sectioning content element, if the stack is not empty</b> - Run these steps:</td></tr>
<tr id="4-5-1"><td>4.5.1.</td><td>If the <i>current section</i> has no heading, create an implied heading and let that be the heading for the <i>current section<i>.</td></tr>
<tr id="4-5-2"><td>4.5.2.</td><td>Pop the top element from the stack, and let the <i>current outline owner</i> be that element.</td></tr>
<tr id="4-5-3"><td>4.5.3.</td><td>Let <i>current section</i> be the last section in the outline of the <i>current outline owner</i> element.</td></tr>
<tr id="4-5-4"><td>4.5.4.</td><td>Append the outline of the sectioning content element being exited to the <i>current section</i>. (This does not change which section is the last section in the outline.)</td></tr>

<!-- 4.6. enter a sectioning root element -->
<tr><td>.</td><td>.</td></tr>
<tr id="4-6"><td>4.6.</td><td><b>When entering a sectioning root element</b> - Run these steps:</td></tr>
<tr id="4-6-1"><td>4.6.1.</td><td>If <i>current outline owner</i> is not null, push <i>current outline owner</i> onto the stack.</td></tr>
<tr id="4-6-2"><td>4.6.2.</td><td>Let <i>current outline owner</i> be the element that is being entered.</td></tr>
<tr id="4-6-3"><td>4.6.3.</td><td>Let <i>current outline owner</i>'s parent section be <i>current section</i>.</td></tr>
<tr id="4-6-4"><td>4.6.4.</td><td>Let <i>current section</i> be a newly created section for the <i>current outline owner</i> element.</td></tr>
<tr id="4-6-5"><td>4.6.5.</td><td>Let there be a new outline for the new <i>current outline owner</i>, initialized with just the new <i>current section</i> as the only section in the outline.</td></tr>

<!-- 4.7. exit a sectioning root element, stack not empty -->
<tr><td>.</td><td>.</td></tr>
<tr id="4-7"><td>4.7.</td><td><b>When exiting a sectioning root element, if the stack is not empty</b> - Run these steps:</td></tr>
<tr id="4-7-1"><td>4.7.1.</td><td>If the <i>current section</i> has no heading, create an implied heading and let that be the heading of the <i>current section</i>.</td></tr>
<tr id="4-7-2"><td>4.7.2.</td><td>Let <i>current section</i> be <i>current outline owner</i>'s parent section.</td></tr>
<tr id="4-7-3"><td>4.7.3.</td><td>Pop the top element from the stack, and let the <i>current outline owner</i> be that element.</td></tr>

<!-- 4.8. exit a sectioning element, stack is empty -->
<tr><td>.</td><td>.</td></tr>
<tr id="4-8"><td>4.8.</td><td><b>When exiting a sectioning content element or a sectioning root element (when the stack is empty)</b></td></tr>
<tr id="4-8-1"><td>4.8.1.</td><td>Note - The <i>current outline owner</i> is the element being exited, and it is the sectioning content element or a sectioning root element at the root of the subtree for which an outline is being generated.</td></tr>
<tr id="4-8-2"><td>4.8.2.</td><td>If the <i>current section</i> has no heading, create an implied heading and let that be the heading for the <i>current section</i>.</td></tr>
<tr id="4-8-3"><td>4.8.3.</td><td>Skip to the next step in the overall set of steps. (The walk is over.)</td></tr>

<!-- 4.9. enter a heading content element -->
<tr><td>.</td><td>.</td></tr>
<tr id="4-9"><td>4.9.</td><td><b>When entering a heading content element</b></td></tr>
<tr id="4-9-1"><td>4.9.1.</td><td>If the <i>current section</i> has no heading, let the element being entered be the heading for the <i>current section</i>.</td></tr>
<tr id="4-9-2"><td>4.9.2.</td><td>Otherwise,</td></tr>
<tr id="4-9-2-1"><td>4.9.2.1.</td><td>if the heading of the last section of the outline of the <i>current outline owner</i> is an implied heading, or</td></tr>
<tr id="4-9-2-2"><td>4.9.2.2.</td><td>if the element being entered has a rank equal to or higher than the heading of the last section of the outline of the <i>current outline owner</i>, then</td></tr>
<tr id="4-9-2-3"><td>4.9.2.3.</td><td>create a new section and append it to the outline of the <i>current outline owner</i> element, so that this new section is the new last section of that outline.</td></tr>
<tr id="4-9-2-4"><td>4.9.2.4.</td><td>Let <i>current section</i> be that new section.</td></tr>
<tr id="4-9-2-5"><td>4.9.2.5.</td><td>Let the element being entered be the new heading for the <i>current section</i>.</td></tr>
<tr id="4-9-3"><td>4.9.3.</td><td>Otherwise, run these substeps:</td></tr>
<tr id="4-9-3-1"><td>4.9.3.1.</td><td>Let the <i>candidate section</i> be <i>current section</i></td></tr>
<tr id="4-9-3-2"><td>4.9.3.2.</td><td>Heading loop:</td></tr>
<tr id="4-9-3-2-1"><td>4.9.3.2.1.</td><td>If the element being entered has a rank lower than the rank of the heading of the <i>candidate section</i>, then</td></tr>
<tr id="4-9-3-2-1-1"><td>4.9.3.2.1.1.</td><td>create a new section, and append it to <i>candidate section</i>. (This does not change which section is the last section in the outline.)</td></tr>
<tr id="4-9-3-2-1-2"><td>4.9.3.2.1.2.</td><td>Let <i>current section</i> be this new section.</td></tr>
<tr id="4-9-3-2-1-3"><td>4.9.3.2.1.3.</td><td>Let the element being entered be the new heading for the <i>current section</i>.</td></tr>
<tr id="4-9-3-2-1-4"><td>4.9.3.2.1.4.</td><td>Abort these substeps.</td></tr>
<tr id="4-9-3-2-2"><td>4.9.3.2.2.</td><td>Let <i>new candidate section</i> be the section that contains <i>candidate section</i> in the outline of the <i>current outline owner</i>.</td></tr>
<tr id="4-9-3-2-3"><td>4.9.3.2.3.</td><td>Let <i>candidate section</i> be <i>new candidate section</i>.</td></tr>
<tr id="4-9-3-2-4"><td>4.9.3.2.4.</td><td>Return to the step labeled <i>Heading loop</i> (4.9.3.2).</td></tr>
<tr id="4-9-4"><td>4.9.4.</td><td>Push the element being entered onto the stack. (This causes the algorithm to skip any descendants of the element.)</td></tr>
<tr id="4-9-5"><td>4.9.5.</td><td>Note - Recall that h1 has the highest rank, and h6 has the lowest rank.</td></tr>

<!-- 4.10. otherwise -->
<tr><td>.</td><td>.</td></tr>
<tr id="4-10"><td>4.10.</td><td><b>Otherwise</b></td></tr>
<tr id="4-10-1"><td>4.10.1.</td><td>Do nothing.</td></tr>

<!-- 4.11. associate nodes with sections -->
<tr><td>.</td><td>.</td></tr>
<tr id="4-11"><td>4.11.</td><td>In addition, whenever the walk exits a node, after doing the steps above, if the node is not associated with a section yet, associate the node with the section <i>current section</i>.</td></tr>

<!-- 5. associate non-elements with sections -->
<tr id="5"><td>5.</td><td>Associate all non-element nodes that are in the subtree for which an outline is being created with the section with which their parent element is associated.</td></tr>

<!-- 6. associate nodes with headings -->
<tr id="6"><td>6.</td><td>Associate all nodes in the subtree with the heading of the section with which they are associated, if any.</td></tr>
</table>

The tree of sections created by the algorithm above, or a proper subset thereof, must be used when generating document outlines, for example when generating tables of contents.
