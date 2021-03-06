..  _syntax:

.. index::
   single: Types of tags

Types of tags
=============

The syntax is inspired by Mustache_. The template is created in Microsoft Word or other software that can save a docx.

.. _Mustache: https://mustache.github.io/

Introduction
------------

With this template (input.docx):

.. code-block:: text

    Hello {name} !

And given the following data (data.json):

.. code-block:: javascript

    {
        name:'John'
    }

docxtemplater will produce (output.docx):

.. code-block:: text

    Hello John !

Conditions
----------

Conditions start with a pound and end with a slash. That is `{#hasKitty}` starts a condition and `{/hasKitty}` ends it.

.. code-block:: text

    {#hasKitty}Cat’s name: {kitty}{/hasKitty}
    {#hasDog}Dog’s name: {dog}{/hasDog}

and this data:

.. code-block:: javascript

    {
        "first_name":"Jane",
        "hasKitty": true,
        "kitty": "Minie"
        "hasDog": false,
        "dog": null
    }

renders the following:

.. code-block:: text

    Cat’s name: Minie

For a more detailled explanation about Conditions, have a look at `Sections`_

You can also have "else" blocks with  `Inverted Sections`_

Loops
-----

In docxtemplater, conditions and loops use the same syntax called Sections

The following template:

.. code-block:: text

    {#products}
        {name}, {price} €
    {/products}

Given the following data:

.. code-block:: javascript

    {
        "products": [
            { name: "Windows", price: 100 },
            { name: "Mac OSX", price: 200 },
            { name: "Ubuntu", price: 0 }
        ]
    }

will result in:

.. code-block:: text

    Windows, 100 €
    Mac OSX, 200 €
    Ubuntu, 0€

To loop over an array containing primitive data (ex: string):

.. code-block:: javascript

   {
      "products": [
          "Windows",
          "Mac OSX",
          "Ubuntu"
      ]
   }

.. code-block:: text

   {#products} {.} {/products}

Will result in:

.. code-block:: text

    Windows Mac OSX Ubuntu

Sections
--------

A section begins with a pound and ends with a slash. That is {#person} begins a "person" section while {/person} ends it.

The section behaves in the following way:

+----------------------+---------------------------+------------------+
| Type of the value    | the section is shown      | scope            |
+======================+===========================+==================+
| falsy or empty array | never                     |                  |
+----------------------+---------------------------+------------------+
| non empty array      | for each element of array | element of array |
+----------------------+---------------------------+------------------+
| object               | once                      | the object       |
+----------------------+---------------------------+------------------+
| other truthy value   | once                      | unchanged        |
+----------------------+---------------------------+------------------+

This table shows for each type of value, what is the condition for the section to be changed and what is the scope of that section.

If the value is of type **boolean**, the section is shown **once if the value is true**, and the scope of the section is **unchanged**.

If we have the section

.. code-block:: text

    {#hasProduct}
        {price} €
    {/hasProduct}

Given the following data:

.. code-block:: javascript

    {
        "hasProduct": true,
        "price": 10
    }

Since hasProduct is a boolean, the section is shown once if `hasProduct` is `true`.
Since the scope is unchanged, the subsection `{price} €` will render as `10 €`


Inverted Sections
-----------------

An inverted section begins with a caret (hat) and ends with a slash. That is {^person} begins a "person" inverted section while {/person} ends it.

While sections can be used to render text one or more times based on the value of the key, inverted sections may render text once based on the inverse value of the key. That is, they will be rendered if the key doesn't exist, is false, or is an empty list. The scope of an inverted section is unchanged.

Template:

.. code-block:: text

    {#repo}
      <b>{name}</b>
    {/repo}
    {^repo}
      No repos :(
    {/repo}

Data:

.. code-block:: javascript

    {
      "repo": []
    }

Output:

.. code-block:: javascript

    No repos :(

Sections and newlines
---------------------

New lines are kept inside sections, so the template:

.. code-block:: text

    {#repo}
      {name}>
    {/repo}
    {^repo}
      No repos :(
    {/repo}

Data:

.. code-block:: javascript

    {
      "repo": [
          {name: "John"},
          {name: "Jane"},
      ]
    }

Will actually render

.. code-block:: text

    NL
      John
    NL
    NL
      Jane
    NL

(where NL represents an emptyline)

The easiest to make this work is to enable the paragraphLoop option, like this :


.. code-block:: javascript

    // Now, all sections in the form of :
    // {#section}
    // something
    // {/section}
    // will keep just the inner paragraphs, and drop the newlines of the outer section
    const doc = new Docxtemplater(zip, {paragraphLoop: true});

An other less recommended way if you don't want to set this option, is to
remove the new lines after the start of the section and before the end of the
section.

For our example , that would be:

.. code-block:: text

    {#repo} {name}
    {/repo} {^repo} No repos :( {/repo}

Raw XML syntax
--------------

It is possible to insert raw (unescaped) XML, for example to render a complex table, an equation, ...

With the ``rawXML`` syntax the whole current paragraph (``w:p``) is replaced by the XML passed in the value.

.. code-block:: text

    {@rawXml}

with this data:

.. code-block:: javascript

    doc.render({
        rawXml : `
        <w:p>
            <w:pPr>
                <w:rPr>
                    <w:color w:val="FF0000"/>
                </w:rPr>
            </w:pPr>
            <w:r>
                <w:rPr>
                    <w:color w:val="FF0000"/>
                </w:rPr>
                <w:t>
                    My custom
                </w:t>
            </w:r>
            <w:r>
                <w:rPr>
                    <w:color w:val="00FF00"/>
                </w:rPr>
                <w:t>
                    XML
                </w:t>
            </w:r>
        </w:p>
        `
    })

This will loop over the first parent <w:p> tag

If you want to insert HTML styled input, you can also use the docxtemplater html module: https://docxtemplater.com/modules/html/

Lambdas
-------

.. code-block:: javascript

    const doc = new Docxtemplater(zip);
    doc.render({
      userGreeting: (scope) => {
        return "How is it going, " + scope.user + " ? ";
      },
      users: [
        {
          name: "John",
        },
        {
          name: "Mary",
        },
      ],
    });


With the following template :

```txt
{#users}
{userGreeting}
{/}
```

It will call the function userGreeting twice, with the current scope as first argument, and the scopeManager as the second argument.

Set Delimiter
-------------

Set Delimiter tags start and end with an equal sign and change the tag delimiters from { and } to custom strings.

Consider the following contrived example:

.. code-block:: text

    * {default_tags}
    {=<% %>=}
    * <% erb_style_tags %>
    <%={ }=%>
    * { default_tags_again }

Here we have a list with three items. The first item uses the default tag style, the second uses erb style as defined by the Set Delimiter tag, and the third returns to the default style after yet another Set Delimiter declaration.

Custom delimiters may not contain whitespace or the equals sign.

It is also possible to `change the delimiters by using docxtemplater options object`_.

.. _`change the delimiters by using docxtemplater options object`: configuration.html#custom-delimiters

Dash syntax
-----------

When using sections, docxtemplater will try to find on what element to loop over by itself:

If between the two tags {#tag}______{/tag}

 * there is a tag ``<w:tc>`` , that means that your loop is inside a table, and it will loop over ``<w:tr>`` (table row).
 * by default, it will loop over ``<w:t>``, which is the default Text Tag

With the Dash syntax you can specify the tag you want to loop on:
For example, if you want to loop on paragraphs (``w:p``), so that each of the loop creates a new paragraph, you can write:

.. code-block:: text

    {-w:p loop} {inner} {/loop}

