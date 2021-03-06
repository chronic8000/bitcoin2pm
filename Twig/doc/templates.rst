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
values when the template is evaluated, and tags, which control the logic of
the template.

Below is a minimal template that illustrates a few basics. We will cover the
details later in that document:

.. code-block:: jinja

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
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

Variables
---------

The application passes variables to the templates you can mess around in the
template. Variables may have attributes or elements on them you can access
too. How a variable looks like, heavily depends on the application providing
those.

You can use a dot (``.``) to access attributes of a variable, alternative the
so-called "subscript" syntax (``[]``) can be used. The following lines do the
same:

.. code-block:: jinja

    {{ foo.bar }}
    {{ foo['bar'] }}

.. note::

    It's important to know that the curly braces are *not* part of the
    variable but the print statement. If you access variables inside tags
    don't put the braces around.

If a variable or attribute does not exist you will get back a ``null`` value
(which can be tested with the ``none`` expression).

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

   ``foo['bar']`` on the other hand works mostly the same with the a small
   difference in the order:

   * check if ``foo`` is an array and ``bar`` a valid element;
   * if not, return a ``null`` value.

   Using the alternative syntax is also useful to dynamically get attributes
   from arrays:

   .. code-block:: jinja

        foo[bar]

Twig always references the following variables:

* ``_self``: references the current template;
* ``_context``: references the current context;
* ``_charset``: references the current charset.

Filters
-------

Variables can by modified by **filters**. Filters are separated from the
variable by a pipe symbol (``|``) and may have optional arguments in
parentheses. Multiple filters can be chained. The output of one filter is
applied to the next.

``{{ name|striptags|title }}`` for example will remove all HTML tags from the
``name`` and title-cases it. Filters that accept arguments have parentheses
around the arguments, like a function call. This example will join a list by
commas: ``{{ list|join(', ') }}``.

The built-in filters section below describes all the built-in filters.

Comments
--------

To comment-out part of a line in a template, use the comment syntax ``{# ...
#}``. This is useful to comment out parts of the template for debugging or to
add information for other template designers or yourself:

.. code-block:: jinja

    {# note: disabled template because we no longer use this
        {% for user in users %}
            ...
        {% endfor %}
    #}

Whitespace Control
------------------

.. versionadded:: 1.1
    Tag level whitespace control was added in Twig 1.1.

The first newline after a template tag is removed automatically (like in PHP.)
Whitespace is not further modified by the template engine, so each whitespace
(spaces, tabs, newlines etc.) is returned unchanged.

Use the ``spaceless`` tag to remove whitespace between HTML tags:

.. code-block:: jinja

    {% spaceless %}
        <div>
            <strong>foo</strong>
        </div>
    {% endspaceless %}

    {# output will be <div><strong>foo</strong></div> #}

In addition to the spaceless tag you can also control whitespace on a per tag 
level.  By using the whitespace control modifier on your tags you can trim
leading and or trailing whitespace from any tag type:

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

For bigger sections it makes sense to mark a block ``raw``. For example to put
Twig syntax as example into a template you can use this snippet:

.. code-block:: jinja

    {% raw %}
      <ul>
      {% for item in seq %}
        <li>{{ item }}</li>
      {% endfor %}
      </ul>
    {% endraw %}

Template Inheritance
--------------------

The most powerful part of Twig is template inheritance. Template inheritance
allows you to build a base "skeleton" template that contains all the common
elements of your site and defines **blocks** that child templates can
override.

Sounds complicated but is very basic. It's easiest to understand it by
starting with an example.

Base Template
~~~~~~~~~~~~~

This template, which we'll call ``base.html``, defines a simple HTML skeleton
document that you might use for a simple two-column page. It's the job of
"child" templates to fill the empty blocks with content:

.. code-block:: jinja

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
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
          &copy; Copyright 2009 by <a href="http://domain.invalid/">you</a>.
        {% endblock %}
      </div>
    </body>
    </html>

In this example, the ``{% block %}`` tags define four blocks that child
templates can fill in. All the ``block`` tag does is to tell the template
engine that a child template may override those portions of the template.

Child Template
~~~~~~~~~~~~~~

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

The ``{% extends %}`` tag is the key here. It tells the template engine that
this template "extends" another template. When the template system evaluates
this template, first it locates the parent. The extends tag should be the
first tag in the template.

The filename of the template depends on the template loader. For example the
``Twig_Loader_Filesystem`` allows you to access other templates by giving the
filename. You can access templates in subdirectories with a slash:

.. code-block:: jinja

    {% extends "layout/default.html" %}

But this behavior can depend on the application embedding Twig. Note that
since the child template doesn't define the ``footer`` block, the value from
the parent template is used instead.

You can't define multiple ``{% block %}`` tags with the same name in the same
template. This limitation exists because a block tag works in "both"
directions. That is, a block tag doesn't just provide a hole to fill - it also
defines the content that fills the hole in the *parent*. If there were two
similarly-named ``{% block %}`` tags in a template, that template's parent
wouldn't know which one of the blocks' content to use.  Block names should
consist of alphanumeric characters, and underscores. Dashes are not permitted.

If you want to print a block multiple times you can however use the
``block`` function:

.. code-block:: jinja

    <title>{% block title %}{% endblock %}</title>
    <h1>{{ block('title') }}</h1>
    {% block body %}{% endblock %}

Like PHP, Twig does not support multiple inheritance. So you can only have one
extends tag called per rendering.

Parent Blocks
~~~~~~~~~~~~~

It's possible to render the contents of the parent block by using the ``parent``
function. This gives back the results of the parent block:

.. code-block:: jinja

    {% block sidebar %}
      <h3>Table Of Contents</h3>
      ...
      {{ parent() }}
    {% endblock %}

Named Block End-Tags
~~~~~~~~~~~~~~~~~~~~

Twig allows you to put the name of the block after the end tag for better
readability:

.. code-block:: jinja

    {% block sidebar %}
      {% block inner_sidebar %}
          ...
      {% endblock inner_sidebar %}
    {% endblock sidebar %}

However the name after the ``endblock`` word must match the block name.

Block Nesting and Scope
~~~~~~~~~~~~~~~~~~~~~~~

Blocks can be nested for more complex layouts. Per default, blocks have access
to variables from outer scopes:

.. code-block:: jinja

    {% for item in seq %}
      <li>{% block loop_item %}{{ item }}{% endblock %}</li>
    {% endfor %}

Block Shortcuts
~~~~~~~~~~~~~~~

For blocks with few content, it's possible to have a shortcut syntax. The
following constructs do the same:

.. code-block:: jinja

    {% block title %}
      {{ page_title|title }}
    {% endblock %}

.. code-block:: jinja

    {% block title page_title|title %}

Dynamic Inheritance
~~~~~~~~~~~~~~~~~~~

Twig supports dynamic inheritance by using a variable as the base template:

.. code-block:: jinja

    {% extends some_var %}

If the variable evaluates to a ``Twig_Template`` object, Twig will use it as
the parent template::

    // {% extends layout %}

    $layout = $twig->loadTemplate('some_layout_template.twig');

    $twig->display('template.twig', array('layout' => $layout));

Conditional Inheritance
~~~~~~~~~~~~~~~~~~~~~~~

As a matter of fact, the template name can be any valid expression. So, it's
also possible to make the inheritance mechanism conditional:

.. code-block:: jinja

    {% extends standalone ? "minimum.html" : "base.html" %}

In this example, the template will extend the "minimum.html" layout template
if the ``standalone`` variable evaluates to ``true``, and "base.html"
otherwise.

Import Context Behavior
-----------------------

Per default included templates are passed the current context.

The context that is passed to the included template includes variables defined
in the template:

.. code-block:: jinja

    {% for box in boxes %}
      {% include "render_box.html" %}
    {% endfor %}

The included template ``render_box.html`` is able to access ``box``.

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
the following chars (``>``, ``<``, ``&``, or ``"``) you **have to** escape it unless
the variable contains well-formed and trusted HTML. Escaping works by piping
the variable through the ``|e`` filter: ``{{ user.username|e }}``.

Working with Automatic Escaping
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Whether automatic escaping is enabled or not, you can mark a section of a
template to be escaped or not by using the ``autoescape`` tag:

.. code-block:: jinja

    {% autoescape true %}
      Everything will be automatically escaped in this block
    {% endautoescape %}

    {% autoescape false %}
      Everything will be outputed as is in this block
    {% endautoescape %}

    {% autoescape true js %}
      Everything will be automatically escaped in this block
      using the js escaping strategy
    {% endautoescape %}

When automatic escaping is enabled everything is escaped by default except for
values explicitly marked as safe. Those can be marked in the template by using
the ``|raw`` filter.

Functions returning template data (like macros and ``parent``) always return
safe markup.

.. note::

    Twig is smart enough to not escape an already escaped value by the
    ``escape`` filter.

.. note::

    The chapter for developers give more information about when and how
    automatic escaping is applied.

List of Control Structures
--------------------------

A control structure refers to all those things that control the flow of a
program - conditionals (i.e. ``if``/``elseif``/``else``), ``for``-loops, as well as
things like blocks. Control structures appear inside ``{% ... %}`` blocks.

For
~~~

Loop over each item in a sequence. For example, to display a list of users
provided in a variable called ``users``:

.. code-block:: jinja

    <h1>Members</h1>
    <ul>
      {% for user in users %}
        <li>{{ user.username|e }}</li>
      {% endfor %}
    </ul>

.. note::

    A sequence can be either an array or an object implementing the
    ``Traversable`` interface.

If you do need to iterate over a sequence of numbers, you can use the ``..``
operator:

.. code-block:: jinja

    {% for i in 0..10 %}
      * {{ i }}
    {% endfor %}

The above snippet of code would print all numbers from 0 to 10.

It can be also useful with letters:

.. code-block:: jinja

    {% for letter in 'a'..'z' %}
      * {{ letter }}
    {% endfor %}

The ``..`` operator can take any expression at both sides:

.. code-block:: jinja

    {% for letter in 'a'|upper..'z'|upper %}
      * {{ letter }}
    {% endfor %}

.. tip:

    If you need a step different from 1, you can use the ``range`` function
    instead.

Inside of a ``for`` loop block you can access some special variables:

===================== =============================================================
Variable              Description
===================== =============================================================
``loop.index``        The current iteration of the loop. (1 indexed)
``loop.index0``       The current iteration of the loop. (0 indexed)
``loop.revindex``     The number of iterations from the end of the loop (1 indexed)
``loop.revindex0``    The number of iterations from the end of the loop (0 indexed)
``loop.first``        True if first iteration
``loop.last``         True if last iteration
``loop.length``       The number of items in the sequence
``loop.parent``       The parent context
===================== =============================================================

.. note::

    The ``loop.length``, ``loop.revindex``, ``loop.revindex0``, and
    ``loop.last`` variables are only available for PHP arrays, or objects that
    implement the ``Countable`` interface.

.. note::

    Unlike in PHP it's not possible to ``break`` or ``continue`` in a loop.

If no iteration took place because the sequence was empty, you can render a
replacement block by using ``else``:

.. code-block:: jinja

    <ul>
      {% for user in users %}
        <li>{{ user.username|e }}</li>
      {% else %}
        <li><em>no user found</em></li>
      {% endfor %}
    </ul>

By default, a loop iterates over the values of the sequence. You can iterate
on keys by using the ``keys`` filter:

.. code-block:: jinja

    <h1>Members</h1>
    <ul>
      {% for key in users|keys %}
        <li>{{ key }}</li>
      {% endfor %}
    </ul>

You can also access both keys and values:

.. code-block:: jinja

    <h1>Members</h1>
    <ul>
      {% for key, user in users %}
        <li>{{ key }}: {{ user.username|e }}</li>
      {% endfor %}
    </ul>

If
~~

The ``if`` statement in Twig is comparable with the if statements of PHP. In
the simplest form you can use it to test if a variable is not empty:

.. code-block:: jinja

    {% if users %}
      <ul>
        {% for user in users %}
          <li>{{ user.username|e }}</li>
        {% endfor %}
      </ul>
    {% endif %}

.. note::

    If you want to test if the variable is defined, use ``if users is
    defined`` instead.

For multiple branches ``elseif`` and ``else`` can be used like in PHP. You can use
more complex ``expressions`` there too:

.. code-block:: jinja

    {% if kenny.sick %}
        Kenny is sick.
    {% elseif kenny.dead %}
        You killed Kenny!  You bastard!!!
    {% else %}
        Kenny looks okay --- so far
    {% endif %}

Macros
~~~~~~

Macros are comparable with functions in regular programming languages. They
are useful to put often used HTML idioms into reusable elements to not repeat
yourself.

Here is a small example of a macro that renders a form element:

.. code-block:: jinja

    {% macro input(name, value, type, size) %}
        <input type="{{ type|default('text') }}" name="{{ name }}" value="{{ value|e }}" size="{{ size|default(20) }}" />
    {% endmacro %}

Macros differs from native PHP functions in a few ways:

* Default argument values are defined by using the ``default`` filter in the
  macro body;

* Arguments of a macro are always optional.

But as PHP functions, macros don't have access to the current template
variables.

.. tip::

    You can pass the whole context as an argument by using the special
    ``_context`` variable.

Macros can be defined in any template, and need to be "imported" before being
used (see the Import section for more information):

.. code-block:: jinja

    {% import "forms.html" as forms %}

The above ``import`` call imports the "forms.html" file (which can contain only
macros, or a template and some macros), and import the functions as items of
the ``forms`` variable.

The macro can then be called at will:

.. code-block:: jinja

    <p>{{ forms.input('username') }}</p>
    <p>{{ forms.input('password', none, 'password') }}</p>

If macros are defined and used in the same template, you can use the
special ``_self`` variable, without importing them:

.. code-block:: jinja

    <p>{{ _self.input('username') }}</p>

When you want to use a macro in another one from the same file, use the ``_self``
variable:

.. code-block:: jinja

    {% macro input(name, value, type, size) %}
      <input type="{{ type|default('text') }}" name="{{ name }}" value="{{ value|e }}" size="{{ size|default(20) }}" />
    {% endmacro %}

    {% macro wrapped_input(name, value, type, size) %}
        <div class="field">
            {{ _self.input(name, value, type, size) }}
        </div>
    {% endmacro %}

When the macro is defined in another file, you need to import it:

.. code-block:: jinja

    {# forms.html #}

    {% macro input(name, value, type, size) %}
      <input type="{{ type|default('text') }}" name="{{ name }}" value="{{ value|e }}" size="{{ size|default(20) }}" />
    {% endmacro %}

    {# shortcuts.html #}

    {% macro wrapped_input(name, value, type, size) %}
        {% import "forms.html" as forms %}
        <div class="field">
            {{ forms.input(name, value, type, size) }}
        </div>
    {% endmacro %}

Filters
~~~~~~~

Filter sections allow you to apply regular Twig filters on a block of template
data. Just wrap the code in the special ``filter`` section:

.. code-block:: jinja

    {% filter upper %}
      This text becomes uppercase
    {% endfilter %}

You can also chain filters:

.. code-block:: jinja

    {% filter lower|escape %}
      <strong>SOME TEXT</strong>
    {% endfilter %}

It should return ``&lt;strong&gt;some text&lt;/strong&gt;``.

Assignments
~~~~~~~~~~~

Inside code blocks you can also assign values to variables. Assignments use
the ``set`` tag and can have multiple targets:

.. code-block:: jinja

    {% set foo = 'foo' %}

    {% set foo = [1, 2] %}

    {% set foo = {'foo': 'bar'} %}

    {% set foo = 'foo' ~ 'bar' %}

    {% set foo, bar = 'foo', 'bar' %}

The ``set`` tag can also be used to 'capture' chunks of text:

.. code-block:: jinja

    {% set foo %}
      <div id="pagination">
        ...
      </div>
    {% endset %}

.. caution::

    If you enable automatic output escaping, Twig will only consider the
    content to be safe when capturing chunks of text.

Extends
~~~~~~~

The ``extends`` tag can be used to extend a template from another one. You can
have multiple of them in a file but only one of them may be executed at the
time. There is no support for multiple inheritance. See the section about
Template inheritance above for more information.

Block
~~~~~

Blocks are used for inheritance and act as placeholders and replacements at
the same time. They are documented in detail as part of the section about
Template inheritance above.

Include
~~~~~~~

The ``include`` statement is useful to include a template and return the
rendered content of that file into the current namespace:

.. code-block:: jinja

    {% include 'header.html' %}
      Body
    {% include 'footer.html' %}

Included templates have access to the variables of the active context.

You can add additional variables by passing them after the ``with`` keyword:

.. code-block:: jinja

    {# the foo template will have access to the variables from the current context and the foo one #}
    {% include 'foo' with {'foo': 'bar'} %}

    {% set vars = {'foo': 'bar'} %}
    {% include 'foo' with vars %}

You can disable access to the context by appending the ``only`` keyword:

.. code-block:: jinja

    {# only the foo variable will be accessible #}
    {% include 'foo' with {'foo': 'bar'} only %}

.. code-block:: jinja

    {# no variable will be accessible #}
    {% include 'foo' only %}

.. tip::

    When including a template created by an end user, you should consider
    sandboxing it. More information in the "Twig for Developers" chapter.

The template name can be any valid Twig expression:

.. code-block:: jinja

    {% include some_var %}
    {% include ajax ? 'ajax.html' : 'not_ajax.html' %}

And if the expression evaluates to a ``Twig_Template`` object, Twig will use it
directly::

    // {% include template %}

    $template = $twig->loadTemplate('some_template.twig');

    $twig->loadTemplate('template.twig')->display(array('template' => $template));

Import
~~~~~~

Twig supports putting often used code into macros. These macros can go into
different templates and get imported from there.

There are two ways to import templates. You can import the complete template
into a variable or request specific macros from it.

Imagine we have a helper module that renders forms (called ``forms.html``):

.. code-block:: jinja

    {% macro input(name, value, type, size) %}
        <input type="{{ type|default('text') }}" name="{{ name }}" value="{{ value|e }}" size="{{ size|default(20) }}" />
    {% endmacro %}

    {% macro textarea(name, value, rows) %}
        <textarea name="{{ name }}" rows="{{ rows|default(10) }}" cols="{{ cols|default(40) }}">{{ value|e }}</textarea>
    {% endmacro %}

The easiest and most flexible is importing the whole module into a variable.
That way you can access the attributes:

.. code-block:: jinja

    {% import 'forms.html' as forms %}

    <dl>
        <dt>Username</dt>
        <dd>{{ forms.input('username') }}</dd>
        <dt>Password</dt>
        <dd>{{ forms.input('password', none, 'password') }}</dd>
    </dl>
    <p>{{ forms.textarea('comment') }}</p>

Alternatively you can import names from the template into the current
namespace:

.. code-block:: jinja

    {% from 'forms.html' import input as input_field, textarea %}

    <dl>
        <dt>Username</dt>
        <dd>{{ input_field('username') }}</dd>
        <dt>Password</dt>
        <dd>{{ input_field('password', type='password') }}</dd>
    </dl>
    <p>{{ textarea('comment') }}</p>

Importing is not needed if the macros and the template are defined in the same
file; use the special ``_self`` variable instead:

.. code-block:: jinja

    {# index.html template #}

    {% macro textarea(name, value, rows) %}
        <textarea name="{{ name }}" rows="{{ rows|default(10) }}" cols="{{ cols|default(40) }}">{{ value|e }}</textarea>
    {% endmacro %}

    <p>{{ _self.textarea('comment') }}</p>

But you can still create an alias by importing from the ``_self`` variable:

.. code-block:: jinja

    {# index.html template #}

    {% macro textarea(name, value, rows) %}
        <textarea name="{{ name }}" rows="{{ rows|default(10) }}" cols="{{ cols|default(40) }}">{{ value|e }}</textarea>
    {% endmacro %}

    {% import _self as forms %}

    <p>{{ forms.textarea('comment') }}</p>

Expressions
-----------

Twig allows basic expressions everywhere. These work very similar to regular
PHP and even if you're not working with PHP you should feel comfortable with
it.

The operator precedence is as follows, with the lowest-precedence operators
listed first: ``or``, ``and``, ``==``, ``!=``, ``<``, ``>``, ``>=``, ``<=``, ``in``, ``+``, ``-``,
``~``, ``*``, ``/``, ``%``, ``//``, ``is``, ``..``, and ``**``.

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

For ``if`` statements, ``for`` filtering or ``if`` expressions it can be useful to
combine multiple expressions:

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

Tests
~~~~~

The ``is`` operator performs tests. Tests can be used to test a variable agains