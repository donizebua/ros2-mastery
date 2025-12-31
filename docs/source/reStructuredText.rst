================
reStructuredText
================

.. # with overline, for parts

.. * with overline, for chapters

.. = for sections

.. - for subsections

.. ^ for subsubsections

.. " for paragraphs

ini adalah bla bla bla
bleblelbe
blublublub

-------------
Inline Markup
-------------

* *text*
* **text**
* ``text``

* This is a bulleted list.
* It has two items, the second
  item uses two lines.

1. This is a numbered list.
2. It has two items too.

#. This is a numbered list.
#. It has two items too.

* this is
* a list

  * with a nested list
  * and some subitems

* and here the parent list continues

term (up to a line of text)
   Definition of the term, which must be indented

   and can even consist of multiple paragraphs

next term
   Description.

| These lines are
| broken exactly like in
| the source file.

This is a normal text paragraph. The next paragraph is a code sample::

   It is not processed in any way, except
   that the indentation is removed.

   It can span multiple lines.

This is a normal text paragraph again.

>>> 1 + 1
2

+------------------------+------------+----------+----------+
| Header row, column 1   | Header 2   | Header 3 | Header 4 |
| (header rows optional) |            |          |          |
+========================+============+==========+==========+
| body row 1, column 1   | column 2   | column 3 | column 4 |
+------------------------+------------+----------+----------+
| body row 2             | ...        | ...      |          |
+------------------------+------------+----------+----------+

This is a paragraph that contains `a link`_.

.. _a link: https://domain.invalid/

`Link text <https://domain.invalid/>`__

