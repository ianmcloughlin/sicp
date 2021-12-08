(section_13)=
# Formulating Abstractions with Higher-Order Procedures

We have seen that procedures are, in effect, abstractions that describe compound operations on numbers independent of the particular numbers. For example, when we


<tt><a name="index_term_0962"></a>(define&#160;(cube&#160;x)&#160;(*&#160;x&#160;x&#160;x))<br></tt>


we are not talking about the cube of a particular number, but rather about a method for obtaining the cube of any number. Of course we could get along without ever defining this procedure, by always writing expressions such as


<tt>(*&#160;3&#160;3&#160;3)<br>
(*&#160;x&#160;x&#160;x)<br>
(*&#160;y&#160;y&#160;y)&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;<br></tt>


and never mentioning <tt>cube</tt> explicitly. This would place us at a serious disadvantage, forcing us to work always at the level of the particular operations that happen to be primitives in the language (multiplication, in this case) rather than in terms of higher-level operations. Our programs would be able to compute cubes, but our language would lack the ability to express the concept of cubing. One of the things we should demand from a powerful programming language is the ability to build abstractions by assigning names to common patterns and then to work in terms of the abstractions directly. Procedures provide this ability. This is why all but the most primitive programming languages include mechanisms for defining procedures.


Yet even in numerical processing we will be severely limited in our ability to create abstractions if we are restricted to procedures whose parameters must be numbers. Often the same programming pattern will be used with a number of different procedures. To express such patterns as concepts, we will need to construct procedures that can accept procedures as arguments or return procedures as values. Procedures that manipulate procedures are called <a name="index_term_0964"></a>*higher-order procedures*. This section shows how higher-order procedures can serve as powerful abstraction mechanisms, vastly increasing the expressive power of our language.


<a name="%_sec_1.3.1"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_1.3.1">1.3.1&#160;&#160;Procedures as Arguments</a></h3>

<a name="index_term_0966"></a><a name="index_term_0968"></a> Consider the following three procedures. The first computes the sum of the integers from <tt>a</tt> through <tt>b</tt>:


<tt><a name="index_term_0970"></a>(define&#160;(sum-integers&#160;a&#160;b)<br>
&#160;&#160;(if&#160;(&gt;&#160;a&#160;b)<br>
&#160;&#160;&#160;&#160;&#160;&#160;0<br>
&#160;&#160;&#160;&#160;&#160;&#160;(+&#160;a&#160;(sum-integers&#160;(+&#160;a&#160;1)&#160;b))))<br></tt>


The second computes the sum of the cubes of the integers in the given range:


<tt><a name="index_term_0972"></a>(define&#160;(sum-cubes&#160;a&#160;b)<br>
&#160;&#160;(if&#160;(&gt;&#160;a&#160;b)<br>
&#160;&#160;&#160;&#160;&#160;&#160;0<br>
&#160;&#160;&#160;&#160;&#160;&#160;(+&#160;(cube&#160;a)&#160;(sum-cubes&#160;(+&#160;a&#160;1)&#160;b))))<br></tt>


The third computes the sum of a sequence of terms in the series

<div align="left">
  <img src="img/chapter_1_image_26.gif" border="0">
</div>

which converges to <img src="img/book_09.gif" border="0">/8 (very slowly):<a name="call_footnote_Temp_90" href="#footnote_Temp_90" id="call_footnote_Temp_90"><sup><small>49</small></sup></a>


<tt><a name="index_term_0978"></a>(define&#160;(pi-sum&#160;a&#160;b)<br>
&#160;&#160;(if&#160;(&gt;&#160;a&#160;b)<br>
&#160;&#160;&#160;&#160;&#160;&#160;0<br>
&#160;&#160;&#160;&#160;&#160;&#160;(+&#160;(/&#160;1.0&#160;(*&#160;a&#160;(+&#160;a&#160;2)))&#160;(pi-sum&#160;(+&#160;a&#160;4)&#160;b))))<br></tt>


These three procedures clearly share a common underlying pattern. They are for the most part identical, differing only in the name of the procedure, the function of <tt>a</tt> used to compute the term to be added, and the function that provides the next value of <tt>a</tt>. We could generate each of the procedures by filling in slots in the same template:


<tt>(define&#160;(&lt;*name*&gt;&#160;a&#160;b)<br>
&#160;&#160;(if&#160;(&gt;&#160;a&#160;b)<br>
&#160;&#160;&#160;&#160;&#160;&#160;0<br>
&#160;&#160;&#160;&#160;&#160;&#160;(+&#160;(&lt;*term*&gt;&#160;a)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(&lt;*name*&gt;&#160;(&lt;*next*&gt;&#160;a)&#160;b))))<br></tt>


<a name="index_term_0980"></a>The presence of such a common pattern is strong evidence that there is a useful abstraction waiting to be brought to the surface. Indeed, mathematicians long ago identified the abstraction of <a name="index_term_0982"></a><a name="index_term_0984"></a>*summation of a series* and invented "sigma <a name="index_term_0986"></a><a name="index_term_0988"></a>notation," for example

<div align="left">
  <img src="img/chapter_1_image_27.gif" border="0">
</div>

to express this concept. The power of sigma notation is that it allows mathematicians to deal with the concept of summation itself rather than only with particular sums -- for example, to formulate general results about sums that are independent of the particular series being summed.


Similarly, as program designers, we would like our language to be powerful enough so that we can write a procedure that expresses the concept of summation itself rather than only procedures that compute particular sums. We can do so readily in our procedural language by taking the common template shown above and transforming the "slots" into formal parameters:


<tt><a name="index_term_0990"></a>(define&#160;(sum&#160;term&#160;a&#160;next&#160;b)<br>
&#160;&#160;(if&#160;(&gt;&#160;a&#160;b)<br>
&#160;&#160;&#160;&#160;&#160;&#160;0<br>
&#160;&#160;&#160;&#160;&#160;&#160;(+&#160;(term&#160;a)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(sum&#160;term&#160;(next&#160;a)&#160;next&#160;b))))<br></tt>


Notice that <tt>sum</tt> takes as its arguments the lower and upper bounds <tt>a</tt>&#160;and&#160;<tt>b</tt> together with the procedures <tt>term</tt> and <tt>next</tt>. We can use <tt>sum</tt> just as we would any procedure. For example, we can use it (along with a procedure <tt>inc</tt> that increments its argument by 1) to define <tt>sum-cubes</tt>:


<tt><a name="index_term_0992"></a>(define&#160;(inc&#160;n)&#160;(+&#160;n&#160;1))<br>
<a name="index_term_0994"></a>(define&#160;(sum-cubes&#160;a&#160;b)<br>
&#160;&#160;(sum&#160;cube&#160;a&#160;inc&#160;b))<br></tt>


Using this, we can compute the sum of the cubes of the integers from 1 to 10:


<tt>(sum-cubes&#160;1&#160;10)<br>
<i>3025</i><br></tt>


With the aid of an identity procedure to compute the term, we can define <tt>sum-integers</tt> in terms of <tt>sum</tt>:


<tt><a name="index_term_0996"></a>(define&#160;(identity&#160;x)&#160;x)<br>
<br>
<a name="index_term_0998"></a>(define&#160;(sum-integers&#160;a&#160;b)<br>
&#160;&#160;(sum&#160;identity&#160;a&#160;inc&#160;b))<br></tt>


Then we can add up the integers from 1 to 10:


<tt>(sum-integers&#160;1&#160;10)<br>
<i>55</i><br></tt>


We can also define <tt>pi-sum</tt> in the same way:<a name="call_footnote_Temp_91" href="#footnote_Temp_91" id="call_footnote_Temp_91"><sup><small>50</small></sup></a>


<tt><a name="index_term_1000"></a>(define&#160;(pi-sum&#160;a&#160;b)<br>
&#160;&#160;(define&#160;(pi-term&#160;x)<br>
&#160;&#160;&#160;&#160;(/&#160;1.0&#160;(*&#160;x&#160;(+&#160;x&#160;2))))<br>
&#160;&#160;(define&#160;(pi-next&#160;x)<br>
&#160;&#160;&#160;&#160;(+&#160;x&#160;4))<br>
&#160;&#160;(sum&#160;pi-term&#160;a&#160;pi-next&#160;b))<br></tt>


Using these procedures, we can compute an approximation to <img src="img/book_09.gif" border="0">:


<tt>(*&#160;8&#160;(pi-sum&#160;1&#160;1000))<br>
<i>3.139592655589783</i><br></tt>


Once we have <tt>sum</tt>, we can use it as a building block in formulating further concepts. For instance, the <a name="index_term_1002"></a>definite integral of a function *f* between the limits *a* and *b* can be approximated numerically using the formula

<div align="left">
  <img src="img/chapter_1_image_28.gif" border="0">
</div>

for small values of *d**x*. We can express this directly as a procedure:


<tt><a name="index_term_1004"></a>(define&#160;(integral&#160;f&#160;a&#160;b&#160;dx)<br>
&#160;&#160;(define&#160;(add-dx&#160;x)&#160;(+&#160;x&#160;dx))<br>
&#160;&#160;(*&#160;(sum&#160;f&#160;(+&#160;a&#160;(/&#160;dx&#160;2.0))&#160;add-dx&#160;b)<br>
&#160;&#160;&#160;&#160;&#160;dx))<br>
(integral&#160;cube&#160;0&#160;1&#160;0.01)<br>
<i>.24998750000000042</i><br>
(integral&#160;cube&#160;0&#160;1&#160;0.001)<br>
<i>.249999875000001</i><br></tt>


(The exact value of the integral of <tt>cube</tt> between 0 and 1 is 1/4.)


<a name="%_thm_1.29"></a> **Exercise 1.29.**&#160;&#160;<a name="index_term_1006"></a>Simpson's Rule is a more accurate method of numerical integration than the method illustrated above. Using Simpson's Rule, the integral of a function *f* between *a* and *b* is approximated as

<div align="left">
  <img src="img/chapter_1_image_29.gif" border="0">
</div>

where *h* = (*b* - *a*)/*n*, for some even integer *n*, and *y*<sub>*k*</sub> = *f*(*a* + *k**h*). (Increasing *n* increases the accuracy of the approximation.) Define a procedure that takes as arguments *f*, *a*, *b*, and *n* and returns the value of the integral, computed using Simpson's Rule. Use your procedure to integrate <tt>cube</tt> between 0 and 1 (with *n* = 100 and *n* = 1000), and compare the results to those of the <tt>integral</tt> procedure shown above.


<a name="%_thm_1.30"></a> **Exercise 1.30.**&#160;&#160;<a name="index_term_1008"></a>The <tt>sum</tt> procedure above generates a linear recursion. The procedure can be rewritten so that the sum is performed iteratively. Show how to do this by filling in the missing expressions in the following definition:


<tt>(define&#160;(sum&#160;term&#160;a&#160;next&#160;b)<br>
&#160;&#160;(define&#160;(iter&#160;a&#160;result)<br>
&#160;&#160;&#160;&#160;(if&#160;&lt;*??*&gt;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&lt;*??*&gt;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(iter&#160;&lt;*??*&gt;&#160;&lt;*??*&gt;)))<br>
&#160;&#160;(iter&#160;&lt;*??*&gt;&#160;&lt;*??*&gt;))<br></tt>


<a name="%_thm_1.31"></a> **Exercise 1.31.**&#160;&#160; <a name="index_term_1010"></a><br>
a.&#160;&#160;The <tt>sum</tt> procedure is only the simplest of a vast number of similar abstractions that can be captured as higher-order procedures.<a name="call_footnote_Temp_95" href="#footnote_Temp_95" id="call_footnote_Temp_95"><sup><small>51</small></sup></a> Write an analogous procedure called <tt>product</tt> that returns the product of the values of a function at points over a given range. Show how to define <a name="index_term_1012"></a><tt>factorial</tt> in terms of <tt>product</tt>. Also use <tt>product</tt> to compute approximations to <a name="index_term_1014"></a><img src="img/book_09.gif" border="0"> using the formula<a name="call_footnote_Temp_96" href="#footnote_Temp_96" id="call_footnote_Temp_96"><sup><small>52</small></sup></a>

<div align="left">
  <img src="img/chapter_1_image_30.gif" border="0">
</div>

b.&#160;&#160;If your <tt>product</tt> procedure generates a recursive process, write one that generates an iterative process. If it generates an iterative process, write one that generates a recursive process.


<a name="%_thm_1.32"></a> **Exercise 1.32.**&#160;&#160;<a name="index_term_1018"></a><a name="index_term_1020"></a><a name="index_term_1022"></a>a. Show that <tt>sum</tt> and <tt>product</tt> (exercise&#160;<a href="#%_thm_1.31">1.31</a>) are both special cases of a still more general notion called <tt>accumulate</tt> that combines a collection of terms, using some general accumulation function:


<tt>(accumulate&#160;combiner&#160;null-value&#160;term&#160;a&#160;next&#160;b)<br></tt>


<tt>Accumulate</tt> takes as arguments the same term and range specifications as <tt>sum</tt> and <tt>product</tt>, together with a <tt>combiner</tt> procedure (of two arguments) that specifies how the current term is to be combined with the accumulation of the preceding terms and a <tt>null-value</tt> that specifies what base value to use when the terms run out. Write <tt>accumulate</tt> and show how <tt>sum</tt> and <tt>product</tt> can both be defined as simple calls to <tt>accumulate</tt>.


b. If your <tt>accumulate</tt> procedure generates a recursive process, write one that generates an iterative process. If it generates an iterative process, write one that generates a recursive process.


<a name="%_thm_1.33"></a> **Exercise 1.33.**&#160;&#160;<a name="index_term_1024"></a>You can obtain an even more general version of <tt>accumulate</tt> (exercise&#160;<a href="#%_thm_1.32">1.32</a>) by introducing the notion of a <a name="index_term_1026"></a>*filter* on the terms to be combined. That is, combine only those terms derived from values in the range that satisfy a specified condition. The resulting <tt>filtered-accumulate</tt> abstraction takes the same arguments as accumulate, together with an additional predicate of one argument that specifies the filter. Write <tt>filtered-accumulate</tt> as a procedure. Show how to express the following using <tt>filtered-accumulate</tt>:


a. the sum of the squares of the prime numbers in the interval *a* to *b* (assuming that you have a <tt>prime?</tt> predicate already written)


b. the product of all the positive integers less than *n* <a name="index_term_1028"></a>that are relatively prime to&#160;*n* (i.e., all positive integers *i* &lt; *n* such that *G**C**D*(*i*,*n*) = 1).


<a name="%_sec_1.3.2"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_1.3.2">1.3.2&#160;&#160;Constructing Procedures Using <tt>Lambda</tt></a></h3>

In using <tt>sum</tt> as in section&#160;<a href="#%_sec_1.3.1">1.3.1</a>, it seems terribly awkward to have to define trivial procedures such as <tt>pi-term</tt> and <tt>pi-next</tt> just so we can use them as arguments to our higher-order procedure. Rather than define <tt>pi-next</tt> and <tt>pi-term</tt>, it would be more convenient to have a way to directly specify "the procedure that returns its input incremented by 4" and "the procedure that returns the reciprocal of its input times its input plus 2." We can do this by introducing the special form <tt>lambda</tt>, which creates procedures. Using <tt>lambda</tt> we can describe what we want as


<tt>(lambda&#160;(x)&#160;(+&#160;x&#160;4))<br></tt>


and


<tt>(lambda&#160;(x)&#160;(/&#160;1.0&#160;(*&#160;x&#160;(+&#160;x&#160;2))))<br></tt>


Then our <tt>pi-sum</tt> procedure can be expressed without defining any auxiliary procedures as


<tt><a name="index_term_1030"></a>(define&#160;(pi-sum&#160;a&#160;b)<br>
&#160;&#160;(sum&#160;(lambda&#160;(x)&#160;(/&#160;1.0&#160;(*&#160;x&#160;(+&#160;x&#160;2))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;a<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda&#160;(x)&#160;(+&#160;x&#160;4))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;b))<br></tt>


Again using <tt>lambda</tt>, we can write the <tt>integral</tt> procedure without having to define the auxiliary procedure <tt>add-dx</tt>:


<tt><a name="index_term_1032"></a>(define&#160;(integral&#160;f&#160;a&#160;b&#160;dx)<br>
&#160;&#160;(*&#160;(sum&#160;f<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(+&#160;a&#160;(/&#160;dx&#160;2.0))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda&#160;(x)&#160;(+&#160;x&#160;dx))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;b)<br>
&#160;&#160;&#160;&#160;&#160;dx))<br></tt>


<a name="index_term_1034"></a><a name="index_term_1036"></a><a name="index_term_1038"></a><a name="index_term_1040"></a><a name="index_term_1042"></a>In general, <tt>lambda</tt> is used to create procedures in the same way as <tt>define</tt>, except that <a name="index_term_1044"></a>no name is specified for the procedure:


<tt>(lambda&#160;(&lt;*formal-parameters*&gt;)&#160;&lt;*body*&gt;)<br></tt>


The resulting procedure is just as much a procedure as one that is created using <tt>define</tt>. The only difference is that it has not been associated with any name in the environment. In fact,


<a name="index_term_1046"></a>


<tt>(define&#160;(plus4&#160;x)&#160;(+&#160;x&#160;4))<br></tt>


is equivalent to


<tt>(define&#160;plus4&#160;(lambda&#160;(x)&#160;(+&#160;x&#160;4)))<br></tt>


We can read a <tt>lambda</tt> expression as follows:


<tt>&#160;&#160;&#160;&#160;(lambda&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(x)&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(+&#160;&#160;&#160;&#160;x&#160;&#160;&#160;&#160;&#160;4))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;<img src="img/book_16.gif" border="0">&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;<img src="img/book_16.gif" border="0">&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;<img src="img/book_16.gif" border="0">&#160;&#160;&#160;&#160;<img src="img/book_16.gif" border="0">&#160;&#160;&#160;&#160;<img src="img/book_16.gif" border="0"><br>
&#160;the&#160;procedure&#160;&#160;&#160;of&#160;an&#160;argument&#160;</tt>x&#160;&#160;that&#160;adds&#160;&#160;<tt>x</tt>&#160;and&#160;4<br>


<a name="index_term_1048"></a><a name="index_term_1050"></a><a name="index_term_1052"></a>Like any expression that has a procedure as its value, a <tt>lambda</tt> expression can be used as the operator in a combination such as


<tt>((lambda&#160;(x&#160;y&#160;z)&#160;(+&#160;x&#160;y&#160;(square&#160;z)))&#160;1&#160;2&#160;3)<br>
<i>12</i><br></tt>


or, more generally, in any context where we would normally use a procedure name.<a name="call_footnote_Temp_99" href="#footnote_Temp_99" id="call_footnote_Temp_99"><sup><small>53</small></sup></a>


<a name="%_sec_Temp_100"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_100">Using <tt>let</tt> to create local variables</a></h4>

<a name="index_term_1058"></a><a name="index_term_1060"></a> Another use of <tt>lambda</tt> is in creating local variables. We often need local variables in our procedures other than those that have been bound as formal parameters. For example, suppose we wish to compute the function

<div align="left">
  <img src="img/chapter_1_image_31.gif" border="0">
</div>

which we could also express as

<div align="left">
  <img src="img/chapter_1_image_32.gif" border="0">
</div>

In writing a procedure to compute *f*, we would like to include as local variables not only *x* and *y* but also the names of intermediate quantities like *a* and *b*. One way to accomplish this is to use an auxiliary procedure to bind the local variables:


<tt>(define&#160;(f&#160;x&#160;y)<br>
&#160;&#160;(define&#160;(f-helper&#160;a&#160;b)<br>
&#160;&#160;&#160;&#160;(+&#160;(*&#160;x&#160;(square&#160;a))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(*&#160;y&#160;b)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(*&#160;a&#160;b)))<br>
&#160;&#160;(f-helper&#160;(+&#160;1&#160;(*&#160;x&#160;y))&#160;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(-&#160;1&#160;y)))<br></tt>


Of course, we could use a <tt>lambda</tt> expression to specify an anonymous procedure for binding our local variables. The body of <tt>f</tt> then becomes a single call to that procedure:


<tt>(define&#160;(f&#160;x&#160;y)<br>
&#160;&#160;((lambda&#160;(a&#160;b)<br>
&#160;&#160;&#160;&#160;&#160;(+&#160;(*&#160;x&#160;(square&#160;a))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(*&#160;y&#160;b)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(*&#160;a&#160;b)))<br>
&#160;&#160;&#160;(+&#160;1&#160;(*&#160;x&#160;y))<br>
&#160;&#160;&#160;(-&#160;1&#160;y)))<br></tt>


This construct is so useful that there is a special form called <tt>let</tt> to make its use more convenient. Using <tt>let</tt>, the <tt>f</tt> procedure could be written as


<tt>(define&#160;(f&#160;x&#160;y)<br>
&#160;&#160;(let&#160;((a&#160;(+&#160;1&#160;(*&#160;x&#160;y)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(b&#160;(-&#160;1&#160;y)))<br>
&#160;&#160;&#160;&#160;(+&#160;(*&#160;x&#160;(square&#160;a))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(*&#160;y&#160;b)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(*&#160;a&#160;b))))<br></tt>


<a name="index_term_1062"></a><a name="index_term_1064"></a>The general form of a <tt>let</tt> expression is


<tt>(let&#160;((&lt;*var<sub>1</sub>*&gt;&#160;&lt;*exp<sub>1</sub>*&gt;)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(&lt;*var<sub>2</sub>*&gt;&#160;&lt;*exp<sub>2</sub>*&gt;)<br>
&#160;&#160;&#160;&#160;&#160;&#160;<img src="img/book_18.gif" border="0"><br>
&#160;&#160;&#160;&#160;&#160;&#160;(&lt;*var<sub>*n*</sub>*&gt;&#160;&lt;*exp<sub>*n*</sub>*&gt;))<br>
&#160;&#160;&#160;&lt;*body*&gt;)<br></tt>


which can be thought of as saying

<table border="0">
  <tr>
    <td valign="top">let</td>

    <td valign="top">&lt;*var<sub>1</sub>*&gt; have the value &lt;*exp<sub>1</sub>*&gt; and</td>
  </tr>

  <tr>
    <td valign="top"></td>

    <td valign="top">&lt;*var<sub>2</sub>*&gt; have the value &lt;*exp<sub>2</sub>*&gt; and</td>
  </tr>

  <tr>
    <td valign="top"></td>

    <td valign="top"><img src="img/book_18.gif" border="0"></td>
  </tr>

  <tr>
    <td valign="top"></td>

    <td valign="top">&lt;*var<sub>*n*</sub>*&gt; have the value &lt;*exp<sub>*n*</sub>*&gt;</td>
  </tr>

  <tr>
    <td valign="top">in</td>

    <td valign="top">&lt;*body*&gt;</td>
  </tr>
</table>

The first part of the <tt>let</tt> expression is a list of name-expression pairs. When the <tt>let</tt> is evaluated, each name is associated with the value of the corresponding expression. The body of the <tt>let</tt> is evaluated with these names bound as local variables. The way this happens is that the <tt>let</tt> expression is interpreted as an alternate syntax for


<tt>((lambda&#160;(&lt;*var<sub>1</sub>*&gt;&#160;</tt>...&lt;*var<sub>*n*</sub>*&gt;)<br>
&#160;&#160;&#160;&#160;&lt;*body*&gt;)<br>
&#160;&lt;*exp<sub>1</sub>*&gt;<br>
&#160;<img src="img/book_18.gif" border="0"><br>
&#160;&lt;*exp<sub>*n*</sub>*&gt;)<br>


No new mechanism is required in the interpreter in order to provide local variables. A <a name="index_term_1066"></a><a name="index_term_1068"></a><tt>let</tt> expression is simply syntactic sugar for the underlying <tt>lambda</tt> application.


<a name="index_term_1070"></a><a name="index_term_1072"></a>We can see from this equivalence that the scope of a variable specified by a <tt>let</tt> expression is the body of the <tt>let</tt>. This implies that:

<ul>
  <li>
    <tt>Let</tt> allows one to bind variables as locally as possible to where they are to be used. For example, if the value of <tt>x</tt> is 5, the value of the expression

    
<tt>(+&#160;(let&#160;((x&#160;3))<br>
    &#160;&#160;&#160;&#160;&#160;(+&#160;x&#160;(*&#160;x&#160;10)))<br>
    &#160;&#160;&#160;x)<br></tt>

    
is 38. Here, the <tt>x</tt> in the body of the <tt>let</tt> is 3, so the value of the <tt>let</tt> expression is 33. On the other hand, the <tt>x</tt> that is the second argument to the outermost <tt>+</tt> is still&#160;5.

  </li>

  <li>The variables' values are computed outside the <tt>let</tt>. This matters when the expressions that provide the values for the local variables depend upon variables having the same names as the local variables themselves. For example, if the value of <tt>x</tt> is 2, the expression

    
<tt>(let&#160;((x&#160;3)<br>
    &#160;&#160;&#160;&#160;&#160;&#160;(y&#160;(+&#160;x&#160;2)))<br>
    &#160;&#160;(*&#160;x&#160;y))<br></tt>

    
will have the value 12 because, inside the body of the <tt>let</tt>, <tt>x</tt> will be 3 and <tt>y</tt> will be 4 (which is the outer <tt>x</tt> plus 2).

  </li>
</ul>

<a name="index_term_1074"></a><a name="index_term_1076"></a>Sometimes we can use internal definitions to get the same effect as with <tt>let</tt>. For example, we could have defined the procedure <tt>f</tt> above as


<tt>(define&#160;(f&#160;x&#160;y)<br>
&#160;&#160;(define&#160;a&#160;(+&#160;1&#160;(*&#160;x&#160;y)))<br>
&#160;&#160;(define&#160;b&#160;(-&#160;1&#160;y))<br>
&#160;&#160;(+&#160;(*&#160;x&#160;(square&#160;a))<br>
&#160;&#160;&#160;&#160;&#160;(*&#160;y&#160;b)<br>
&#160;&#160;&#160;&#160;&#160;(*&#160;a&#160;b)))<br></tt>


We prefer, however, to use <tt>let</tt> in situations like this and to use internal <tt>define</tt> only for internal procedures.<a name="call_footnote_Temp_101" href="#footnote_Temp_101" id="call_footnote_Temp_101"><sup><small>54</small></sup></a>


<a name="%_thm_1.34"></a> **Exercise 1.34.**&#160;&#160;Suppose we define the procedure


<tt>(define&#160;(f&#160;g)<br>
&#160;&#160;(g&#160;2))<br></tt>


Then we have


<tt>(f&#160;square)<br>
<i>4</i><br>
<br>
(f&#160;(lambda&#160;(z)&#160;(*&#160;z&#160;(+&#160;z&#160;1))))<br>
<i>6</i><br></tt>


What happens if we (perversely) ask the interpreter to evaluate the combination <tt>(f f)</tt>? Explain.


<a name="%_sec_1.3.3"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_1.3.3">1.3.3&#160;&#160;Procedures as General Methods</a></h3>

<a name="index_term_1078"></a><a name="index_term_1080"></a> We introduced compound procedures in section&#160;<a href="chapter_1_section_1.html#%_sec_1.1.4">1.1.4</a> as a mechanism for abstracting patterns of numerical operations so as to make them independent of the particular numbers involved. With higher-order procedures, such as the <tt>integral</tt> procedure of section&#160;<a href="#%_sec_1.3.1">1.3.1</a>, we began to see a more powerful kind of abstraction: procedures used to express general methods of computation, independent of the particular functions involved. In this section we discuss two more elaborate examples -- general methods for finding zeros and fixed points of functions -- and show how these methods can be expressed directly as procedures.


<a name="%_sec_Temp_103"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_103">Finding roots of equations by the half-interval method</a></h4>

<a name="index_term_1082"></a> The *half-interval method* is a simple but powerful technique for finding roots of an equation *f*(*x*) = 0, where *f* is a continuous function. The idea is that, if we are given points *a* and *b* such that *f*(*a*) &lt; 0 &lt; *f*(*b*), then *f* must have at least one zero between *a* and *b*. To locate a zero, let *x* be the average of *a* and *b* and compute *f*(*x*). If *f*(*x*) &gt; 0, then *f* must have a zero between *a* and *x*. If *f*(*x*) &lt; 0, then *f* must have a zero between *x* and *b*. Continuing in this way, we can identify smaller and smaller intervals on which *f* must have a zero. When we reach a point where the interval is small enough, the process stops. Since the interval of uncertainty is reduced by half at each step of the process, the number of steps required grows as <img src="img/book_03.gif" border="0">(<tt>log</tt>( *L*/*T*)), where *L* is the length of the original interval and *T* is the error tolerance (that is, the size of the interval we will consider "small enough"). Here is a procedure that implements this strategy:


<tt><a name="index_term_1084"></a>(define&#160;(search&#160;f&#160;neg-point&#160;pos-point)<br>
&#160;&#160;(let&#160;((midpoint&#160;(average&#160;neg-point&#160;pos-point)))<br>
&#160;&#160;&#160;&#160;(if&#160;(close-enough?&#160;neg-point&#160;pos-point)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;midpoint<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(let&#160;((test-value&#160;(f&#160;midpoint)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cond&#160;((positive?&#160;test-value)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(search&#160;f&#160;neg-point&#160;midpoint))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((negative?&#160;test-value)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(search&#160;f&#160;midpoint&#160;pos-point))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;midpoint))))))<br></tt>


We assume that we are initially given the function *f* together with points at which its values are negative and positive. We first compute the midpoint of the two given points. Next we check to see if the given interval is small enough, and if so we simply return the midpoint as our answer. Otherwise, we compute as a test value the value of *f* at the midpoint. If the test value is positive, then we continue the process with a new interval running from the original negative point to the midpoint. If the test value is negative, we continue with the interval from the midpoint to the positive point. Finally, there is the possibility that the test value is&#160;0, in which case the midpoint is itself the root we are searching for.


To test whether the endpoints are "close enough" we can use a procedure similar to the one used in section&#160;<a href="chapter_1_section_1.html#%_sec_1.1.7">1.1.7</a> for computing square roots:<a name="call_footnote_Temp_104" href="#footnote_Temp_104" id="call_footnote_Temp_104"><sup><small>55</small></sup></a>


<tt>(define&#160;(close-enough?&#160;x&#160;y)<br>
&#160;&#160;(&lt;&#160;(abs&#160;(-&#160;x&#160;y))&#160;0.001))<br></tt>


<tt>Search</tt> is awkward to use directly, because we can accidentally give it points at which *f*'s values do not have the required sign, in which case we get a wrong answer. Instead we will use <tt>search</tt> via the following procedure, which checks to see which of the endpoints has a negative function value and which has a positive value, and calls the <tt>search</tt> procedure accordingly. If the function has the same sign on the two given points, the half-interval method cannot be used, in which case the procedure signals an error.<a name="call_footnote_Temp_105" href="#footnote_Temp_105" id="call_footnote_Temp_105"><sup><small>56</small></sup></a>


<tt><a name="index_term_1092"></a>(define&#160;(half-interval-method&#160;f&#160;a&#160;b)<br>
&#160;&#160;(let&#160;((a-value&#160;(f&#160;a))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(b-value&#160;(f&#160;b)))<br>
&#160;&#160;&#160;&#160;(cond&#160;((and&#160;(negative?&#160;a-value)&#160;(positive?&#160;b-value))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(search&#160;f&#160;a&#160;b))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((and&#160;(negative?&#160;b-value)&#160;(positive?&#160;a-value))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(search&#160;f&#160;b&#160;a))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Values&#160;are&#160;not&#160;of&#160;opposite&#160;sign&quot;&#160;a&#160;b)))))<br></tt>


<a name="index_term_1094"></a>The following example uses the half-interval method to approximate <img src="img/book_09.gif" border="0"> as the root between 2 and 4 of <tt>sin</tt> *x* = 0:


<tt>(half-interval-method&#160;sin&#160;2.0&#160;4.0)<br>
<i>3.14111328125</i><br></tt>


Here is another example, using the half-interval method to search for a root of the equation *x*<sup>3</sup> - 2*x* - 3 = 0 between 1 and 2:


<tt>(half-interval-method&#160;(lambda&#160;(x)&#160;(-&#160;(*&#160;x&#160;x&#160;x)&#160;(*&#160;2&#160;x)&#160;3))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;1.0<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;2.0)<br>
<i>1.89306640625</i><br></tt>


<a name="%_sec_Temp_106"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_106">Finding fixed points of functions</a></h4>

<a name="index_term_1096"></a><a name="index_term_1098"></a> A number *x* is called a *fixed point* of a function *f* if *x* satisfies the equation *f*(*x*) = *x*. For some functions *f* we can locate a fixed point by beginning with an initial guess and applying *f* repeatedly,

<div align="left">
  <img src="img/chapter_1_image_33.gif" border="0">
</div>

until the value does not change very much. Using this idea, we can devise a procedure <tt>fixed-point</tt> that takes as inputs a function and an initial guess and produces an approximation to a fixed point of the function. We apply the function repeatedly until we find two successive values whose difference is less than some prescribed tolerance:


<tt>(define&#160;tolerance&#160;0.00001)<br>
<a name="index_term_1100"></a>(define&#160;(fixed-point&#160;f&#160;first-guess)<br>
&#160;&#160;(define&#160;(close-enough?&#160;v1&#160;v2)<br>
&#160;&#160;&#160;&#160;(&lt;&#160;(abs&#160;(-&#160;v1&#160;v2))&#160;tolerance))<br>
&#160;&#160;(define&#160;(try&#160;guess)<br>
&#160;&#160;&#160;&#160;(let&#160;((next&#160;(f&#160;guess)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;(close-enough?&#160;guess&#160;next)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;next<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(try&#160;next))))<br>
&#160;&#160;(try&#160;first-guess))<br></tt>


<a name="index_term_1102"></a><a name="index_term_1104"></a>For example, we can use this method to approximate the fixed point of the cosine function, starting with 1 as an initial approximation:<a name="call_footnote_Temp_107" href="#footnote_Temp_107" id="call_footnote_Temp_107"><sup><small>57</small></sup></a>


<tt><a name="index_term_1112"></a><a name="index_term_1114"></a>(fixed-point&#160;cos&#160;1.0)<br>
<i>.7390822985224023</i><br></tt>


Similarly, we can find a solution to the equation *y* = <tt>sin</tt> *y* + <tt>cos</tt> *y*:


<tt><a name="index_term_1116"></a><a name="index_term_1118"></a>(fixed-point&#160;(lambda&#160;(y)&#160;(+&#160;(sin&#160;y)&#160;(cos&#160;y)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;1.0)<br>
<i>1.2587315962971173</i><br></tt>


The fixed-point process is reminiscent of the process we used for finding square roots in section&#160;<a href="chapter_1_section_1.html#%_sec_1.1.7">1.1.7</a>. Both are based on the idea of repeatedly improving a guess until the result satisfies some criterion. In fact, we can readily formulate the <a name="index_term_1120"></a>square-root computation as a fixed-point search. Computing the square root of some number *x* requires finding a *y* such that *y*<sup>2</sup> = *x*. Putting this equation into the equivalent form *y* = *x*/*y*, we recognize that we are looking for a fixed point of the function<a name="call_footnote_Temp_108" href="#footnote_Temp_108" id="call_footnote_Temp_108"><sup><small>58</small></sup></a> *y* <img src="img/book_17.gif" border="0"> *x*/*y*, and we can therefore try to compute square roots as


<tt>(define&#160;(sqrt&#160;x)<br>
&#160;&#160;(fixed-point&#160;(lambda&#160;(y)&#160;(/&#160;x&#160;y))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;1.0))<br></tt>


Unfortunately, this fixed-point search does not converge. Consider an initial guess *y*<sub>1</sub>. The next guess is *y*<sub>2</sub> = *x*/*y*<sub>1</sub> and the next guess is *y*<sub>3</sub> = *x*/*y*<sub>2</sub> = *x*/(*x*/*y*<sub>1</sub>) = *y*<sub>1</sub>. This results in an infinite loop in which the two guesses *y*<sub>1</sub> and *y*<sub>2</sub> repeat over and over, oscillating about the answer.


One way to control such oscillations is to prevent the guesses from changing so much. Since the answer is always between our guess *y* and *x*/*y*, we can make a new guess that is not as far from *y* as *x*/*y* by averaging *y* with *x*/*y*, so that the next guess after *y* is (1/2)(*y* + *x*/*y*) instead of *x*/*y*. The process of making such a sequence of guesses is simply the process of looking for a fixed point of *y* <img src="img/book_17.gif" border="0"> (1/2)(*y* + *x*/*y*):


<tt><a name="index_term_1126"></a>(define&#160;(sqrt&#160;x)<br>
&#160;&#160;(fixed-point&#160;(lambda&#160;(y)&#160;(average&#160;y&#160;(/&#160;x&#160;y)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;1.0))<br></tt>


(Note that *y* = (1/2)(*y* + *x*/*y*) is a simple transformation of the equation *y* = *x*/*y*; to derive it, add *y* to both sides of the equation and divide by&#160;2.)


With this modification, the square-root procedure works. In fact, if we unravel the definitions, we can see that the sequence of approximations to the square root generated here is precisely the same as the one generated by our original square-root procedure of section&#160;<a href="chapter_1_section_1.html#%_sec_1.1.7">1.1.7</a>. This approach of averaging successive approximations to a solution, a technique we that we call <a name="index_term_1128"></a>*average damping*, often aids the convergence of fixed-point searches.


<a name="%_thm_1.35"></a> **Exercise 1.35.**&#160;&#160;<a name="index_term_1130"></a><a name="index_term_1132"></a>Show that the golden ratio <img src="img/book_11.gif" border="0"> (section&#160;<a href="chapter_1_section_2.html#%_sec_1.2.2">1.2.2</a>) is a fixed point of the transformation *x* <img src="img/book_17.gif" border="0"> 1 + 1/*x*, and use this fact to compute <img src="img/book_11.gif" border="0"> by means of the <tt>fixed-point</tt> procedure.


<a name="%_thm_1.36"></a> **Exercise 1.36.**&#160;&#160;Modify <tt>fixed-point</tt> so that it prints the sequence of approximations it generates, using the <tt>newline</tt> and <tt>display</tt> primitives shown in exercise&#160;<a href="chapter_1_section_2.html#exercise_1_22">1.22</a>. Then find a solution to *x*<sup>*x*</sup> = 1000 by finding a fixed point of *x* <img src="img/book_17.gif" border="0"> <tt>log</tt>(1000)/<tt>log</tt>(*x*). (Use Scheme's <a name="index_term_1134"></a><a name="index_term_1136"></a>primitive <tt>log</tt> procedure, which computes natural logarithms.) Compare the number of steps this takes with and without average damping. (Note that you cannot start <tt>fixed-point</tt> with a guess of 1, as this would cause division by <tt>log</tt>(1) = 0.)


<a name="%_thm_1.37"></a> **Exercise 1.37.**&#160;&#160;<a name="index_term_1138"></a>a. An infinite *continued fraction* is an expression of the form

<div align="left">
  <img src="img/chapter_1_image_34.gif" border="0">
</div>

<a name="index_term_1140"></a><a name="index_term_1142"></a>As an example, one can show that the infinite continued fraction expansion with the *N*<sub>*i*</sub> and the *D*<sub>*i*</sub> all equal to 1 produces 1/<img src="img/book_11.gif" border="0">, where <img src="img/book_11.gif" border="0"> is the golden ratio (described in section&#160;<a href="chapter_1_section_2.html#%_sec_1.2.2">1.2.2</a>). One way to approximate an infinite continued fraction is to truncate the expansion after a given number of terms. Such a truncation -- a so-called **k*-term finite continued fraction* -- has the form

<div align="left">
  <img src="img/chapter_1_image_35.gif" border="0">
</div>

Suppose that <tt>n</tt> and <tt>d</tt> are procedures of one argument (the term index *i*) that return the *N*<sub>*i*</sub> and *D*<sub>*i*</sub> of the terms of the continued fraction. Define a procedure <tt>cont-frac</tt> such that evaluating <tt>(cont-frac n d k)</tt> computes the value of the *k*-term finite continued fraction. Check your procedure by approximating 1/<img src="img/book_11.gif" border="0"> using


<tt>(cont-frac&#160;(lambda&#160;(i)&#160;1.0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda&#160;(i)&#160;1.0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;k)<br></tt>


for successive values of <tt>k</tt>. How large must you make <tt>k</tt> in order to get an approximation that is accurate to 4 decimal places?


b. If your <tt>cont-frac</tt> procedure generates a recursive process, write one that generates an iterative process. If it generates an iterative process, write one that generates a recursive process.


<a name="%_thm_1.38"></a> **Exercise 1.38.**&#160;&#160;<a name="index_term_1144"></a>In 1737, the Swiss mathematician Leonhard Euler published a memoir *De Fractionibus Continuis*, which included a <a name="index_term_1146"></a><a name="index_term_1148"></a>continued fraction expansion for *e* - 2, where *e* is the base of the natural logarithms. In this fraction, the *N*<sub>*i*</sub> are all 1, and the *D*<sub>*i*</sub> are successively 1, 2, 1, 1, 4, 1, 1, 6, 1, 1, 8, <tt>...</tt>. Write a program that uses your <tt>cont-frac</tt> procedure from exercise&#160;<a href="#%_thm_1.37">1.37</a> to approximate *e*, based on Euler's expansion.


<a name="%_thm_1.39"></a> **Exercise 1.39.**&#160;&#160;<a name="index_term_1150"></a><a name="index_term_1152"></a><a name="index_term_1154"></a>A continued fraction representation of the tangent function was published in 1770 by the German mathematician J.H. Lambert:

<div align="left">
  <img src="img/chapter_1_image_36.gif" border="0">
</div>

where *x* is in radians. Define a procedure <tt>(tan-cf x k)</tt> that computes an approximation to the tangent function based on Lambert's formula. <tt>K</tt> specifies the number of terms to compute, as in exercise&#160;<a href="#%_thm_1.37">1.37</a>.


<a name="%_sec_1.3.4"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_1.3.4">1.3.4&#160;&#160;Procedures as Returned Values</a></h3>

<a name="index_term_1156"></a><a name="index_term_1158"></a> The above examples demonstrate how the ability to pass procedures as arguments significantly enhances the expressive power of our programming language. We can achieve even more expressive power by creating procedures whose returned values are themselves procedures.


We can illustrate this idea by looking again at the fixed-point example described at the end of section&#160;<a href="#%_sec_1.3.3">1.3.3</a>. We formulated a new version of the square-root procedure as a fixed-point search, starting with the observation that <img src="img/book_13.gif" border="0">*x* is a fixed-point of the function *y* <img src="img/book_17.gif" border="0"> *x*/*y*. Then we used average damping to make the approximations converge. Average damping is a useful general technique in itself. Namely, given a function&#160;*f*, we consider the function whose value at *x* is equal to the average of *x* and *f*(*x*).


We can express the idea of average damping by means of the following procedure:


<tt><a name="index_term_1160"></a>(define&#160;(average-damp&#160;f)<br>
&#160;&#160;(lambda&#160;(x)&#160;(average&#160;x&#160;(f&#160;x))))<br></tt>


<tt>Average-damp</tt> is a procedure that takes as its argument a procedure <tt>f</tt> and returns as its value a procedure (produced by the <tt>lambda</tt>) that, when applied to a number <tt>x</tt>, produces the average of <tt>x</tt> and <tt>(f x)</tt>. For example, applying <tt>average-damp</tt> to the <tt>square</tt> procedure produces a procedure whose value at some number *x* is the average of *x* and *x*<sup>2</sup>. Applying this resulting procedure to 10 returns the average of 10 and 100, or 55:<a name="call_footnote_Temp_114" href="#footnote_Temp_114" id="call_footnote_Temp_114"><sup><small>59</small></sup></a>


<tt>((average-damp&#160;square)&#160;10)<br>
<i>55</i><br></tt>


<a name="index_term_1168"></a>Using <tt>average-damp</tt>, we can reformulate the square-root procedure as follows:


<tt><a name="index_term_1170"></a>(define&#160;(sqrt&#160;x)<br>
&#160;&#160;(fixed-point&#160;(average-damp&#160;(lambda&#160;(y)&#160;(/&#160;x&#160;y)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;1.0))<br></tt>


Notice how this formulation makes explicit the three ideas in the method: fixed-point search, average damping, and the function *y* <img src="img/book_17.gif" border="0"> *x*/*y*. It is instructive to compare this formulation of the square-root method with the original version given in section&#160;<a href="chapter_1_section_1.html#%_sec_1.1.7">1.1.7</a>. Bear in mind that these procedures express the same process, and notice how much clearer the idea becomes when we express the process in terms of these abstractions. In general, there are many ways to formulate a process as a procedure. Experienced programmers know how to choose procedural formulations that are particularly perspicuous, and where useful elements of the process are exposed as separate entities that can be reused in other applications. As a simple example of reuse, notice that the cube root of *x* is a fixed point of the function *y* <img src="img/book_17.gif" border="0"> *x*/*y*<sup>2</sup>, so we can immediately generalize our square-root procedure to one that extracts <a name="index_term_1172"></a><a name="index_term_1174"></a>cube roots:<a name="call_footnote_Temp_115" href="#footnote_Temp_115" id="call_footnote_Temp_115"><sup><small>60</small></sup></a>


<tt><a name="index_term_1176"></a>(define&#160;(cube-root&#160;x)<br>
&#160;&#160;(fixed-point&#160;(average-damp&#160;(lambda&#160;(y)&#160;(/&#160;x&#160;(square&#160;y))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;1.0))<br></tt>


<a name="%_sec_Temp_116"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_116">Newton's method</a></h4>

<a name="index_term_1178"></a> When we first introduced the square-root procedure, in section&#160;<a href="chapter_1_section_1.html#%_sec_1.1.7">1.1.7</a>, we mentioned that this was a special case of *Newton's method*. If *x* <img src="img/book_17.gif" border="0"> *g*(*x*) is a differentiable function, then a solution of the equation *g*(*x*) = 0 is a fixed point of the function *x* <img src="img/book_17.gif" border="0"> *f*(*x*) where

<div align="left">
  <img src="img/chapter_1_image_37.gif" border="0">
</div>

and *D**g*(*x*) is the derivative of *g* evaluated at *x*. <a name="index_term_1180"></a>Newton's method is the use of the fixed-point method we saw above to approximate a solution of the equation by finding a fixed point of the function *f*.<a name="call_footnote_Temp_117" href="#footnote_Temp_117" id="call_footnote_Temp_117"><sup><small>61</small></sup></a> For many functions *g* and for sufficiently good initial guesses for *x*, Newton's method converges very rapidly to a solution of *g*(*x*) = 0.<a name="call_footnote_Temp_118" href="#footnote_Temp_118" id="call_footnote_Temp_118"><sup><small>62</small></sup></a>


<a name="index_term_1186"></a><a name="index_term_1188"></a><a name="index_term_1190"></a>In order to implement Newton's method as a procedure, we must first express the idea of derivative. Note that "derivative," like average damping, is something that transforms a function into another function. For instance, the derivative of the function *x* <img src="img/book_17.gif" border="0"> *x*<sup>3</sup> is the function *x* <img src="img/book_17.gif" border="0"> 3*x*<sup>2</sup>. In general, if *g* is a function and *d**x* is a small number, then the derivative *D**g* of *g* is the function whose value at any number *x* is given (in the limit of small *d**x*) by

<div align="left">
  <img src="img/chapter_1_image_38.gif" border="0">
</div>

Thus, we can express the idea of derivative (taking *d**x* to be, say, 0.00001) as the procedure


<tt><a name="index_term_1192"></a>(define&#160;(deriv&#160;g)<br>
&#160;&#160;(lambda&#160;(x)<br>
&#160;&#160;&#160;&#160;(/&#160;(-&#160;(g&#160;(+&#160;x&#160;dx))&#160;(g&#160;x))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;dx)))<br></tt>


along with the definition


<tt>(define&#160;dx&#160;0.00001)<br></tt>


Like <tt>average-damp</tt>, <tt>deriv</tt> is a procedure that takes a procedure as argument and returns a procedure as value. For example, to approximate the derivative of *x* <img src="img/book_17.gif" border="0"> *x*<sup>3</sup> at 5 (whose exact value is 75) we can evaluate


<tt><a name="index_term_1194"></a>(define&#160;(cube&#160;x)&#160;(*&#160;x&#160;x&#160;x))<br>
((deriv&#160;cube)&#160;5)<br>
<i>75.00014999664018</i><br></tt>


With the aid of <tt>deriv</tt>, we can express Newton's method as a fixed-point process:


<tt><a name="index_term_1196"></a>(define&#160;(newton-transform&#160;g)<br>
&#160;&#160;(lambda&#160;(x)<br>
&#160;&#160;&#160;&#160;(-&#160;x&#160;(/&#160;(g&#160;x)&#160;((deriv&#160;g)&#160;x)))))<br>
<a name="index_term_1198"></a>(define&#160;(newtons-method&#160;g&#160;guess)<br>
&#160;&#160;(fixed-point&#160;(newton-transform&#160;g)&#160;guess))<br></tt>


The <tt>newton-transform</tt> procedure expresses the formula at the beginning of this section, and <tt>newtons-method</tt> is readily defined in terms of this. It takes as arguments a procedure that computes the function for which we want to find a zero, together with an initial guess. For instance, to find the <a name="index_term_1200"></a>square root of *x*, we can use Newton's method to find a zero of the function *y* <img src="img/book_17.gif" border="0"> *y*<sup>2</sup> - *x* starting with an initial guess of 1.<a name="call_footnote_Temp_119" href="#footnote_Temp_119" id="call_footnote_Temp_119"><sup><small>63</small></sup></a> This provides yet another form of the square-root procedure:


<tt><a name="index_term_1202"></a>(define&#160;(sqrt&#160;x)<br>
&#160;&#160;(newtons-method&#160;(lambda&#160;(y)&#160;(-&#160;(square&#160;y)&#160;x))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;1.0))<br></tt>


<a name="%_sec_Temp_120"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_120">Abstractions and first-class procedures</a></h4>

We've seen two ways to express the square-root computation as an instance of a more general method, once as a fixed-point search and once using Newton's method. Since Newton's method was itself expressed as a fixed-point process, we actually saw two ways to compute square roots as fixed points. Each method begins with a function and finds a <a name="index_term_1204"></a>fixed point of some transformation of the function. We can express this general idea itself as a procedure:


<tt><a name="index_term_1206"></a>(define&#160;(fixed-point-of-transform&#160;g&#160;transform&#160;guess)<br>
&#160;&#160;(fixed-point&#160;(transform&#160;g)&#160;guess))<br></tt>


This very general procedure takes as its arguments a procedure <tt>g</tt> that computes some function, a procedure that transforms <tt>g</tt>, and an initial guess. The returned result is a fixed point of the transformed function.


<a name="index_term_1208"></a>Using this abstraction, we can recast the first square-root computation from this section (where we look for a fixed point of the average-damped version of *y* <img src="img/book_17.gif" border="0"> *x*/*y*) as an instance of this general method:


<tt><a name="index_term_1210"></a>(define&#160;(sqrt&#160;x)<br>
&#160;&#160;(fixed-point-of-transform&#160;(lambda&#160;(y)&#160;(/&#160;x&#160;y))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;average-damp<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;1.0))<br></tt>


<a name="index_term_1212"></a>Similarly, we can express the second square-root computation from this section (an instance of Newton's method that finds a fixed point of the Newton transform of *y* <img src="img/book_17.gif" border="0"> *y*<sup>2</sup> - *x*) as


<tt><a name="index_term_1214"></a><a name="index_term_1216"></a>(define&#160;(sqrt&#160;x)<br>
&#160;&#160;(fixed-point-of-transform&#160;(lambda&#160;(y)&#160;(-&#160;(square&#160;y)&#160;x))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;newton-transform<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;1.0))<br></tt>


We began section&#160;<a href="#%_sec_1.3">1.3</a> with the observation that compound procedures are a crucial abstraction mechanism, because they permit us to express general methods of computing as explicit elements in our programming language. Now we've seen how higher-order procedures permit us to manipulate these general methods to create further abstractions.


As programmers, we should be alert to opportunities to identify the underlying abstractions in our programs and to build upon them and generalize them to create more powerful abstractions. This is not to say that one should always write programs in the most abstract way possible; expert programmers know how to choose the level of abstraction appropriate to their task. But it is important to be able to think in terms of these abstractions, so that we can be ready to apply them in new contexts. The significance of higher-order procedures is that they enable us to represent these abstractions explicitly as elements in our programming language, so that they can be handled just like other computational elements.


In general, programming languages impose restrictions on the ways in which computational elements can be manipulated. Elements with the fewest restrictions are said to have <a name="index_term_1218"></a>*first-class* status. Some of the "rights and privileges" of first-class elements are:<a name="call_footnote_Temp_121" href="#footnote_Temp_121" id="call_footnote_Temp_121"><sup><small>64</small></sup></a>

<ul>
  <li>They may be named by variables.</li>

  <li>They may be passed as arguments to procedures.</li>

  <li>They may be returned as the results of procedures.</li>

  <li>They may be included in data structures.<a name="call_footnote_Temp_122" href="#footnote_Temp_122" id="call_footnote_Temp_122"><sup><small>65</small></sup></a></li>
</ul>

<a name="index_term_1222"></a><a name="index_term_1224"></a>Lisp, unlike other common programming languages, awards procedures full first-class status. This poses challenges for efficient implementation, but the resulting gain in expressive power is enormous.<a name="call_footnote_Temp_123" href="#footnote_Temp_123" id="call_footnote_Temp_123"><sup><small>66</small></sup></a>


<a name="%_thm_1.40"></a> **Exercise 1.40.**&#160;&#160;Define a procedure <tt>cubic</tt> that can be used together with the <tt>newtons-method</tt> procedure in expressions of the form


<tt>(newtons-method&#160;(cubic&#160;a&#160;b&#160;c)&#160;1)<br></tt>


to approximate zeros of the cubic *x*<sup>3</sup> + *a**x*<sup>2</sup> + *b**x* + *c*.


<a name="%_thm_1.41"></a> **Exercise 1.41.**&#160;&#160;Define a procedure <tt>double</tt> that takes a procedure of one argument as argument and returns a procedure that applies the original procedure twice. For example, if <tt>inc</tt> is a procedure that adds 1 to its argument, then <tt>(double inc)</tt> should be a procedure that adds 2. What value is returned by


<tt>(((double&#160;(double&#160;double))&#160;inc)&#160;5)<br></tt>


<a name="%_thm_1.42"></a> **Exercise 1.42.**&#160;&#160;<a name="index_term_1226"></a><a name="index_term_1228"></a>Let *f* and *g* be two one-argument functions. The *composition* *f* after *g* is defined to be the function *x* <img src="img/book_17.gif" border="0"> *f*(*g*(*x*)). Define a procedure <tt>compose</tt> that implements composition. For example, if <tt>inc</tt> is a procedure that adds 1 to its argument,


<tt>((compose&#160;square&#160;inc)&#160;6)<br>
<i>49</i><br></tt>


<a name="%_thm_1.43"></a> **Exercise 1.43.**&#160;&#160;<a name="index_term_1230"></a>If *f* is a numerical function and *n* is a positive integer, then we can form the *n*th repeated application of *f*, which is defined to be the function whose value at *x* is *f*(*f*(<tt>...</tt>(*f*(*x*))<tt>...</tt>)). For example, if *f* is the function *x* <img src="img/book_17.gif" border="0"> *x* + 1, then the *n*th repeated application of *f* is the function *x* <img src="img/book_17.gif" border="0"> *x* + *n*. If *f* is the operation of squaring a number, then the *n*th repeated application of *f* is the function that raises its argument to the 2<sup>*n*</sup>th power. Write a procedure that takes as inputs a procedure that computes *f* and a positive integer *n* and returns the procedure that computes the *n*th repeated application of *f*. Your procedure should be able to be used as follows:


<tt>((repeated&#160;square&#160;2)&#160;5)<br>
<i>625</i><br></tt>


Hint: You may find it convenient to use <tt>compose</tt> from exercise&#160;<a href="#%_thm_1.42">1.42</a>.


<a name="%_thm_1.44"></a> **Exercise 1.44.**&#160;&#160;<a name="index_term_1232"></a><a name="index_term_1234"></a><a name="index_term_1236"></a>The idea of *smoothing* a function is an important concept in signal processing. If *f* is a function and *d**x* is some small number, then the smoothed version of *f* is the function whose value at a point *x* is the average of *f*(*x* - *d**x*), *f*(*x*), and *f*(*x* + *d**x*). Write a procedure <tt>smooth</tt> that takes as input a procedure that computes *f* and returns a procedure that computes the smoothed *f*. It is sometimes valuable to repeatedly smooth a function (that is, smooth the smoothed function, and so on) to obtained the **n*-fold smoothed function*. Show how to generate the *n*-fold smoothed function of any given function using <tt>smooth</tt> and <tt>repeated</tt> from exercise&#160;<a href="#%_thm_1.43">1.43</a>.


<a name="%_thm_1.45"></a> **Exercise 1.45.**&#160;&#160;We saw in section&#160;<a href="#%_sec_1.3.3">1.3.3</a> that attempting to compute square roots by naively finding a fixed point of *y* <img src="img/book_17.gif" border="0"> *x*/*y* does not converge, and that this can be fixed by average damping. The same method works for finding cube roots as fixed points of the average-damped *y* <img src="img/book_17.gif" border="0"> *x*/*y*<sup>2</sup>. Unfortunately, the process does not work for <a name="index_term_1238"></a><a name="index_term_1240"></a>fourth roots -- a single average damp is not enough to make a fixed-point search for *y* <img src="img/book_17.gif" border="0"> *x*/*y*<sup>3</sup> converge. On the other hand, if we average damp twice (i.e., use the average damp of the average damp of *y* <img src="img/book_17.gif" border="0"> *x*/*y*<sup>3</sup>) the fixed-point search does converge. Do some experiments to determine how many average damps are required to compute <a name="index_term_1242"></a><a name="index_term_1244"></a>*n*th roots as a fixed-point search based upon repeated average damping of *y* <img src="img/book_17.gif" border="0"> *x*/*y*<sup>*n*-1</sup>. Use this to implement a simple procedure for computing *n*th roots using <tt>fixed-point</tt>, <tt>average-damp</tt>, and the <tt>repeated</tt> procedure of exercise&#160;<a href="#%_thm_1.43">1.43</a>. Assume that any arithmetic operations you need are available as primitives.


<a name="%_thm_1.46"></a> **Exercise 1.46.**&#160;&#160;<a name="index_term_1246"></a><a name="index_term_1248"></a><a name="index_term_1250"></a><a name="index_term_1252"></a>Several of the numerical methods described in this chapter are instances of an extremely general computational strategy known as *iterative improvement*. Iterative improvement says that, to compute something, we start with an initial guess for the answer, test if the guess is good enough, and otherwise improve the guess and continue the process using the improved guess as the new guess. Write a procedure <tt>iterative-improve</tt> that takes two procedures as arguments: a method for telling whether a guess is good enough and a method for improving a guess. <tt>Iterative-improve</tt> should return as its value a procedure that takes a guess as argument and keeps improving the guess until it is good enough. Rewrite the <tt>sqrt</tt> procedure of section&#160;<a href="chapter_1_section_1.html#%_sec_1.1.7">1.1.7</a> and the <tt>fixed-point</tt> procedure of section&#160;<a href="#%_sec_1.3.3">1.3.3</a> in terms of <tt>iterative-improve</tt>.

<div class="smallprint">
  <hr>
</div>

<div class="footnote">
  
<a name="footnote_Temp_90" href="#call_footnote_Temp_90" id="footnote_Temp_90"><sup><small>49</small></sup></a> This series, <a name="index_term_0974"></a><a name="index_term_0976"></a>usually written in the equivalent form (<img src="img/book_09.gif" border="0">/4) = 1 - (1/3) + (1/5) - (1/7) + <tt>���</tt>, is due to Leibniz. We'll see how to use this as the basis for some fancy numerical tricks in section&#160;<a href="chapter_3_section_5.html#%_sec_3.5.3">3.5.3</a>.

  
<a name="footnote_Temp_91" href="#call_footnote_Temp_91" id="footnote_Temp_91"><sup><small>50</small></sup></a> Notice that we have used block structure (section&#160;<a href="chapter_1_section_1.html#%_sec_1.1.8">1.1.8</a>) to embed the definitions of <tt>pi-next</tt> and <tt>pi-term</tt> within <tt>pi-sum</tt>, since these procedures are unlikely to be useful for any other purpose. We will see how to get rid of them altogether in section&#160;<a href="#%_sec_1.3.2">1.3.2</a>.

  
<a name="footnote_Temp_95" href="#call_footnote_Temp_95" id="footnote_Temp_95"><sup><small>51</small></sup></a> The intent of exercises&#160;<a href="#%_thm_1.31">1.31</a>-<a href="#%_thm_1.33">1.33</a> is to demonstrate the expressive power that is attained by using an appropriate abstraction to consolidate many seemingly disparate operations. However, though accumulation and filtering are elegant ideas, our hands are somewhat tied in using them at this point since we do not yet have data structures to provide suitable means of combination for these abstractions. We will return to these ideas in section&#160;<a href="chapter_2_section_2.html#%_sec_2.2.3">2.2.3</a> when we show how to use *sequences* as interfaces for combining filters and accumulators to build even more powerful abstractions. We will see there how these methods really come into their own as a powerful and elegant approach to designing programs.

  
<a name="footnote_Temp_96" href="#call_footnote_Temp_96" id="footnote_Temp_96"><sup><small>52</small></sup></a> This formula was discovered by the seventeenth-century <a name="index_term_1016"></a>English mathematician John Wallis.

  
<a name="footnote_Temp_99" href="#call_footnote_Temp_99" id="footnote_Temp_99"><sup><small>53</small></sup></a> It would be clearer and less intimidating to people learning Lisp if a name more obvious than <tt>lambda</tt>, such as <tt>make-procedure</tt>, were used. But the convention is firmly entrenched. The notation is adopted from the <a name="index_term_1054"></a><img src="img/book_06.gif" border="0"> calculus, a <a name="index_term_1056"></a>mathematical formalism introduced by the mathematical logician Alonzo Church (1941). Church developed the <img src="img/book_06.gif" border="0"> calculus to provide a rigorous foundation for studying the notions of function and function application. The <img src="img/book_06.gif" border="0"> calculus has become a basic tool for mathematical investigations of the semantics of programming languages.

  
<a name="footnote_Temp_101" href="#call_footnote_Temp_101" id="footnote_Temp_101"><sup><small>54</small></sup></a> Understanding internal definitions well enough to be sure a program means what we intend it to mean requires a more elaborate model of the evaluation process than we have presented in this chapter. The subtleties do not arise with internal definitions of procedures, however. We will return to this issue in section&#160;<a href="chapter_4_section_1.html#%_sec_4.1.6">4.1.6</a>, after we learn more about evaluation.

  
<a name="footnote_Temp_104" href="#call_footnote_Temp_104" id="footnote_Temp_104"><sup><small>55</small></sup></a> We have used 0.001 as a representative "small" number to indicate a tolerance for the acceptable error in a calculation. The appropriate tolerance for a real calculation depends upon the problem to be solved and the limitations of the computer and the algorithm. This is often <a name="index_term_1086"></a>a very subtle consideration, requiring help from a numerical analyst or some other kind of magician.

  
<a name="footnote_Temp_105" href="#call_footnote_Temp_105" id="footnote_Temp_105"><sup><small>56</small></sup></a> This <a name="index_term_1088"></a><a name="index_term_1090"></a>can be accomplished using <tt>error</tt>, which takes as arguments a number of items that are printed as error messages.

  
<a name="footnote_Temp_107" href="#call_footnote_Temp_107" id="footnote_Temp_107"><sup><small>57</small></sup></a> Try this during a boring lecture: Set your calculator to <a name="index_term_1106"></a><a name="index_term_1108"></a><a name="index_term_1110"></a>radians mode and then repeatedly press the <tt>cos</tt> button until you obtain the fixed point.

  
<a name="footnote_Temp_108" href="#call_footnote_Temp_108" id="footnote_Temp_108"><sup><small>58</small></sup></a> <img src="img/book_17.gif" border="0"> <a name="index_term_1122"></a><a name="index_term_1124"></a>(pronounced "maps to") is the mathematician's way of writing <tt>lambda</tt>. *y* <img src="img/book_17.gif" border="0"> *x*/*y* means <tt>(lambda(y) (/ x y))</tt>, that is, the function whose value at *y* is *x*/*y*.

  
<a name="footnote_Temp_114" href="#call_footnote_Temp_114" id="footnote_Temp_114"><sup><small>59</small></sup></a> Observe that this is a combination whose operator is itself <a name="index_term_1162"></a><a name="index_term_1164"></a><a name="index_term_1166"></a>a combination. Exercise&#160;<a href="chapter_1_section_1.html#exercise_1_4">1.4</a> already demonstrated the ability to form such combinations, but that was only a toy example. Here we begin to see the real need for such combinations -- when applying a procedure that is obtained as the value returned by a higher-order procedure.

  
<a name="footnote_Temp_115" href="#call_footnote_Temp_115" id="footnote_Temp_115"><sup><small>60</small></sup></a> See exercise&#160;<a href="#%_thm_1.45">1.45</a> for a further generalization.

  
<a name="footnote_Temp_117" href="#call_footnote_Temp_117" id="footnote_Temp_117"><sup><small>61</small></sup></a> Elementary calculus books usually describe Newton's method in terms of the sequence of approximations *x*<sub>*n*+1</sub> = *x*<sub>*n*</sub> - *g*(*x*<sub>*n*</sub>)/*D**g*(*x*<sub>*n*</sub>). Having language for talking about processes and using the idea of fixed points simplifies the description of the method.

  
<a name="footnote_Temp_118" href="#call_footnote_Temp_118" id="footnote_Temp_118"><sup><small>62</small></sup></a> Newton's method does not always converge to an answer, but it can be shown that in favorable cases each iteration doubles the number-of-digits accuracy of the approximation to the solution. In such cases, <a name="index_term_1182"></a><a name="index_term_1184"></a>Newton's method will converge much more rapidly than the half-interval method.

  
<a name="footnote_Temp_119" href="#call_footnote_Temp_119" id="footnote_Temp_119"><sup><small>63</small></sup></a> For finding square roots, Newton's method converges rapidly to the correct solution from any starting point.

  
<a name="footnote_Temp_121" href="#call_footnote_Temp_121" id="footnote_Temp_121"><sup><small>64</small></sup></a> The notion of first-class status of programming-language <a name="index_term_1220"></a>elements is due to the British computer scientist Christopher Strachey (1916-1975).

  
<a name="footnote_Temp_122" href="#call_footnote_Temp_122" id="footnote_Temp_122"><sup><small>65</small></sup></a> We'll see examples of this after we introduce data structures in chapter&#160;2.

  
<a name="footnote_Temp_123" href="#call_footnote_Temp_123" id="footnote_Temp_123"><sup><small>66</small></sup></a> The major implementation cost of first-class procedures is that allowing procedures to be returned as values requires reserving storage for a procedure's free variables even while the procedure is not executing. In the Scheme implementation we will study in section&#160;<a href="chapter_4_section_1.html#%_sec_4.1">4.1</a>, these variables are stored in the procedure's environment.

</div>