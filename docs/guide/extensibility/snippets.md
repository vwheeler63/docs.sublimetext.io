---
title: Snippets
---

# Snippets

Whether you are coding or writing the next vampire best-seller, you're
likely to need certain short fragments of text again and again. Use
snippets to save yourself tedious typing. Snippets are smart templates
that will insert text for you and adapt it to their context.

To create a new snippet, select **Tools | Developer | New Snippet...**
Sublime Text will present you with a skeleton for it.

Snippets can be stored under any package's folder, but to keep it simple
while you're learning, you can save them to your `Packages/User` folder.

## Snippets File Format

Snippets typically live in a Sublime Text package. They are simplified
XML files with the extension `.sublime-snippet`. For instance, you could
have a `greeting.sublime-snippet` inside an *Email* package.

The structure of a typical snippet is as follows (including the default
hints Sublime Text inserts for your convenience):

```xml
<snippet>
    <content><![CDATA[Type your snippet here]]></content>
    <!-- Optional: Tab trigger to activate the snippet -->
    <tabTrigger>xyzzy</tabTrigger>
    <!-- Optional: Scope the tab trigger will be active in -->
    <scope>source.python</scope>
    <!-- Optional: Description to show in the menu -->
    <description>My Fancy Snippet</description>
</snippet>
```

The `snippet` element contains all the information Sublime Text needs in
order to know *what* to insert, *whether* to insert and *when*. Let's
look at each of these parts in turn.

`content`
: The actual snippet. Snippets can range from simple to fairly complex
  templates. We'll look at examples of both later.
  
  Keep the following in mind when writing your own snippets:
  
  - If you want to get a literal `$`, you have to escape it like
    this: `\$`.
  
  - When writing a snippet that contains indentation, always use
    tabs. When the snippet is inserted, the tabs will be
    transformed into spaces if the option
    `translate_tabs_to_spaces` is `true`.
  
  - The `content` must be included in a `<![CDATA[…]]>` section.
    Snippets won't work if you don't do this\!
  
  - The `content` of your snippet must not contain `]]>` because
    this string of characters will prematurely close the
    `<![CDATA[…]]>` section, resulting in an XML error. To work
    around this pitfall, you can insert an undefined variable into
    the string like this: `]]$NOT_DEFINED>`. This modified string
    passes through the XML parser without closing the content
    element's `<![CDATA[…]]>` section, but Sublime Text will
    replace `$NOT_DEFINED` with an empty string before inserting
    the snippet into your file. In other words, `]]$NOT_DEFINED>`
    in your snippet file `content` will be written as `]]>` when
    you trigger the snippet.

`tabTrigger`
: Defines the sequence of keys that must be pressed to insert this snippet. 
  After typing this sequence, the snippet will kick in as soon as you hit
  the <Key k="tab" /> key.

`scope`
: Scope selector determining the context where the snippet will be active.
  See [Scopes][] for more information.

`description`
: Used when showing the snippet in the Snippets menu. If not present, Sublime
  Text defaults to the file name of the snippet.

[Scopes]: /guide/extensibility/syntaxdefs.md#scopes

With this information, you can start writing your own snippets as described in
the next sections.

::: tip Note
In the interest of brevity, we're only including the ``content``
element's text in examples unless otherwise noted.
:::


## Snippet Features

### Environment Variables

Snippets have access to contextual information in the form of
environment variables. The values of variables listed below are set
automatically by Sublime Text.


|      Variable        |                           Description                                 |
| -------------------- | --------------------------------------------------------------------- |
| `$SELECTION`         | The text that was selected when the snippet was triggered.            |
| `$TM_CURRENT_LINE`   | Content of the cursor's line when the snippet was triggered.          |
| `$TM_CURRENT_WORD`   | Word under the cursor when the snippet was triggered.                 |
| `$TM_DIRECTORY`      | Directory name of the file being edited. (since 3154)                 |
| `$TM_FILENAME`       | Name of the file being edited, including extension.                   |
| `$TM_FILEPATH`       | Path to the file being edited.                                        |
| `$TM_FULLNAME`       | User's user name.                                                     |
| `$TM_LINE_INDEX`     | Column where the snippet is being inserted, 0 based.                  |
| `$TM_LINE_NUMBER`    | Row where the snippet is being inserted, 1 based.                     |
| `$TM_SELECTED_TEXT`  | An alias for **$SELECTION**.                                          |
| `$TM_SCOPE`          | The scope of the beginning of each selected region. (since 3154)      |
| `$TM_SOFT_TABS`      | `YES` if `translate_tabs_to_spaces` is true, otherwise `NO`.          |
| `$TM_TAB_SIZE`       | Spaces per-tab (controlled by the `tab_size` option).                 |


::: tip Note
Packages can define more, environment variables via [Shell Variables in Metafiles][shell-vars].

[shell-vars]: ../../reference/metadata#shell-variables
:::


Let's see a simple example of a snippet using variables:

```perl
=================================
USER NAME:          $TM_FULLNAME
FILE NAME:          $TM_FILENAME
 TAB SIZE:          $TM_TAB_SIZE
SOFT TABS:          $TM_SOFT_TABS
=================================

# Output:
=============================
USER NAME:          guillermo
FILE NAME:          test.txt
 TAB SIZE:          4
SOFT TABS:          YES
=============================
```


### Fields

With the help of field markers, you can cycle through positions within the
snippet by pressing the <Key k="tab" /> key. Fields are used to walk you through the
customization of a snippet after it's been inserted.

```perl
First Name: $1
Second Name: $2
Address: $3
```

In the example above, the cursor will jump to `$1` if you press <Key k="tab" />
once. If you press <Key k="tab" /> a second time, it will advance to `$2`, etc. You
can also move backwards in the series with <Key k="shift+tab" />. If you press
<Key k="tab" /> after the highest tab stop, Sublime Text will place the cursor at the
end of the snippet's content so that you can resume normal editing.

If you want to control where the exit point should be, use the `$0` mark. By
default, this is the end of the snippet.

You can break out of the field cycle any time by pressing <Key k="escape" />.

### Mirrored Fields

Identical field markers mirror each other: when you edit the first one, the rest
will be populated in real time with the same value.

```perl
First Name: $1
Second Name: $2
Address: $3
User name: $1
```

In this example, "User name" will be filled out with the same value as "First
Name".

### Placeholders

By expanding the field syntax a little bit, you can define default
values for a field. Placeholders are useful whenever there's a general
case for your snippet, but you still want to keep it customizable.

```perl
First Name: ${1:Guillermo}
Second Name: ${2:López}
Address: ${3:Main Street 1234}
User name: $1
```

Variables can be used as placeholders:

```perl
First Name: ${1:Guillermo}
Second Name: ${2:López}
Address: ${3:Main Street 1234}
User name: ${4:$TM_FULLNAME}
```

And you can nest placeholders within other placeholders too:

```perl
Test: ${1:Nested ${2:Placeholder}}
```


### Substitutions

In addition to the place holder syntax, tab stops can specify more
complex operations with substitutions. Use substitutions to dynamically
generate text based on a mirrored tab stop. Of course, the tab stop you
want to use as variable has to be mirrored somewhere else in the
snippet.

The substitution syntax has the following syntaxes:

- `${var_name/regex/format_string/}`
- `${var_name/regex/format_string/options}`

**var_name**
: The variable name: `1`, `2`… or an environment variable such as `TM_FILENAME` or `SELECTION`.

**regex**
: Perl-style regular expression: See the [Boost library documentation for
  regular expressions][boost-regex].

**format_string**
: See the [Boost library documentation for format strings][boost-format].

**options**
: Optional. May be any of the following:

  **i**
  : Case-insensitive regex.

  **g**
  : Replace all occurrences of ``regex``.

  **m**
  : Don't ignore newlines in the string.

[boost-regex]: https://www.boost.org/doc/libs/release/libs/regex/doc/html/boost_regex/syntax/perl_syntax.html

[boost-format]: https://www.boost.org/doc/libs/release/libs/regex/doc/html/boost_regex/format/boost_format_syntax.html

With substitutions you can, for instance, underline text effortlessly:

```perl

      Original: ${1:Hey, Joe!}
Transformation: ${1/./=/g}

# Output:

      Original: Hey, Joe!
Transformation: =========
```

Another more complex example can translate snail_case to Tile Case With Spaces.
Basically, it combines two separate expressions and replaces into one.
It also illustrates that replaces may occur before the actual tabstop.

<pre class="language-perl line-numbers">
<code>
Transformation: ${1/^(\w)|(?:_(\w))/(?1\u$1:)(?2 \u$2:)/g}
      Original: ${1:text_in_snail_case}

# Output:

Transformation: Text In Snail Case
      Original: text_in_snail_case
</code>
</pre>

```perl
Transformation: ${1/^(\w)|(?:_(\w))/(?1\u$1:)(?2 \u$2:)/g}
      Original: ${1:text_in_snail_case}

# Output:

Transformation: Text In Snail Case
      Original: text_in_snail_case
```

You can also use environment variables with substitutions:

```perl
# In file MyModule.js:

Transformation: ${TM_FILENAME/(\w+)\.js/\1/g}

# Output:

Transformation: MyModule
```

### Passing custom arguments

Command Palette items, key bindings and plugins
can pass values 
of fields (`$1` or `${1:placeholder}`)
or custom arguments (`$any_variable`)
to snippets,
when inserting them via [insert_snippet][] command.

The following examples insert `"Hello World!"` to buffer.

Let's assume a generic snippet _Packages/User/My Snippet.sublime-snippet_
which provides a field `$1` 
and expects a custom argument `$subject`:

```xml
<snippet>
   <content><![CDATA[$1 $subject!]]></content>
   <description>Snippet with 2 fields</description>
</snippet>
```

It can be triggered by key bindings with static values being passed.

```json
[
  { 
    "keys": ["ctrl+k", "ctrl+w"],
    "command": "insert_snippet",
    "args": {
      "name": "Packages/User/My Snippet.sublime-snippet",
      "1": "Hello",
      "subject": "World"
    }
  }
]
```

Plugins can pass dynamically determined values.

```py
import sublime_plugin

class MyInsertSnippetCommand(sublime_plugin.TextCommand):
  def run(self, edit):
    field1 = "Hello"
    
    self.view.run_command(
      cmd="insert_snippet",
      args={
        "name": "Packages/User/My Snippet.sublime-snippet",
        "1": field1,
        "subject": self.get_subject()
      }
    )

  def get_subject(self):
    return "World"
```

Field values and arguments can also be passed to inline snippet content.

```py
import sublime_plugin

class MyInsertSnippetCommand(sublime_plugin.TextCommand):
  def run(self, edit):

    snippet_content = "$1 $subject!"

    field1 = "Hello"
    
    self.view.run_command(
      cmd="insert_snippet",
      args={
        "contents": snippet_content,
        "1": field1,
        "subject": self.get_subject()
      }
    )

  def get_subject(self):
    return "World"
```

[insert_snippet]: ../../reference/commands#insert_snippet
