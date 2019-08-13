
[2.4. Common microsyntaxes](https://w3c.github.io/html/infrastructure.html#common-microsyntaxes)

[2.4.2. Boolean attributes](https://w3c.github.io/html/infrastructure.html#sec-boolean-attributes)

* the presence of a boolean attribute on an element represents the true value,
  and the absence of the attribute represents the false value
* value must either be the empty string, or a value that is an ascii case-insensitive
  match for the attribute's canonical name with no leaing or trailing white space
* note - `true` and `false` as values are not allowed
* note - `checked`, `checked=""`, `checked="checked"` all represent the true value
* as soon as a boolean attribute is specified, it represents the true value -
  the actual value of the attribute is of no interest -
  to represent a false value, the attribute has to be omitted altogether
* as a consequence, each element always has an unlimited number of boolean
  attributes that have the false value - there is no undefined boolean attribute
* e.g. `<input checked disabled="">` - mixed styles are allowed

notes

* see [hidden attribute](#hidden-attribute)
