---
title: Introducing Developers to XQuery in MarkLogic
author: Dave
layout: post
permalink: /2012/05/03/introducing-developers-to-xquery-in-marklogic/
categories:
  - For Beginners
tags:
  - introduction
  - learning
  - MarkLogic
  - XPath
  - XQuery
---
In the last few month&#8217;s I have helped a number of developers enter the world of XQuery.  Here are the first three websites that I point people to on their first day as well as some discussions on sequences and FWLOR statements that often occur.

### Intro Sites

[W3School.com&#8217;s XPath Tutorial ][1]and [W3Schools.com&#8217;s XQuery Tutorial][2] &#8212; While not directly related to the W3C, the XPath and XQuery tutorials here are a pretty good non-intimidating introduction.  (There is even a link to an XML tutorial for those who need some work on fundamentals)  Teaching XPath before XQuery is essential to stop people from overusing the &#8220;*where*&#8221; clause of FLWOR statements that could otherwise be done with an XPath predicate.  You can try out W3School&#8217;s code snippets first by inserting their  XML into MarkLogic under &#8220;books.xml&#8221; by copying something like the following into the Query Console


  <pre><code class="language-xquery xquery">let $books := 
  &lt;bookstore&gt;
    &lt;book category="COOKING"&gt;
      &lt;title lang="en"&gt;Everyday Italian&lt;/title&gt;
      &lt;author&gt;Giada De Laurentiis&lt;/author&gt;
      &lt;year&gt;2005&lt;/year&gt;
      &lt;price&gt;30.00&lt;/price&gt;
    &lt;/book&gt;
    ... other books from W3Schools example ...
  &lt;/bookstore&gt;

return
  xdmp:document-insert("books.xml", $books)</code></pre>


Once the person is comfortable with the basics, I almost always plunge them into the excellent [Ninja Tutorial][3] on the MarkLogic community site.  This tutorial reinforces XPath and XQuery while also expanding to things like JSON and search, which are nice to have on your radar early.

The next thing I do is make sure the developer has the most recent [MarkLogic XQuery functions reference][4] bookmarked.

### The &#8220;Everything is a Sequence&#8221; Discussion

I can usually skip this discussion with people who enjoy reading [language specifications with formal grammars][5] or [railroad diagrams][6] (you can generate railroad diagrams for XQuery 1.0 with this excellent tool, use the tabs to page through the interface), but understanding sequences and where they fit into XQuery can really help new developers when they are reading and writing code their first day.

All literals are sequences, and don&#8217;t need parenthesis if there is only one item or if the expression is the Query Body of the Main Module


  <pre><code class="language-text text">1                 (: a sequence of 1 xs:integer literal :)
"Hello World"     (: a sequence of 1 xs:string literal :)</code></pre>


Sequences collapse.  Sequences don&#8217;t nest like Arrays in other languages. The following evaluate to equivalent sequences:


  <pre><code class="language-text text">("a", "b", "c, "d")
("a", (), "b", "c", "d")
("a", ( "b", "c"), "d")</code></pre>


All expressions are sequences.  Literals, Path Expressions, FLWOR statements, IF/ELSEs , TYPESWITCHes, and COMPARISON all evaluate to sequences, even if they are just returning a sequence of count 1.  These language structures all return sequences, combine into sequences, and even have sequences inside of them:


  <pre><code class="language-xquery xquery">(: returns a sequence of perfect squares :)
for $i in (1 to 10)
return
  $i * $i</code></pre>



  <pre><code class="language-xquery xquery">(: Two statements expressed as a sequence :)
(
  for $book in fn:doc("books.xml")//book
  return
    fn:concat( $book/title, ": ", $book/author), 

  if(fn:exists(//book[fn:contains(./title, "SQL")]) then
    "Yuck"
  else
    "Hooray!"
)</code></pre>



  <pre><code class="language-text text">(: Some of the locations in FWLOR that could be considered sequences :)
for $x in SEQUENCE
let $y := SEQUENCE
return
  SEQUENCE

(: Sequences inside IF/ELSEs :)
if( BooleanExpression ) then
  SEQUENCE
else
  SEQUENCE</code></pre>


Because FLWOR statements both contain and *are* sequences, you can quickly compose them:


  <pre><code class="language-xquery xquery">(: IF statement inside FLWOR statements :)
for $x in (1 to 10)
return
  if($x ne 2) then
    $x * $x
  else
    "the number of years I have loved XQuery"</code></pre>


Composing expressions of course quickly turns into the next discussion:

### The &#8220;Think Like Functions&#8221; Discussion

When I see developers doing the following I usually start the discussion about functions and functional languages:


  <pre><code class="language-xquery xquery">(: I assume local mathematical functions
   functionA, functionB, and functionC are
   defined above :)

(: this is too procedural :)
let $accumulator := 1
let $accumulator := functionA($accumulator)
let $accumulator := functionB($accumulator)
let $accumulator := functionC($accumulator)
return
  $accumulator</code></pre>


The problem is that XQuery is a functional language and variables like $accumulator are immutable.  That means that new memory is allocated for each assignment of an expression to the variable unlike Assembly or C which alter the actual memory address on assignment.  XQuery is just making multiple redundant copies.  The above code will work, but following this practice tends to lead to unnecessary performance problems if not simply inelegant code.  I try to get people to think of the XQuery syntax more like the math they learned in high school where functions can compose f( g( x ) ) :


  <pre><code class="language-xquery xquery">(: now you're thinking with functions :)
functionC( functionB( functionA( 1 )))</code></pre>


Though eventually for style and maintainability purposes I&#8217;ll try to lead the developer to write the following:


  <pre><code class="language-xquery xquery">(: maybe more maintainable, 
   especially when IF and FLWOR statements 
   get mixed in :)
functionC (
  functionB (
    functionA (
      1
    )
  )
)</code></pre>


### The &#8220;XPath first&#8221; Discussion

XPath expressions evaluate to sequences, like just about everything else in XQuery.  XPath also allows quick filtering with XPath predicates, the thing in the square brackets.  As a result many simple FLWOR statements can just be reduced to a single XPath expression which I think encourages developers to move away from XQuery code that is limited to previous experiences with SQL.


  <pre><code class="language-xquery xquery">for $book in //book
where $book/author eq "J K. Rowling"
return
  $book</code></pre>


I take issue with two things in the above code.  First, the FLWOR statement just returns the the iterator without doing anything with it or to it.  Second, the *where* clause could just as easily be done in a predicate.  In early versions of many XQuery interpreters (not just MarkLogic) predicates were significantly faster than *where* clauses.  This isn&#8217;t really true any more but I still suggest the following one line aleternative:


  <pre><code class="language-xquery xquery">//book[author eq "J K. Rowling"]</code></pre>


* * *

Updates:  
Fixed URL link to Ninja tutorial (2012-05-04)

 [1]: http://www.w3schools.com/xpath/xpath_intro.asp
 [2]: http://w3schools.com/xquery/default.asp
 [3]: http://community.marklogic.com/try/ninja/index
 [4]: http://community.marklogic.com/pubs/5.0/apidocs/All.html
 [5]: http://www.w3.org/TR/xquery/#nt-bnf
 [6]: http://railroad.my28msec.com/rr/ui