# blogc

A blog compiler.


## Design Notes

The main idea is simple: a source file is read by the compiler, and a result file is written by the compiler.

The source file must provide (almost) all the data needed to build the result file, including any variables. The result file is built using a template, that defines how the information provided by the source file should be used to generate a reasonable result file.

The compiler will always generate a single file, no matter how many source files are provided. If more than one source file is provided, the template must know how to convert them to a single result file.

The templates can define blocks. These are the block rules:

- If something is defined outside of blocks, it should be always used.
- If something is defined inside a block, it should only be used if the block matches the compiler argument expectations, e.g.:
    - ``single_source`` should be used if just one source file is provided.
    - ``multiple_sources`` should be used if more than one source file is provided, being used once for each source file.
    - ``multiple_sources_once`` should be used if more than one source file is provided, but only once.
- Template blocks can be defined multiple times in the same template, but can't be nested.

The variables defined in the source file are only available inside of blocks. If something does not depends on the source files, and is global, it must be hardcoded in the template, for the sake of simplicity.

The templates can use conditional statements: ``{% if variable %}`` and ``{% endif %}``. They check if a variable is defined or not. As variables are not available outside of blocks, these conditional statements can't be defined outside of blocks.

Variables are not available in ``multiple_sources_once`` blocks, because it is not possible to guess which source file to get the variables from.

As the compiler is output-agnostic, Atom feeds and sitemaps should be generated using templates as well.

The content defined in source files must be written as pre-formatted text. Make sure to enclose the content with ``<pre>`` and ``</pre>`` tags in your templates.

The compiler is designed to be easily used with any POSIX-compatible implementation of ``make``.


### Source file syntax

```
TITLE: My nice post
DATE: 2007-04-05T12:30-02:00
----
test content.

more test content.
```

If more than one source file is provided, and they have the ``DATE`` variable required by the compiler, it will be used to sort the source files, if needed. Otherwise, the file name will be used to sort the source files.

The ``DATE`` variable is an ISO-8601 date-time, with seconds, and always in UTC. If you want to show the date of your posts in your blog, you can use the ``DATE`` variable, but it won't be nicely formated, it will always be an ISO-8601 date-time.

Variables are single-line, and all the whitespace characters, including tabs, before the leading non-whitespace character and after the trailing non-whitespace character will be removed.

Pre-formatted content is available in template blocks as the ``CONTENT`` variable.


### Template file syntax

```html
<html>
    <head>
        {% block single_source %}
        <title>My cool blog >> {{ TITLE }}</title>
        {% endblock %}
        {% block multiple_sources %}
        <title>My cool blog - Main page</title>
        {% endblock %}
    </head>
    <body>
        <h1>My cool blog</h1>
        {% block single_source %}
        <h2>{{ TITLE }}</h2>
        {% if DATE %}<h4>Published in: {{ DATE }}</h4>{% endif %}
        <pre>{{ CONTENT }}</pre>
        {% endblock %}
        {% block multiple_sources_once %}<ul>{% endblock %}
        {% block multiple_sources %}<p><a href="{{ FILENAME }}.html">{{ TITLE }}</a>{% if DATE %} - {{ DATE }}{% endif %}</p>{% endblock %}
        {% block multiple_sources_once %}</ul>{% endblock %}
    </body>
</html>
```

This template would generate a list of posts, if multiple source files were provided, and single pages for each post, if only one source file was provided. The ``FILENAME`` variable is generated by the compiler, and is the source file name without its last extension.
