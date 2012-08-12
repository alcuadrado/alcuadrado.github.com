---
layout: default
title: "Brainfuck beware: JavaScript is after you!"
description: How to write any javascript only with ()[]{}!+ charcters.
permalink: /blog/2012/08/09/non-alphanumeric-javascript.html
---

**tl;dr** I just made a tool to transform any javascript code into an equivalent sequence
of `()[]{}!+` characters. You can try it
[here](/files/hieroglyphy), or grab it from
[github](https://github.com/alcuadrado/hieroglyphy) or
[npm](https://npmjs.org/package/hieroglyphy). Keep on reading if you want
 to know how it works.

Non alphanumeric JavaScript
---------------------------

> What do you know about non-alphanumeric XSS?

The other day one of [my friends](http://mfsec.com.ar/) asked me that question
on IRC, pointing me to some articles on [sla.ckers.org](http://sla.ckers.org)
where they tried to create some scripts like `alert(1)` with non-alphanumeric
characters.

As a security researcher and a penetration tester, he insisted that extending
that concept to any javascript source would be really useful for bypassing
[IDSs](http://en.wikipedia.org/wiki/Intrusion_detection_system),
[IPSs](http://en.wikipedia.org/wiki/Intrusion_prevention_system) and
[WAFs](https://www.owasp.org/index.php/Web_Application_Firewall). So challange
accepted!

Alphabet
--------

Many alphabets could do the job, but just for fun, I tried to keep it as small
as possible, using only the following characters:

* `[` and `]` to access array elements, objects properties, get numbers and
cast elements to strings.
* `(` and `)` to call functions and avoid parsing errors.
* `+` to append strings, sum and cast elements to numbers.
* `!` to cast elements to booleans.
* `{` and `}` to get `NaN` and the infamous string `"[object Object]"`


Numbers
-------

To start our journey to the world of brackets, lets represent the numbers with
our new alphabet.

`0` is easily obtained by casting an empty array like this `+[]`. In a similar
way, we can cast the empty array to boolean to get `true`, and then to `1` with
`+!![]`. Those numbers, along with `+` would be enough to get every natural.

But if we take advantage of JavaScript coercion of types, we can reduce the size
of the sequence of the numbers in two ways.

First, if we add a number and a boolean, both operands would be casted to
numbers. So instead of using sums of ones to generate larger values, we can
add just a `1` and a sequence of `true`s (we can use more than one `true` at a
time beacuse addition is left-to-right assosiative). For instance, here is `4`:
`!+[]+!![]+!![]+!![]`.

The second idea is to get strings representing large numbers and cast them in
order to get a shorter sequence of symbols. Once we obtained all the possible
digits like we did above with `4`, we can get the desired string by adding the
first digit to `[]` (to make it a character), and combinig all of them with `+`
(with the necessary parens). Once again, the left-to-right assosiativite would
save us lots of chars. Finally, we only need to cast that. Doing this, `12`
would look like this: `+((+!![]+[])+(!+[]+!![]))`.

The second idea is to reuse what we've done above in order to get a shorter
sequence of symbols. The main purpose of doing this is to represent bigger
numbers without the need to sum `1` each time to get to our number, so instead
we get it's string representation and cast it to number. For example,
representing `12` adding ones would be:
`(!+[]+!![]+!![]+!![]+!![]+!![]+!![]+!![]+!![]+!![]+!![]+!![])`, but by resuing
`1` and `2` we can be represen it like this: `+((+!![]+[])+(!+[]+!![]))`. Here,
we have casted the first digit to string, added the second, and then, converted
everything to a number. Speaking in terms of code, on the first case we did a
simple sum: `(1+1+1+1+1+1+1+1+1+1+1+1)`; and on the second one we concatenated
two strings and casted them into a number like this: `+("1"+2)`.

Having said that, here is a table of all the possible digits:

<div class="codeblock">

{% highlight javascript linenos=table anchorlinenos=true lineanchors=codeblock2 %}
0 +[]
1 +!![]
2 !+[]+!![]
3 !+[]+!![]+!![]
4 !+[]+!![]+!![]+!![]
5 !+[]+!![]+!![]+!![]+!![]
6 !+[]+!![]+!![]+!![]+!![]+!![]
7 !+[]+!![]+!![]+!![]+!![]+!![]+!![]
8 !+[]+!![]+!![]+!![]+!![]+!![]+!![]+!![]
9 !+[]+!![]+!![]+!![]+!![]+!![]+!![]+!![]+!![]
{% endhighlight %}

</div>

Base elements and strings
-------------------------

Now that we have numbers, lets go for more interesting elements from which we
can obtain characters:

* `true` as we have already seen, can be obtained from `!![]`
* `false` from `![]`
* `undefined` by accessing to non-existing element to an array: `[][+[]]`
* `NaN` is the result of trying to cast an object to number: `+{}`
* `"[object Object]"` with `{}+[]`

Casting them to string (when necessary) and accessing those like arrays will
give us single characters, from which we can even get more strings! These are
`(the space)`, `"["`, `"]"`, `"a"`, `"b"`, `"c"`, `"d"`, `"e"`, `"f"`, `"i"`,
`"j"`, `"l"`, `"n"`, `"N"`, `"o"`, `"O"`, `"r"`, `"s"`, `"t"` and `"u"`. By
combining them with numbers we can get `"1e100"` and `"1e1000"`, which when
casted to numbers would result in `1e+100` and `Infinity`. And by casting them
back to strings we can manage to get `"y"`, `"I"` and `"+"`.

Gathering functions from available characters
-------------------------------------------------

By combining those characters, we can only get these JavaScript functions and
type names: `"call"`, `"concat"`, `"constructor"`, `"join"`, `"slice"` and
`"sort"`.

Playing with our alphabet and these strings, we can get the following functions:

* `Function` from `array["sort"]["constructor"]`
* `Array` from `array["constructor"]`
* `Bolean` from `false["constructor"]`
* `Number` from `0["constructor"]`
* `Object` fom `{}["constructor"]`
* `String` fom `string["constructor"]`
* `Function.prototype.call` from `f["call"]`
* `String.prototype.concat` from `string["concat"]`
* `Array.prototype.join` from `array["join"]`
* `Array.prototype.slice`  from `array["slice"]`
* `Array.prototype.sort` from `array["sort"]`

Unluckily, none of these functions would give us new characters, but don't loose
your hope yet!

Exploting the DOM for fun and characters
----------------------------------------

If we sacrifice some portabilty and constraint the scripts to webpages, we can
take for granted that DOM elements would be available, and get the remaining
characters.

One interesting function that becames available is `window.unescape` which would
give us all the ASCII characters by calling
`window.unescape("%" + HEXA_ASCII_VAL)`.

All we are missing to get `unescape` is the `"p"` character. So once again we
make a trade-off, sacrificing some more portability to get it. If we know that
we are in a webpage served over HTTP or HTTPS we can asume that by casting
`window.location` to string, and getting its third character we would obtain the
precious `"p"`.

But how can we obtain the `window.location` object if we don't have access to
`window` yet? Luckly JavaScript, being so premissive, would give that object by
doing this:

<div class="codeblock">

{% highlight javascript linenos=table anchorlinenos=true lineanchors=codeblock8 %}
Function("return location")()
{% endhighlight %}

</div>

And with `location` now we can have three more characters `"h"`, `"p"`, `"/"`,
`escape` and `unescape` functions!

If we could get the character `"%"` we would be able to get the rest by calling
`unescape("%" + HEXA_ASCII_VALUE)`. Luckly, escaping `"["` yields the string
`"%5B`, and from that, we can obtain the percentage sign.

Now, we can reach any ASCII character like this:

<div class="codeblock">

{% highlight javascript linenos=table anchorlinenos=true lineanchors=codeblock3 %}
[][(![]+[])[+[]+!![]+!![]+!![]]+({}+[])[+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]][({}+[])[!+[]+!+[]+!+[]+!+[]+!+[]]+({}+[])[+!![]]+([][+[]]+[])[+!![]]+(![]+[])[+[]+!![]+!![]+!![]]+(!![]+[])[+[]]+(!![]+[])[+!![]]+([][+[]]+[])[+[]]+({}+[])[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+({}+[])[+!![]]+(!![]+[])[+!![]]]((!![]+[])[+!![]]+(!![]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+([][+[]]+[])[+[]]+(!![]+[])[+!![]]+([][+[]]+[])[+!![]]+({}+[])[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+([][+[]]+[])[+[]]+([][+[]]+[])[+!![]]+(!![]+[])[!+[]+!![]+!![]]+(![]+[])[+[]+!![]+!![]+!![]]+({}+[])[!+[]+!+[]+!+[]+!+[]+!+[]]+(+{}+[])[+!![]]+([]+[][(![]+[])[+[]+!![]+!![]+!![]]+({}+[])[+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]][({}+[])[!+[]+!+[]+!+[]+!+[]+!+[]]+({}+[])[+!![]]+([][+[]]+[])[+!![]]+(![]+[])[+[]+!![]+!![]+!![]]+(!![]+[])[+[]]+(!![]+[])[+!![]]+([][+[]]+[])[+[]]+({}+[])[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+({}+[])[+!![]]+(!![]+[])[+!![]]]((!![]+[])[+!![]]+(!![]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+([][+[]]+[])[+[]]+(!![]+[])[+!![]]+([][+[]]+[])[+!![]]+({}+[])[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+(![]+[])[+[]+!![]+!![]]+({}+[])[+!![]]+({}+[])[!+[]+!+[]+!+[]+!+[]+!+[]]+(+{}+[])[+!![]]+(!![]+[])[+[]]+([][+[]]+[])[!+[]+!![]+!![]+!![]+!![]]+({}+[])[+!![]]+([][+[]]+[])[+!![]])())[!+[]+!![]+!![]]+(!![]+[])[!+[]+!![]+!![]])()([][(![]+[])[+[]+!![]+!![]+!![]]+({}+[])[+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]][({}+[])[!+[]+!+[]+!+[]+!+[]+!+[]]+({}+[])[+!![]]+([][+[]]+[])[+!![]]+(![]+[])[+[]+!![]+!![]+!![]]+(!![]+[])[+[]]+(!![]+[])[+!![]]+([][+[]]+[])[+[]]+({}+[])[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+({}+[])[+!![]]+(!![]+[])[+!![]]]((!![]+[])[+!![]]+(!![]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+([][+[]]+[])[+[]]+(!![]+[])[+!![]]+([][+[]]+[])[+!![]]+({}+[])[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+(!![]+[])[!+[]+!![]+!![]]+(![]+[])[+[]+!![]+!![]+!![]]+({}+[])[!+[]+!+[]+!+[]+!+[]+!+[]]+(+{}+[])[+!![]]+([]+[][(![]+[])[+[]+!![]+!![]+!![]]+({}+[])[+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]][({}+[])[!+[]+!+[]+!+[]+!+[]+!+[]]+({}+[])[+!![]]+([][+[]]+[])[+!![]]+(![]+[])[+[]+!![]+!![]+!![]]+(!![]+[])[+[]]+(!![]+[])[+!![]]+([][+[]]+[])[+[]]+({}+[])[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+({}+[])[+!![]]+(!![]+[])[+!![]]]((!![]+[])[+!![]]+(!![]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+([][+[]]+[])[+[]]+(!![]+[])[+!![]]+([][+[]]+[])[+!![]]+({}+[])[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+(![]+[])[+[]+!![]+!![]]+({}+[])[+!![]]+({}+[])[!+[]+!+[]+!+[]+!+[]+!+[]]+(+{}+[])[+!![]]+(!![]+[])[+[]]+([][+[]]+[])[!+[]+!![]+!![]+!![]+!![]]+({}+[])[+!![]]+([][+[]]+[])[+!![]])())[!+[]+!![]+!![]]+(!![]+[])[!+[]+!![]+!![]])()(({}+[])[+[]])[+[]]+HEXA_VALUE)
{% endhighlight %}

</div>

Finally, all we need to transform a script into symbols, is reading it as a
string, encoding it in our alphabet, and use `Function` as `eval`.

Hieroglyphy
-----------

With the findings in this article, I've made a tool for encoding scripts,
strings and numbers into this alphabet. It's available at
[github](https://github.com/alcuadrado/hieroglyphy), so feel free to fork and
modify it. You can also try it online
[here](http://patriciopalladino.com/files/hieroglyphy).

Room from improvement
---------------------

Both this article and Hieroglyphy are just proof of concepts, there is plenty of
room from improvments:

* Once we were able to generate all ASCII characters, no effort was made to get
    the shortest representation of any of them.
* When targeting modern browsers only, `btoa` would be a great help
    yielding lots of characters in shorter sequences.
* Depending on the target, one may select a bigger alphabet for reducing the
    encoding size.
* If we know the domain where the script would be run, more characters can be
    graved from it.
<!-- * When working with XSS during a pentest, you will find appropiate to
    easily get the characters of your need by using the current domain. -->

Acknowledgments
---------------

Thanks to Matt for giving me the initial idea, helping me with the development
and the redaction of this article.

Don't miss his future entry on how can you use Hieroglyphy for XSS!
