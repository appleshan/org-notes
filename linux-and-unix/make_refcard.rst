---
title: Make Refcard
source: http://www.schacherer.de/frank/technology/tools/make.html
---

Make Refcard
============

You can get all the info on this page in more detail from man make or
from the `GNU Make
Manual <http://www.gnu.org/software/make/manual/make.html>`__.
A nice intro to makefiles, that works through one example in detail (and is not
focusing on compiling C programs) can also be found on
`jfranken <https://web.archive.org/web/20180119050809/http://www.jfranken.de/homepages/johannes/vortraege/make.en.html>`__.

.. contents:: **Contents**
   :depth: 2

Overview
--------

**make** is a tool to automate rebuilding of files that are depended on other files.
The dependency rules are written in a ``Makefile``.
`Rules <#rules>`__ consist of `targets <#targets>`__ that depend on so-called `prerequisites <#prerequisites>`__.
Whenever any of the prerequisites changed,
the target will be rebuilt,
using `shell <#shell>`__ `commands <#commands>`__ given by the rule
(or using global commands given for the involved name `patterns <#patterns>`__, if no specific rule was found).
Since a prerequisite can be the target for another rule,
you can build large trees of prerequisites,
and make can infer,
which files to update,
if some file in the tree changed.

Comments
--------

A ``#`` at line start turns the line into a comment line that will be
ignored by make.
Use backslashes ``\`` at the end of a line to split a
long line into several lines on screen.
Lists are spearated by white
space.

Targets
-------

Normally,
make expects the targets and prerequisites to be files,
so it can compare their timestamps to decide if one of the prerequisites is newer than the target,
and the target has to be updated using the rule.

If you specify a target for which no file exists (a so called **phony** target),
there is no timestamps to compare,
and the rule will *always* be run.
You can also treat a target for which a file exists as phony (forcing its rule to be always run),
by assigning it to the .PHONY special name.
Also,
a target which ultimately depends on phony targets can not compare timestamps,
and will always be run.

Special names
~~~~~~~~~~~~~

Special names are target names that **make** treat their prerequisites in a special way.
They are all starting with a dot and written in UPPER CASE,
and there are many more than the ones listed here.

+---------------------------+---------------------------------------------------------------------------------------------------------------------------+
| Special name              | Explain                                                                                                                   |
+===========================+===========================================================================================================================+
| ``.PHONY``                | Prerequisites of .PHONY are considered to be phony targets.                                                               |
|                           | ``make`` will run its commands regardless of whether a file with that name exists or what its last-modification time is.  |
+---------------------------+---------------------------------------------------------------------------------------------------------------------------+
| ``.IGNORE``               | Prerequisites for .IGNORE ignore errors in execution of the commands run for those particular files.                      |
+---------------------------+---------------------------------------------------------------------------------------------------------------------------+
| ``.EXPORT_ALL_VARIABLES`` | This tells ``make`` to export all variables to child processes.                                                           |
|                           | It is more of a general option and uses no prerequisites.                                                                 |
+---------------------------+---------------------------------------------------------------------------------------------------------------------------+

Prerequisites
-------------

Here is an example of prerequisites.

|Prerequisites|

For this in one-lined notation, you will get the following Makefile:

.. code:: Make

    # target    prerequisite              commands
    # ------------------------------------------------------
    house     : roof plumbing electrics;  @echo $@; touch $@
    plumbing  : pipes basement;           @echo $@; touch $@
    roof      : walls;                    @echo $@; touch $@
    electrics : walls wires;              @echo $@; touch $@
    walls     : basement bricks;          @echo $@; touch $@
    pipes     : ;                         @echo $@; touch $@
    basement  : ;                         @echo $@; touch $@
    bricks    : ;                         @echo $@; touch $@
    wires     : ;                         @echo $@; touch $@

The touch command serves to create the targets as files,
so that when you ran ``make plumbing``,
and then ``make house``,
plumbing,
pipes and basement would already exist and not have to be build again.

Commands
--------

Commands can be any shell commands.

- ``@`` at the beginning of a command means "don't print the line before executing",
- ``-`` means "don't exit on error".

Rules
-----

Rules can be written in one line for short rules

.. code:: Make

    target(s) : [prerequisites] [; shell-command(s)]

Or in the full format for more involved commands (note the TAB)

.. code:: Make

    target(s) : [prerequisites]
    [TAB shell-command]
    [TAB shell-command]
    ...

Make starts with the first rule that has a name not starting with a dot,
if not invoked for a specific rule.
The other rules are processed because their targets appear as prerequisites of this goal,
and so on.
If some rule is not needed for this,
that rule is not processed.
If several targets are given for a rule,
it's as if there were as many rules,
each with one target.

You can split each rule into two parts:

- An *implicit* rule stating the prerequisites, and
- An *explicit* one for the commands.

Any rules in the last example lead to the same commands and differed in their prerequisites only.
For those rules that have no prerequisites (e.g. ``bricks``) you don't even need an implicit rule.
The explicit ones can be pooled,
because of their commands being all identical.
Thus you get a shorter and pretty clear Makefile:

.. code:: Make

    # An explicit rule assigns the commands for several targets
    house plumbing roof electrics walls pipes basement bricks wires: ; @echo $@; touch $@

    # Implicit rules state the prerequisites
    house:     roof plumbing electrics
    plumbing:  pipes basement
    roof:      walls
    electrics: walls wires
    walls:     basement bricks

Macros
------

Use ``=`` or ``:=`` to assign values to variables (so called *macros*),
depending on if potentially contained variables and functions should be expanded at *using* or *declaration* time.
Fringe space is stripped.
Macros assigned with ``=`` must be declared above any uses,
or they will still be empty.

To retrieve the stored value, write ``$(myvar)``.
Macros are expanded by substituting the assigned values textually for the name.
To have make executing the value (like a function),
write ``$(call   myvar)``.

Patterns
--------

Often you have long lists of files that all have a similar form (similar extensions, names, etc),
and have to be processed in the same way.
In this case,
you will not want to write a rule for every single file,
what you want is a rule that says "for files that look like that, do this".
Patterns allow you to do this.
A target pattern is composed of a \`%' between a prefix and a suffix,
either or both of which may be empty.
For example:

.. code:: Make

    %.class: %.java; javac $<

The way it works is that any name that matches the target pattern will invoke the rule.
The part of the name that matched the wildcard will be substituted for the wildcard in the prerequisites.

Pattern rules may have more than one target.
Unlike normal rules,
this does not act as many different rules with the same prerequisites and commands.
If a pattern rule has multiple targets,
\`make' knows that the rule's commands are responsible for making all of the targets.
The commands are executed only once to make all the targets.

The used to be special suffix rules.
These are now superceded by pattern rules.

Functions
---------

Often,
simple patterns are not enough,
and you will want to mangle filenames in various other ways.
Here is a bunch of built-in functions for this purpose.

+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| Function                                 | Purpose                                                                                                    |
+==========================================+============================================================================================================+
| ``$(subst from,to,text)``                | Replace from with to in text.                                                                              |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(patsubst pattern,replacement,text)`` | Replace words matching pattern with replacement in text.                                                   |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(strip string)``                      | Remove excess whitespace characters from string.                                                           |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(findstring find,text)``              | Locate find in text.                                                                                       |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(filter pattern...,text)``            | Select words in text that match one of the pattern words.                                                  |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(filter-out pattern...,text)``        | Select words in text that do not match any of the pattern words.                                           |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(sort list)``                         | Sort the words in list lexicographically, removing duplicates.                                             |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(dir names...)``                      | Extract the directory part of each file name.                                                              |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(notdir names...)``                   | Extract the non-directory part of each file name.                                                          |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(suffix names...)``                   | Extract the suffix (the last dot and following characters) of each file name.                              |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(basename names...)``                 | Extract the base name (name without suffix) of each file name.                                             |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(addsuffix suffix,names...)``         | Append suffix to each word in names.                                                                       |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(addprefix prefix,names...)``         | Prepend prefix to each word in names.                                                                      |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(join list1,list2)``                  | Join two parallel lists of words.                                                                          |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(word n,text)``                       | Extract the nth word (one-origin) of text.                                                                 |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(words text)``                        | Count the number of words in text.                                                                         |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(wordlist s,e,text)``                 | Returns the list of words in text from s to e.                                                             |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(firstword names...)``                | Extract the first word of names.                                                                           |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(wildcard pattern...)``               | Find file names matching a shell file name pattern (not a \`%' pattern).                                   |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(error text...)``                     | When this function is evaluated, make generates a fatal error with the message text.                       |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(warning text...)``                   | When this function is evaluated, make generates a warning with the message text.                           |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(shell command)``                     | Execute a shell command and return its output.                                                             |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(origin variable)``                   | Return a string describing how the make variable variable was defined.                                     |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(foreach var,words,text)``            | Evaluate text with var bound to each word in words, and concatenate the results.                           |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+
| ``$(call var,param,...)``                | Evaluate the variable var replacing any references to $(1),$(2) with the first, second, etc. param values. |
+------------------------------------------+------------------------------------------------------------------------------------------------------------+

Variables
---------

There are some predefined variables for use in rules
(called *dynamic macros*, because they look a bit like macros and there contents are set dynamically during rule evaluation):

+------------+-------------------------------------------------------------------------------------------------+
| Variable   | Purpose                                                                                         |
+============+=================================================================================================+
| ``$@``     | The name of the target.                                                                         |
+------------+-------------------------------------------------------------------------------------------------+
| ``$%``     | The target member name, when the target is an archive member.                                   |
+------------+-------------------------------------------------------------------------------------------------+
| ``$<``     | The name of the first (or only) prerequisite.                                                   |
+------------+-------------------------------------------------------------------------------------------------+
| ``$?``     | The names of all the prerequisites that are newer than the target, with spaces between them.    |
+------------+-------------------------------------------------------------------------------------------------+
| ``$^``     | The names of all the prerequisites, with spaces between them.                                   |
| ``$+``     | The value of ``$^`` omits duplicate prerequisites, while ``$+`` retains them and preserves their order. |
+------------+-------------------------------------------------------------------------------------------------+
| ``$\*``    | The stem with which an implicit rule matches.                                                   |
+------------+-------------------------------------------------------------------------------------------------+
| ``$(@D)``  | The directory part and the file-within-directory part of ``$@``                                 |
| ``$(@F)``  |                                                                                                 |
+------------+-------------------------------------------------------------------------------------------------+
| ``$(\*D)`` | The directory part and the file-within-directory part of ``$\*``                                |
| ``$(\*F)`` |                                                                                                 |
+------------+-------------------------------------------------------------------------------------------------+
| ``$(%D)``  | The directory part and the file-within-directory part of ``$%``                                 |
| ``$(%F)``  |                                                                                                 |
+------------+-------------------------------------------------------------------------------------------------+
| ``$(<D)``  | The directory part and the file-within-directory part of ``$<``                                 |
| ``$(<F)``  |                                                                                                 |
+------------+-------------------------------------------------------------------------------------------------+
| ``$(^D)``  | The directory part and the file-within-directory part of ``$^``                                 |
| ``$(^F)``  |                                                                                                 |
+------------+-------------------------------------------------------------------------------------------------+
| ``$(+D)``  | The directory part and the file-within-directory part of ``$+``                                 |
| ``$(+F)``  |                                                                                                 |
+------------+-------------------------------------------------------------------------------------------------+
| ``$(?D)``  | The directory part and the file-within-directory part of ``$?``                                 |
| ``$(?F)``  |                                                                                                 |
+------------+-------------------------------------------------------------------------------------------------+

Includes
--------

When your project is large, a single giant makefile can become rather unwieldy.
You can split your makefiles into several files and in-line those during runtime.
When make encounters an ``include``-command,
it will stop processing the current Makefile,
read the included Makefile and then continue where it left off.
If you don't want it to abort when the included Makefile's missing,
just say ``-include Makefile(s)`` (the minus sign generally make s make ignore errors).

shell syntax in make
--------------------

Escaping variables in make: When using $ variables inside make
(for shell commands, or Perl special vars)
write $$ instead of $.

Make treats every line as running in a new sub-shell,
and thus forgetting about the previous lines.
This will shoot down shell scripts that have loops or if statements spanning several lines.
So you have to put your whole conditional on one line.
Remember,
when writing one-line shell conditionals,
you have to end every block and condition with a semicolon.
So

.. code:: bash

    for x in a b c do
        echo $x
    done

becomes

.. code:: Make

    for x in a b c;\
    do\
        echo $$x;\
    done

Example
-------

.. code:: Make

    # Example Makefile

    sourcefiles = Main.java Gui.java Logic.java
    compiler=javac
    jc=$(compiler -warn)

    all: $(sourcefiles) docs clean

    # A phony target, not really the name of a file. It is
    # just a name for some commands to be executed when you make an explicit
    # request.

    clean:
            -@ $(RM) *~
            -@ $(RM) *.class

    # % is the wildcard char for targets or prerequisites (like
    # in SQL), $< is the current prerequisite (points to the target on the
    # left), $@; is the current target (looks a bit like a target for
    # shooting)

    %.class: %.java; $(jc) $<

.. |Prerequisites| image:: img/make.png
   :width: 421px
   :height: 340px
