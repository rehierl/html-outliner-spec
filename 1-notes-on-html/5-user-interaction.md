
[5. User interaction](https://w3c.github.io/html/editing.html#editing)

<a href="hidden-attribute">
[5.1. The **hidden** attribute](https://w3c.github.io/html/editing.html#the-hidden-attribute)</a>

* all html elements may have the hidden content attribute set
* it is a [boolean attribute](#boolean-attributes) (see 2.4.2)
* specifies, that an element is not yet, or no longer directly relevant to the
  page's current state, or it is used to declare content to be reused by other
  parts of the page - as opposed to being directly accessed by the user
* user agents should not render such elements
* typically implemented using style sheets (CSS)
* if something is marked hidden, it is hidden from all presentations
  including, for instance, screen readers
* non-hidden elements must not link to hidden ones
