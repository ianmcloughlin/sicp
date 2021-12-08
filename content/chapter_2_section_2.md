(section_22)=
# Hierarchical Data and the Closure Property

As we have seen, pairs provide a primitive "glue" that we can use to construct compound data objects. Figure&nbsp;<a href="#%_fig_2.2">2.2</a> shows a standard way to visualize a <a name="index_term_1526"></a>pair -- in this case, the pair formed by <tt>(cons 1 2)</tt>. In this representation, which is called <a name="index_term_1528"></a>*box-and-pointer notation*, each object is shown as a <a name="index_term_1530"></a>*pointer* to a box. The box for a primitive object contains a representation of the object. For example, the box for a number contains a numeral. The box for a pair is actually a double box, the left part containing (a pointer to) the <tt>car</tt> of the pair and the right part containing the <tt>cdr</tt>.


We have already seen that <tt>cons</tt> can be used to combine not only numbers but pairs as well. (You made use of this fact, or should have, in doing exercises&nbsp;<a href="chapter_2_section_1.html#exercise_2_2">2.2</a> and&nbsp;<a href="chapter_2_section_1.html#exercise_2_3">2.3</a>.) As a consequence, pairs provide a universal building block from which we can construct all sorts of data structures. Figure&nbsp;<a href="#%_fig_2.3">2.3</a> shows two ways to use pairs to combine the numbers 1, 2, 3, and 4.


<a name="%_fig_2.2"></a>

<div align="left">
  <div align="left">
    **Figure 2.2:**&nbsp;&nbsp;Box-and-pointer representation of <tt>(cons 1 2)</tt>.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_11.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="%_fig_2.3"></a>

<div align="left">
  <div align="left">
    **Figure 2.3:**&nbsp;&nbsp;Two ways to combine 1, 2, 3, and 4 using pairs.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_12.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

The ability to create pairs whose elements are pairs is the essence of list structure's importance as a representational tool. We refer to this ability as the <a name="index_term_1532"></a><a name="index_term_1534"></a>*closure property* of <tt>cons</tt>. In general, an operation for combining data objects satisfies the closure property if the results of combining things with that operation can themselves be combined using the same operation.<a name="call_footnote_Temp_154" href="#footnote_Temp_154" id="call_footnote_Temp_154"><sup><small>6</small></sup></a> Closure is the key to power in any means of combination because it permits us to create <a name="index_term_1538"></a><a name="index_term_1540"></a>*hierarchical* structures -- structures made up of parts, which themselves are made up of parts, and so on.


From the outset of chapter&nbsp;1, we've made essential use of closure in dealing with procedures, because all but the very simplest programs rely on the fact that the elements of a combination can themselves be combinations. In this section, we take up the consequences of closure for compound data. We describe some conventional techniques for using pairs to represent sequences and trees, and we exhibit a graphics language that illustrates closure in a vivid way.<a name="call_footnote_Temp_155" href="#footnote_Temp_155" id="call_footnote_Temp_155"><sup><small>7</small></sup></a>


<a name="%_sec_2.2.1"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_2.2.1">2.2.1&nbsp;&nbsp;Representing Sequences</a></h3>

<a name="%_fig_2.4"></a>

<div align="left">
  <div align="left">
    **Figure 2.4:**&nbsp;&nbsp;The sequence 1, 2, 3, 4 represented as a chain of pairs.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_13.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

One of the useful structures we can build with pairs is a <a name="index_term_1554"></a><a name="index_term_1556"></a><a name="index_term_1558"></a>*sequence* -- an ordered collection of data objects. There are, of course, many ways to represent sequences in terms of pairs. One particularly straightforward representation is illustrated in figure&nbsp;<a href="#%_fig_2.4">2.4</a>, where the sequence 1, 2, 3, 4 is represented as a chain of pairs. The <tt>car</tt> of each pair is the corresponding item in the chain, and the <tt>cdr</tt> of the pair is the next pair in the chain. The <tt>cdr</tt> of the final pair signals the end of the sequence by pointing to a distinguished value that is not a pair, represented in box-and-pointer diagrams as a diagonal line <a name="index_term_1560"></a>and in programs as the value of the variable <a name="index_term_1562"></a><a name="index_term_1564"></a><tt>nil</tt>. The entire sequence is constructed by nested <tt>cons</tt> operations:


<tt>(cons&nbsp;1<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;2<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;3<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;4&nbsp;nil))))<br></tt>


Such a sequence of pairs, formed by nested <tt>cons</tt>es, is called a <a name="index_term_1566"></a>*list*, and Scheme provides a primitive called <a name="index_term_1568"></a><a name="index_term_1570"></a><tt>list</tt> to help in constructing lists.<a name="call_footnote_Temp_156" href="#footnote_Temp_156" id="call_footnote_Temp_156"><sup><small>8</small></sup></a> The above sequence could be produced by <tt>(list 1 2 3 4)</tt>. In general,


<tt>(list&nbsp;&lt;*a<sub>1</sub>*&gt;&nbsp;&lt;*a<sub>2</sub>*&gt;&nbsp;</tt>... &lt;*a<sub>*n*</sub>*&gt;)<br>


is equivalent to


<tt>(cons&nbsp;&lt;*a<sub>1</sub>*&gt;&nbsp;(cons&nbsp;&lt;*a<sub>2</sub>*&gt;&nbsp;(cons&nbsp;</tt>... (cons&nbsp;&lt;*a<sub>*n*</sub>*&gt;&nbsp;nil)&nbsp;<tt>...</tt>)))<br>


Lisp systems conventionally print lists by printing the sequence of <a name="index_term_1576"></a>elements, enclosed in parentheses. Thus, the data object in figure&nbsp;<a href="#%_fig_2.4">2.4</a> is printed as <tt>(1 2 3 4)</tt>:


<tt>(define&nbsp;one-through-four&nbsp;(list&nbsp;1&nbsp;2&nbsp;3&nbsp;4))<br>
<br>
one-through-four<br>
<i>(1&nbsp;2&nbsp;3&nbsp;4)</i><br></tt>


Be careful not to confuse the expression <tt>(list 1 2 3 4)</tt> with the list <tt>(1 2 3 4)</tt>, which is the result obtained when the expression is evaluated. Attempting to evaluate the expression <tt>(1 2 3 4)</tt> will signal an error when the interpreter tries to apply the procedure <tt>1</tt> to arguments <tt>2</tt>, <tt>3</tt>, and <tt>4</tt>.


We can think of <a name="index_term_1578"></a><a name="index_term_1580"></a><tt>car</tt> as selecting the first item in the list, and of <a name="index_term_1582"></a><tt>cdr</tt> as selecting the sublist consisting of all but the first item. Nested applications of <tt>car</tt> and <tt>cdr</tt> can be used to extract the second, third, and subsequent items in the list.<a name="call_footnote_Temp_157" href="#footnote_Temp_157" id="call_footnote_Temp_157"><sup><small>9</small></sup></a> The constructor <a name="index_term_1592"></a><tt>cons</tt> makes a list like the original one, but with an additional item at the beginning.


<tt>(car&nbsp;one-through-four)<br>
<i>1</i><br>
<br>
(cdr&nbsp;one-through-four)<br>
<i>(2&nbsp;3&nbsp;4)</i><br>
(car&nbsp;(cdr&nbsp;one-through-four))<br>
<i>2</i><br>
<br>
(cons&nbsp;10&nbsp;one-through-four)<br>
<i>(10&nbsp;1&nbsp;2&nbsp;3&nbsp;4)</i><br>
<br>
(cons&nbsp;5&nbsp;one-through-four)<br>
<i>(5&nbsp;1&nbsp;2&nbsp;3&nbsp;4)</i><br></tt>


The value of <tt>nil</tt>, used to terminate the chain of pairs, can be thought of as a sequence of no elements, the <a name="index_term_1594"></a><a name="index_term_1596"></a>*empty list*. The word *nil* is a contraction of the Latin word *nihil*, which means "nothing."<a name="call_footnote_Temp_158" href="#footnote_Temp_158" id="call_footnote_Temp_158"><sup><small>10</small></sup></a>


<a name="%_sec_Temp_159"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_159">List operations</a></h4>

<a name="index_term_1602"></a><a name="index_term_1604"></a> The use of pairs to represent sequences of elements as lists is accompanied by conventional programming techniques for manipulating lists by successively <a name="index_term_1606"></a><a name="index_term_1608"></a>"<tt>cdr</tt>ing down" the lists. For example, the procedure <a name="index_term_1610"></a><tt>list-ref</tt> takes as arguments a list and a number *n* and returns the *n*th item of the list. It is customary to number the elements of the list beginning with 0. The method for computing <tt>list-ref</tt> is the following:

<ul>
  <li>For *n* = 0, <tt>list-ref</tt> should return the <tt>car</tt> of the list.</li>

  <li>Otherwise, <tt>list-ref</tt> should return the (*n* - 1)st item of the <tt>cdr</tt> of the list.</li>
</ul>

<tt><a name="index_term_1612"></a>(define&nbsp;(list-ref&nbsp;items&nbsp;n)<br>
&nbsp;&nbsp;(if&nbsp;(=&nbsp;n&nbsp;0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(car&nbsp;items)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list-ref&nbsp;(cdr&nbsp;items)&nbsp;(-&nbsp;n&nbsp;1))))<br>
(define&nbsp;squares&nbsp;(list&nbsp;1&nbsp;4&nbsp;9&nbsp;16&nbsp;25))<br>
<br>
(list-ref&nbsp;squares&nbsp;3)<br>
<i>16</i><br></tt>


Often we <tt>cdr</tt> down the whole list. To aid in this, Scheme includes a primitive predicate <a name="index_term_1614"></a><a name="index_term_1616"></a><a name="index_term_1618"></a><tt>null?</tt>, which tests whether its argument is the empty list. The procedure <a name="index_term_1620"></a><a name="index_term_1622"></a><tt>length</tt>, which returns the number of items in a list, illustrates this typical pattern of use:


<tt><a name="index_term_1624"></a>(define&nbsp;(length&nbsp;items)<br>
&nbsp;&nbsp;(if&nbsp;(null?&nbsp;items)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;0<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(+&nbsp;1&nbsp;(length&nbsp;(cdr&nbsp;items)))))<br>
(define&nbsp;odds&nbsp;(list&nbsp;1&nbsp;3&nbsp;5&nbsp;7))<br>
<br>
(length&nbsp;odds)<br>
<i>4</i><br></tt>


The <tt>length</tt> procedure implements a simple recursive plan. The reduction step is:

<ul>
  <li>The <tt>length</tt> of any list is 1 plus the <tt>length</tt> of the <tt>cdr</tt> of the list.</li>
</ul>

This is applied successively until we reach the base case:

<ul>
  <li>The <tt>length</tt> of the empty list is 0.</li>
</ul>

We could also compute <tt>length</tt> in an iterative style:


<tt><a name="index_term_1626"></a>(define&nbsp;(length&nbsp;items)<br>
&nbsp;&nbsp;(define&nbsp;(length-iter&nbsp;a&nbsp;count)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;(null?&nbsp;a)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;count<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(length-iter&nbsp;(cdr&nbsp;a)&nbsp;(+&nbsp;1&nbsp;count))))<br>
&nbsp;&nbsp;(length-iter&nbsp;items&nbsp;0))<br></tt>


Another conventional programming technique is to <a name="index_term_1628"></a><a name="index_term_1630"></a>"<tt>cons</tt> up" an answer list while <tt>cdr</tt>ing down a list, as in the procedure <a name="index_term_1632"></a><a name="index_term_1634"></a><tt>append</tt>, which takes two lists as arguments and combines their elements to make a new list:


<tt>(append&nbsp;squares&nbsp;odds)<br>
<i>(1&nbsp;4&nbsp;9&nbsp;16&nbsp;25&nbsp;1&nbsp;3&nbsp;5&nbsp;7)</i><br>
<br>
(append&nbsp;odds&nbsp;squares)<br>
<i>(1&nbsp;3&nbsp;5&nbsp;7&nbsp;1&nbsp;4&nbsp;9&nbsp;16&nbsp;25)</i><br></tt>


<tt>Append</tt> is also implemented using a recursive plan. To <tt>append</tt> lists <tt>list1</tt> and <tt>list2</tt>, do the following:

<ul>
  <li>If <tt>list1</tt> is the empty list, then the result is just <tt>list2</tt>.</li>

  <li>Otherwise, <tt>append</tt> the <tt>cdr</tt> of <tt>list1</tt> and <tt>list2</tt>, and <tt>cons</tt> the <tt>car</tt> of <tt>list1</tt> onto the result:</li>
</ul>

<tt><a name="index_term_1636"></a>(define&nbsp;(append&nbsp;list1&nbsp;list2)<br>
&nbsp;&nbsp;(if&nbsp;(null?&nbsp;list1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;list2<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;(car&nbsp;list1)&nbsp;(append&nbsp;(cdr&nbsp;list1)&nbsp;list2))))<br></tt>


<a name="exercise_2_17">**Exercise 2.17**</a> Define a procedure <a name="index_term_1638"></a><a name="index_term_1640"></a><tt>last-pair</tt> that returns the list that contains only the last element of a given (nonempty) list:


<tt>(last-pair&nbsp;(list&nbsp;23&nbsp;72&nbsp;149&nbsp;34))<br>
<i>(34)</i><br></tt>


<a name="exercise_2_18">**Exercise 2.18**</a> Define a procedure <a name="index_term_1642"></a><a name="index_term_1644"></a><tt>reverse</tt> that takes a list as argument and returns a list of the same elements in reverse order:


<tt>(reverse&nbsp;(list&nbsp;1&nbsp;4&nbsp;9&nbsp;16&nbsp;25))<br>
<i>(25&nbsp;16&nbsp;9&nbsp;4&nbsp;1)</i><br></tt>


<a name="exercise_2_19">**Exercise 2.19**</a> Consider the <a name="index_term_1646"></a>change-counting program of section <a href="chapter_1_section_2.html#%_sec_1.2.2">1.2.2</a>. It would be nice to be able to easily change the currency used by the program, so that we could compute the number of ways to change a British pound, for example. As the program is written, the knowledge of the currency is distributed partly into the procedure <tt>first-denomination</tt> and partly into the procedure <tt>count-change</tt> (which knows that there are five kinds of U.S. coins). It would be nicer to be able to supply a list of coins to be used for making change.


We want to rewrite the procedure <tt>cc</tt> so that its second argument is a list of the values of the coins to use rather than an integer specifying which coins to use. We could then have lists that defined each kind of currency:


<tt>(define&nbsp;us-coins&nbsp;(list&nbsp;50&nbsp;25&nbsp;10&nbsp;5&nbsp;1))<br>
(define&nbsp;uk-coins&nbsp;(list&nbsp;100&nbsp;50&nbsp;20&nbsp;10&nbsp;5&nbsp;2&nbsp;1&nbsp;0.5))<br></tt>


We could then call <tt>cc</tt> as follows:


<tt>(cc&nbsp;100&nbsp;us-coins)<br>
<i>292</i><br></tt>


To do this will require changing the program <tt>cc</tt> somewhat. It will still have the same form, but it will access its second argument differently, as follows:


<tt>(define&nbsp;(cc&nbsp;amount&nbsp;coin-values)<br>
&nbsp;&nbsp;(cond&nbsp;((=&nbsp;amount&nbsp;0)&nbsp;1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;((or&nbsp;(&lt;&nbsp;amount&nbsp;0)&nbsp;(no-more?&nbsp;coin-values))&nbsp;0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(else<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(+&nbsp;(cc&nbsp;amount<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(except-first-denomination&nbsp;coin-values))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cc&nbsp;(-&nbsp;amount<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(first-denomination&nbsp;coin-values))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;coin-values)))))<br></tt>


Define the procedures <tt>first-denomination</tt>, <tt>except-first-denomination</tt>, and <tt>no-more?</tt> in terms of primitive operations on list structures. Does the order of the list <tt>coin-values</tt> affect the answer produced by <tt>cc</tt>? Why or why not?


<a name="exercise_2_20">**Exercise 2.20**</a> <a name="index_term_1648"></a><a name="index_term_1650"></a><a name="index_term_1652"></a><a name="index_term_1654"></a>The procedures <tt>+</tt>, <tt>*</tt>, and <tt>list</tt> take arbitrary numbers of arguments. One way to define such procedures is to use <tt>define</tt> with *dotted-tail notation*. In a procedure definition, a parameter list that has a dot before the last parameter name indicates that, when the procedure is called, the initial parameters (if any) will have as values the initial arguments, as usual, but the final parameter's value will be a *list* of any remaining arguments. For instance, given the definition


<tt>(define&nbsp;(f&nbsp;x&nbsp;y&nbsp;.&nbsp;z)&nbsp;*&lt;body&gt;*)<br></tt>


the procedure <tt>f</tt> can be called with two or more arguments. If we evaluate


<tt>(f&nbsp;1&nbsp;2&nbsp;3&nbsp;4&nbsp;5&nbsp;6)<br></tt>


then in the body of <tt>f</tt>, <tt>x</tt> will be 1, <tt>y</tt> will be 2, and <tt>z</tt> will be the list <tt>(3 4 5 6)</tt>. Given the definition


<tt>(define&nbsp;(g&nbsp;.&nbsp;w)&nbsp;*&lt;body&gt;*)<br></tt>


the procedure <tt>g</tt> can be called with zero or more arguments. If we evaluate


<tt>(g&nbsp;1&nbsp;2&nbsp;3&nbsp;4&nbsp;5&nbsp;6)<br></tt>


then in the body of <tt>g</tt>, <tt>w</tt> will be the list <tt>(1 2 3 4 5 6)</tt>.<a name="call_footnote_Temp_164" href="#footnote_Temp_164" id="call_footnote_Temp_164"><sup><small>11</small></sup></a>


Use this notation to write a procedure <tt>same-parity</tt> that takes one or more integers and returns a list of all the arguments that have the same even-odd parity as the first argument. For example,


<tt>(same-parity&nbsp;1&nbsp;2&nbsp;3&nbsp;4&nbsp;5&nbsp;6&nbsp;7)<br>
<i>(1&nbsp;3&nbsp;5&nbsp;7)</i><br>
<br>
(same-parity&nbsp;2&nbsp;3&nbsp;4&nbsp;5&nbsp;6&nbsp;7)<br>
<i>(2&nbsp;4&nbsp;6)</i><br></tt>


<a name="%_sec_Temp_165"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_165">Mapping over lists</a></h4>

<a name="index_term_1658"></a><a name="index_term_1660"></a> One extremely useful operation is to apply some transformation to each element in a list and generate the list of results. For instance, the following procedure scales each number in a list by a given factor:


<tt><a name="index_term_1662"></a>(define&nbsp;(scale-list&nbsp;items&nbsp;factor)<br>
&nbsp;&nbsp;(if&nbsp;(null?&nbsp;items)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nil<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;(*&nbsp;(car&nbsp;items)&nbsp;factor)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(scale-list&nbsp;(cdr&nbsp;items)&nbsp;factor))))<br>
(scale-list&nbsp;(list&nbsp;1&nbsp;2&nbsp;3&nbsp;4&nbsp;5)&nbsp;10)<br>
<i>(10&nbsp;20&nbsp;30&nbsp;40&nbsp;50)</i><br></tt>


We can abstract this general idea and capture it as a common pattern expressed as a higher-order procedure, just as in section <a href="chapter_1_section_3.html#%_sec_1.3">1.3</a>. The higher-order procedure here is called <tt>map</tt>. <tt>Map</tt> takes as arguments a procedure of one argument and a list, and returns a list of the results produced by applying the procedure to each element in the list:<a name="call_footnote_Temp_166" href="#footnote_Temp_166" id="call_footnote_Temp_166"><sup><small>12</small></sup></a>


<tt><a name="index_term_1666"></a>(define&nbsp;(map&nbsp;proc&nbsp;items)<br>
&nbsp;&nbsp;(if&nbsp;(null?&nbsp;items)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nil<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;(proc&nbsp;(car&nbsp;items))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(map&nbsp;proc&nbsp;(cdr&nbsp;items)))))<br>
(map&nbsp;abs&nbsp;(list&nbsp;-10&nbsp;2.5&nbsp;-11.6&nbsp;17))<br>
<i>(10&nbsp;2.5&nbsp;11.6&nbsp;17)</i><br>
(map&nbsp;(lambda&nbsp;(x)&nbsp;(*&nbsp;x&nbsp;x))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;1&nbsp;2&nbsp;3&nbsp;4))<br>
<i>(1&nbsp;4&nbsp;9&nbsp;16)</i><br></tt>


Now we can give a new definition of <tt>scale-list</tt> in terms of <tt>map</tt>:


<tt><a name="index_term_1668"></a>(define&nbsp;(scale-list&nbsp;items&nbsp;factor)<br>
&nbsp;&nbsp;(map&nbsp;(lambda&nbsp;(x)&nbsp;(*&nbsp;x&nbsp;factor))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;items))<br></tt>


<tt>Map</tt> is an important construct, not only because it captures a common pattern, but because it establishes a higher level of abstraction in dealing with lists. In the original definition of <tt>scale-list</tt>, the recursive structure of the program draws attention to the element-by-element processing of the list. Defining <tt>scale-list</tt> in terms of <tt>map</tt> suppresses that level of detail and emphasizes that scaling transforms a list of elements to a list of results. The difference between the two definitions is not that the computer is performing a different process (it isn't) but that we think about the process differently. In effect, <tt>map</tt> helps establish an abstraction barrier that isolates the implementation of procedures that transform lists from the details of how the elements of the list are extracted and combined. Like the barriers shown in figure&nbsp;<a href="chapter_2_section_1.html#%_fig_2.1">2.1</a>, this abstraction gives us the flexibility to change the low-level details of how sequences are implemented, while preserving the conceptual framework of operations that transform sequences to sequences. section <a href="#%_sec_2.2.3">2.2.3</a> expands on this use of sequences as a framework for organizing programs.


<a name="exercise_2_21">**Exercise 2.21**</a> The procedure <tt>square-list</tt> takes a list of numbers as argument and returns a list of the squares of those numbers.


<tt>(square-list&nbsp;(list&nbsp;1&nbsp;2&nbsp;3&nbsp;4))<br>
<i>(1&nbsp;4&nbsp;9&nbsp;16)</i><br></tt>


Here are two different definitions of <tt>square-list</tt>. Complete both of them by filling in the missing expressions:


<tt>(define&nbsp;(square-list&nbsp;items)<br>
&nbsp;&nbsp;(if&nbsp;(null?&nbsp;items)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nil<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;&lt;*??*&gt;&nbsp;&lt;*??*&gt;)))<br>
(define&nbsp;(square-list&nbsp;items)<br>
&nbsp;&nbsp;(map&nbsp;&lt;*??*&gt;&nbsp;&lt;*??*&gt;))<br></tt>


<a name="exercise_2_22">**Exercise 2.22**</a> Louis Reasoner tries to rewrite the first <tt>square-list</tt> procedure of exercise&nbsp;<a href="#%_thm_2.21">2.21</a> so that it evolves an iterative process:


<tt>(define&nbsp;(square-list&nbsp;items)<br>
&nbsp;&nbsp;(define&nbsp;(iter&nbsp;things&nbsp;answer)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;(null?&nbsp;things)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;answer<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(iter&nbsp;(cdr&nbsp;things)&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;(square&nbsp;(car&nbsp;things))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;answer))))<br>
&nbsp;&nbsp;(iter&nbsp;items&nbsp;nil))<br></tt>


Unfortunately, defining <tt>square-list</tt> this way produces the answer list in the reverse order of the one desired. Why?


Louis then tries to fix his bug by interchanging the arguments to <tt>cons</tt>:


<tt>(define&nbsp;(square-list&nbsp;items)<br>
&nbsp;&nbsp;(define&nbsp;(iter&nbsp;things&nbsp;answer)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;(null?&nbsp;things)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;answer<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(iter&nbsp;(cdr&nbsp;things)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;answer<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(square&nbsp;(car&nbsp;things))))))<br>
&nbsp;&nbsp;(iter&nbsp;items&nbsp;nil))<br></tt>


This doesn't work either. Explain.


<a name="exercise_2_23">**Exercise 2.23**</a> The procedure <a name="index_term_1670"></a><tt>for-each</tt> is similar to <tt>map</tt>. It takes as arguments a procedure and a list of elements. However, rather than forming a list of the results, <tt>for-each</tt> just applies the procedure to each of the elements in turn, from left to right. The values returned by applying the procedure to the elements are not used at all -- <tt>for-each</tt> is used with procedures that perform an action, such as printing. For example,


<tt>(for-each&nbsp;(lambda&nbsp;(x)&nbsp;(newline)&nbsp;(display&nbsp;x))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;57&nbsp;321&nbsp;88))<br>
<i>57</i><br>
<i>321</i><br>
<i>88</i><br></tt>


The value returned by the call to <tt>for-each</tt> (not illustrated above) can be something arbitrary, such as true. Give an implementation of <tt>for-each</tt>.


<a name="%_sec_2.2.2"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_2.2.2">2.2.2&nbsp;&nbsp;Hierarchical Structures</a></h3>

<a name="index_term_1672"></a><a name="index_term_1674"></a><a name="index_term_1676"></a><a name="index_term_1678"></a> The representation of sequences in terms of lists generalizes naturally to represent sequences whose elements may themselves be sequences. For example, we can regard the object <tt>((1 2) 3 4)</tt> constructed by


<tt>(cons&nbsp;(list&nbsp;1&nbsp;2)&nbsp;(list&nbsp;3&nbsp;4))<br></tt>


as a list of three items, the first of which is itself a list, <tt>(1 2)</tt>. Indeed, this is suggested by the form in which the result is printed by the interpreter. Figure&nbsp;<a href="#%_fig_2.5">2.5</a> shows the representation of this structure in terms of pairs.


<a name="%_fig_2.5"></a>

<div align="left">
  <div align="left">
    **Figure 2.5:**&nbsp;&nbsp;Structure formed by <tt>(cons (list 1 2) (list 3 4))</tt>.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_15.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

Another way to think of sequences whose elements are sequences is as *trees*. The elements of the sequence are the branches of the tree, and elements that are themselves sequences are subtrees. Figure&nbsp;<a href="#%_fig_2.6">2.6</a> shows the structure in figure&nbsp;<a href="#%_fig_2.5">2.5</a> viewed as a tree.


<a name="%_fig_2.6"></a>

<div align="left">
  <div align="left">
    **Figure 2.6:**&nbsp;&nbsp;The list structure in figure&nbsp;<a href="#%_fig_2.5">2.5</a> viewed as a tree.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_16.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="index_term_1680"></a>Recursion is a natural tool for dealing with tree structures, since we can often reduce operations on trees to operations on their branches, which reduce in turn to operations on the branches of the branches, and so on, until we reach the leaves of the tree. As an example, compare the <tt>length</tt> procedure of section <a href="#%_sec_2.2.1">2.2.1</a> with the <a name="index_term_1682"></a><a name="index_term_1684"></a><tt>count-leaves</tt> procedure, which returns the total number of leaves of a tree:


<tt>(define&nbsp;x&nbsp;(cons&nbsp;(list&nbsp;1&nbsp;2)&nbsp;(list&nbsp;3&nbsp;4)))<br>
<br>
(length&nbsp;x)<br>
<i>3</i><br>
(count-leaves&nbsp;x)<br>
<i>4</i><br>
<br>
(list&nbsp;x&nbsp;x)<br>
<i>(((1&nbsp;2)&nbsp;3&nbsp;4)&nbsp;((1&nbsp;2)&nbsp;3&nbsp;4))</i><br>
<br>
(length&nbsp;(list&nbsp;x&nbsp;x))<br>
<i>2</i><br>
<br>
(count-leaves&nbsp;(list&nbsp;x&nbsp;x))<br>
<i>8</i><br></tt>


To implement <tt>count-leaves</tt>, recall the recursive plan for computing <tt>length</tt>:

<ul>
  <li><tt>Length</tt> of a list <tt>x</tt> is 1 plus <tt>length</tt> of the <tt>cdr</tt> of <tt>x</tt>.</li>

  <li><tt>Length</tt> of the empty list is 0.</li>
</ul>

<tt>Count-leaves</tt> is similar. The value for the empty list is the same:

<ul>
  <li><tt>Count-leaves</tt> of the empty list is 0.</li>
</ul>

But in the reduction step, where we strip off the <tt>car</tt> of the list, we must take into account that the <tt>car</tt> may itself be a tree whose leaves we need to count. Thus, the appropriate reduction step is

<ul>
  <li><tt>Count-leaves</tt> of a tree <tt>x</tt> is <tt>count-leaves</tt> of the <tt>car</tt> of <tt>x</tt> plus <tt>count-leaves</tt> of the <tt>cdr</tt> of <tt>x</tt>.</li>
</ul>

Finally, by taking <tt>car</tt>s we reach actual leaves, so we need another base case:

<ul>
  <li><tt>Count-leaves</tt> of a leaf is 1.</li>
</ul>

To aid in writing recursive procedures on trees, Scheme provides the primitive predicate <a name="index_term_1686"></a><a name="index_term_1688"></a><tt>pair?</tt>, which tests whether its argument is a pair. Here is the complete procedure:<a name="call_footnote_Temp_170" href="#footnote_Temp_170" id="call_footnote_Temp_170"><sup><small>13</small></sup></a>


<tt><a name="index_term_1690"></a>(define&nbsp;(count-leaves&nbsp;x)<br>
&nbsp;&nbsp;(cond&nbsp;((null?&nbsp;x)&nbsp;0)&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;((not&nbsp;(pair?&nbsp;x))&nbsp;1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(else&nbsp;(+&nbsp;(count-leaves&nbsp;(car&nbsp;x))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(count-leaves&nbsp;(cdr&nbsp;x))))))<br></tt>


<a name="exercise_2_24">**Exercise 2.24**</a> Suppose we evaluate the expression <tt>(list 1 (list 2 (list 3 4)))</tt>. Give the result printed by the interpreter, the corresponding box-and-pointer structure, and the interpretation of this as a tree (as in figure&nbsp;<a href="#%_fig_2.6">2.6</a>).


<a name="exercise_2_25">**Exercise 2.25**</a> Give combinations of <tt>car</tt>s and <tt>cdr</tt>s that will pick 7 from each of the following lists:


<tt>(1&nbsp;3&nbsp;(5&nbsp;7)&nbsp;9)<br>
<br>
((7))<br>
<br>
(1&nbsp;(2&nbsp;(3&nbsp;(4&nbsp;(5&nbsp;(6&nbsp;7))))))<br></tt>


<a name="exercise_2_26">**Exercise 2.26**</a> Suppose we define <tt>x</tt> and <tt>y</tt> to be two lists:


<tt>(define&nbsp;x&nbsp;(list&nbsp;1&nbsp;2&nbsp;3))<br>
(define&nbsp;y&nbsp;(list&nbsp;4&nbsp;5&nbsp;6))<br></tt>


What result is printed by the interpreter in response to evaluating each of the following expressions:


<tt>(append&nbsp;x&nbsp;y)<br>
<br>
(cons&nbsp;x&nbsp;y)<br>
<br>
(list&nbsp;x&nbsp;y)<br></tt>


<a name="exercise_2_27">**Exercise 2.27**</a> Modify your <tt>reverse</tt> procedure of exercise&nbsp;<a href="#%_thm_2.18">2.18</a> to produce a <a name="index_term_1692"></a><a name="index_term_1694"></a><tt>deep-reverse</tt> procedure that takes a list as argument and returns as its value the list with its elements reversed and with all sublists deep-reversed as well. For example,


<tt>(define&nbsp;x&nbsp;(list&nbsp;(list&nbsp;1&nbsp;2)&nbsp;(list&nbsp;3&nbsp;4)))<br>
<br>
x<br>
<i>((1&nbsp;2)&nbsp;(3&nbsp;4))</i><br>
<br>
(reverse&nbsp;x)<br>
<i>((3&nbsp;4)&nbsp;(1&nbsp;2))</i><br>
<br>
(deep-reverse&nbsp;x)<br>
<i>((4&nbsp;3)&nbsp;(2&nbsp;1))</i><br></tt>


<a name="exercise_2_28">**Exercise 2.28**</a> Write a procedure <a name="index_term_1696"></a><a name="index_term_1698"></a><tt>fringe</tt> that takes as argument a tree (represented as a list) and returns a list whose elements are all the leaves of the tree arranged in left-to-right order. For example,


<tt>(define&nbsp;x&nbsp;(list&nbsp;(list&nbsp;1&nbsp;2)&nbsp;(list&nbsp;3&nbsp;4)))<br>
<br>
(fringe&nbsp;x)<br>
<i>(1&nbsp;2&nbsp;3&nbsp;4)</i><br>
<br>
(fringe&nbsp;(list&nbsp;x&nbsp;x))<br>
<i>(1&nbsp;2&nbsp;3&nbsp;4&nbsp;1&nbsp;2&nbsp;3&nbsp;4)</i><br></tt>


<a name="exercise_2_29">**Exercise 2.29**</a> <a name="index_term_1700"></a>A binary mobile consists of two branches, a left branch and a right branch. Each branch is a rod of a certain length, from which hangs either a weight or another binary mobile. We can represent a binary mobile using compound data by constructing it from two branches (for example, using <tt>list</tt>):


<tt>(define&nbsp;(make-mobile&nbsp;left&nbsp;right)<br>
&nbsp;&nbsp;(list&nbsp;left&nbsp;right))<br></tt>


A branch is constructed from a <tt>length</tt> (which must be a number) together with a <tt>structure</tt>, which may be either a number (representing a simple weight) or another mobile:


<tt>(define&nbsp;(make-branch&nbsp;length&nbsp;structure)<br>
&nbsp;&nbsp;(list&nbsp;length&nbsp;structure))<br></tt>


a.&nbsp;&nbsp;Write the corresponding selectors <tt>left-branch</tt> and <tt>right-branch</tt>, which return the branches of a mobile, and <tt>branch-length</tt> and <tt>branch-structure</tt>, which return the components of a branch.


b.&nbsp;&nbsp;Using your selectors, define a procedure <tt>total-weight</tt> that returns the total weight of a mobile.


c.&nbsp;&nbsp;A mobile is said to be <a name="index_term_1702"></a>*balanced* if the torque applied by its top-left branch is equal to that applied by its top-right branch (that is, if the length of the left rod multiplied by the weight hanging from that rod is equal to the corresponding product for the right side) and if each of the submobiles hanging off its branches is balanced. Design a predicate that tests whether a binary mobile is balanced.


d.&nbsp;&nbsp;Suppose we change the representation of mobiles so that the constructors are


<tt>(define&nbsp;(make-mobile&nbsp;left&nbsp;right)<br>
&nbsp;&nbsp;(cons&nbsp;left&nbsp;right))<br>
(define&nbsp;(make-branch&nbsp;length&nbsp;structure)<br>
&nbsp;&nbsp;(cons&nbsp;length&nbsp;structure))<br></tt>


How much do you need to change your programs to convert to the new representation?


<a name="%_sec_Temp_177"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_177">Mapping over trees</a></h4>

<a name="index_term_1704"></a><a name="index_term_1706"></a> Just as <tt>map</tt> is a powerful abstraction for dealing with sequences, <tt>map</tt> together with recursion is a powerful abstraction for dealing with trees. For instance, the <tt>scale-tree</tt> procedure, analogous to <tt>scale-list</tt> of section <a href="#%_sec_2.2.1">2.2.1</a>, takes as arguments a numeric factor and a tree whose leaves are numbers. It returns a tree of the same shape, where each number is multiplied by the factor. The recursive plan for <tt>scale-tree</tt> is similar to the one for <tt>count-leaves</tt>:


<tt><a name="index_term_1708"></a>(define&nbsp;(scale-tree&nbsp;tree&nbsp;factor)<br>
&nbsp;&nbsp;(cond&nbsp;((null?&nbsp;tree)&nbsp;nil)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;((not&nbsp;(pair?&nbsp;tree))&nbsp;(*&nbsp;tree&nbsp;factor))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(else&nbsp;(cons&nbsp;(scale-tree&nbsp;(car&nbsp;tree)&nbsp;factor)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(scale-tree&nbsp;(cdr&nbsp;tree)&nbsp;factor)))))<br>
(scale-tree&nbsp;(list&nbsp;1&nbsp;(list&nbsp;2&nbsp;(list&nbsp;3&nbsp;4)&nbsp;5)&nbsp;(list&nbsp;6&nbsp;7))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;10)<br>
<i>(10&nbsp;(20&nbsp;(30&nbsp;40)&nbsp;50)&nbsp;(60&nbsp;70))</i><br></tt>


Another way to implement <tt>scale-tree</tt> is to regard the tree as a sequence of sub-trees and use <tt>map</tt>. We map over the sequence, scaling each sub-tree in turn, and return the list of results. In the base case, where the tree is a leaf, we simply multiply by the factor:


<tt><a name="index_term_1710"></a>(define&nbsp;(scale-tree&nbsp;tree&nbsp;factor)<br>
&nbsp;&nbsp;(map&nbsp;(lambda&nbsp;(sub-tree)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;(pair?&nbsp;sub-tree)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(scale-tree&nbsp;sub-tree&nbsp;factor)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;sub-tree&nbsp;factor)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tree))<br></tt>


Many tree operations can be implemented by similar combinations of sequence operations and recursion.


<a name="exercise_2_30">**Exercise 2.30**</a> Define a procedure <tt>square-tree</tt> analogous to the <tt>square-list</tt> procedure of exercise&nbsp;<a href="#%_thm_2.21">2.21</a>. That is, <tt>square-list</tt> should behave as follows:


<tt>(square-tree<br>
&nbsp;(list&nbsp;1<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;2&nbsp;(list&nbsp;3&nbsp;4)&nbsp;5)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;6&nbsp;7)))<br>
<i>(1&nbsp;(4&nbsp;(9&nbsp;16)&nbsp;25)&nbsp;(36&nbsp;49))</i><br></tt>


Define <tt>square-tree</tt> both directly (i.e., without using any higher-order procedures) and also by using <tt>map</tt> and recursion.


<a name="exercise_2_31">**Exercise 2.31**</a> Abstract your answer to exercise&nbsp;<a href="#%_thm_2.30">2.30</a> to produce a procedure <a name="index_term_1712"></a><tt>tree-map</tt> with the property that <tt>square-tree</tt> could be defined as


<tt>(define&nbsp;(square-tree&nbsp;tree)&nbsp;(tree-map&nbsp;square&nbsp;tree))<br></tt>


<a name="exercise_2_32">**Exercise 2.32**</a> We can represent a <a name="index_term_1714"></a>set as a list of distinct elements, and we can represent the set of all subsets of the set as a list of lists. For example, if the set is <tt>(1&nbsp;2&nbsp;3)</tt>, then the set of all subsets is <tt>(() (3) (2) (2&nbsp;3) (1) (1&nbsp;3) (1&nbsp;2) (1&nbsp;2&nbsp;3))</tt>. Complete the following definition of a procedure that generates the set of subsets of a set and give a clear explanation of why it works:


<tt><a name="index_term_1716"></a>(define&nbsp;(subsets&nbsp;s)<br>
&nbsp;&nbsp;(if&nbsp;(null?&nbsp;s)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;nil)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((rest&nbsp;(subsets&nbsp;(cdr&nbsp;s))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(append&nbsp;rest&nbsp;(map&nbsp;&lt;*??*&gt;&nbsp;rest)))))<br></tt>


<a name="%_sec_2.2.3"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_2.2.3">2.2.3&nbsp;&nbsp;Sequences as Conventional Interfaces</a></h3>

<a name="index_term_1718"></a><a name="index_term_1720"></a> In working with compound data, we've stressed how data abstraction permits us to design programs without becoming enmeshed in the details of data representations, and how abstraction preserves for us the flexibility to experiment with alternative representations. In this section, we introduce another powerful design principle for working with data structures -- the use of *conventional interfaces*.


In section <a href="chapter_1_section_3.html#%_sec_1.3">1.3</a> we saw how program abstractions, implemented as higher-order procedures, can capture common patterns in programs that deal with numerical data. Our ability to formulate analogous operations for working with compound data depends crucially on the style in which we manipulate our data structures. Consider, for example, the following procedure, analogous to the <tt>count-leaves</tt> procedure of section <a href="#%_sec_2.2.2">2.2.2</a>, which takes a tree as argument and computes the sum of the squares of the leaves that are odd:


<tt><a name="index_term_1722"></a>(define&nbsp;(sum-odd-squares&nbsp;tree)<br>
&nbsp;&nbsp;(cond&nbsp;((null?&nbsp;tree)&nbsp;0)&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;((not&nbsp;(pair?&nbsp;tree))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;(odd?&nbsp;tree)&nbsp;(square&nbsp;tree)&nbsp;0))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(else&nbsp;(+&nbsp;(sum-odd-squares&nbsp;(car&nbsp;tree))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(sum-odd-squares&nbsp;(cdr&nbsp;tree))))))<br></tt>


On the surface, this procedure is very different from the following one, which constructs a list of all the even Fibonacci numbers *F**i**b*(*k*), where *k* is less than or equal to a given integer *n*:


<tt><a name="index_term_1724"></a>(define&nbsp;(even-fibs&nbsp;n)<br>
&nbsp;&nbsp;(define&nbsp;(next&nbsp;k)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;(&gt;&nbsp;k&nbsp;n)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nil<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((f&nbsp;(fib&nbsp;k)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;(even?&nbsp;f)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;f&nbsp;(next&nbsp;(+&nbsp;k&nbsp;1)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(next&nbsp;(+&nbsp;k&nbsp;1))))))<br>
&nbsp;&nbsp;(next&nbsp;0))<br></tt>


Despite the fact that these two procedures are structurally very different, a more abstract description of the two computations reveals a great deal of similarity. The first program

<ul>
  <li>enumerates the leaves of a tree;</li>

  <li>filters them, selecting the odd ones;</li>

  <li>squares each of the selected ones; and</li>

  <li>accumulates the results using <tt>+</tt>, starting with 0.</li>
</ul>

The second program

<ul>
  <li>enumerates the integers from 0 to *n*;</li>

  <li>computes the Fibonacci number for each integer;</li>

  <li>filters them, selecting the even ones; and</li>

  <li>accumulates the results using <tt>cons</tt>, starting with the empty list.</li>
</ul>

<a name="index_term_1726"></a><a name="index_term_1728"></a>A signal-processing engineer would find it natural to conceptualize these processes in terms of signals flowing through a cascade of stages, each of which implements part of the program plan, as shown in figure&nbsp;<a href="#%_fig_2.7">2.7</a>. In <tt>sum-odd-squares</tt>, we begin with an <a name="index_term_1730"></a>*enumerator*, which generates a "signal" consisting of the leaves of a given tree. This signal is passed through a <a name="index_term_1732"></a>*filter*, which eliminates all but the odd elements. The resulting signal is in turn passed through a <a name="index_term_1734"></a>*map*, which is a "transducer" that applies the <tt>square</tt> procedure to each element. The output of the map is then fed to an <a name="index_term_1736"></a>*accumulator*, which combines the elements using <tt>+</tt>, starting from an initial 0. The plan for <tt>even-fibs</tt> is analogous.


<a name="%_fig_2.7"></a>

<div align="left">
  <div align="left">
    **Figure 2.7:**&nbsp;&nbsp;The signal-flow plans for the procedures <tt>sum-odd-squares</tt> (top) and <tt>even-fibs</tt> (bottom) reveal the commonality between the two programs.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_17.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

Unfortunately, the two procedure definitions above fail to exhibit this signal-flow structure. For instance, if we examine the <tt>sum-odd-squares</tt> procedure, we find that the enumeration is implemented partly by the <tt>null?</tt> and <tt>pair?</tt> tests and partly by the tree-recursive structure of the procedure. Similarly, the accumulation is found partly in the tests and partly in the addition used in the recursion. In general, there are no distinct parts of either procedure that correspond to the elements in the signal-flow description. Our two procedures decompose the computations in a different way, spreading the enumeration over the program and mingling it with the map, the filter, and the accumulation. If we could organize our programs to make the signal-flow structure manifest in the procedures we write, this would increase the conceptual clarity of the resulting code.


<a name="%_sec_Temp_181"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_181">Sequence Operations</a></h4>

<a name="index_term_1738"></a> The key to organizing programs so as to more clearly reflect the signal-flow structure is to concentrate on the "signals" that flow from one stage in the process to the next. If we represent these signals as lists, then we can use list operations to implement the processing at each of the stages. For instance, we can implement the mapping stages of the signal-flow diagrams using the <tt>map</tt> procedure from section <a href="#%_sec_2.2.1">2.2.1</a>:


<tt>(map&nbsp;square&nbsp;(list&nbsp;1&nbsp;2&nbsp;3&nbsp;4&nbsp;5))<br>
<i>(1&nbsp;4&nbsp;9&nbsp;16&nbsp;25)</i><br></tt>


Filtering a sequence to select only those elements that satisfy a given predicate is accomplished by


<tt><a name="index_term_1740"></a>(define&nbsp;(filter&nbsp;predicate&nbsp;sequence)<br>
&nbsp;&nbsp;(cond&nbsp;((null?&nbsp;sequence)&nbsp;nil)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;((predicate&nbsp;(car&nbsp;sequence))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;(car&nbsp;sequence)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(filter&nbsp;predicate&nbsp;(cdr&nbsp;sequence))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(else&nbsp;(filter&nbsp;predicate&nbsp;(cdr&nbsp;sequence)))))<br></tt>


For example,


<tt>(filter&nbsp;odd?&nbsp;(list&nbsp;1&nbsp;2&nbsp;3&nbsp;4&nbsp;5))<br>
<i>(1&nbsp;3&nbsp;5)</i><br></tt>


Accumulations can be implemented by


<tt><a name="index_term_1742"></a>(define&nbsp;(accumulate&nbsp;op&nbsp;initial&nbsp;sequence)<br>
&nbsp;&nbsp;(if&nbsp;(null?&nbsp;sequence)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;initial<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(op&nbsp;(car&nbsp;sequence)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(accumulate&nbsp;op&nbsp;initial&nbsp;(cdr&nbsp;sequence)))))<br>
(accumulate&nbsp;+&nbsp;0&nbsp;(list&nbsp;1&nbsp;2&nbsp;3&nbsp;4&nbsp;5))<br>
<i>15</i><br>
(accumulate&nbsp;*&nbsp;1&nbsp;(list&nbsp;1&nbsp;2&nbsp;3&nbsp;4&nbsp;5))<br>
<i>120</i><br>
(accumulate&nbsp;cons&nbsp;nil&nbsp;(list&nbsp;1&nbsp;2&nbsp;3&nbsp;4&nbsp;5))<br>
<i>(1&nbsp;2&nbsp;3&nbsp;4&nbsp;5)</i><br></tt>


All that remains to implement signal-flow diagrams is to enumerate the sequence of elements to be processed. For <tt>even-fibs</tt>, we need to generate the sequence of integers in a given range, which we can do as follows:


<tt><a name="index_term_1744"></a>(define&nbsp;(enumerate-interval&nbsp;low&nbsp;high)<br>
&nbsp;&nbsp;(if&nbsp;(&gt;&nbsp;low&nbsp;high)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nil<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;low&nbsp;(enumerate-interval&nbsp;(+&nbsp;low&nbsp;1)&nbsp;high))))<br>
(enumerate-interval&nbsp;2&nbsp;7)<br>
<i>(2&nbsp;3&nbsp;4&nbsp;5&nbsp;6&nbsp;7)</i><br></tt>


To enumerate the leaves of a tree, we can use<a name="call_footnote_Temp_182" href="#footnote_Temp_182" id="call_footnote_Temp_182"><sup><small>14</small></sup></a>


<tt><a name="index_term_1748"></a><a name="index_term_1750"></a>(define&nbsp;(enumerate-tree&nbsp;tree)<br>
&nbsp;&nbsp;(cond&nbsp;((null?&nbsp;tree)&nbsp;nil)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;((not&nbsp;(pair?&nbsp;tree))&nbsp;(list&nbsp;tree))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(else&nbsp;(append&nbsp;(enumerate-tree&nbsp;(car&nbsp;tree))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(enumerate-tree&nbsp;(cdr&nbsp;tree))))))<br>
(enumerate-tree&nbsp;(list&nbsp;1&nbsp;(list&nbsp;2&nbsp;(list&nbsp;3&nbsp;4))&nbsp;5))<br>
<i>(1&nbsp;2&nbsp;3&nbsp;4&nbsp;5)</i><br></tt>


Now we can reformulate <tt>sum-odd-squares</tt> and <tt>even-fibs</tt> as in the signal-flow diagrams. For <tt>sum-odd-squares</tt>, we enumerate the sequence of leaves of the tree, filter this to keep only the odd numbers in the sequence, square each element, and sum the results:


<tt><a name="index_term_1752"></a>(define&nbsp;(sum-odd-squares&nbsp;tree)<br>
&nbsp;&nbsp;(accumulate&nbsp;+<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;0<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(map&nbsp;square<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(filter&nbsp;odd?<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(enumerate-tree&nbsp;tree)))))<br></tt>


For <tt>even-fibs</tt>, we enumerate the integers from 0 to *n*, generate the Fibonacci number for each of these integers, filter the resulting sequence to keep only the even elements, and accumulate the results into a list:


<tt><a name="index_term_1754"></a>(define&nbsp;(even-fibs&nbsp;n)<br>
&nbsp;&nbsp;(accumulate&nbsp;cons<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nil<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(filter&nbsp;even?<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(map&nbsp;fib<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(enumerate-interval&nbsp;0&nbsp;n)))))<br></tt>


The value of expressing programs as sequence operations is that this helps us make program designs that are modular, that is, designs that are constructed by combining relatively independent pieces. We can encourage modular design by providing a library of standard components together with a conventional interface for connecting the components in flexible ways.


<a name="index_term_1756"></a><a name="index_term_1758"></a>Modular construction is a powerful strategy for controlling complexity in engineering design. In real signal-processing applications, for example, designers regularly build systems by cascading elements selected from standardized families of filters and transducers. Similarly, sequence operations provide a library of standard program elements that we can mix and match. For instance, we can reuse pieces from the <tt>sum-odd-squares</tt> and <tt>even-fibs</tt> procedures in a program that constructs a list of the squares of the first *n* + 1 Fibonacci numbers:


<tt>(define&nbsp;(list-fib-squares&nbsp;n)<br>
&nbsp;&nbsp;(accumulate&nbsp;cons<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nil<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(map&nbsp;square<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(map&nbsp;fib<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(enumerate-interval&nbsp;0&nbsp;n)))))<br>
(list-fib-squares&nbsp;10)<br>
<i>(0&nbsp;1&nbsp;1&nbsp;4&nbsp;9&nbsp;25&nbsp;64&nbsp;169&nbsp;441&nbsp;1156&nbsp;3025)</i><br></tt>


We can rearrange the pieces and use them in computing the product of the odd integers in a sequence:


<tt>(define&nbsp;(product-of-squares-of-odd-elements&nbsp;sequence)<br>
&nbsp;&nbsp;(accumulate&nbsp;*<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(map&nbsp;square<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(filter&nbsp;odd?&nbsp;sequence))))<br>
(product-of-squares-of-odd-elements&nbsp;(list&nbsp;1&nbsp;2&nbsp;3&nbsp;4&nbsp;5))<br>
<i>225</i><br></tt>


We can also formulate conventional data-processing applications in terms of sequence operations. Suppose we have a sequence of personnel records and we want to find the salary of the highest-paid programmer. Assume that we have a selector <tt>salary</tt> that returns the salary of a record, and a predicate <tt>programmer?</tt> that tests if a record is for a programmer. Then we can write


<tt>(define&nbsp;(salary-of-highest-paid-programmer&nbsp;records)<br>
&nbsp;&nbsp;(accumulate&nbsp;max<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;0<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(map&nbsp;salary<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(filter&nbsp;programmer?&nbsp;records))))<br></tt>


These examples give just a hint of the vast range of operations that can be expressed as sequence operations.<a name="call_footnote_Temp_183" href="#footnote_Temp_183" id="call_footnote_Temp_183"><sup><small>15</small></sup></a>


Sequences, implemented here as lists, serve as a conventional interface that permits us to combine processing modules. Additionally, when we uniformly represent structures as sequences, we have localized the data-structure dependencies in our programs to a small number of sequence operations. By changing these, we can experiment with alternative representations of sequences, while leaving the overall design of our programs intact. We will exploit this capability in section <a href="chapter_3_section_5.html#%_sec_3.5">3.5</a>, when we generalize the sequence-processing paradigm to admit infinite sequences.


<a name="exercise_2_33">**Exercise 2.33**</a> Fill in the missing expressions to complete the following definitions of some basic list-manipulation operations as accumulations:


<tt><a name="index_term_1766"></a>(define&nbsp;(map&nbsp;p&nbsp;sequence)<br>
&nbsp;&nbsp;(accumulate&nbsp;(lambda&nbsp;(x&nbsp;y)&nbsp;&lt;*??*&gt;)&nbsp;nil&nbsp;sequence))<br>
<a name="index_term_1768"></a>(define&nbsp;(append&nbsp;seq1&nbsp;seq2)<br>
&nbsp;&nbsp;(accumulate&nbsp;cons&nbsp;&lt;*??*&gt;&nbsp;&lt;*??*&gt;))<br>
<a name="index_term_1770"></a>(define&nbsp;(length&nbsp;sequence)<br>
&nbsp;&nbsp;(accumulate&nbsp;&lt;*??*&gt;&nbsp;0&nbsp;sequence))<br></tt>


<a name="exercise_2_34">**Exercise 2.34**</a> <a name="index_term_1772"></a>Evaluating a polynomial in *x* at a given value of *x* can be formulated as an accumulation. We evaluate the polynomial

<div align="left">
  <img src="img/chapter_2_image_18.gif" border="0">
</div>

using a well-known algorithm called <a name="index_term_1774"></a>*Horner's rule*, which structures the computation as

<div align="left">
  <img src="img/chapter_2_image_19.gif" border="0">
</div>

In other words, we start with *a*<sub>*n*</sub>, multiply by *x*, add *a*<sub>*n*-1</sub>, multiply by *x*, and so on, until we reach *a*<sub>0</sub>.<a name="call_footnote_Temp_186" href="#footnote_Temp_186" id="call_footnote_Temp_186"><sup><small>16</small></sup></a> Fill in the following template to produce a procedure that evaluates a polynomial using Horner's rule. Assume that the coefficients of the polynomial are arranged in a sequence, from *a*<sub>0</sub> through *a*<sub>*n*</sub>.


<tt>(define&nbsp;(horner-eval&nbsp;x&nbsp;coefficient-sequence)<br>
&nbsp;&nbsp;(accumulate&nbsp;(lambda&nbsp;(this-coeff&nbsp;higher-terms)&nbsp;&lt;*??*&gt;)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;0<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;coefficient-sequence))<br></tt>


For example, to compute 1 + 3*x* + 5*x*<sup>3</sup> + *x*<sup>5</sup> at *x* = 2 you would evaluate


<tt>(horner-eval&nbsp;2&nbsp;(list&nbsp;1&nbsp;3&nbsp;0&nbsp;5&nbsp;0&nbsp;1))<br></tt>


<a name="exercise_2_35">**Exercise 2.35**</a> Redefine <tt>count-leaves</tt> from section <a href="#%_sec_2.2.2">2.2.2</a> as an accumulation:


<tt><a name="index_term_1792"></a>(define&nbsp;(count-leaves&nbsp;t)<br>
&nbsp;&nbsp;(accumulate&nbsp;&lt;*??*&gt;&nbsp;&lt;*??*&gt;&nbsp;(map&nbsp;&lt;*??*&gt;&nbsp;&lt;*??*&gt;)))<br></tt>


<a name="exercise_2_36">**Exercise 2.36**</a> The procedure <tt>accumulate-n</tt> is similar to <tt>accumulate</tt> except that it takes as its third argument a sequence of sequences, which are all assumed to have the same number of elements. It applies the designated accumulation procedure to combine all the first elements of the sequences, all the second elements of the sequences, and so on, and returns a sequence of the results. For instance, if <tt>s</tt> is a sequence containing four sequences, <tt>((1 2 3) (4 5 6) (7 8 9) (10 11 12)),</tt> then the value of <tt>(accumulate-n + 0 s)</tt> should be the sequence <tt>(22 26 30)</tt>. Fill in the missing expressions in the following definition of <tt>accumulate-n</tt>:


<tt><a name="index_term_1794"></a>(define&nbsp;(accumulate-n&nbsp;op&nbsp;init&nbsp;seqs)<br>
&nbsp;&nbsp;(if&nbsp;(null?&nbsp;(car&nbsp;seqs))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nil<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;(accumulate&nbsp;op&nbsp;init&nbsp;&lt;*??*&gt;)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(accumulate-n&nbsp;op&nbsp;init&nbsp;&lt;*??*&gt;))))<br></tt>


<a name="exercise_2_37">**Exercise 2.37**</a> <a name="index_term_1796"></a><a name="index_term_1798"></a><a name="index_term_1800"></a>Suppose we represent vectors *v* = (*v*<sub>*i*</sub>) as sequences of numbers, and matrices *m* = (*m*<sub>*i**j*</sub>) as sequences of vectors (the rows of the matrix). For example, the matrix

<div align="left">
  <img src="img/chapter_2_image_20.gif" border="0">
</div>

is represented as the sequence <tt>((1 2 3 4) (4 5 6 6) (6 7 8 9))</tt>. With this representation, we can use sequence operations to concisely express the basic matrix and vector operations. These operations (which are described in any book on matrix algebra) are the following:

<div align="left">
  <img src="img/chapter_2_image_21.gif" border="0">
</div>

We can define the dot product as<a name="call_footnote_Temp_190" href="#footnote_Temp_190" id="call_footnote_Temp_190"><sup><small>17</small></sup></a>


<tt><a name="index_term_1802"></a>(define&nbsp;(dot-product&nbsp;v&nbsp;w)<br>
&nbsp;&nbsp;(accumulate&nbsp;+&nbsp;0&nbsp;(map&nbsp;*&nbsp;v&nbsp;w)))<br></tt>


Fill in the missing expressions in the following procedures for computing the other matrix operations. (The procedure <tt>accumulate-n</tt> is defined in exercise&nbsp;<a href="#%_thm_2.36">2.36</a>.)


<tt><a name="index_term_1804"></a>(define&nbsp;(matrix-*-vector&nbsp;m&nbsp;v)<br>
&nbsp;&nbsp;(map&nbsp;&lt;*??*&gt;&nbsp;m))<br>
<a name="index_term_1806"></a>(define&nbsp;(transpose&nbsp;mat)<br>
&nbsp;&nbsp;(accumulate-n&nbsp;&lt;*??*&gt;&nbsp;&lt;*??*&gt;&nbsp;mat))<br>
<a name="index_term_1808"></a>(define&nbsp;(matrix-*-matrix&nbsp;m&nbsp;n)<br>
&nbsp;&nbsp;(let&nbsp;((cols&nbsp;(transpose&nbsp;n)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;(map&nbsp;&lt;*??*&gt;&nbsp;m)))<br></tt>


<a name="exercise_2_38">**Exercise 2.38**</a> <a name="index_term_1810"></a><a name="index_term_1812"></a>The <tt>accumulate</tt> procedure is also known as <tt>fold-right</tt>, because it combines the first element of the sequence with the result of combining all the elements to the right. There is also a <tt>fold-left</tt>, which is similar to <tt>fold-right</tt>, except that it combines elements working in the opposite direction:


<tt><a name="index_term_1814"></a>(define&nbsp;(fold-left&nbsp;op&nbsp;initial&nbsp;sequence)<br>
&nbsp;&nbsp;(define&nbsp;(iter&nbsp;result&nbsp;rest)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;(null?&nbsp;rest)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(iter&nbsp;(op&nbsp;result&nbsp;(car&nbsp;rest))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cdr&nbsp;rest))))<br>
&nbsp;&nbsp;(iter&nbsp;initial&nbsp;sequence))<br></tt>


What are the values of


<tt>(fold-right&nbsp;/&nbsp;1&nbsp;(list&nbsp;1&nbsp;2&nbsp;3))<br>
(fold-left&nbsp;/&nbsp;1&nbsp;(list&nbsp;1&nbsp;2&nbsp;3))<br>
(fold-right&nbsp;list&nbsp;nil&nbsp;(list&nbsp;1&nbsp;2&nbsp;3))<br>
(fold-left&nbsp;list&nbsp;nil&nbsp;(list&nbsp;1&nbsp;2&nbsp;3))<br></tt>


Give a property that <tt>op</tt> should satisfy to guarantee that <tt>fold-right</tt> and <tt>fold-left</tt> will produce the same values for any sequence.


<a name="exercise_2_39">**Exercise 2.39**</a>  Complete the following definitions of <tt>reverse</tt> <a name="index_term_1816"></a>(exercise&nbsp;<a href="#%_thm_2.18">2.18</a>) in terms of <tt>fold-right</tt> and <tt>fold-left</tt> from exercise&nbsp;<a href="#%_thm_2.38">2.38</a>:


<tt>(define&nbsp;(reverse&nbsp;sequence)<br>
&nbsp;&nbsp;(fold-right&nbsp;(lambda&nbsp;(x&nbsp;y)&nbsp;&lt;*??*&gt;)&nbsp;nil&nbsp;sequence))<br>
(define&nbsp;(reverse&nbsp;sequence)<br>
&nbsp;&nbsp;(fold-left&nbsp;(lambda&nbsp;(x&nbsp;y)&nbsp;&lt;*??*&gt;)&nbsp;nil&nbsp;sequence))<br></tt>


<a name="%_sec_Temp_193"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_193">Nested Mappings</a></h4>

<a name="index_term_1818"></a> We can extend the sequence paradigm to include many computations that are commonly expressed using nested loops.<a name="call_footnote_Temp_194" href="#footnote_Temp_194" id="call_footnote_Temp_194"><sup><small>18</small></sup></a> Consider this problem: Given a positive integer *n*, find all ordered pairs of distinct positive integers *i* and *j*, where 1<u>&lt;</u> *j*&lt; *i*<u>&lt;</u> *n*, such that *i* + *j* is prime. For example, if *n* is 6, then the pairs are the following:

<div align="left">
  <img src="img/chapter_2_image_22.gif" border="0">
</div>

A natural way to organize this computation is to generate the sequence of all ordered pairs of positive integers less than or equal to *n*, filter to select those pairs whose sum is prime, and then, for each pair (*i*, *j*) that passes through the filter, produce the triple (*i*,*j*,*i* + *j*).


Here is a way to generate the sequence of pairs: For each integer *i*<u>&lt;</u> *n*, enumerate the integers *j*&lt;*i*, and for each such *i* and *j* generate the pair (*i*,*j*). In terms of sequence operations, we map along the sequence <tt>(enumerate-interval 1 n)</tt>. For each *i* in this sequence, we map along the sequence <tt>(enumerate-interval 1 (- i 1))</tt>. For each *j* in this latter sequence, we generate the pair <tt>(list i j)</tt>. This gives us a sequence of pairs for each *i*. Combining all the sequences for all the *i* (by accumulating with <tt>append</tt>) produces the required sequence of pairs:<a name="call_footnote_Temp_195" href="#footnote_Temp_195" id="call_footnote_Temp_195"><sup><small>19</small></sup></a>


<tt>(accumulate&nbsp;append<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nil<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(map&nbsp;(lambda&nbsp;(i)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(map&nbsp;(lambda&nbsp;(j)&nbsp;(list&nbsp;i&nbsp;j))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(enumerate-interval&nbsp;1&nbsp;(-&nbsp;i&nbsp;1))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(enumerate-interval&nbsp;1&nbsp;n)))<br></tt>


The combination of mapping and accumulating with <tt>append</tt> is so common in this sort of program that we will isolate it as a separate procedure:


<tt><a name="index_term_1826"></a>(define&nbsp;(flatmap&nbsp;proc&nbsp;seq)<br>
&nbsp;&nbsp;(accumulate&nbsp;append&nbsp;nil&nbsp;(map&nbsp;proc&nbsp;seq)))<br></tt>


Now filter this sequence of pairs to find those whose sum is prime. The filter predicate is called for each element of the sequence; its argument is a pair and it must extract the integers from the pair. Thus, the predicate to apply to each element in the sequence is


<tt>(define&nbsp;(prime-sum?&nbsp;pair)<br>
&nbsp;&nbsp;(prime?&nbsp;(+&nbsp;(car&nbsp;pair)&nbsp;(cadr&nbsp;pair))))<br></tt>


Finally, generate the sequence of results by mapping over the filtered pairs using the following procedure, which constructs a triple consisting of the two elements of the pair along with their sum:


<tt>(define&nbsp;(make-pair-sum&nbsp;pair)<br>
&nbsp;&nbsp;(list&nbsp;(car&nbsp;pair)&nbsp;(cadr&nbsp;pair)&nbsp;(+&nbsp;(car&nbsp;pair)&nbsp;(cadr&nbsp;pair))))<br></tt>


Combining all these steps yields the complete procedure:


<tt><a name="index_term_1828"></a>(define&nbsp;(prime-sum-pairs&nbsp;n)<br>
&nbsp;&nbsp;(map&nbsp;make-pair-sum<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(filter&nbsp;prime-sum?<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(flatmap<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(i)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(map&nbsp;(lambda&nbsp;(j)&nbsp;(list&nbsp;i&nbsp;j))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(enumerate-interval&nbsp;1&nbsp;(-&nbsp;i&nbsp;1))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(enumerate-interval&nbsp;1&nbsp;n)))))<br></tt>


Nested mappings are also useful for sequences other than those that enumerate intervals. Suppose we wish to generate all the <a name="index_term_1830"></a><a name="index_term_1832"></a>permutations of a set *S*; that is, all the ways of ordering the items in the set. For instance, the permutations of {1,2,3} are {1,2,3}, { 1,3,2}, {2,1,3}, { 2,3,1}, { 3,1,2}, and { 3,2,1}. Here is a plan for generating the permutations of&nbsp;*S*: For each item *x* in *S*, recursively generate the sequence of permutations of *S* - *x*,<a name="call_footnote_Temp_196" href="#footnote_Temp_196" id="call_footnote_Temp_196"><sup><small>20</small></sup></a> and adjoin *x* to the front of each one. This yields, for each *x* in *S*, the sequence of permutations of *S* that begin with&nbsp;*x*. Combining these sequences for all *x* gives all the permutations of&nbsp;*S*:<a name="call_footnote_Temp_197" href="#footnote_Temp_197" id="call_footnote_Temp_197"><sup><small>21</small></sup></a>


<tt><a name="index_term_1840"></a>(define&nbsp;(permutations&nbsp;s)<br>
&nbsp;&nbsp;(if&nbsp;(null?&nbsp;s)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;empty&nbsp;set?*<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;nil)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;sequence&nbsp;containing&nbsp;empty&nbsp;set*<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(flatmap&nbsp;(lambda&nbsp;(x)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(map&nbsp;(lambda&nbsp;(p)&nbsp;(cons&nbsp;x&nbsp;p))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(permutations&nbsp;(remove&nbsp;x&nbsp;s))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;s)))<br></tt>


Notice how this strategy reduces the problem of generating permutations of *S* to the problem of generating the permutations of sets with fewer elements than *S*. In the terminal case, we work our way down to the empty list, which represents a set of no elements. For this, we generate <tt>(list nil)</tt>, which is a sequence with one item, namely the set with no elements. The <tt>remove</tt> procedure used in <tt>permutations</tt> returns all the items in a given sequence except for a given item. This can be expressed as a simple filter:


<tt><a name="index_term_1842"></a>(define&nbsp;(remove&nbsp;item&nbsp;sequence)<br>
&nbsp;&nbsp;(filter&nbsp;(lambda&nbsp;(x)&nbsp;(not&nbsp;(=&nbsp;x&nbsp;item)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sequence))<br></tt>


<a name="exercise_2_40">**Exercise 2.40**</a> Define a procedure <a name="index_term_1844"></a><tt>unique-pairs</tt> that, given an integer *n*, generates the sequence of pairs (*i*,*j*) with 1<u>&lt;</u> *j*&lt; *i*<u>&lt;</u> *n*. Use <tt>unique-pairs</tt> to simplify the definition of <tt>prime-sum-pairs</tt> given above.


<a name="exercise_2_41">**Exercise 2.41**</a> Write a procedure to find all ordered triples of distinct positive integers *i*, *j*, and&nbsp;*k* less than or equal to a given integer *n* that sum to a given integer *s*.


<a name="exercise_2_42">**Exercise 2.42**</a> <a name="%_fig_2.8"></a>

<div align="left">
  <div align="left">
    **Figure 2.8:**&nbsp;&nbsp;A solution to the eight-queens puzzle.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_23.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

The <a name="index_term_1846"></a><a name="index_term_1848"></a><a name="index_term_1850"></a>"eight-queens puzzle" asks how to place eight queens on a chessboard so that no queen is in check from any other (i.e., no two queens are in the same row, column, or diagonal). One possible solution is shown in figure&nbsp;<a href="#%_fig_2.8">2.8</a>. One way to solve the puzzle is to work across the board, placing a queen in each column. Once we have placed *k* - 1 queens, we must place the *k*th queen in a position where it does not check any of the queens already on the board. We can formulate this approach recursively: Assume that we have already generated the sequence of all possible ways to place *k* - 1 queens in the first *k* - 1 columns of the board. For each of these ways, generate an extended set of positions by placing a queen in each row of the *k*th column. Now filter these, keeping only the positions for which the queen in the *k*th column is safe with respect to the other queens. This produces the sequence of all ways to place *k* queens in the first *k* columns. By continuing this process, we will produce not only one solution, but all solutions to the puzzle.


We implement this solution as a procedure <tt>queens</tt>, which returns a sequence of all solutions to the problem of placing *n* queens on an *n*� *n* chessboard. <tt>Queens</tt> has an internal procedure <tt>queen-cols</tt> that returns the sequence of all ways to place queens in the first *k* columns of the board.


<tt><a name="index_term_1852"></a>(define&nbsp;(queens&nbsp;board-size)<br>
&nbsp;&nbsp;(define&nbsp;(queen-cols&nbsp;k)&nbsp;&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;(=&nbsp;k&nbsp;0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;empty-board)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(filter<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(positions)&nbsp;(safe?&nbsp;k&nbsp;positions))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(flatmap<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(rest-of-queens)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(map&nbsp;(lambda&nbsp;(new-row)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(adjoin-position&nbsp;new-row&nbsp;k&nbsp;rest-of-queens))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(enumerate-interval&nbsp;1&nbsp;board-size)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(queen-cols&nbsp;(-&nbsp;k&nbsp;1))))))<br>
&nbsp;&nbsp;(queen-cols&nbsp;board-size))<br></tt>


In this procedure <tt>rest-of-queens</tt> is a way to place *k* - 1 queens in the first *k* - 1 columns, and <tt>new-row</tt> is a proposed row in which to place the queen for the *k*th column. Complete the program by implementing the representation for sets of board positions, including the procedure <tt>adjoin-position</tt>, which adjoins a new row-column position to a set of positions, and <tt>empty-board</tt>, which represents an empty set of positions. You must also write the procedure <tt>safe?</tt>, which determines for a set of positions, whether the queen in the *k*th column is safe with respect to the others. (Note that we need only check whether the new queen is safe -- the other queens are already guaranteed safe with respect to each other.)


<a name="exercise_2_43">**Exercise 2.43**</a> Louis Reasoner is having a terrible time doing exercise&nbsp;<a href="#%_thm_2.42">2.42</a>. His <tt>queens</tt> procedure seems to work, but it runs extremely slowly. (Louis never does manage to wait long enough for it to solve even the 6� 6 case.) When Louis asks Eva Lu Ator for help, she points out that he has interchanged the order of the nested mappings in the <tt>flatmap</tt>, writing it as


<tt>(flatmap<br>
&nbsp;(lambda&nbsp;(new-row)<br>
&nbsp;&nbsp;&nbsp;(map&nbsp;(lambda&nbsp;(rest-of-queens)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(adjoin-position&nbsp;new-row&nbsp;k&nbsp;rest-of-queens))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(queen-cols&nbsp;(-&nbsp;k&nbsp;1))))<br>
&nbsp;(enumerate-interval&nbsp;1&nbsp;board-size))<br></tt>


Explain why this interchange makes the program run slowly. Estimate how long it will take Louis's program to solve the eight-queens puzzle, assuming that the program in exercise&nbsp;<a href="#%_thm_2.42">2.42</a> solves the puzzle in time *T*.


<a name="%_sec_2.2.4"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_2.2.4">2.2.4&nbsp;&nbsp;Example: A Picture Language</a></h3>

<a name="index_term_1854"></a> This section presents a simple language for drawing pictures that illustrates the power of data abstraction and closure, and also exploits higher-order procedures in an essential way. The language is designed to make it easy to experiment with patterns such as the ones in figure&nbsp;<a href="#%_fig_2.9">2.9</a>, which are composed of repeated elements that are shifted and scaled.<a name="call_footnote_Temp_202" href="#footnote_Temp_202" id="call_footnote_Temp_202"><sup><small>22</small></sup></a> In this language, the data objects being combined are represented as procedures rather than as list structure. Just as <tt>cons</tt>, which satisfies the <a name="index_term_1860"></a>closure property, allowed us to easily build arbitrarily complicated list structure, the operations in this language, which also satisfy the closure property, allow us to easily build arbitrarily complicated patterns.


<a name="%_fig_2.9"></a>

<div align="left">
  <div align="left">
    **Figure 2.9:**&nbsp;&nbsp;Designs generated with the picture language.
  </div>

  <table width="100%">
    <tr>
      <td>
        <div align="left">
          <img src="img/chapter_2_image_24.gif" border="0"> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="img/chapter_2_image_25.gif" border="0">&nbsp;
        </div>
      </td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="%_sec_Temp_203"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_203">The picture language</a></h4>

When we began our study of programming in section <a href="chapter_1_section_1.html#%_sec_1.1">1.1</a>, we emphasized the importance of describing a language by focusing on the language's primitives, its means of combination, and its means of abstraction. We'll follow that framework here.


Part of the elegance of this picture language is that there is only one kind of element, called a <a name="index_term_1862"></a>*painter*. A painter draws an image that is shifted and scaled to fit within a designated <a name="index_term_1864"></a>parallelogram-shaped frame. For example, there's a primitive painter we'll call <tt>wave</tt> that makes a crude line drawing, as shown in figure&nbsp;<a href="#%_fig_2.10">2.10</a>. The actual shape of the drawing depends on the frame -- all four images in figure&nbsp;<a href="#%_fig_2.10">2.10</a> are produced by the same <tt>wave</tt> painter, but with respect to four different frames. Painters can be more elaborate than this: The primitive painter called <tt>rogers</tt> paints a picture of MIT's founder, William Barton Rogers, as shown in figure&nbsp;<a href="#%_fig_2.11">2.11</a>.<a name="call_footnote_Temp_204" href="#footnote_Temp_204" id="call_footnote_Temp_204"><sup><small>23</small></sup></a> The four images in figure&nbsp;<a href="#%_fig_2.11">2.11</a> are drawn with respect to the same four frames as the <tt>wave</tt> images in figure&nbsp;<a href="#%_fig_2.10">2.10</a>.


<a name="index_term_1876"></a>To combine images, we use various operations that construct new painters from given painters. For example, the <a name="index_term_1878"></a><tt>beside</tt> operation takes two painters and produces a new, compound painter that draws the first painter's image in the left half of the frame and the second painter's image in the right half of the frame. Similarly, <a name="index_term_1880"></a><tt>below</tt> takes two painters and produces a compound painter that draws the first painter's image below the second painter's image. Some operations transform a single painter to produce a new painter. For example, <a name="index_term_1882"></a><tt>flip-vert</tt> takes a painter and produces a painter that draws its image upside-down, and <a name="index_term_1884"></a><tt>flip-horiz</tt> produces a painter that draws the original painter's image left-to-right reversed.


<a name="%_fig_2.10"></a>

<div align="left">
  <div align="left">
    **Figure 2.10:**&nbsp;&nbsp;Images produced by the <tt>wave</tt> painter, with respect to four different frames. The frames, shown with dotted lines, are not part of the images.
  </div>

  <table width="100%">
    <tr>
      <td>
        <div align="left">
          <img src="img/chapter_2_image_26.gif" border="0"> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="img/chapter_2_image_27.gif" border="0">&nbsp;
        </div>

        <div align="left">
          <img src="img/chapter_2_image_28.gif" border="0"> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="img/chapter_2_image_29.gif" border="0">&nbsp;
        </div>
      </td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="%_fig_2.11"></a>

<div align="left">
  <div align="left">
    **Figure 2.11:**&nbsp;&nbsp;Images of William Barton Rogers, founder and first president of MIT, painted with respect to the same four frames as in figure&nbsp;<a href="#%_fig_2.10">2.10</a> (original image reprinted with the permission of the MIT Museum).
  </div>

  <table width="100%">
    <tr>
      <td>
        <div align="left">
          <img src="img/chapter_2_image_30.gif" border="0"> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="img/chapter_2_image_31.gif" border="0">&nbsp;
        </div>

        <div align="left">
          <img src="img/chapter_2_image_32.gif" border="0"> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="img/chapter_2_image_33.gif" border="0">&nbsp;
        </div>
      </td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

Figure&nbsp;<a href="#%_fig_2.12">2.12</a> shows the drawing of a painter called <tt>wave4</tt> that is built up in two stages starting from <tt>wave</tt>:


<tt>(define&nbsp;wave2&nbsp;(beside&nbsp;wave&nbsp;(flip-vert&nbsp;wave)))<br>
(define&nbsp;wave4&nbsp;(below&nbsp;wave2&nbsp;wave2))<br></tt>


<a name="%_fig_2.12"></a>

<div align="left">
  <div align="left">
    **Figure 2.12:**&nbsp;&nbsp;Creating a complex figure, starting from the <tt>wave</tt> painter of figure&nbsp;<a href="#%_fig_2.10">2.10</a>.
  </div>

  <table width="100%">
    <tr>
      <td>
        <div align="left">
          <img src="img/chapter_2_image_34.gif" border="0"> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="img/chapter_2_image_35.gif" border="0">&nbsp;
        </div>

        
<tt>(define&nbsp;wave2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(define&nbsp;wave4<br>
        &nbsp;&nbsp;(beside&nbsp;wave&nbsp;(flip-vert&nbsp;wave)))&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(below&nbsp;wave2&nbsp;wave2))<br></tt>

      </td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="index_term_1886"></a>In building up a complex image in this manner we are exploiting the fact that painters are closed under the language's means of combination. The <tt>beside</tt> or <tt>below</tt> of two painters is itself a painter; therefore, we can use it as an element in making more complex painters. As with building up list structure using <tt>cons</tt>, the closure of our data under the means of combination is crucial to the ability to create complex structures while using only a few operations.


Once we can combine painters, we would like to be able to abstract typical patterns of combining painters. We will implement the painter operations as Scheme procedures. This means that we don't need a special abstraction mechanism in the picture language: Since the means of combination are ordinary Scheme procedures, we automatically have the capability to do anything with painter operations that we can do with procedures. For example, we can abstract the pattern in <tt>wave4</tt> as


<tt><a name="index_term_1888"></a>(define&nbsp;(flipped-pairs&nbsp;painter)<br>
&nbsp;&nbsp;(let&nbsp;((painter2&nbsp;(beside&nbsp;painter&nbsp;(flip-vert&nbsp;painter))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;(below&nbsp;painter2&nbsp;painter2)))<br></tt>


and define <tt>wave4</tt> as an instance of this pattern:


<tt>(define&nbsp;wave4&nbsp;(flipped-pairs&nbsp;wave))<br></tt>


We can also define recursive operations. Here's one that makes painters split and branch towards the right as shown in figures&nbsp;<a href="#%_fig_2.13">2.13</a> and &nbsp;<a href="#%_fig_2.14">2.14</a>:


<tt><a name="index_term_1890"></a>(define&nbsp;(right-split&nbsp;painter&nbsp;n)<br>
&nbsp;&nbsp;(if&nbsp;(=&nbsp;n&nbsp;0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;painter<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((smaller&nbsp;(right-split&nbsp;painter&nbsp;(-&nbsp;n&nbsp;1))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(beside&nbsp;painter&nbsp;(below&nbsp;smaller&nbsp;smaller)))))<br></tt>


<a name="%_fig_2.13"></a>

<div align="left">
  <div align="left">
    **Figure 2.13:**&nbsp;&nbsp;Recursive plans for <tt>right-split</tt> and <tt>corner-split</tt>.
  </div>

  <table width="100%">
    <tr>
      <td>
        <div align="left">
          <img src="img/chapter_2_image_36.gif" border="0"> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="img/chapter_2_image_37.gif" border="0">&nbsp;
        </div>

        
<tt>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;right-split&nbsp;*n*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;corner-split&nbsp;*n*<br></tt>

      </td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

We can produce balanced patterns by branching upwards as well as towards the right (see exercise&nbsp;<a href="#%_thm_2.44">2.44</a> and figures&nbsp;<a href="#%_fig_2.13">2.13</a> and &nbsp;<a href="#%_fig_2.14">2.14</a>):


<tt><a name="index_term_1892"></a>(define&nbsp;(corner-split&nbsp;painter&nbsp;n)<br>
&nbsp;&nbsp;(if&nbsp;(=&nbsp;n&nbsp;0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;painter<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((up&nbsp;(up-split&nbsp;painter&nbsp;(-&nbsp;n&nbsp;1)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(right&nbsp;(right-split&nbsp;painter&nbsp;(-&nbsp;n&nbsp;1))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((top-left&nbsp;(beside&nbsp;up&nbsp;up))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(bottom-right&nbsp;(below&nbsp;right&nbsp;right))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(corner&nbsp;(corner-split&nbsp;painter&nbsp;(-&nbsp;n&nbsp;1))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(beside&nbsp;(below&nbsp;painter&nbsp;top-left)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(below&nbsp;bottom-right&nbsp;corner))))))<br></tt>


<a name="%_fig_2.14"></a>

<div align="left">
  <div align="left">
    **Figure 2.14:**&nbsp;&nbsp;The recursive operations <tt>right-split</tt> and <tt>corner-split</tt> applied to the painters <tt>wave</tt> and <tt>rogers</tt>. Combining four <tt>corner-split</tt> figures produces symmetric <tt>square-limit</tt> designs as shown in figure&nbsp;<a href="#%_fig_2.9">2.9</a>.
  </div>

  <table width="100%">
    <tr>
      <td>
        <div align="left">
          <img src="img/chapter_2_image_38.gif" border="0"> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="img/chapter_2_image_39.gif" border="0">&nbsp;
        </div>

        
<tt>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(right-split&nbsp;wave&nbsp;4)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(right-split&nbsp;rogers&nbsp;4)<br></tt>

        <div align="left">
          <img src="img/chapter_2_image_40.gif" border="0"> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="img/chapter_2_image_41.gif" border="0">&nbsp;
        </div>

        
<tt>&nbsp;&nbsp;&nbsp;&nbsp;(corner-split&nbsp;wave&nbsp;4)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(corner-split&nbsp;rogers&nbsp;4)<br></tt>

      </td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

By placing four copies of a <tt>corner-split</tt> appropriately, we obtain a pattern called <tt>square-limit</tt>, whose application to <tt>wave</tt> and <tt>rogers</tt> is shown in figure&nbsp;<a href="#%_fig_2.9">2.9</a>:


<tt><a name="index_term_1894"></a>(define&nbsp;(square-limit&nbsp;painter&nbsp;n)<br>
&nbsp;&nbsp;(let&nbsp;((quarter&nbsp;(corner-split&nbsp;painter&nbsp;n)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((half&nbsp;(beside&nbsp;(flip-horiz&nbsp;quarter)&nbsp;quarter)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(below&nbsp;(flip-vert&nbsp;half)&nbsp;half))))<br></tt>


<a name="exercise_2_44">**Exercise 2.44**</a> Define the procedure <a name="index_term_1896"></a><tt>up-split</tt> used by <tt>corner-split</tt>. It is similar to <tt>right-split</tt>, except that it switches the roles of <tt>below</tt> and <tt>beside</tt>.


<a name="%_sec_Temp_206"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_206">Higher-order operations</a></h4>

<a name="index_term_1898"></a> In addition to abstracting patterns of combining painters, we can work at a higher level, abstracting patterns of combining painter operations. That is, we can view the painter operations as elements to manipulate and can write means of combination for these elements -- procedures that take painter operations as arguments and create new painter operations.


For example, <tt>flipped-pairs</tt> and <tt>square-limit</tt> each arrange four copies of a painter's image in a square pattern; they differ only in how they orient the copies. One way to abstract this pattern of painter combination is with the following procedure, which takes four one-argument painter operations and produces a painter operation that transforms a given painter with those four operations and arranges the results in a square. <tt>Tl</tt>, <tt>tr</tt>, <tt>bl</tt>, and <tt>br</tt> are the transformations to apply to the top left copy, the top right copy, the bottom left copy, and the bottom right copy, respectively.


<tt><a name="index_term_1900"></a>(define&nbsp;(square-of-four&nbsp;tl&nbsp;tr&nbsp;bl&nbsp;br)<br>
&nbsp;&nbsp;(lambda&nbsp;(painter)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((top&nbsp;(beside&nbsp;(tl&nbsp;painter)&nbsp;(tr&nbsp;painter)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(bottom&nbsp;(beside&nbsp;(bl&nbsp;painter)&nbsp;(br&nbsp;painter))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(below&nbsp;bottom&nbsp;top))))<br></tt>


Then <tt>flipped-pairs</tt> can be defined in terms of <tt>square-of-four</tt> as follows:<a name="call_footnote_Temp_207" href="#footnote_Temp_207" id="call_footnote_Temp_207"><sup><small>24</small></sup></a>


<tt><a name="index_term_1904"></a>(define&nbsp;(flipped-pairs&nbsp;painter)<br>
&nbsp;&nbsp;(let&nbsp;((combine4&nbsp;(square-of-four&nbsp;identity&nbsp;flip-vert<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;identity&nbsp;flip-vert)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;(combine4&nbsp;painter)))<br></tt>


and <tt>square-limit</tt> can be expressed as<a name="call_footnote_Temp_208" href="#footnote_Temp_208" id="call_footnote_Temp_208"><sup><small>25</small></sup></a>


<tt><a name="index_term_1906"></a>(define&nbsp;(square-limit&nbsp;painter&nbsp;n)<br>
&nbsp;&nbsp;(let&nbsp;((combine4&nbsp;(square-of-four&nbsp;flip-horiz&nbsp;identity<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rotate180&nbsp;flip-vert)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;(combine4&nbsp;(corner-split&nbsp;painter&nbsp;n))))<br></tt>


<a name="exercise_2_45">**Exercise 2.45**</a> <tt>Right-split</tt> and <tt>up-split</tt> can be expressed as instances of a general splitting operation. Define a procedure <a name="index_term_1908"></a><tt>split</tt> with the property that evaluating


<tt>(define&nbsp;right-split&nbsp;(split&nbsp;beside&nbsp;below))<br>
(define&nbsp;up-split&nbsp;(split&nbsp;below&nbsp;beside))<br></tt>


produces procedures <tt>right-split</tt> and <tt>up-split</tt> with the same behaviors as the ones already defined.


<a name="%_sec_Temp_210"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_210">Frames</a></h4>

<a name="index_term_1910"></a> Before we can show how to implement painters and their means of combination, we must first consider <a name="index_term_1912"></a>frames. A frame can be described by three vectors -- an origin vector and two edge vectors. The origin vector specifies the offset of the frame's origin from some absolute origin in the plane, and the edge vectors specify the offsets of the frame's corners from its origin. If the edges are perpendicular, the frame will be rectangular. Otherwise the frame will be a more general parallelogram.


Figure&nbsp;<a href="#%_fig_2.15">2.15</a> shows a frame and its associated vectors. In accordance with data abstraction, we need not be specific yet about how frames are represented, other than to say that there is a constructor <a name="index_term_1914"></a><tt>make-frame</tt>, which takes three vectors and produces a frame, and three corresponding selectors <a name="index_term_1916"></a><tt>origin-frame</tt>, <a name="index_term_1918"></a><tt>edge1-frame</tt>, and <a name="index_term_1920"></a><tt>edge2-frame</tt> (see exercise&nbsp;<a href="#%_thm_2.47">2.47</a>).


<a name="%_fig_2.15"></a>

<div align="left">
  <div align="left">
    **Figure 2.15:**&nbsp;&nbsp;A frame is described by three vectors -- an origin and two edges.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_42.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="index_term_1922"></a>We will use coordinates in the unit square (0<u>&lt;</u> *x*,*y*<u>&lt;</u> 1) to specify images. With each frame, we associate a <a name="index_term_1924"></a>*frame coordinate map*, which will be used to shift and scale images to fit the frame. The map transforms the unit square into the frame by mapping the vector <strong>*v*</strong> = (*x*,*y*) to the vector sum

<div align="left">
  <img src="img/chapter_2_image_43.gif" border="0">
</div>

For example, (0,0) is mapped to the origin of the frame, (1,1) to the vertex diagonally opposite the origin, and (0.5,0.5) to the center of the frame. We can create a frame's coordinate map with the following procedure:<a name="call_footnote_Temp_211" href="#footnote_Temp_211" id="call_footnote_Temp_211"><sup><small>26</small></sup></a>


<tt><a name="index_term_1926"></a>(define&nbsp;(frame-coord-map&nbsp;frame)<br>
&nbsp;&nbsp;(lambda&nbsp;(v)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(add-vect<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(origin-frame&nbsp;frame)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(add-vect&nbsp;(scale-vect&nbsp;(xcor-vect&nbsp;v)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(edge1-frame&nbsp;frame))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(scale-vect&nbsp;(ycor-vect&nbsp;v)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(edge2-frame&nbsp;frame))))))<br></tt>


Observe that applying <tt>frame-coord-map</tt> to a frame returns a procedure that, given a vector, returns a vector. If the argument vector is in the unit square, the result vector will be in the frame. For example,


<tt>((frame-coord-map&nbsp;a-frame)&nbsp;(make-vect&nbsp;0&nbsp;0))<br></tt>


returns the same vector as


<tt>(origin-frame&nbsp;a-frame)<br></tt>


<a name="exercise_2_46">**Exercise 2.46**</a> <a name="index_term_1928"></a><a name="index_term_1930"></a>A two-dimensional vector <strong>v</strong> running from the origin to a point can be represented as a pair consisting of an *x*-coordinate and a *y*-coordinate. Implement a data abstraction for vectors by giving a constructor <a name="index_term_1932"></a><tt>make-vect</tt> and corresponding selectors <a name="index_term_1934"></a><tt>xcor-vect</tt> and <a name="index_term_1936"></a><tt>ycor-vect</tt>. In terms of your selectors and constructor, implement procedures <a name="index_term_1938"></a><tt>add-vect</tt>, <a name="index_term_1940"></a><tt>sub-vect</tt>, and <a name="index_term_1942"></a><tt>scale-vect</tt> that perform the operations vector addition, vector subtraction, and multiplying a vector by a scalar:

<div align="left">
  <img src="img/chapter_2_image_44.gif" border="0">
</div>

<a name="exercise_2_47">**Exercise 2.47**</a> Here are two possible constructors for frames:


<tt><a name="index_term_1944"></a>(define&nbsp;(make-frame&nbsp;origin&nbsp;edge1&nbsp;edge2)<br>
&nbsp;&nbsp;(list&nbsp;origin&nbsp;edge1&nbsp;edge2))<br>
<br>
(define&nbsp;(make-frame&nbsp;origin&nbsp;edge1&nbsp;edge2)<br>
&nbsp;&nbsp;(cons&nbsp;origin&nbsp;(cons&nbsp;edge1&nbsp;edge2)))<br></tt>


For each constructor supply the appropriate selectors to produce an implementation for frames.


<a name="%_sec_Temp_214"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_214">Painters</a></h4>

<a name="index_term_1946"></a> A painter is represented as a procedure that, given a frame as argument, draws a particular image shifted and scaled to fit the frame. That is to say, if <tt>p</tt> is a painter and <tt>f</tt> is a frame, then we produce <tt>p</tt>'s image in <tt>f</tt> by calling <tt>p</tt> with <tt>f</tt> as argument.


The details of how primitive painters are implemented depend on the particular characteristics of the graphics system and the type of image to be drawn. For instance, suppose we have a procedure <a name="index_term_1948"></a><tt>draw-line</tt> that draws a line on the screen between two specified points. Then we can create painters for line drawings, such as the <tt>wave</tt> painter in figure&nbsp;<a href="#%_fig_2.10">2.10</a>, from lists of line segments as follows:<a name="call_footnote_Temp_215" href="#footnote_Temp_215" id="call_footnote_Temp_215"><sup><small>27</small></sup></a>


<tt><a name="index_term_1950"></a>(define&nbsp;(segments-&gt;painter&nbsp;segment-list)<br>
&nbsp;&nbsp;(lambda&nbsp;(frame)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(for-each<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(segment)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(draw-line<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;((frame-coord-map&nbsp;frame)&nbsp;(start-segment&nbsp;segment))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;((frame-coord-map&nbsp;frame)&nbsp;(end-segment&nbsp;segment))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;segment-list)))<br></tt>


The segments are given using coordinates with respect to the unit square. For each segment in the list, the painter transforms the segment endpoints with the frame coordinate map and draws a line between the transformed points.


Representing painters as procedures erects a powerful abstraction barrier in the picture language. We can create and intermix all sorts of primitive painters, based on a variety of graphics capabilities. The details of their implementation do not matter. Any procedure can serve as a painter, provided that it takes a frame as argument and draws something scaled to fit the frame.<a name="call_footnote_Temp_216" href="#footnote_Temp_216" id="call_footnote_Temp_216"><sup><small>28</small></sup></a>


<a name="exercise_2_48">**Exercise 2.48**</a> <a name="index_term_1952"></a>A directed line segment in the plane can be represented as a pair of vectors -- the vector running from the origin to the start-point of the segment, and the vector running from the origin to the end-point of the segment. Use your vector representation from exercise&nbsp;<a href="#%_thm_2.46">2.46</a> to define a representation for segments with a constructor <a name="index_term_1954"></a><tt>make-segment</tt> and selectors <a name="index_term_1956"></a><tt>start-segment</tt> and <a name="index_term_1958"></a><tt>end-segment</tt>.


<a name="exercise_2_49">**Exercise 2.49**</a> Use <tt>segments-&gt;painter</tt> to define the following primitive painters:


a.&nbsp;&nbsp;The painter that draws the outline of the designated frame.


b.&nbsp;&nbsp;The painter that draws an "X" by connecting opposite corners of the frame.


c.&nbsp;&nbsp;The painter that draws a diamond shape by connecting the midpoints of the sides of the frame.


d.&nbsp;&nbsp;The <tt>wave</tt> painter.


<a name="%_sec_Temp_219"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_219">Transforming and combining painters</a></h4>

<a name="index_term_1960"></a> An operation on painters (such as <tt>flip-vert</tt> or <tt>beside</tt>) works by creating a painter that invokes the original painters with respect to frames derived from the argument frame. Thus, for example, <tt>flip-vert</tt> doesn't have to know how a painter works in order to flip it -- it just has to know how to turn a frame upside down: The flipped painter just uses the original painter, but in the inverted frame.


Painter operations are based on the procedure <tt>transform-painter</tt>, which takes as arguments a painter and information on how to transform a frame and produces a new painter. The transformed painter, when called on a frame, transforms the frame and calls the original painter on the transformed frame. The arguments to <tt>transform-painter</tt> are points (represented as vectors) that specify the corners of the new frame: When mapped into the frame, the first point specifies the new frame's origin and the other two specify the ends of its edge vectors. Thus, arguments within the unit square specify a frame contained within the original frame.


<tt><a name="index_term_1962"></a>(define&nbsp;(transform-painter&nbsp;painter&nbsp;origin&nbsp;corner1&nbsp;corner2)<br>
&nbsp;&nbsp;(lambda&nbsp;(frame)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((m&nbsp;(frame-coord-map&nbsp;frame)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((new-origin&nbsp;(m&nbsp;origin)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(painter<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-frame&nbsp;new-origin<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(sub-vect&nbsp;(m&nbsp;corner1)&nbsp;new-origin)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(sub-vect&nbsp;(m&nbsp;corner2)&nbsp;new-origin)))))))<br></tt>


Here's how to flip painter images vertically:


<tt><a name="index_term_1964"></a>(define&nbsp;(flip-vert&nbsp;painter)<br>
&nbsp;&nbsp;(transform-painter&nbsp;painter<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;0.0&nbsp;1.0)&nbsp;&nbsp;&nbsp;*;&nbsp;new&nbsp;<tt>origin</tt>*<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;1.0&nbsp;1.0)&nbsp;&nbsp;&nbsp;*;&nbsp;new&nbsp;end&nbsp;of&nbsp;<tt>edge1</tt>*<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;0.0&nbsp;0.0)))&nbsp;*;&nbsp;new&nbsp;end&nbsp;of&nbsp;<tt>edge2</tt>*<br></tt>


Using <tt>transform-painter</tt>, we can easily define new transformations. For example, we can define a painter that shrinks its image to the upper-right quarter of the frame it is given:


<tt><a name="index_term_1966"></a>(define&nbsp;(shrink-to-upper-right&nbsp;painter)<br>
&nbsp;&nbsp;(transform-painter&nbsp;painter<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;0.5&nbsp;0.5)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;1.0&nbsp;0.5)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;0.5&nbsp;1.0)))<br></tt>


Other transformations rotate images counterclockwise by 90 degrees<a name="call_footnote_Temp_220" href="#footnote_Temp_220" id="call_footnote_Temp_220"><sup><small>29</small></sup></a>


<tt><a name="index_term_1968"></a>(define&nbsp;(rotate90&nbsp;painter)<br>
&nbsp;&nbsp;(transform-painter&nbsp;painter<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;1.0&nbsp;0.0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;1.0&nbsp;1.0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;0.0&nbsp;0.0)))<br></tt>


or squash images towards the center of the frame:<a name="call_footnote_Temp_221" href="#footnote_Temp_221" id="call_footnote_Temp_221"><sup><small>30</small></sup></a>


<tt><a name="index_term_1970"></a>(define&nbsp;(squash-inwards&nbsp;painter)<br>
&nbsp;&nbsp;(transform-painter&nbsp;painter<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;0.0&nbsp;0.0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;0.65&nbsp;0.35)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;0.35&nbsp;0.65)))<br></tt>


Frame transformation is also the key to defining means of combining two or more painters. The <tt>beside</tt> procedure, for example, takes two painters, transforms them to paint in the left and right halves of an argument frame respectively, and produces a new, compound painter. When the compound painter is given a frame, it calls the first transformed painter to paint in the left half of the frame and calls the second transformed painter to paint in the right half of the frame:


<tt><a name="index_term_1972"></a>(define&nbsp;(beside&nbsp;painter1&nbsp;painter2)<br>
&nbsp;&nbsp;(let&nbsp;((split-point&nbsp;(make-vect&nbsp;0.5&nbsp;0.0)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((paint-left<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(transform-painter&nbsp;painter1<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;0.0&nbsp;0.0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;split-point<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;0.0&nbsp;1.0)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(paint-right<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(transform-painter&nbsp;painter2<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;split-point<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;1.0&nbsp;0.0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-vect&nbsp;0.5&nbsp;1.0))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(frame)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(paint-left&nbsp;frame)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(paint-right&nbsp;frame)))))<br></tt>


Observe how the painter data abstraction, and in particular the representation of painters as procedures, makes <tt>beside</tt> easy to implement. The <tt>beside</tt> procedure need not know anything about the details of the component painters other than that each painter will draw something in its designated frame.


<a name="exercise_2_50">**Exercise 2.50**</a> Define the transformation <a name="index_term_1974"></a><tt>flip-horiz</tt>, which flips painters horizontally, and transformations that rotate painters counterclockwise by 180 degrees and 270 degrees.


<a name="exercise_2_51">**Exercise 2.51**</a> Define the <a name="index_term_1976"></a><tt>below</tt> operation for painters. <tt>Below</tt> takes two painters as arguments. The resulting painter, given a frame, draws with the first painter in the bottom of the frame and with the second painter in the top. Define <tt>below</tt> in two different ways -- first by writing a procedure that is analogous to the <tt>beside</tt> procedure given above, and again in terms of <tt>beside</tt> and suitable rotation operations (from exercise&nbsp;<a href="#%_thm_2.50">2.50</a>).


<a name="%_sec_Temp_224"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_224">Levels of language for robust design</a></h4>

The picture language exercises some of the critical ideas we've introduced about abstraction with procedures and data. The fundamental data abstractions, painters, are implemented using procedural representations, which enables the language to handle different basic drawing capabilities in a uniform way. The means of combination satisfy the closure property, which permits us to easily build up complex designs. Finally, all the tools for abstracting procedures are available to us for abstracting means of combination for painters.


We have also obtained a glimpse of another crucial idea about languages and program design. This is the approach of <a name="index_term_1978"></a><a name="index_term_1980"></a>*stratified design*, the notion that a complex system should be structured as a sequence of levels that are described using a sequence of languages. Each level is constructed by combining parts that are regarded as primitive at that level, and the parts constructed at each level are used as primitives at the next level. The language used at each level of a stratified design has primitives, means of combination, and means of abstraction appropriate to that level of detail.


Stratified design pervades the engineering of complex systems. For example, in computer engineering, resistors and transistors are combined (and described using a language of analog circuits) to produce parts such as and-gates and or-gates, which form the primitives of a language for digital-circuit design.<a name="call_footnote_Temp_225" href="#footnote_Temp_225" id="call_footnote_Temp_225"><sup><small>31</small></sup></a> These parts are combined to build processors, bus structures, and memory systems, which are in turn combined to form computers, using languages appropriate to computer architecture. Computers are combined to form distributed systems, using languages appropriate for describing network interconnections, and so on.


As a tiny example of stratification, our picture language uses primitive elements (primitive painters) that are created using a language that specifies points and lines to provide the lists of line segments for <tt>segments-&gt;painter</tt>, or the shading details for a painter like <tt>rogers</tt>. The bulk of our description of the picture language focused on combining these primitives, using geometric combiners such as <tt>beside</tt> and <tt>below</tt>. We also worked at a higher level, regarding <tt>beside</tt> and <tt>below</tt> as primitives to be manipulated in a language whose operations, such as <tt>square-of-four</tt>, capture common patterns of combining geometric combiners.


<a name="index_term_1982"></a>Stratified design helps make programs *robust*, that is, it makes it likely that small changes in a specification will require correspondingly small changes in the program. For instance, suppose we wanted to change the image based on <tt>wave</tt> shown in figure&nbsp;<a href="#%_fig_2.9">2.9</a>. We could work at the lowest level to change the detailed appearance of the <tt>wave</tt> element; we could work at the middle level to change the way <tt>corner-split</tt> replicates the <tt>wave</tt>; we could work at the highest level to change how <tt>square-limit</tt> arranges the four copies of the corner. In general, each level of a stratified design provides a different vocabulary for expressing the characteristics of the system, and a different kind of ability to change it.


<a name="exercise_2_52">**Exercise 2.52**</a> Make changes to the square limit of <tt>wave</tt> shown in figure&nbsp;<a href="#%_fig_2.9">2.9</a> by working at each of the levels described above. In particular:


a.&nbsp;&nbsp;Add some segments to the primitive <tt>wave</tt> painter of exercise &nbsp;<a href="#%_thm_2.49">2.49</a> (to add a smile, for example).


b.&nbsp;&nbsp;Change the pattern constructed by <tt>corner-split</tt> (for example, by using only one copy of the <tt>up-split</tt> and <tt>right-split</tt> images instead of two).


c.&nbsp;&nbsp;Modify the version of <tt>square-limit</tt> that uses <tt>square-of-four</tt> so as to assemble the corners in a different pattern. (For example, you might make the big Mr. Rogers look outward from each corner of the square.)

<div class="smallprint">
  <hr>
</div>

<div class="footnote">
  
<a name="footnote_Temp_154" href="#call_footnote_Temp_154" id="footnote_Temp_154"><sup><small>6</small></sup></a> The use of the word <a name="index_term_1536"></a>"closure" here comes from abstract algebra, where a set of elements is said to be closed under an operation if applying the operation to elements in the set produces an element that is again an element of the set. The Lisp community also (unfortunately) uses the word "closure" to describe a totally unrelated concept: A closure is an implementation technique for representing procedures with free variables. We do not use the word "closure" in this second sense in this book.

  
<a name="footnote_Temp_155" href="#call_footnote_Temp_155" id="footnote_Temp_155"><sup><small>7</small></sup></a> The notion that a means of <a name="index_term_1542"></a>combination should satisfy closure is a straightforward idea. Unfortunately, the data combiners provided in many popular programming languages do not satisfy closure, or make closure cumbersome to exploit. In <a name="index_term_1544"></a>Fortran or <a name="index_term_1546"></a>Basic, one typically combines data elements by assembling them into arrays -- but one cannot form arrays whose elements are themselves arrays. <a name="index_term_1548"></a>Pascal and <a name="index_term_1550"></a>C admit structures whose elements are structures. However, this requires that the programmer manipulate pointers explicitly, and adhere to the restriction that each field of a structure can contain only elements of a prespecified form. Unlike Lisp with its pairs, these languages have no built-in general-purpose glue that makes it easy to manipulate compound data in a uniform way. This limitation lies behind Alan <a name="index_term_1552"></a>Perlis's comment in his foreword to this book: "In Pascal the plethora of declarable data structures induces a specialization within functions that inhibits and penalizes casual cooperation. It is better to have 100 functions operate on one data structure than to have 10 functions operate on 10 data structures."

  
<a name="footnote_Temp_156" href="#call_footnote_Temp_156" id="footnote_Temp_156"><sup><small>8</small></sup></a> In this book, we use *list* to mean a chain of pairs terminated by the end-of-list marker. In contrast, the term <a name="index_term_1572"></a><a name="index_term_1574"></a>*list structure* refers to any data structure made out of pairs, not just to lists.

  
<a name="footnote_Temp_157" href="#call_footnote_Temp_157" id="footnote_Temp_157"><sup><small>9</small></sup></a> Since nested applications of <tt>car</tt> and <tt>cdr</tt> are cumbersome to write, Lisp dialects provide abbreviations for them -- for instance, <a name="index_term_1584"></a><a name="index_term_1586"></a>

  <div align="left">
    <img src="img/chapter_2_image_14.gif" border="0">
  </div>

  
The names of all such procedures start with <tt>c</tt> and end with <tt>r</tt>. Each <tt>a</tt> between them stands for a <a name="index_term_1588"></a><a name="index_term_1590"></a><tt>car</tt> operation and each <tt>d</tt> for a <tt>cdr</tt> operation, to be applied in the same order in which they appear in the name. The names <tt>car</tt> and <tt>cdr</tt> persist because simple combinations like <tt>cadr</tt> are pronounceable.

  
<a name="footnote_Temp_158" href="#call_footnote_Temp_158" id="footnote_Temp_158"><sup><small>10</small></sup></a> It's remarkable how much energy in the standardization of Lisp dialects has been dissipated in arguments that are literally over nothing: Should <tt>nil</tt> be an ordinary name? Should the value of <tt>nil</tt> be a symbol? Should it be a list? Should it be a pair? <a name="index_term_1598"></a>In Scheme, <tt>nil</tt> is an ordinary name, which we use in this section as a variable whose value is the end-of-list marker (just as <tt>true</tt> is an ordinary variable that has a true value). Other dialects of Lisp, including Common Lisp, treat <tt>nil</tt> as a special symbol. The <a name="index_term_1600"></a>authors of this book, who have endured too many language standardization brawls, would like to avoid the entire issue. Once we have introduced quotation in section <a href="chapter_2_section_3.html#%_sec_2.3">2.3</a>, we will denote the empty list as <tt>'()</tt> and dispense with the variable <tt>nil</tt> entirely.

  
<a name="footnote_Temp_164" href="#call_footnote_Temp_164" id="footnote_Temp_164"><sup><small>11</small></sup></a> To define <tt>f</tt> and <tt>g</tt> using <a name="index_term_1656"></a><tt>lambda</tt> we would write

  
<tt>(define&nbsp;f&nbsp;(lambda&nbsp;(x&nbsp;y&nbsp;.&nbsp;z)&nbsp;&lt;*body*&gt;))<br>
  (define&nbsp;g&nbsp;(lambda&nbsp;w&nbsp;&lt;*body*&gt;))<br></tt>

  
<a name="footnote_Temp_166" href="#call_footnote_Temp_166" id="footnote_Temp_166"><sup><small>12</small></sup></a> Scheme standardly provides a <a name="index_term_1664"></a><tt>map</tt> procedure that is more general than the one described here. This more general <tt>map</tt> takes a procedure of *n* arguments, together with *n* lists, and applies the procedure to all the first elements of the lists, all the second elements of the lists, and so on, returning a list of the results. For example:

  
<tt>(map&nbsp;+&nbsp;(list&nbsp;1&nbsp;2&nbsp;3)&nbsp;(list&nbsp;40&nbsp;50&nbsp;60)&nbsp;(list&nbsp;700&nbsp;800&nbsp;900))<br>
  <i>(741&nbsp;852&nbsp;963)</i><br>
  <br>
  (map&nbsp;(lambda&nbsp;(x&nbsp;y)&nbsp;(+&nbsp;x&nbsp;(*&nbsp;2&nbsp;y)))<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;1&nbsp;2&nbsp;3)<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;4&nbsp;5&nbsp;6))<br>
  <i>(9&nbsp;12&nbsp;15)</i><br></tt>

  
<a name="footnote_Temp_170" href="#call_footnote_Temp_170" id="footnote_Temp_170"><sup><small>13</small></sup></a> The order of the first two clauses in the <tt>cond</tt> matters, since the empty list satisfies <tt>null?</tt> and also is not a pair.

  
<a name="footnote_Temp_182" href="#call_footnote_Temp_182" id="footnote_Temp_182"><sup><small>14</small></sup></a> This is, in fact, precisely the <a name="index_term_1746"></a><tt>fringe</tt> procedure from exercise&nbsp;<a href="#%_thm_2.28">2.28</a>. Here we've renamed it to emphasize that it is part of a family of general sequence-manipulation procedures.

  
<a name="footnote_Temp_183" href="#call_footnote_Temp_183" id="footnote_Temp_183"><sup><small>15</small></sup></a> <a name="index_term_1760"></a>Richard Waters (1979) developed a program that automatically analyzes traditional <a name="index_term_1762"></a>Fortran programs, viewing them in terms of maps, filters, and accumulations. He found that fully 90 percent of the code in the Fortran Scientific Subroutine Package fits neatly into this paradigm. One of the reasons for the success of Lisp as a programming language is that lists provide a standard medium for expressing ordered collections so that they can be manipulated using higher-order operations. The programming language <a name="index_term_1764"></a>APL owes much of its power and appeal to a similar choice. In APL all data are represented as arrays, and there is a universal and convenient set of generic operators for all sorts of array operations.

  
<a name="footnote_Temp_186" href="#call_footnote_Temp_186" id="footnote_Temp_186"><sup><small>16</small></sup></a> According to <a name="index_term_1776"></a>Knuth (1981), this rule was formulated by <a name="index_term_1778"></a>W. G. Horner early in the nineteenth century, but the method was actually used by Newton over a hundred years earlier. Horner's rule evaluates the polynomial using fewer additions and multiplications than does the straightforward method of first computing *a*<sub>*n*</sub> *x*<sup>*n*</sup>, then adding *a*<sub>*n*-1</sub>*x*<sup>*n*-1</sup>, and so on. In fact, it is possible to prove that any algorithm for evaluating arbitrary polynomials must use at least as many additions and multiplications as does Horner's rule, and thus Horner's rule is an <a name="index_term_1780"></a><a name="index_term_1782"></a>optimal algorithm for polynomial evaluation. This was proved (for the number of additions) by <a name="index_term_1784"></a>A. M. Ostrowski in a 1954 paper that essentially founded the modern study of optimal algorithms. The analogous statement for multiplications was proved by <a name="index_term_1786"></a>V. Y. Pan in 1966. The book by <a name="index_term_1788"></a>Borodin and <a name="index_term_1790"></a>Munro (1975) provides an overview of these and other results about optimal algorithms.

  
<a name="footnote_Temp_190" href="#call_footnote_Temp_190" id="footnote_Temp_190"><sup><small>17</small></sup></a> This definition uses the extended version of <tt>map</tt> described in footnote&nbsp;<a href="#footnote_Temp_166">12</a>.

  
<a name="footnote_Temp_194" href="#call_footnote_Temp_194" id="footnote_Temp_194"><sup><small>18</small></sup></a> This approach to nested mappings was shown to us by <a name="index_term_1820"></a>David Turner, whose languages <a name="index_term_1822"></a>KRC and <a name="index_term_1824"></a>Miranda provide elegant formalisms for dealing with these constructs. The examples in this section (see also exercise&nbsp;<a href="#%_thm_2.42">2.42</a>) are adapted from Turner 1981. In section <a href="chapter_3_section_5.html#%_sec_3.5.3">3.5.3</a>, we'll see how this approach generalizes to infinite sequences.

  
<a name="footnote_Temp_195" href="#call_footnote_Temp_195" id="footnote_Temp_195"><sup><small>19</small></sup></a> We're representing a pair here as a list of two elements rather than as a Lisp pair. Thus, the "pair" (*i*,*j*) is represented as <tt>(list i j)</tt>, not <tt>(cons i j)</tt>.

  
<a name="footnote_Temp_196" href="#call_footnote_Temp_196" id="footnote_Temp_196"><sup><small>20</small></sup></a> The set *S* - *x* is the set of all elements of *S*, excluding *x*.

  
<a name="footnote_Temp_197" href="#call_footnote_Temp_197" id="footnote_Temp_197"><sup><small>21</small></sup></a> <a name="index_term_1834"></a><a name="index_term_1836"></a><a name="index_term_1838"></a>Semicolons in Scheme code are used to introduce *comments*. Everything from the semicolon to the end of the line is ignored by the interpreter. In this book we don't use many comments; we try to make our programs self-documenting by using descriptive names.

  
<a name="footnote_Temp_202" href="#call_footnote_Temp_202" id="footnote_Temp_202"><sup><small>22</small></sup></a> The picture language is based on the language <a name="index_term_1856"></a>Peter Henderson created to construct images like <a name="index_term_1858"></a>M.C. Escher's "Square Limit" woodcut (see Henderson 1982). The woodcut incorporates a repeated scaled pattern, similar to the arrangements drawn using the <tt>square-limit</tt> procedure in this section.

  
<a name="footnote_Temp_204" href="#call_footnote_Temp_204" id="footnote_Temp_204"><sup><small>23</small></sup></a> <a name="index_term_1866"></a><a name="index_term_1868"></a>William Barton Rogers (1804-1882) was the founder and first president of MIT. A geologist and talented teacher, he taught at William and Mary College and at the University of Virginia. In 1859 he moved to Boston, where he had more time for research, worked on a plan for establishing a "polytechnic institute," and served as Massachusetts's first State Inspector of Gas Meters.

  
When MIT was established in 1861, Rogers was elected its first president. Rogers espoused an ideal of "useful learning" that was different from the university education of the time, with its overemphasis on the classics, which, as he wrote, "stand in the way of the broader, higher and more practical instruction and discipline of the natural and social sciences." This education was likewise to be different from narrow trade-school education. In Rogers's words:

  <blockquote>
    
The world-enforced distinction between the practical and the scientific worker is utterly futile, and the whole experience of modern times has demonstrated its utter worthlessness.

  </blockquote>

  
Rogers served as president of MIT until 1870, when he resigned due to ill health. In 1878 the second president of MIT, <a name="index_term_1870"></a>John Runkle, resigned under the pressure of a financial crisis brought on by the Panic of 1873 and strain of fighting off attempts by Harvard to take over MIT. Rogers returned to hold the office of president until 1881.

  
Rogers collapsed and died while addressing MIT's graduating class at the commencement exercises of 1882. Runkle quoted Rogers's last words in a memorial address delivered that same year:

  <blockquote>
    
"As I stand here today and see what the Institute is, <tt>...</tt> I call to mind the beginnings of science. I remember one hundred and fifty years ago Stephen Hales published a pamphlet on the subject of illuminating gas, in which he stated that his researches had demonstrated that 128 grains of bituminous coal -- " <a name="index_term_1872"></a>

    
"Bituminous coal," these were his last words on earth. Here he bent forward, as if consulting some notes on the table before him, then slowly regaining an erect position, threw up his hands, and was translated from the scene of his earthly labors and triumphs to "the tomorrow of death," where the mysteries of life are solved, and the disembodied spirit finds unending satisfaction in contemplating the new and still unfathomable mysteries of the infinite future.

  </blockquote>In the words of Francis A. Walker <a name="index_term_1874"></a>(MIT's third president):

  <blockquote>
    
All his life he had borne himself most faithfully and heroically, and he died as so good a knight would surely have wished, in harness, at his post, and in the very part and act of public duty.

  </blockquote>

  
<a name="footnote_Temp_207" href="#call_footnote_Temp_207" id="footnote_Temp_207"><sup><small>24</small></sup></a> Equivalently, we could write

  
<tt><a name="index_term_1902"></a>(define&nbsp;flipped-pairs<br>
  &nbsp;&nbsp;(square-of-four&nbsp;identity&nbsp;flip-vert&nbsp;identity&nbsp;flip-vert))<br></tt>

  
<a name="footnote_Temp_208" href="#call_footnote_Temp_208" id="footnote_Temp_208"><sup><small>25</small></sup></a> <tt>Rotate180</tt> rotates a painter by 180 degrees (see exercise&nbsp;<a href="#%_thm_2.50">2.50</a>). Instead of <tt>rotate180</tt> we could say <tt>(compose flip-vert flip-horiz)</tt>, using the <tt>compose</tt> procedure from exercise&nbsp;<a href="chapter_1_section_3.html#exercise_1_42">1.42</a>.

  
<a name="footnote_Temp_211" href="#call_footnote_Temp_211" id="footnote_Temp_211"><sup><small>26</small></sup></a> <tt>Frame-coord-map</tt> uses the vector operations described in exercise&nbsp;<a href="#%_thm_2.46">2.46</a> below, which we assume have been implemented using some representation for vectors. Because of data abstraction, it doesn't matter what this vector representation is, so long as the vector operations behave correctly.

  
<a name="footnote_Temp_215" href="#call_footnote_Temp_215" id="footnote_Temp_215"><sup><small>27</small></sup></a> <tt>Segments-&gt;painter</tt> uses the representation for line segments described in exercise&nbsp;<a href="#%_thm_2.48">2.48</a> below. It also uses the <tt>for-each</tt> procedure described in exercise&nbsp;<a href="#%_thm_2.23">2.23</a>.

  
<a name="footnote_Temp_216" href="#call_footnote_Temp_216" id="footnote_Temp_216"><sup><small>28</small></sup></a> For example, the <tt>rogers</tt> painter of figure&nbsp;<a href="#%_fig_2.11">2.11</a> was constructed from a gray-level image. For each point in a given frame, the <tt>rogers</tt> painter determines the point in the image that is mapped to it under the frame coordinate map, and shades it accordingly. By allowing different types of painters, we are capitalizing on the abstract data idea discussed in section <a href="chapter_2_section_1.html#%_sec_2.1.3">2.1.3</a>, where we argued that a rational-number representation could be anything at all that satisfies an appropriate condition. Here we're using the fact that a painter can be implemented in any way at all, so long as it draws something in the designated frame. section <a href="chapter_2_section_1.html#%_sec_2.1.3">2.1.3</a> also showed how pairs could be implemented as procedures. Painters are our second example of a procedural representation for data.

  
<a name="footnote_Temp_220" href="#call_footnote_Temp_220" id="footnote_Temp_220"><sup><small>29</small></sup></a> <tt>Rotate90</tt> is a pure rotation only for square frames, because it also stretches and shrinks the image to fit into the rotated frame.

  
<a name="footnote_Temp_221" href="#call_footnote_Temp_221" id="footnote_Temp_221"><sup><small>30</small></sup></a> The diamond-shaped images in figures&nbsp;<a href="#%_fig_2.10">2.10</a> and&nbsp;<a href="#%_fig_2.11">2.11</a> were created with <tt>squash-inwards</tt> applied to <tt>wave</tt> and <tt>rogers</tt>.

  
<a name="footnote_Temp_225" href="#call_footnote_Temp_225" id="footnote_Temp_225"><sup><small>31</small></sup></a> section <a href="chapter_3_section_3.html#%_sec_3.3.4">3.3.4</a> describes one such language.

</div>