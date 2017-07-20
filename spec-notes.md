
TODO - notes on certain aspects of the outline algorithm

* [W3C, HTML 5.2, Editor's Draft, 3 May 2017](https://w3c.github.io/html)
* these notes refer to [outliner-steps.md](./outliner-steps.md)

### step 4.2.1. and 4.11.

* leaves open whether to associate hidden elements with any section
* probably not

### step 4.11.

* 4.11. refers to "node" in general, which includes non-element nodes.
* 5. specifically refers to non-element nodes.
* in short: 4.11. should use the word "element" instead of "node"

### step 6.

* this step is not necessary, because it should be possible to use
  node.section.heading
* and node.section should have already been set by 4.11. and/or 5.
