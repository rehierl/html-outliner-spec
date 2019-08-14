
[3.2.4. Content models](https://w3c.github.io/html/dom.html#content-models)

[3.2.4.2.2. **Flow content** (FC)](https://w3c.github.io/html/dom.html#flow-content)

* elements - **dialog, h1-6, section, div, text, ...**

```
a, abbr, address, area (if descendant of map), article, aside
audio, b, bdi, bdo, blockquote, br, button, canvas, cite, code, data
datalist, del, details, dfn, dialog, div, dl, em, embed, fieldset, figure
footer, form, h1, h2, h3, h4, h5, h6, header, hr, i, iframe, img
input, ins, kbd, label, link (if allowed inside body), main, map, mark
MathML, math, menu, meter, nav, noscript, object, ol, output, p, picture
pre, progress, q, ruby, s, samp, script, section, select, small, span
strong, style, sub, sup, svg, table, template, textarea, time, u, ul
var, video, wbr, text
```

*notes*

* a mere listing of elements to classify them as 'flow content'
* won't state what the actual meaning of flow content is

*notes*

* contains all HCEs, SCEs, SREs (`body` and `td` not included)
* **FCEs, but no PCEs** - address, article, aside, blockquote, details, dialog,
  div, dl, fieldset, figure, footer, form, h1-6, header, hr, main, menu, nav, ol,
  p, pre, section, style, table, ul - **especially** - HCEs, SCEs, and SREs
* **FCEs, but no PalpCEs** - area, br, datalist, del, dialog, hr, link, menu,
  noscript, picture, script, style, template, wbr

[3.2.4.2.3. **Sectioning content** (SC)](https://w3c.github.io/html/dom.html#sectioning-content)

* elements - **article, aside, nav, section**
* content that defines the scope of headings (3.2.4.2.4) and footers (4.3.8)
* each SC element **potentially has a heading and an outline**
* a sectioning root can also have an outline

*notes*

* content models - **FC** - nav and aside must not contain a main element

[4.3.9. **Sectioning roots** (SR)](https://w3c.github.io/html/sections.html#sectioning-roots)

* elements - **blockquote, body, details, dialog, fieldset, figure, td**
* inner sections and headings do not contribute to the outlines of their ancestors
* can have their own outlines

*notes*

* does not state, similar to SCEs, that they can have a heading
* content models - **FC** - additional requirements for details, fieldset and
  figure elements

[3.2.4.2.4. **Heading content** (HC)](https://w3c.github.io/html/dom.html#heading-content)

* see elements - **h1, h2, h3, h4, h5, h6** - see also 4.3.6
* defines the header of a section (whether explicitly marked up using
  SCs, or implied by the HC itself)

*notes*

* distinction between HC and HE? - HC is the content model (class) and HEs are
  the elements (instances of that class)

[3.2.4.2.5. **Phrasing content** (PC)](https://w3c.github.io/html/dom.html#phrasing-content-2)

* elements - **i, s, span, strong, sub, sup, text, ...**

```
a, abbr, area (if descendant of map), audio, b, bdi, bdo,
br, button, canvas, cite, code, data, datalist, del, dfn, em, embed, i,
iframe, img, input, ins, kbd, label, link (if allowed in body), map,
mark, MathML, math, meter, noscript, object, output, picture, progress,
q, ruby, s, samp, script, select, small, span, strong, sub, sup, svg,
template, textarea, time, u, var, video, wbr, text
```

* phrasing content is the text (i.e. text nodes or nothing) of a document
* including the elements that mark up that text at the intra-paragraph level
* may contain other PCE, but no FCE

*notes*

* contains **no** HCEs, SCEs or SREs
* **PCEs, but no FCEs** - all PCEs are also FCEs, but `(#FC > #PC)`
* **PCEs, but no PalpCEs** - area, br, datalist, del, link, noscript, picture,
  script, template, wbr

[3.2.4.2.8. **Palpable content** (PalpC)](https://w3c.github.io/html/dom.html#palpable-content-2)

* elements - **h1-6, i, s, section, strong, div, text, ...**

```
a, abbr, address, article, aside, audio (if controls present), b,
bdi, bdo, blockquote, button, canvas, cite, code, data, details, dfn, div,
dl (if children include one/more name-value group), em, embed,
fieldset, figure, footer, form, h1, h2, h3, h4, h5, h6, header, i,
iframe, img, input (if type-attrib not hidden), ins, kbd, label,
main, map, mark, MathML, math, meter, nav, object, ol (if children include one/more li),
output, p, pre, progress, q, ruby, s, samp,
section, select, small, span, strong, sub, sup, svg, table, textarea,
time, u, ul (if chilren include one/more li), var, video,
text (that is not inter-element white space)
```

* in general, elements whose content model allows any FC or PC should have at
  least one node in its contents that is palpable content and that does not have
  the hidden attribute specified.
* makes an element non-empty (i.e. text, audio, video, image, controls, canvas)
* not a hard requirement, i.e. may be temporarily empty

*notes*

* palpable - (DE) fühlbar, greifbar, konkret, ...
* contains all HCEs, SCEs, SREs (`body`, `dialog` and `td` not included)
* **PalpCEs, but no FCEs** - details (as the only element)
* **PalpCEs, but no PCEs** - address, article, aside, b, blockquote, details,
  div, dl, fieldset, figure, footer, form, h1-h6, header, main, nav, ol, p, pre,
  section, table, ul - **especially** - HCEs, SCEs, SREs
