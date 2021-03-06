Twig for Template Designers
===========================

This document describes the syntax and semantics of the template engine and
will be most useful as reference to those creating Twig templates.

Synopsis
--------

A template is simply a text file. It can generate any text-based format (HTML,
XML, CSV, LaTeX, etc.). It doesn't have a specific extension, ``.html`` or
``.xml`` are just fine.

A template contains **variables** or **expressions**, which get replaced with
values when the template is evaluated, and **tags**, which control the logic
of the template.

Below is a minimal template that illustrates a few basics. We will cover the
details later on:

.. code-block:: html+jinja

    <!DOCTYPE html>
    <html>
        <head>
            <title>My Webpage</title>
        </head>
        <body>
            <ul id="navigation">
            {% for item in navigation %}
                <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
            {% endfor %}
            </ul>

            <h1>My Webpage</h1>
            {{ a_variable }}
        </body>
    </html>

There are two kinds of delimiters: ``{% ... %}`` and ``{{ ... }}``. The first
one is used to execute statements such as for-loops, the latter prints the
result of an expression to the template.

IDEs Integration
----------------

Many IDEs support syntax highlighting and auto-completion for Twig:

* *Textmate* via the `Twig bundle`_
* *Vim* via the `Jinja syntax plugin`_
* *Netbeans* via the `Twig syntax plugin`_
* *PhpStorm* (native as of 2.1)
* *Eclipse* via the `Twig plugin`_
* *Sublime Text* via the `Twig bundle`_
* *GtkSourceView* via the `Twig language definition`_ (used by gedit and other projects)

Variables
---------

The application passes variables to the templates you can mess around in the
template. Variables may have attributes or elements on them you can access
too. How a variable looks like heavily depends on the application providing
those.

You can use a dot (``.``) to access attributes of a variable (methods or
properties of a PHP object, or items of a PHP array), or the so-called
"subscript" syntax (``[]``):

.. code-block:: jinja

    {{ foo.bar }}
    {{ foo['bar'] }}

.. note::

    It's important to know that the curly braces are *not* part of the
    variable but the print statement. If you access variables inside tags
    don't put the braces around.

If a variable or attribute does not exist you will get back a ``null`` value.

.. sidebar:: Implementation

    For convenience sake ``foo.bar`` does the following things on the PHP
    layer:

    * check if ``foo`` is an array and ``bar`` a valid element;
    * if not, and if ``foo`` is an object, check that ``bar`` is a valid property;
    * if not, and if ``foo`` is an object, check that ``bar`` is a valid method
      (even if ``bar`` is the constructor - use ``__construct()`` instead);
    * if not, and if ``foo`` is an object, check that ``getBar`` is a valid method;
    * if not, and if ``foo`` is an object, check that ``isBar`` is a valid method;
    * if not, return a ``null`` value.

    ``foo['bar']`` on the other hand only works with PHP arrays:

    * check if ``foo`` is an array and ``bar`` a valid element;
    * if not, return a ``null`` value.

.. note::

    If you want to get a dynamic attribute on a variable, use the
    :doc:`attribute<functions/attribute>` function instead.

Global Variables
~~~~~~~~~~~~~~~~

The following variables are always available in templates:

* ``_self``: references the current template;
* ``_context``: references the current context;
* ``_charset``: references the current charset.

Setting Variables
~~~~~~~~~~~~~~~~~

You can assign values to variables inside code blocks. Assignments use the
:doc:`set<tags/set>` tag:

.. code-block:: jinja

    {% set foo = 'foo' %}
    {% set foo = [1, 2] %}
    {% set foo = {'foo': 'bar'} %}

Filters
-------

Variables can be modified by **filters**. Filters are separated from the
variable by a pipe symbol (``|``) and may have optional arguments in
parentheses. Multiple filters can be chained. The output of one filter is
applied to the next.

The following example removes all HTML tags from the ``name`` and title-cases
it:

.. code-block:: jinja

    {{ name|striptags|title }}

Filters that accept arguments have parentheses around the arguments. This
example will join a list by commas:

.. code-block:: jinja

    {{ list|join(', ') }}

To apply a filter on a section of code, wrap it with the
:doc:`filter<tags/filter>` tag:

.. code-block:: jinja

    {% filter upper %}
      This text becomes uppercase
    {% endfilter %}

Go to the :doc:`filters<filters/index>` page to learn more about the built-in
filters.

Functions
---------

Functions can be called to generate content. Functions are called by their
name followed by parentheses (``()``) and may have arguments.

For instance, the ``range`` function returns a list containing an arithmetic
progression of integers:

.. code-block:: jinja

    {% for i in range(0, 3) %}
        {{ i }},
    {% endfor %}

Go to the :doc:`functions<functions/index>` page to learn more about the
built-in functions.

Control Structure
-----------------

A control structure refers to all those things that control the flow of a
program - conditionals (i.e. ``if``/``elseif``/``else``), ``for``-loops, as
well as things like blocks. Control structures appear inside ``{% ... %}``
blocks.

For example, to display a list of users provided in a variable called
``users``, use the :doc:`for<tags/for>` tag:

.. code-block:: jinja

    <h1>Members</h1>
    <ul>
        {% for user in users %}
            <li>{{ user.username|e }}</li>
        {% endfor %}
    </ul>

The :doc:`if<tags/if>` tag can be used to test an expression:

.. code-block:: jinja

    {% if users|length > 0 %}
        <ul>
            {% for user in users %}
                <li>{{ user.username|e }}</li>
            {% endfor %}
        </ul>
    {% endif %}

Go to the :doc:`tags<tags/index>` page to learn more about the built-in tags.

Comments
--------

To comment-out part of a line in a template, use the comment syntax ``{# ...
#}``. This is useful for debugging or to add information for other template
designers or yourself:

.. code-block:: jinja

    {# note: disabled template because we no longer use this
        {% for user in users %}
            ...
        {% endfor %}
    #}

Including other Templates
-------------------------

The :doc:`include<tags/include>` tag is useful to include a template and
return the rendered content of that template into the current one:

.. code-block:: jinja

    {% include 'sidebar.html' %}

Per default included templates are passed the current context.

The context that is passed to the included template includes variables defined
in the template:

.. code-block:: jinja

    {% for box in boxes %}
        {% include "render_box.html" %}
    {% endfor %}

The included template ``render_box.html`` is able to access ``box``.

The filename of the template depends on the template loader. For instance, the
``Twig_Loader_Filesystem`` allows you to access other templates by giving the
filename. You can access templates in subdirectories with a slash:

.. code-block:: jinja

    {% include "sections/articles/sidebar.html" %}

This behavior depends on the application embedding Twig.

Template Inheritance
--------------------

The most powerful part of Twig is template inheritance. Template inheritance
allows you to build a base "skeleton" template that contains all the common
elements of your site and defines **blocks** that child templates can
override.

Sounds complicated but is very basic. It's easiest to understand it by
starting with an example.

Let's define a base template, ``base.html``, which defines a simple HTML
skeleton document that you might use for a simple two-column page:

.. code-block:: html+jinja

    <!DOCTYPE html>
    <html>
        <head>
            {% block head %}
                <link rel="stylesheet" href="style.css" />
                <title>{% block title %}{% endblock %} - My Webpage</title>
            {% endblock %}
        </head>
        <body>
            <div id="content">{% block content %}{% endblock %}</div>
            <div id="footer">
                {% block footer %}
                    &copy; Copyright 2011 by <a href="http://domain.invalid/">you</a>.
                {% endblock %}
            </div>
        </body>
    </html>

In this example, the :doc:`{% block %}<tags/block>` tags define four blocks
that child templates can fill in. All the ``block`` tag does is to tell the
template engine that a child template may override those portions of the
template.

A child template might look like this:

.. code-block:: jinja

    {% extends "base.html" %}

    {% block title %}Index{% endblock %}
    {% block head %}
        {{ parent() }}
        <style type="text/css">
            .important { color: #336699; }
        </style>
    {% endblock %}
    {% block content %}
        <h1>Index</h1>
        <p class="important">
            Welcome on my awesome homepage.
        </p>
    {% endblock %}

The :doc:`{% extends %}<tags/extends>` tag is the key here. It tells the
template engine that this template "extends" another template. When the
template system evaluates this template, first it locates the parent. The
extends tag should be the first tag in the template.

Note that since the child template doesn't define the ``footer`` block, the
value from the parent template is used instead.

It's possible to render the contents of the parent block by using the
:doc:`parent<functions/parent>` function. This gives back the results of the
parent block:

.. code-block:: jinja

    {% block sidebar %}
        <h3>Table Of Contents</h3>
        ...
        {{ parent() }}
    {% endblock %}

.. tip::

    The documentation page for the :doc:`extends<tags/extends>` tag describes
    more advanced features like block nesting, scope, dynamic inheritance, and
    conditional inheritance.

HTML Escaping
-------------

When generating HTML from templates, there's always a risk that a variable
will include characters that affect the resulting HTML. There are two
approaches: manually escaping each variable or automatically escaping
everything by default.

Twig supports both, automatic escaping is enabled by default.

.. note::

    Automatic escaping is only supported if the *escaper* extension has been
    enabled (which is the default).

Working with Manual Escaping
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If manual escaping is enabled it's **your** responsibility to escape variables
if needed. What to escape? If you have a variable that *may* include any of
the following chars (``>``, ``<``, ``&``, or ``"``) you **have to** escape it
unless the variable contains well-formed and trusted HTML. Escaping works by
piping the variable through the :doc:`escape<filters/escape>` or ``e`` filter:

.. code-block:: jinja

    {{ user.username|e }}
    {{ user.username|e('js') }}

Working with Automatic Escaping
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Whether automatic escaping is enabled or not, you can mark a section of a
template to be escaped or not by using the :doc:`autoescape<tags/autoescape>`
tag:

.. code-block:: jinja

    {% autoescape true %}
        Everything will be automatically escaped in this block
    {% endautoescape %}

Escaping
--------

It is sometimes desirable or even necessary to have Twig ignore parts it would
otherwise handle as variables or blocks. For example if the default syntax is
used and you want to use ``{{`` as raw string in the template and not start a
variable you have to use a trick.

The easiest way is to output the variable delimiter (``{{``) by using a variable
expression:

.. code-block:: jinja

    {{ '{{' }}

For bigger sections it makes sense to mark a block :doc:`raw<tags/raw>`.

Macros
------

Macros are comparable with functions in regular programming languages. They
are useful to put often used HTML idioms into reusable elements to not repeat
yourself.

A macro is defined via the :doc:`macro<tags/macro>` tag. Here is a small
example of a macro that renders a form element:

.. code-block:: jinja

    {% macro input(name, value, type, size) %}
        <input type="{{ type|default('text') }}" name="{{ name }}" value="{{ value|e }}" size="{{ size|default(20) }}" />
    {% endmacro %}

Macros can be defined in any template, and need to be "imported" before being
used via the :doc:`import<tags/import>` tag:

.. code-block:: jinja

    {% import "forms.html" as forms %}

    <p>{{ forms.input('username') }}</p>

Alternatively you can import names from the template into the current
namespace via the :doc:`from<tags/from>` tag:

.. code-block:: jinja

    {% from 'forms.html' import input as input_field, textarea %}

    <dl>
        <dt>Username</dt>
        <dd>{{ input_field('username') }}</dd>
        <dt>Password</dt>
        <dd>{{ input_field('password', type='password') }}</dd>
    </dl>
    <p>{{ textarea('comment') }}</p>

Expressions
-----------

Twig allows expressions everywhere. These work very similar to regular PHP and
even if you're not working with PHP you should feel comfortable with it.

.. note::

    The operator precedence is as follows, with the lowest-precedence
    operators listed first: ``or``, ``and``, ``==``, ``!=``, ``<``, ``>``,
    ``>=``, ``<=``, ``in``, ``+``, ``-``, ``~``, ``*``, ``/``, ``%``, ``//``,
    ``is``, ``..``, and ``**``.

Literals
~~~~~~~~

The simplest form of expressions are literals. Literals are representations
for PHP types such as strings, numbers, and arrays. The following literals
exist:

* ``"Hello World"``: Everything between two double or single quotes is a
  string. They are useful whenever you need a string in the template (for
  example as arguments to function calls, filters or just to extend or
  include a template).

* ``42`` / ``42.23``: Integers and floating point numbers are created by just
  writing the number down. If a dot is present the number is a float,
  otherwise an integer.

* ``["foo", "bar"]``: Arrays are defined by a sequence of expressions
  separated by a comma (``,``) and wrapped with squared brackets (``[]``).

* ``{"foo": "bar"}``: Hashes are defined by a list of keys and values
  separated by a comma (``,``) and wrapped with curly braces (``{}``). A value
  can be any valid expression.

* ``true`` / ``false``: ``true`` represents the true value, ``false``
  represents the false value.

* ``none``: ``none`` represents no specific value (the equivalent of ``null`` in
  PHP). This is the value returned when a variable does not exist.

Arrays and hashes can be nested:

.. code-block:: jinja

    {% set foo = [1, {"foo": "bar"}] %}

Math
~~~~

Twig allows you to calculate with values. This is rarely useful in templates
but exists for completeness' sake. The following operators are supported:

* ``+``: Adds two objects together (the operands are casted to numbers). ``{{
  1 + 1 }}`` is ``2``.

* ``-``: Substracts the second number from the first one. ``{{ 3 - 2 }}`` is
  ``1``.

* ``/``: Divides two numbers. The return value will be a floating point
  number. ``{{ 1 / 2 }}`` is ``{{ 0.5 }}``.

* ``%``: Calculates the remainder of an integer division. ``{{ 11 % 7 }}`` is
  ``4``.

* ``//``: Divides two numbers and returns the truncated integer result. ``{{
  20 // 7 }}`` is ``2``.

* ``*``: Multiplies the left operand with the right one. ``{{ 2 * 2 }}`` would
  return ``4``.

* ``**``: Raises the left operand to the power of the right operand. ``{{ 2**3
  }}`` would return ``8``.

Logic
~~~~~

You can combine multiple expressions with the following operators:

* ``and``: Returns true if the left and the right operands are both true.

* ``or``: Returns true if the left or the right operand is true.

* ``not``: Negates a statement.

* ``(expr)``: Groups an expression.

Comparisons
~~~~~~~~~~~

The following comparison operators are supported in any expression: ``==``,
``!=``, ``<``, ``>``, ``>=``, and ``<=``.

Containment Operator
~~~~~~~~~~~~~~~~~~~~

The ``in`` operator performs containment test.

It returns ``true`` if the left operand is contained in the right:

.. code-block:: jinja

    {# returns true #}

    {{ 1 in [1, 2, 3] }}

    {{ 'cd' in 'abcde' }}

.. tip::

    You can use this filter to perform a containment test on strings, arrays,
    or objects implementing the ``Traversable`` interface.

To perform a negative test, use the ``not in`` operator:

.. code-block:: jinja

    {% if 1 not in [1, 2, 3] %}

    {# is equivalent to #}
    {% if not (1 in [1, 2, 3]) %}

Test Operator
~~~~~~~~~~~~~

The ``is`` operator performs tests. Tests can be used to test a variable against
a common expression. The right operand is name of the test:

.. code-block:: jinja

    {# find out if a variable is odd #}

    {{ name is odd }}

Tests can accept arguments too:

.. code-block:: jinja

    {% if loop.index is divisibleby(3) %}

Tests can be negated by using the ``not in`` operator:

.. code-block:: jinja

    {% if loop.index is not divisibleby(3) %}

    {# is equivalent to #}
    {% if not (loop.index is divisibleby(3)) %}

Go to the :doc:`tests<tests/index>` page to learn more about the built-in
tests.

Other Operators
~~~~~~~~~~~~~~~

The following operators are very useful but don't fit into any of the other
categories:

* ``..``: Creates a sequence based on the operand before and after the
  operator (this is just syntactic sugar for the :doc:`range<functions/range>`
  function).

* ``|``: Applies a filter.

* ``~``: Converts all operands into strings and concatenates them. ``{{ "Hello
  " ~ name ~ "!" }}`` would return (assuming ``name`` is ``'John'``) ``Hello
  John!``.

* ``.``, ``[]``: Gets an attribute of an object.

* ``?:``: The PHP ternary operator: ``{{ foo ? 'yes' : 'no' }}``

Whitespace Control
------------------

.. versionadded:: 1.1
    Tag level whitespace control was added in Twig 1.1.

The first newline after a template tag is removed automatically (like in PHP.)
Whitespace is not further modified by the template engine, so each whitespace
(spaces, tabs, newlines etc.) is returned unchanged.

Use the ``spaceless`` tag to remove whitespace *between HTML tags*:

.. code-block:: jinja

    {% spaceless %}
        <div>
            <strong>foo</strong>
        </div>
    {% endspaceless %}

    {# output will be <div><strong>foo</strong></div> #}

In addition to the spaceless tag you can also control whitespace on a per tag
level. By using the whitespace control modifier on your tags, you can trim
leading and or trailing whitespace:

.. code-block:: jinja

    {% set value = 'no spaces' %}
    {#- No leading/trailing whitespace -#}
    {%- if true -%}
        {{- value -}}
    {%- endif -%}

    {# output 'no spaces' #}

The above sample shows the default whitespace control modifier, and how you can
use it to remove whitespace around tags.  Trimming space will consume all whitespace
for that side of the tag.  It is possible to use whitespace trimming on one side
of a tag:

.. code-block:: jinja

    {% set value = 'no spaces' %}
    <li>    {{- value }}    </li>

    {# outputs '<li>no spaces    </li>' #}

Extensions
----------

Twig can be easily extended.

If you are looking for new tags, filters, or functions, have a look at the Twig official
`extension repository`_.

If you want to create your own, read :doc:`extensions`.

.. _`Twig bundle`:              https://github.com/Anomareh/PHP-Twig.tmbundle
.. _`Jinja syntax plugin`:      http://jinja.pocoo.org/2/documentation/integration
.. _`Twig syntax plugin`:       https://github.com/blogsh/Twig-netbeans
.. _`Twig plugin`:              https://github.com/pulse00/Twig-Eclipse-Plugin
.. _`Twig language definition`: https://github.com/gabrielcorpse/gedit-twig-template-language
.. _`extension repository`:     http://github.com/fabpot/Twig-extensions
