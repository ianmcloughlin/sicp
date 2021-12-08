(section_21)=
# Introduction to Data Abstraction

In section <a href="chapter_1_section_1.html#%_sec_1.1.8">1.1.8</a>, we noted that a procedure used as an element in creating a more complex procedure could be regarded not only as a collection of particular operations but also as a procedural abstraction. That is, the details of how the procedure was implemented could be suppressed, and the particular procedure itself could be replaced by any other procedure with the same overall behavior. In other words, we could make an abstraction that would separate the way the procedure would be used from the details of how the procedure would be implemented in terms of more primitive procedures. The analogous notion for compound data is called <a name="index_term_1280"></a>*data abstraction*. Data abstraction is a methodology that enables us to isolate how a compound data object is used from the details of how it is constructed from more primitive data objects.


The basic idea of data abstraction is to structure the programs that are to use compound data objects so that they operate on <a name="index_term_1282"></a><a name="index_term_1284"></a>"abstract data." That is, our programs should use data in such a way as to make no assumptions about the data that are not strictly necessary for performing the task at hand. At the same time, a <a name="index_term_1286"></a><a name="index_term_1288"></a>"concrete" data representation is defined independent of the programs that use the data. The interface between these two parts of our system will be a set of procedures, called <a name="index_term_1290"></a>*selectors* and <a name="index_term_1292"></a>*constructors*, that implement the abstract data in terms of the concrete representation. To illustrate this technique, we will consider how to design a set of procedures for manipulating rational numbers.


<a name="%_sec_2.1.1"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_2.1.1">2.1.1&nbsp;&nbsp;Example: Arithmetic Operations for Rational Numbers</a></h3>

<a name="index_term_1294"></a><a name="index_term_1296"></a><a name="index_term_1298"></a>Suppose we want to do arithmetic with rational numbers. We want to be able to add, subtract, multiply, and divide them and to test whether two rational numbers are equal.


Let us begin by assuming that we already have a way of constructing a rational number from a numerator and a denominator. We also assume that, given a rational number, we have a way of extracting (or selecting) its numerator and its denominator. Let us further assume that the constructor and selectors are available as procedures:

<ul>
  <li style="list-style: none"><a name="index_term_1300"></a></li>

  <li>
    <tt>(make-rat &lt;*n*&gt; &lt;*d*&gt;)</tt> returns the rational number whose numerator is the integer <tt>&lt;*n*&gt;</tt> and whose denominator is the integer <tt>&lt;*d*&gt;</tt>.

    
<a name="index_term_1302"></a>

  </li>

  <li>
    <tt>(numer &lt;*x*&gt;)</tt> returns the numerator of the rational number <tt>&lt;*x*&gt;</tt>.

    
<a name="index_term_1304"></a>

  </li>

  <li><tt>(denom &lt;*x*&gt;)</tt> returns the denominator of the rational number <tt>&lt;*x*&gt;</tt>.</li>
</ul>

We are using here a powerful strategy of synthesis: <a name="index_term_1306"></a>*wishful thinking*. We haven't yet said how a rational number is represented, or how the procedures <tt>numer</tt>, <tt>denom</tt>, and <tt>make-rat</tt> should be implemented. Even so, if we did have these three procedures, we could then add, subtract, multiply, divide, and test equality by using the following relations:

<div align="left">
  <img src="img/chapter_2_image_01.gif" border="0">
</div>

<div align="left">
  <img src="img/chapter_2_image_02.gif" border="0">
</div>

<div align="left">
  <img src="img/chapter_2_image_03.gif" border="0">
</div>

<div align="left">
  <img src="img/chapter_2_image_04.gif" border="0">
</div>

<div align="left">
  <img src="img/chapter_2_image_05.gif" border="0">
</div>

We can express these rules as procedures:


<tt><a name="index_term_1308"></a>(define&nbsp;(add-rat&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;(make-rat&nbsp;(+&nbsp;(*&nbsp;(numer&nbsp;x)&nbsp;(denom&nbsp;y))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;(numer&nbsp;y)&nbsp;(denom&nbsp;x)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;(denom&nbsp;x)&nbsp;(denom&nbsp;y))))<br>
<a name="index_term_1310"></a>(define&nbsp;(sub-rat&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;(make-rat&nbsp;(-&nbsp;(*&nbsp;(numer&nbsp;x)&nbsp;(denom&nbsp;y))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;(numer&nbsp;y)&nbsp;(denom&nbsp;x)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;(denom&nbsp;x)&nbsp;(denom&nbsp;y))))<br>
<a name="index_term_1312"></a>(define&nbsp;(mul-rat&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;(make-rat&nbsp;(*&nbsp;(numer&nbsp;x)&nbsp;(numer&nbsp;y))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;(denom&nbsp;x)&nbsp;(denom&nbsp;y))))<br>
<a name="index_term_1314"></a>(define&nbsp;(div-rat&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;(make-rat&nbsp;(*&nbsp;(numer&nbsp;x)&nbsp;(denom&nbsp;y))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;(denom&nbsp;x)&nbsp;(numer&nbsp;y))))<br>
<a name="index_term_1316"></a>(define&nbsp;(equal-rat?&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;(=&nbsp;(*&nbsp;(numer&nbsp;x)&nbsp;(denom&nbsp;y))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;(numer&nbsp;y)&nbsp;(denom&nbsp;x))))<br></tt>


Now we have the operations on rational numbers defined in terms of the selector and constructor procedures <tt>numer</tt>, <tt>denom</tt>, and <tt>make-rat</tt>. But we haven't yet defined these. What we need is some way to glue together a numerator and a denominator to form a rational number.


<a name="%_sec_Temp_132"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_132">Pairs</a></h4>

To enable us to implement the concrete level of our data abstraction, our language provides a compound structure called a <a name="index_term_1318"></a>*pair*, which can be constructed with the primitive procedure <a name="index_term_1320"></a><a name="index_term_1322"></a><tt>cons</tt>. This procedure takes two arguments and returns a compound data object that contains the two arguments as parts. Given a pair, we can extract the parts using the primitive procedures <a name="index_term_1324"></a><a name="index_term_1326"></a><tt>car</tt> and <a name="index_term_1328"></a><a name="index_term_1330"></a><tt>cdr</tt>.<a name="call_footnote_Temp_133" href="#footnote_Temp_133" id="call_footnote_Temp_133"><sup><small>2</small></sup></a> Thus, we can use <tt>cons</tt>, <tt>car</tt>, and <tt>cdr</tt> as follows:


<tt>(define&nbsp;x&nbsp;(cons&nbsp;1&nbsp;2))<br>
<br>
(car&nbsp;x)<br>
<i>1</i><br>
<br>
(cdr&nbsp;x)<br>
<i>2</i><br></tt>


Notice that a pair is a data object that can be given a name and manipulated, just like a primitive data object. Moreover, <tt>cons</tt> can be used to form pairs whose elements are pairs, and so on:


<tt>(define&nbsp;x&nbsp;(cons&nbsp;1&nbsp;2))<br>
<br>
(define&nbsp;y&nbsp;(cons&nbsp;3&nbsp;4))<br>
<br>
(define&nbsp;z&nbsp;(cons&nbsp;x&nbsp;y))<br>
<br>
(car&nbsp;(car&nbsp;z))<br>
<i>1</i><br>
<br>
(car&nbsp;(cdr&nbsp;z))<br>
<i>3</i><br></tt>


In section <a href="chapter_2_section_2.html#%_sec_2.2">2.2</a> we will see how this ability to combine pairs means that pairs can be used as general-purpose building blocks to create all sorts of complex data structures. The single compound-data primitive *pair*, implemented by the procedures <tt>cons</tt>, <tt>car</tt>, and <tt>cdr</tt>, is the only glue we need. Data objects constructed from pairs are called <a name="index_term_1342"></a><a name="index_term_1344"></a>*list-structured* data.


<a name="%_sec_Temp_134"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_134">Representing rational numbers</a></h4>

<a name="index_term_1346"></a>Pairs offer a natural way to complete the rational-number system. Simply represent a rational number as a pair of two integers: a numerator and a denominator. Then <tt>make-rat</tt>, <tt>numer</tt>, and <tt>denom</tt> are readily implemented as follows:<a name="call_footnote_Temp_135" href="#footnote_Temp_135" id="call_footnote_Temp_135"><sup><small>3</small></sup></a>


<tt><a name="index_term_1348"></a>(define&nbsp;(make-rat&nbsp;n&nbsp;d)&nbsp;(cons&nbsp;n&nbsp;d))<br>
<br>
<a name="index_term_1350"></a>(define&nbsp;(numer&nbsp;x)&nbsp;(car&nbsp;x))<br>
<br>
<a name="index_term_1352"></a>(define&nbsp;(denom&nbsp;x)&nbsp;(cdr&nbsp;x))<br></tt>


Also, in order to display the results of our computations, we can <a name="index_term_1354"></a>print rational numbers by printing the numerator, a slash, and the denominator:<a name="call_footnote_Temp_136" href="#footnote_Temp_136" id="call_footnote_Temp_136"><sup><small>4</small></sup></a>


<tt><a name="index_term_1370"></a>(define&nbsp;(print-rat&nbsp;x)<br>
&nbsp;&nbsp;(newline)<br>
&nbsp;&nbsp;(display&nbsp;(numer&nbsp;x))<br>
&nbsp;&nbsp;(display&nbsp;&quot;/&quot;)<br>
&nbsp;&nbsp;(display&nbsp;(denom&nbsp;x)))<br></tt>


Now we can try our rational-number procedures:


<tt>(define&nbsp;one-half&nbsp;(make-rat&nbsp;1&nbsp;2))<br>
<br>
(print-rat&nbsp;one-half)<br>
<i>1/2</i><br>
<br>
(define&nbsp;one-third&nbsp;(make-rat&nbsp;1&nbsp;3))<br>
(print-rat&nbsp;(add-rat&nbsp;one-half&nbsp;one-third))<br>
<i>5/6</i><br>
<br>
(print-rat&nbsp;(mul-rat&nbsp;one-half&nbsp;one-third))<br>
<i>1/6</i><br>
<br>
(print-rat&nbsp;(add-rat&nbsp;one-third&nbsp;one-third))<br>
<i>6/9</i><br></tt>


<a name="index_term_1372"></a><a name="index_term_1374"></a>As the final example shows, our rational-number implementation does not reduce rational numbers to lowest terms. We can remedy this by changing <tt>make-rat</tt>. If we have a <a name="index_term_1376"></a><tt>gcd</tt> procedure like the one in section <a href="chapter_1_section_2.html#%_sec_1.2.5">1.2.5</a> that produces the greatest common divisor of two integers, we can use <tt>gcd</tt> to reduce the numerator and the denominator to lowest terms before constructing the pair:


<tt><a name="index_term_1378"></a>(define&nbsp;(make-rat&nbsp;n&nbsp;d)<br>
&nbsp;&nbsp;(let&nbsp;((g&nbsp;(gcd&nbsp;n&nbsp;d)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;(/&nbsp;n&nbsp;g)&nbsp;(/&nbsp;d&nbsp;g))))<br></tt>


Now we have


<tt>(print-rat&nbsp;(add-rat&nbsp;one-third&nbsp;one-third))<br>
<i>2/3</i><br></tt>


as desired. This modification was accomplished by changing the constructor <tt>make-rat</tt> without changing any of the procedures (such as <tt>add-rat</tt> and <tt>mul-rat</tt>) that implement the actual operations.


<a name="exercise_2_1">**Exercise 2.1**</a> Define a better version of <tt>make-rat</tt> that handles both positive and negative arguments. <tt>Make-rat</tt> should normalize the sign so that if the rational number is positive, both the numerator and denominator are positive, and if the rational number is negative, only the numerator is negative.


<a name="%_sec_2.1.2"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_2.1.2">2.1.2&nbsp;&nbsp;Abstraction Barriers</a></h3>

<a name="index_term_1380"></a>Before continuing with more examples of compound data and data abstraction, let us consider some of the issues raised by the rational-number example. We defined the rational-number operations in terms of a constructor <tt>make-rat</tt> and selectors <tt>numer</tt> and <tt>denom</tt>. In general, the underlying idea of data abstraction is to identify for each type of data object a basic set of operations in terms of which all manipulations of data objects of that type will be expressed, and then to use only those operations in manipulating the data.


We can envision the structure of the rational-number system as shown in figure&nbsp;<a href="#%_fig_2.1">2.1</a>. The horizontal lines represent *abstraction barriers* that isolate different "levels" of the system. At each level, the barrier separates the programs (above) that use the data abstraction from the programs (below) that implement the data abstraction. Programs that use rational numbers manipulate them solely in terms of the procedures supplied "for public use" by the rational-number package: <tt>add-rat</tt>, <tt>sub-rat</tt>, <tt>mul-rat</tt>, <tt>div-rat</tt>, and <tt>equal-rat?</tt>. These, in turn, are implemented solely in terms of the <a name="index_term_1382"></a><a name="index_term_1384"></a>constructor and selectors <tt>make-rat</tt>, <tt>numer</tt>, and <tt>denom</tt>, which themselves are implemented in terms of pairs. The details of how pairs are implemented are irrelevant to the rest of the rational-number package so long as pairs can be manipulated by the use of <tt>cons</tt>, <tt>car</tt>, and <tt>cdr</tt>. In effect, procedures at each level are the interfaces that define the abstraction barriers and connect the different levels.


<a name="%_fig_2.1"></a>

<div align="left">
  <div align="left">
    **Figure 2.1:**&nbsp;&nbsp;Data-abstraction barriers in the rational-number package.
  </div>

  <table width="100%">
    <tr>
      <td>
        <div align="left">
          <img src="img/chapter_2_image_06.gif" border="0">&nbsp;
        </div>
      </td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

This simple idea has many advantages. One advantage is that it makes programs much easier to maintain and to modify. Any complex data structure can be represented in a variety of ways with the primitive data structures provided by a programming language. Of course, the choice of representation influences the programs that operate on it; thus, if the representation were to be changed at some later time, all such programs might have to be modified accordingly. This task could be time-consuming and expensive in the case of large programs unless the dependence on the representation were to be confined by design to a very few program modules.


<a name="index_term_1386"></a><a name="index_term_1388"></a>For example, an alternate way to address the problem of reducing rational numbers to lowest terms is to perform the reduction whenever we access the parts of a rational number, rather than when we construct it. This leads to different constructor and selector procedures:


<tt><a name="index_term_1390"></a>(define&nbsp;(make-rat&nbsp;n&nbsp;d)<br>
&nbsp;&nbsp;(cons&nbsp;n&nbsp;d))<br>
<a name="index_term_1392"></a>(define&nbsp;(numer&nbsp;x)<br>
&nbsp;&nbsp;(let&nbsp;((g&nbsp;(gcd&nbsp;(car&nbsp;x)&nbsp;(cdr&nbsp;x))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;(/&nbsp;(car&nbsp;x)&nbsp;g)))<br>
<a name="index_term_1394"></a>(define&nbsp;(denom&nbsp;x)<br>
&nbsp;&nbsp;(let&nbsp;((g&nbsp;(gcd&nbsp;(car&nbsp;x)&nbsp;(cdr&nbsp;x))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;(/&nbsp;(cdr&nbsp;x)&nbsp;g)))<br></tt>


The difference between this implementation and the previous one lies in when we compute the <tt>gcd</tt>. If in our typical use of rational numbers we access the numerators and denominators of the same rational numbers many times, it would be preferable to compute the <tt>gcd</tt> when the rational numbers are constructed. If not, we may be better off waiting until access time to compute the <tt>gcd</tt>. In any case, when we change from one representation to the other, the procedures <tt>add-rat</tt>, <tt>sub-rat</tt>, and so on do not have to be modified at all.


Constraining the dependence on the representation to a few interface procedures helps us design programs as well as modify them, because it allows us to maintain the flexibility to consider alternate implementations. To continue with our simple example, suppose we are designing a rational-number package and we can't decide initially whether to perform the <tt>gcd</tt> at construction time or at selection time. The data-abstraction methodology gives us a way to defer that decision without losing the ability to make progress on the rest of the system.


<a name="exercise_2_2">**Exercise 2.2**</a> Consider the problem of representing <a name="index_term_1396"></a>line segments in a plane. Each segment is represented as a pair of points: a starting point and an ending point. Define a constructor <a name="index_term_1398"></a><tt>make-segment</tt> and selectors <a name="index_term_1400"></a><tt>start-segment</tt> and <a name="index_term_1402"></a><tt>end-segment</tt> that define the representation of segments in terms of points. Furthermore, a point <a name="index_term_1404"></a>can be represented as a pair of numbers: the *x* coordinate and the *y* coordinate. Accordingly, specify a constructor <a name="index_term_1406"></a><tt>make-point</tt> and selectors <tt>x-point</tt> and <tt>y-point</tt> that define this representation. Finally, using your selectors and constructors, define a procedure <a name="index_term_1408"></a><tt>midpoint-segment</tt> that takes a line segment as argument and returns its midpoint (the point whose coordinates are the average of the coordinates of the endpoints). To try your procedures, you'll need a way to print points:


<tt><a name="index_term_1410"></a>(define&nbsp;(print-point&nbsp;p)<br>
&nbsp;&nbsp;(newline)<br>
&nbsp;&nbsp;(display&nbsp;&quot;(&quot;)<br>
&nbsp;&nbsp;(display&nbsp;(x-point&nbsp;p))<br>
&nbsp;&nbsp;(display&nbsp;&quot;,&quot;)<br>
&nbsp;&nbsp;(display&nbsp;(y-point&nbsp;p))<br>
&nbsp;&nbsp;(display&nbsp;&quot;)&quot;))<br></tt>


<a name="exercise_2_3">**Exercise 2.3**</a> <a name="index_term_1412"></a>Implement a representation for rectangles in a plane. (Hint: You may want to make use of exercise&nbsp;<a href="#%_thm_2.2">2.2</a>.) In terms of your constructors and selectors, create procedures that compute the perimeter and the area of a given rectangle. Now implement a different representation for rectangles. Can you design your system with suitable abstraction barriers, so that the same perimeter and area procedures will work using either representation?


<a name="%_sec_2.1.3"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_2.1.3">2.1.3&nbsp;&nbsp;What Is Meant by Data?</a></h3>

<a name="index_term_1414"></a> We began the rational-number implementation in section <a href="#%_sec_2.1.1">2.1.1</a> by implementing the rational-number operations <tt>add-rat</tt>, <tt>sub-rat</tt>, and so on in terms of three unspecified procedures: <tt>make-rat</tt>, <tt>numer</tt>, and <tt>denom</tt>. At that point, we could think of the operations as being defined in terms of data objects -- numerators, denominators, and rational numbers -- whose behavior was specified by the latter three procedures.


But exactly what is meant by *data*? It is not enough to say "whatever is implemented by the given selectors and constructors." Clearly, not every arbitrary set of three procedures can serve as an appropriate basis for the rational-number implementation. We need to guarantee that, <a name="index_term_1416"></a><a name="index_term_1418"></a><a name="index_term_1420"></a>if we construct a rational number <tt>x</tt> from a pair of integers <tt>n</tt> and <tt>d</tt>, then extracting the <tt>numer</tt> and the <tt>denom</tt> of <tt>x</tt> and dividing them should yield the same result as dividing <tt>n</tt> by <tt>d</tt>. In other words, <tt>make-rat</tt>, <tt>numer</tt>, and <tt>denom</tt> must satisfy the condition that, for any integer <tt>n</tt> and any non-zero integer <tt>d</tt>, if <tt>x</tt> is (<tt>make-rat n d</tt>), then

<div align="left">
  <img src="img/chapter_2_image_07.gif" border="0">
</div>

In fact, this is the only condition <tt>make-rat</tt>, <tt>numer</tt>, and <tt>denom</tt> must fulfill in order to form a suitable basis for a rational-number representation. In general, we can think of data as defined by some collection of selectors and constructors, together with specified conditions that these procedures must fulfill in order to be a valid representation.<a name="call_footnote_Temp_140" href="#footnote_Temp_140" id="call_footnote_Temp_140"><sup><small>5</small></sup></a>


<a name="index_term_1446"></a><a name="index_term_1448"></a>This point of view can serve to define not only "high-level" data objects, such as rational numbers, but lower-level objects as well. <a name="index_term_1450"></a>Consider the notion of a pair, which we used in order to define our rational numbers. We never actually said what a pair was, only that the language supplied procedures <tt>cons</tt>, <tt>car</tt>, and <tt>cdr</tt> for operating on pairs. But the only thing we need to know about these three operations <a name="index_term_1452"></a><a name="index_term_1454"></a><a name="index_term_1456"></a><a name="index_term_1458"></a>is that if we glue two objects together using <tt>cons</tt> we can retrieve the objects using <tt>car</tt> and <tt>cdr</tt>. That is, the operations satisfy the condition that, for any objects <tt>x</tt> and <tt>y</tt>, if <tt>z</tt> is <tt>(cons x y)</tt> then <tt>(car z)</tt> is <tt>x</tt> and <tt>(cdr z)</tt> is <tt>y</tt>. Indeed, we mentioned that these three procedures are included as primitives in our language. However, any triple of procedures that satisfies the above condition can be used as the basis for implementing pairs. This point is illustrated strikingly by the fact that we could implement <tt>cons</tt>, <tt>car</tt>, and <tt>cdr</tt> without using any data structures at all but only using procedures. Here are the definitions:


<tt><a name="index_term_1460"></a>(define&nbsp;(cons&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;(define&nbsp;(dispatch&nbsp;m)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(cond&nbsp;((=&nbsp;m&nbsp;0)&nbsp;x)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;((=&nbsp;m&nbsp;1)&nbsp;y)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(else&nbsp;(error&nbsp;&quot;Argument&nbsp;not&nbsp;0&nbsp;or&nbsp;1&nbsp;--&nbsp;CONS&quot;&nbsp;m))))<br>
&nbsp;&nbsp;dispatch)<br>
<br>
<a name="index_term_1462"></a>(define&nbsp;(car&nbsp;z)&nbsp;(z&nbsp;0))<br>
<br>
<a name="index_term_1464"></a>(define&nbsp;(cdr&nbsp;z)&nbsp;(z&nbsp;1))<br></tt>


This use of procedures corresponds to nothing like our intuitive notion of what data should be. Nevertheless, all we need to do to show that this is a valid way to represent pairs is to verify that these procedures satisfy the condition given above.


The subtle point to notice is that the value returned by <tt>(cons x y)</tt> is a procedure -- namely the internally defined procedure <tt>dispatch</tt>, which takes one argument and returns either <tt>x</tt> or <tt>y</tt> depending on whether the argument is 0 or 1. Correspondingly, <tt>(car z)</tt> is defined to apply <tt>z</tt> to 0. Hence, if <tt>z</tt> is the procedure formed by <tt>(cons x y)</tt>, then <tt>z</tt> applied to 0 will yield <tt>x</tt>. Thus, we have shown that <tt>(car (cons x y))</tt> yields <tt>x</tt>, as desired. Similarly, <tt>(cdr (cons x y))</tt> applies the procedure returned by <tt>(cons x y)</tt> to 1, which returns <tt>y</tt>. Therefore, this procedural implementation of pairs is a valid implementation, and if we access pairs using only <tt>cons</tt>, <tt>car</tt>, and <tt>cdr</tt> we cannot distinguish this implementation from one that uses "real" data structures.


The point of exhibiting the procedural representation of pairs is not that our language works this way (Scheme, and Lisp systems in general, implement pairs directly, for efficiency reasons) but that it could work this way. The procedural representation, although obscure, is a perfectly adequate way to represent pairs, since it fulfills the only conditions that pairs need to fulfill. This example also demonstrates that the ability to manipulate procedures as objects automatically provides the ability to represent compound data. This may seem a curiosity now, but procedural representations of data will play a central role in our programming repertoire. This style of programming is often called <a name="index_term_1466"></a>*message passing*, and we will be using it as a basic tool in chapter&nbsp;3 when we address the issues of modeling and simulation.


<a name="exercise_2_4">**Exercise 2.4**</a> Here is an alternative procedural representation of pairs. For this representation, verify that <tt>(car (cons x y))</tt> yields <tt>x</tt> for any objects <tt>x</tt> and <tt>y</tt>.


<tt><a name="index_term_1468"></a>(define&nbsp;(cons&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;(lambda&nbsp;(m)&nbsp;(m&nbsp;x&nbsp;y)))<br>
<br>
<a name="index_term_1470"></a>(define&nbsp;(car&nbsp;z)<br>
&nbsp;&nbsp;(z&nbsp;(lambda&nbsp;(p&nbsp;q)&nbsp;p)))<br></tt>


<a name="index_term_1472"></a>What is the corresponding definition of <tt>cdr</tt>? (Hint: To verify that this works, make use of the substitution model of section <a href="chapter_1_section_1.html#%_sec_1.1.5">1.1.5</a>.)


<a name="exercise_2_5">**Exercise 2.5**</a> Show that we can represent pairs of nonnegative integers using only numbers and arithmetic operations if we represent the pair *a* and *b* as the integer that is the product 2<sup>*a*</sup> 3<sup>*b*</sup>. Give the corresponding definitions of the procedures <tt>cons</tt>, <tt>car</tt>, and <tt>cdr</tt>.


<a name="exercise_2_6">**Exercise 2.6**</a> In case representing pairs as procedures wasn't mind-boggling enough, consider that, in a language that can manipulate procedures, we can get by without numbers (at least insofar as nonnegative integers are concerned) by implementing 0 and the operation of adding 1 as


<tt>(define&nbsp;zero&nbsp;(lambda&nbsp;(f)&nbsp;(lambda&nbsp;(x)&nbsp;x)))<br>
<br>
(define&nbsp;(add-1&nbsp;n)<br>
&nbsp;&nbsp;(lambda&nbsp;(f)&nbsp;(lambda&nbsp;(x)&nbsp;(f&nbsp;((n&nbsp;f)&nbsp;x)))))<br></tt>


This representation is known as <a name="index_term_1474"></a>*Church numerals*, after its inventor, <a name="index_term_1476"></a>Alonzo Church, the logician who invented the <img src="img/book_06.gif" border="0"> calculus.


Define <tt>one</tt> and <tt>two</tt> directly (not in terms of <tt>zero</tt> and <tt>add-1</tt>). (Hint: Use substitution to evaluate <tt>(add-1 zero)</tt>). Give a direct definition of the addition procedure <tt>+</tt> (not in terms of repeated application of <tt>add-1</tt>).


<a name="%_sec_2.1.4"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_2.1.4">2.1.4&nbsp;&nbsp;Extended Exercise: Interval Arithmetic</a></h3>

<a name="index_term_1478"></a><a name="index_term_1480"></a> Alyssa P. Hacker is designing a system to help people solve engineering problems. One feature she wants to provide in her system is the ability to manipulate inexact quantities (such as measured parameters of physical devices) with known precision, so that when computations are done with such approximate quantities the results will be numbers of known precision.


Electrical engineers will be using Alyssa's system to compute electrical quantities. It is sometimes necessary for them to compute the value of a parallel equivalent resistance *R*<sub>*p*</sub> of two resistors *R*<sub>1</sub> and *R*<sub>2</sub> using the formula <a name="index_term_1482"></a>

<div align="left">
  <img src="img/chapter_2_image_08.gif" border="0">
</div>

Resistance values are usually known only up to some <a name="index_term_1484"></a>tolerance guaranteed by the manufacturer of the resistor. For example, if you buy a resistor labeled "6.8 ohms with 10% tolerance" you can only be sure that the resistor has a resistance between 6.8 - 0.68 = 6.12 and 6.8 + 0.68 = 7.48 ohms. Thus, if you have a 6.8-ohm 10% resistor in parallel with a 4.7-ohm 5% resistor, the resistance of the combination can range from about 2.58 ohms (if the two resistors are at the lower bounds) to about 2.97 ohms (if the two resistors are at the upper bounds).


Alyssa's idea is to implement "interval arithmetic" as a set of arithmetic operations for combining "intervals" (objects that represent the range of possible values of an inexact quantity). The result of adding, subtracting, multiplying, or dividing two intervals is itself an interval, representing the range of the result.


Alyssa postulates the existence of an abstract object called an "interval" that has two endpoints: a lower bound and an upper bound. She also presumes that, given the endpoints of an interval, she can construct the interval using the data constructor <a name="index_term_1486"></a><tt>make-interval</tt>. Alyssa first writes a procedure for adding two intervals. She reasons that the minimum value the sum could be is the sum of the two lower bounds and the maximum value it could be is the sum of the two upper bounds:


<tt><a name="index_term_1488"></a>(define&nbsp;(add-interval&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;(make-interval&nbsp;(+&nbsp;(lower-bound&nbsp;x)&nbsp;(lower-bound&nbsp;y))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(+&nbsp;(upper-bound&nbsp;x)&nbsp;(upper-bound&nbsp;y))))<br></tt>


Alyssa also works out the product of two intervals by finding the minimum and the maximum of the products of the bounds and using them as the bounds of the resulting interval. (<tt>Min</tt> and <tt>max</tt> are <a name="index_term_1490"></a><a name="index_term_1492"></a><a name="index_term_1494"></a><a name="index_term_1496"></a>primitives that find the minimum or maximum of any number of arguments.)


<tt><a name="index_term_1498"></a>(define&nbsp;(mul-interval&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;(let&nbsp;((p1&nbsp;(*&nbsp;(lower-bound&nbsp;x)&nbsp;(lower-bound&nbsp;y)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(p2&nbsp;(*&nbsp;(lower-bound&nbsp;x)&nbsp;(upper-bound&nbsp;y)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(p3&nbsp;(*&nbsp;(upper-bound&nbsp;x)&nbsp;(lower-bound&nbsp;y)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(p4&nbsp;(*&nbsp;(upper-bound&nbsp;x)&nbsp;(upper-bound&nbsp;y))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;(make-interval&nbsp;(min&nbsp;p1&nbsp;p2&nbsp;p3&nbsp;p4)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(max&nbsp;p1&nbsp;p2&nbsp;p3&nbsp;p4))))<br></tt>


To divide two intervals, Alyssa multiplies the first by the reciprocal of the second. Note that the bounds of the reciprocal interval are the reciprocal of the upper bound and the reciprocal of the lower bound, in that order.


<tt><a name="index_term_1500"></a>(define&nbsp;(div-interval&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;(mul-interval&nbsp;x&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-interval&nbsp;(/&nbsp;1.0&nbsp;(upper-bound&nbsp;y))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(/&nbsp;1.0&nbsp;(lower-bound&nbsp;y)))))<br></tt>


<a name="exercise_2_7">**Exercise 2.7**</a> Alyssa's program is incomplete because she has not specified the implementation of the interval abstraction. Here is a definition of the interval constructor:


<tt><a name="index_term_1502"></a>(define&nbsp;(make-interval&nbsp;a&nbsp;b)&nbsp;(cons&nbsp;a&nbsp;b))<br></tt>


Define selectors <a name="index_term_1504"></a><tt>upper-bound</tt> and <a name="index_term_1506"></a><tt>lower-bound</tt> to complete the implementation.


<a name="exercise_2_8">**Exercise 2.8**</a> Using reasoning analogous to Alyssa's, describe how the difference of two intervals may be computed. Define a corresponding subtraction procedure, called <a name="index_term_1508"></a><tt>sub-interval</tt>.


<a name="exercise_2_9">**Exercise 2.9**</a> <a name="index_term_1510"></a>The *width* of an interval is half of the difference between its upper and lower bounds. The width is a measure of the uncertainty of the number specified by the interval. For some arithmetic operations the width of the result of combining two intervals is a function only of the widths of the argument intervals, whereas for others the width of the combination is not a function of the widths of the argument intervals. Show that the width of the sum (or difference) of two intervals is a function only of the widths of the intervals being added (or subtracted). Give examples to show that this is not true for multiplication or division.


<a name="exercise_2_10">**Exercise 2.10**</a> <a name="index_term_1512"></a>Ben Bitdiddle, an expert systems programmer, looks over Alyssa's shoulder and comments that it is not clear what it means to divide by an interval that spans zero. Modify Alyssa's code to check for this condition and to signal an error if it occurs.


<a name="exercise_2_11">**Exercise 2.11**</a> <a name="index_term_1514"></a>In passing, Ben also cryptically comments: "By testing the signs of the endpoints of the intervals, it is possible to break <tt>mul-interval</tt> into nine cases, only one of which requires more than two multiplications." Rewrite this procedure using Ben's suggestion.


After debugging her program, Alyssa shows it to a potential user, who complains that her program solves the wrong problem. He wants a program that can deal with numbers represented as a center value and an additive tolerance; for example, he wants to work with intervals such as 3.5� 0.15 rather than [3.35, 3.65]. Alyssa returns to her desk and fixes this problem by supplying an alternate constructor and alternate selectors:


<tt><a name="index_term_1516"></a>(define&nbsp;(make-center-width&nbsp;c&nbsp;w)<br>
&nbsp;&nbsp;(make-interval&nbsp;(-&nbsp;c&nbsp;w)&nbsp;(+&nbsp;c&nbsp;w)))<br>
<a name="index_term_1518"></a>(define&nbsp;(center&nbsp;i)<br>
&nbsp;&nbsp;(/&nbsp;(+&nbsp;(lower-bound&nbsp;i)&nbsp;(upper-bound&nbsp;i))&nbsp;2))<br>
<a name="index_term_1520"></a>(define&nbsp;(width&nbsp;i)<br>
&nbsp;&nbsp;(/&nbsp;(-&nbsp;(upper-bound&nbsp;i)&nbsp;(lower-bound&nbsp;i))&nbsp;2))<br></tt>


Unfortunately, most of Alyssa's users are engineers. Real engineering situations usually involve measurements with only a small uncertainty, measured as the ratio of the width of the interval to the midpoint of the interval. Engineers usually specify percentage tolerances on the parameters of devices, as in the resistor specifications given earlier.


<a name="exercise_2_12">**Exercise 2.12**</a> Define a constructor <a name="index_term_1522"></a><tt>make-center-percent</tt> that takes a center and a percentage tolerance and produces the desired interval. You must also define a selector <tt>percent</tt> that produces the percentage tolerance for a given interval. The <tt>center</tt> selector is the same as the one shown above.


<a name="exercise_2_13">**Exercise 2.13**</a> Show that under the assumption of small percentage tolerances there is a simple formula for the approximate percentage tolerance of the product of two intervals in terms of the tolerances of the factors. You may simplify the problem by assuming that all numbers are positive.


After considerable work, Alyssa P. Hacker delivers her finished system. Several years later, after she has forgotten all about it, she gets a frenzied call from an irate user, Lem E. Tweakit. It seems that Lem has noticed that the formula for parallel resistors can be written in two <a name="index_term_1524"></a>algebraically equivalent ways:

<div align="left">
  <img src="img/chapter_2_image_09.gif" border="0">
</div>

and

<div align="left">
  <img src="img/chapter_2_image_10.gif" border="0">
</div>

He has written the following two programs, each of which computes the parallel-resistors formula differently:


<tt>(define&nbsp;(par1&nbsp;r1&nbsp;r2)<br>
&nbsp;&nbsp;(div-interval&nbsp;(mul-interval&nbsp;r1&nbsp;r2)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(add-interval&nbsp;r1&nbsp;r2)))<br>
(define&nbsp;(par2&nbsp;r1&nbsp;r2)<br>
&nbsp;&nbsp;(let&nbsp;((one&nbsp;(make-interval&nbsp;1&nbsp;1)))&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;(div-interval&nbsp;one<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(add-interval&nbsp;(div-interval&nbsp;one&nbsp;r1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(div-interval&nbsp;one&nbsp;r2)))))<br></tt>


Lem complains that Alyssa's program gives different answers for the two ways of computing. This is a serious complaint.


<a name="exercise_2_14">**Exercise 2.14**</a> Demonstrate that Lem is right. Investigate the behavior of the system on a variety of arithmetic expressions. Make some intervals *A* and *B*, and use them in computing the expressions *A*/*A* and *A*/*B*. You will get the most insight by using intervals whose width is a small percentage of the center value. Examine the results of the computation in center-percent form (see exercise&nbsp;<a href="#%_thm_2.12">2.12</a>).


<a name="exercise_2_15">**Exercise 2.15**</a> Eva Lu Ator, another user, has also noticed the different intervals computed by different but algebraically equivalent expressions. She says that a formula to compute with intervals using Alyssa's system will produce tighter error bounds if it can be written in such a form that no variable that represents an uncertain number is repeated. Thus, she says, <tt>par2</tt> is a "better" program for parallel resistances than <tt>par1</tt>. Is she right? Why?


<a name="exercise_2_16">**Exercise 2.16**</a> Explain, in general, why equivalent algebraic expressions may lead to different answers. Can you devise an interval-arithmetic package that does not have this shortcoming, or is this task impossible? (Warning: This problem is very difficult.)

<div class="smallprint">
  <hr>
</div>

<div class="footnote">
  
<a name="footnote_Temp_133" href="#call_footnote_Temp_133" id="footnote_Temp_133"><sup><small>2</small></sup></a> The name <a name="index_term_1332"></a><tt>cons</tt> stands for "construct." The names <a name="index_term_1334"></a><tt>car</tt> and <a name="index_term_1336"></a><tt>cdr</tt> derive from the original implementation of Lisp on the <a name="index_term_1338"></a><a name="index_term_1340"></a>IBM 704. That machine had an addressing scheme that allowed one to reference the "address" and "decrement" parts of a memory location. <tt>Car</tt> stands for "Contents of Address part of Register" and <tt>cdr</tt> (pronounced "could-er") stands for "Contents of Decrement part of Register."

  
<a name="footnote_Temp_135" href="#call_footnote_Temp_135" id="footnote_Temp_135"><sup><small>3</small></sup></a> Another way to define the selectors and constructor is

  
<tt>(define&nbsp;make-rat&nbsp;cons)<br>
  (define&nbsp;numer&nbsp;car)<br>
  (define&nbsp;denom&nbsp;cdr)<br></tt>

  
The first definition associates the name <tt>make-rat</tt> with the value of the expression <tt>cons</tt>, which is the primitive procedure that constructs pairs. Thus <tt>make-rat</tt> and <tt>cons</tt> are names for the same primitive constructor.

  
Defining selectors and constructors in this way is efficient: Instead of <tt>make-rat</tt> *calling* <tt>cons</tt>, <tt>make-rat</tt> *is* <tt>cons</tt>, so there is only one procedure called, not two, when <tt>make-rat</tt> is called. On the other hand, doing this defeats debugging aids that trace procedure calls or put breakpoints on procedure calls: You may want to watch <tt>make-rat</tt> being called, but you certainly don't want to watch every call to <tt>cons</tt>.

  
We have chosen not to use this style of definition in this book.

  
<a name="footnote_Temp_136" href="#call_footnote_Temp_136" id="footnote_Temp_136"><sup><small>4</small></sup></a> <a name="index_term_1356"></a><a name="index_term_1358"></a><a name="index_term_1360"></a><a name="index_term_1362"></a><a name="index_term_1364"></a><tt>Display</tt> is the Scheme primitive for printing data. The Scheme primitive <tt>newline</tt> starts a new line for printing. <a name="index_term_1366"></a><a name="index_term_1368"></a>Neither of these procedures returns a useful value, so in the uses of <tt>print-rat</tt> below, we show only what <tt>print-rat</tt> prints, not what the interpreter prints as the value returned by <tt>print-rat</tt>.

  
<a name="footnote_Temp_140" href="#call_footnote_Temp_140" id="footnote_Temp_140"><sup><small>5</small></sup></a> Surprisingly, this idea is very difficult to formulate rigorously. There are two approaches to giving such a formulation. One, pioneered by <a name="index_term_1422"></a>C. A. R. Hoare (1972), is known as the method of <a name="index_term_1424"></a><a name="index_term_1426"></a>*abstract models*. It formalizes the "procedures plus conditions" specification as outlined in the rational-number example above. Note that the condition on the rational-number representation was stated in terms of facts about integers (equality and division). In general, abstract models define new kinds of data objects in terms of previously defined types of data objects. Assertions about data objects can therefore be checked by reducing them to assertions about previously defined data objects. Another approach, introduced by <a name="index_term_1428"></a>Zilles at MIT, by <a name="index_term_1430"></a>Goguen, <a name="index_term_1432"></a>Thatcher, <a name="index_term_1434"></a>Wagner, and <a name="index_term_1436"></a>Wright at IBM (see Thatcher, Wagner, and Wright 1978), and by <a name="index_term_1438"></a>Guttag at Toronto (see Guttag 1977), is called <a name="index_term_1440"></a><a name="index_term_1442"></a>*algebraic specification*. It regards the "procedures" as elements of an abstract algebraic system whose behavior is specified by axioms that correspond to our "conditions," and uses the techniques of abstract algebra to check assertions about data objects. Both methods are surveyed in the paper by <a name="index_term_1444"></a>Liskov and Zilles (1975).

</div>