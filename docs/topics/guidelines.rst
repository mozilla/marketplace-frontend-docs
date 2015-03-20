.. _guidelines:

Coding Guidelines
=================

This section is about coding guidelines, or coding style guide. Marketplace
developers, like many other Mozilla developers, can be strict on following
the style guide. We can be nitty and tell you to put periods at the end of
all your comments, add linebreaks to logically separate your code, or
alphabetize your CSS rules. This is to make our code consistent with itself
and improve readability and ease decisions how to style code.

When in doubt, just follow what the code looks like.

Mozilla Webdev Style Guide
~~~~~~~~~~~~~~~~~~~~~~~~~~

As a foundation, we follow Mozilla's web developer style guide. But we will
have additional guidelines on top of that.

- `JS Style Guide <http://mozweb.readthedocs.org/en/latest/reference/js-style.html>`_
- `HTML Style Guide <http://mozweb.readthedocs.org/en/latest/reference/html-style.html>`_
- `CSS Style Guide <http://mozweb.readthedocs.org/en/latest/reference/css-style.html>`_

Indenting
~~~~~~~~~

We use 2-space indents for HTML, 4-space indents everywhere else.

For a line of code that spans multiple lines, use a visual-based indent or
hanging-ident, preference for visual-based indents.

Visual Indent
-------------

.. code-block:: javascript

    // On the wrapped line, line the parameter to the start of the previous line's
    // function call.
    createShip(captain='Mal', address='Serenity',
               class='Firefly')

Hanging Indent
--------------

.. code-block:: javascript

    // The next line starts 4 spaces after the previous line.
    createShip(
        captain='Mal', address='Serenity', class='Firefly')


AMD Definitions
~~~~~~~~~~~~~~~

We make heavy use of AMD modules in the frontend.

- Docstring at the top of all modules that describes the module.
- The `define` line, the dependency list, and the function signature all begin
on separate lines.
- Generally, alphabetize dependencies in module definitions based on the name
of the dependency module.
- Keep with 80-chars. When a dependency list grows too long, line-break it
and use a visual-based indent.
- If there is a line break in the dependency list, match the line break in
the function signature.

For example:

.. code-block:: javascript

    /*
        Here is what this module is about.
    */
    define('myModule',
        ['foo', 'bar', ..., 'baz',
         'qux'],
        function(foo, bar, ..., baz,
                 qux) {
    });

Comments
~~~~~~~~

Keep comments short and concise. Make them span as few lines as possible. This
can be done by getting rid of excess words like definite articles and pronouns.

.. code-block:: javascript

    /* This comment is too long. */
    // This function is the centralized handler for all apps in a list.

    /* This comment is concise. */
    // Centralized handler for apps in list.

All comments end with a period.

Additional Guidelines
~~~~~~~~~~~~~~~~~~~~~

Some additional guidelines:

- When importing Nunjucks macros from a file, specify each macro rather than
  importing the whole macro file using the :code:`from "x.html" import x` syntax.
- When creating an event to trigger, prefix the event name by the module
  which is triggering it followed by two hyphens, and then the event name. For
  example :code:`builder--post-render`.
- Media queries should be defined at the end of CSS files in order of viewport
  size.
- Stylus variables should be defined all at the top of the file, not in the
  middle.
- Prefix Stylus classes by page or component (e.g., :code:`.big-dropdown-label`
  rather than just :code:`.label`).
- Try to contain as much view logic in JS as you can rather than Nunjucks
  templates. Nunjucks templates usually generate more code than JS.
