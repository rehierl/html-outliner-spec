
Issue - HCEs vs. implied sections.

* [W3C, HTML 5.2, Editor's Draft, 3 May 2017](https://w3c.github.io/html)
* [W3C, HTML 5.2, Working Draft, 9 May 2017](https://www.w3.org/TR/html52/)
* [Nu Html Checker](https://validator.w3.org/nu/)

*acronyms*

- sectioning element (SE) - can be an SRE or an SCE
- sectioning content (SC) - SC element (SCE)
- sectioning root (SR) - SR element (SRE)
- heading content (HC) - HC element (HCE)

*definitions*

- see [outliner-chapter](./outliner-chapter.md) for a definition of the
  expression "first inner/outer X element of element Y"

## About

[4.9.1.](./outliner-steps.md) tells to use the first heading found within an SE
as the current section's heading. - That is independend of which elements might
have appeared before this first heading element.

## Example fragments

**example**

```html
<body>
<p>A</p>
<h1>B</h1>
<p>C</p>
</body>
```

will produce the following outline:

```
1. heading 'B'
```

More importantly, both paragraphs *A* and *C* will be associated with heading
*B*. In case of *C*, that is certainly intended by the author. Automatically
associating paragraph *A* with heading *B* on the other hand could turn out to
be not intended. But that cannot be avoided.
