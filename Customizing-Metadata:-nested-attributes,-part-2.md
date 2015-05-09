# Overview

This builds on the Customizing Metadata: nested attributes, part 1 but instead creates each nested attribute as a complete ActiveFedora:::Base object instead of using blank nodes.

You have a complex RDF data model that has includes descriptive metadata from one resource included in another. As example, let's say you have a book with multiple authors. The book is an RDF resource, and each author of the book is also its own RDF resource with a first and last name.

# The Book