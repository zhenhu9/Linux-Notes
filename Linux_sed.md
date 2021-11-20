# How to Use the sed Command on Linux

It might sound crazy, but the Linux `sed` command is a text or [stream
editor][man7-sed] without an interface.  You can use it from the command line
to manipulate text in files and streams.  We’ll show you how to harness its
power.

[man7-sed]: <http://man7.org/linux/man-pages/man1/sed.1.html>

## The Power of sed

With `sed` you can do all of the following:

-   Select text
-   Substitute text
-   Add lines to text
-   Delete lines from text
-   Modify (or preserve) an original file

We’ve structured our examples to introduce and demonstrate concepts.  Hoever,
the pattern matching and text selection functionalities of `sed` rely heavily
on regular expressions ([regexes][wikipedia_regex]).

[wikipedia_regex]: <https://en.wikipedia.org/wiki/Regular_expression>

**RELATED:** [How to Use Regular Expressions (regexes) on
Linux](https://www.howtogeek.com/661101/how-to-use-regular-expressions-regexes-on-linux/)

## A Simple Example

    $ echo howtodo | sed 's/do/usesed/'

      | data-stream  | sed 'substitution/old/new/' |  ==>  teminal window

    howtousesed

## Selecting Text

A sample text.

    $ less first_sample.txt

    Three birds in a row

    by Stephen Crane

    Three little birds in a row sat musing.
    A man passed near that place.
    Then did the little birds nudge each other.

    They said,"He thinks he can sing."
    They threw back their heads to laugh.
    With quaint countenances,
    They regarded him.
    They were very curious,
    Those three little birds in a row.

1. To select a range with the start and end lines or just a single line:

```
$ sed -n '5,7p' first_sample.txt

Three little birds in a row sat musing.
A man passed near that place.
Then did the little birds nudge each other.
```

The `p` means "print matched lines". By default, `sed` prints all lines and
matched lines. To print only matched lines, use the `-n` (quiet) option.

2. To make multiple range selections, use the `-e` (expression) option:

```
$ sed -n -e '5,7p' -e '9,12p' first_sample.txt

Three little birds in a row sat musing.
A man passed near that place.
Then did the little birds nudge each other.
They said,"He thinks he can sing."
They threw back their heads to laugh.
With quaint countenances,
They regarded him.
```

3. To select rows by multiples of rows by useing tild `~`:

```
$ cat -n first_sample.txt | sed -n '1~4p'

 1	Three birds in a row
 5	Three little birds in a row sat musing.
 9	They said,"He thinks he can sing."
13	They were very curious,
```

**Note**: The first number means the starting line.

4. To match a string from the start of each line:

```
$ sed -n '/^They /p' first_sample.txt

They said,"He thinks he can sing."
They threw back their heads to laugh.
They regarded him.
They were very curious,
```

**Note**: `^` means matching from the start of each line.

## Making Substitutions

1. To substitute the first matched string:

```
$ sed -n 's/birds/cats/p' first_sample.txt

Three cats in a row
Three little cats in a row sat musing.
Then did the little cats nudge each other.
Those three little cats in a row.
```

2. To substitute all matched strings:

```
$ sed -n 's/birds/cats/gp' first_sample.txt

Three cats in a row
Three little cats in a row sat musing.
Then did the little cats nudge each other.
Those three little cats in a row.
```

3. To substitute all matched strings with case-insensitivity:

```
$ sed -n 's/he/she/gip' first_sample.txt

by Stepshen Crane
Tshen did tshe little birds nudge each otsher.
Tshey said,"she thinks she can sing."
Tshey threw back tsheir sheads to laugh.
Tshey regarded him.
Tshey were very curious,
```

4. characters enclosed in square brackets `[]` are interpreted as “any
   character from this list of characters.”

```
$ sed -n 's/[Hh]e/she/gp' first_sample.txt

by Stepshen Crane
Tshen did tshe little birds nudge each otsher.
Tshey said,"she thinks she can sing."
Tshey threw back tsheir sheads to laugh.
Tshey regarded him.
Tshey were very curious,
```

5. To substitute strings in a specified range:

```
$ sed -n '5,7 s/  */ /gp' first_sample.txt
```

**Note**: `*` means zero or more of the preceding character.

6. Here we a single space in the pattern, you’ll see immediately why we have to
   include two spaces:

```
$ sed -n '5,7 s/ */ /gp' first_sample.txt

 T h r e e l i t t l e b i r d s i n a r o w s a t m u s i n g . 
 A m a n p a s s e d n e a r t h a t p l a c e . 
 T h e n d i d t h e l i t t l e b i r d s n u d g e e a c h o t h e r . 
```

7. To substitute in mutiple selected ranges:

```
$ sed -n -e 's/Stephen/Bob/gip' -e 's/Crane/Alisa/gip' first_sample.txt

equvilent to the below,

$ sed -n 's/Stephen/Bob/gip ; s/Crane/Alisa/gip' first_sample.txt

by Bob Crane
by Bob Alisa
```

8. To substitute the specified match in the preceding matched ranges.

```
$ sed -n '/little/ s/[Tt]hree/four/gp' first_sample.txt

four little birds in a row sat musing.
Those four little birds in a row.
```

## More Complex Substitutions

There are shorter ways to do this (more on that later), but we’ll use
the longer way here to demonstrate another concept. Each matched item in
a search pattern (called subexpressions) can be numbered (up to a
maximum of nine items). You can then use these numbers in your `sed`
commands to reference specific subexpressions.

You have to enclose the subexpression in parentheses `()` for this to work. The
parentheses also must be preceded by a backward slash `\` to prevent them from
being treated as a normal character.

The syntex would like the following,

```
    sed 's/\(numbered-part-subexpression\)part-subexpression
          /\numbered-back-reference/' filename
```

1. To output the usernames in `/etc/passwd`:

```
$ sed 's/\([^:]*\).*/\1/' /etc/passwd

root
bin
daemon
...
sshd
example_user
```

**Explanation**

-   **`[^:]*`:** The caret (`^`) in square brackets `[]` means “not” when
    used in a group. A group means any character that isn’t a colon (`:`)
    will be accepted as a match.

-   **`.*`:** means “any character and any number of them.”

-   **`\1`:** This means outputing the numbered matching field.

2. To output the second matching field:

```
$ sed 's/\([^:]*\)\(.*\)/\2/' /etc/passwd

:x:0:0:root:/root:/bin/bash
:x:1:1:bin:/bin:/sbin/nologin
:x:2:2:daemon:/sbin:/sbin/nologin

The following can get the same result,

$ sed 's/:.*//" /etc/passwd
```

3. To switch the two subexpression matching results and insert a delimiter.

```
$ cat second_sample.txt

1,2
3,4
5,6
7,8
9,10
```

```
$ sed 's/^\(.*\),\(.*\)$/\2,\1 /g' second_sample.txt

2,1 
4,3 
6,5 
8,7 
10,9 
```

**Explanation**

-   **`^`:** the caret not in a group (`[]`) means “The start of the line.”

-   **`\(.*\),`:** This means any number of any characters.

-   **`$/`:** The dollar sign (`$`) represents the end of the line and
    will allow our search to continue to the end of the line. We’ve used
    this simply to introduce the dollar sign. We don’t really need it
    here, as the asterisk (`*`) would go to the end of the line in this
    scenario.

-   **`\2,\1 /g'`:** Because we enclosed our two subexpressions
    in parentheses, we can refer to both of them by their numbers.
    Because we want to reverse the order, we type them as
    `second-match,first-match`.

-   **`/g`:** This enables our command to work globally on each line.

4. To use the Cut command (`c`) to substitute entire lines that match
   your search pattern.

```
$ sed -n '/man/c There is a woman.' first_sample.txt

There is a woman.
```

## Inserting Lines and Text


1. To insert new lines after any match, use the Append command (`a`):

```
$ cat second_sample.txt

1,2
3,4
5,6
7,8
9,10
```

```
$ sed '/8/a --> Inserted!' second_sample.txt

1,2
3,4
5,6
7,8
--> Inserted!
9,10

```

2. To insert new lines above any match, use the Insert Command (`i`):

```
$ sed '/8/i --> Inserted!' second_sample.txt

1,2
3,4
5,6
--> Inserted!
7,8
9,10
```

3. The ampersand `&` represents the original matched text:

```
$ sed 's/.*/--> Inserted &/' second_sample.txt

--> Inserted 1,2
--> Inserted 3,4
--> Inserted 5,6
--> Inserted 7,8
--> Inserted 9,10
```

4. To add a blank line between each line, use the `G` command:

```
$ sed 'G' second_sample.txt

1,2

3,4

5,6

7,8

9,10
```

If you want to add two or more blank lines, you can use `G;G`, `G;G;G`,
and so on.

## Deleting Lines

The Delete command (`d`) deletes lines that match a search pattern, or
those specified with line numbers or ranges.

1. To delete the specified line, use the delete command `d`:

```
$ sed '3d' second_sample.txt

1,2
3,4
7,8
9,10
```

2. To delete a range of lines:

```
$ sed '4,5d' second_sample.txt

1,2
3,4
5,6
```

3. To delete lines outside a range, use an exclamation character (`!`):

```
$ sed '3,4!d' second_sample.txt

5,6
7,8
```

## Saving Your Changes

We can save the processed result by redirecting it to a file without changing
the original file. In addition, You can use the In-place option (`-i`) to
backup the original file with a specific file extension enclosed by `''` behind
the option and change the original file.

1. To save your changes, redirect the output to a file:

```
$ sed '/^.*Thr.*$/d' first_sample.txt > new.first_sample.txt
```

2. To modify the original file, use `i` option to specify a backup:

```
$ sed -i'.bak' '/^.*Thr.*$/d' first_sample.txt > first_sample.txt
```

## GNU resources

[GNU sed](https://www.gnu.org/software/sed/manual/sed.html).

## Reference

**DAVE MCKAY**: [how-to-use-the-sed-command-on-linux][dave-sed]

[dave-sed]: <https://www.howtogeek.com/666395/how-to-use-the-sed-command-on-linux>
