(section_25)=
# Systems with Generic Operations

<a name="index_term_2496"></a>In the previous section, we saw how to design systems in which data objects can be represented in more than one way. The key idea is to link the code that specifies the data operations to the several representations by means of generic interface procedures. Now we will see how to use this same idea not only to define operations that are generic over different representations but also to define operations that are generic over different kinds of arguments. We have already seen several different packages of arithmetic operations: the primitive arithmetic (<tt>+</tt>, <tt>-</tt>, <tt>*</tt>, <tt>/</tt>) built into our language, the rational-number arithmetic (<tt>add-rat</tt>, <tt>sub-rat</tt>, <tt>mul-rat</tt>, <tt>div-rat</tt>) of section <a href="chapter_2_section_1.html#%_sec_2.1.1">2.1.1</a>, and the complex-number arithmetic that we implemented in section <a href="chapter_2_section_4.html#%_sec_2.4.3">2.4.3</a>. We will now use data-directed techniques to construct a package of arithmetic operations that incorporates all the arithmetic packages we have already constructed.


Figure&nbsp;<a href="#%_fig_2.23">2.23</a> shows the structure of the system we shall build. Notice the <a name="index_term_2498"></a>abstraction barriers. From the perspective of someone using "numbers," there is a single procedure <tt>add</tt> that operates on whatever numbers are supplied. <tt>Add</tt> is part of a generic interface that allows the separate ordinary-arithmetic, rational-arithmetic, and complex-arithmetic packages to be accessed uniformly by programs that use numbers. Any individual arithmetic package (such as the complex package) may itself be accessed through generic procedures (such as <tt>add-complex</tt>) that combine packages designed for different representations (such as rectangular and polar). Moreover, the structure of the system is additive, so that one can design the individual arithmetic packages separately and combine them to produce a generic arithmetic system. <a name="%_fig_2.23"></a>

<div align="left">
  <div align="left">
    **Figure 2.23:**&nbsp;&nbsp;Generic arithmetic system.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_64.gif" border="0"></td>
    </tr>

    <tr>
      <td><a name="index_term_2500"></a></td>
    </tr>
  </table>
</div>

<a name="%_sec_2.5.1"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_2.5.1">2.5.1&nbsp;&nbsp;Generic Arithmetic Operations</a></h3>

<a name="index_term_2502"></a> The task of designing generic arithmetic operations is analogous to that of designing the generic complex-number operations. We would like, for instance, to have a generic addition procedure <tt>add</tt> that acts like ordinary primitive addition <tt>+</tt> on ordinary numbers, like <tt>add-rat</tt> on rational numbers, and like <tt>add-complex</tt> on complex numbers. We can implement <tt>add</tt>, and the other generic arithmetic operations, by following the same strategy we used in section <a href="chapter_2_section_4.html#%_sec_2.4.3">2.4.3</a> to implement the generic selectors for complex numbers. We will attach a type tag to each kind of number and cause the generic procedure to dispatch to an appropriate package according to the data type of its arguments.


The generic arithmetic procedures are defined as follows:


<tt><a name="index_term_2504"></a>(define&nbsp;(add&nbsp;x&nbsp;y)&nbsp;(apply-generic&nbsp;'add&nbsp;x&nbsp;y))<br>
<a name="index_term_2506"></a>(define&nbsp;(sub&nbsp;x&nbsp;y)&nbsp;(apply-generic&nbsp;'sub&nbsp;x&nbsp;y))<br>
<a name="index_term_2508"></a>(define&nbsp;(mul&nbsp;x&nbsp;y)&nbsp;(apply-generic&nbsp;'mul&nbsp;x&nbsp;y))<br>
<a name="index_term_2510"></a>(define&nbsp;(div&nbsp;x&nbsp;y)&nbsp;(apply-generic&nbsp;'div&nbsp;x&nbsp;y))<br></tt>


<a name="index_term_2512"></a><a name="index_term_2514"></a>We begin by installing a package for handling *ordinary* numbers, that is, the primitive numbers of our language. We will tag these with the symbol <tt>scheme-number</tt>. The arithmetic operations in this package are the primitive arithmetic procedures (so there is no need to define extra procedures to handle the untagged numbers). Since these operations each take two arguments, they are installed in the table keyed by the list <tt>(scheme-number scheme-number)</tt>: <a name="index_term_2516"></a><a name="index_term_2518"></a>


<tt><a name="index_term_2520"></a>(define&nbsp;(install-scheme-number-package)<br>
&nbsp;&nbsp;(define&nbsp;(tag&nbsp;x)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(attach-tag&nbsp;'scheme-number&nbsp;x))&nbsp;&nbsp;&nbsp;&nbsp;<br>
&nbsp;&nbsp;(put&nbsp;'add&nbsp;'(scheme-number&nbsp;scheme-number)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(x&nbsp;y)&nbsp;(tag&nbsp;(+&nbsp;x&nbsp;y))))<br>
&nbsp;&nbsp;(put&nbsp;'sub&nbsp;'(scheme-number&nbsp;scheme-number)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(x&nbsp;y)&nbsp;(tag&nbsp;(-&nbsp;x&nbsp;y))))<br>
&nbsp;&nbsp;(put&nbsp;'mul&nbsp;'(scheme-number&nbsp;scheme-number)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(x&nbsp;y)&nbsp;(tag&nbsp;(*&nbsp;x&nbsp;y))))<br>
&nbsp;&nbsp;(put&nbsp;'div&nbsp;'(scheme-number&nbsp;scheme-number)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(x&nbsp;y)&nbsp;(tag&nbsp;(/&nbsp;x&nbsp;y))))<br>
&nbsp;&nbsp;(put&nbsp;'make&nbsp;'scheme-number<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(x)&nbsp;(tag&nbsp;x)))<br>
&nbsp;&nbsp;'done)<br></tt>


Users of the Scheme-number package will create (tagged) ordinary numbers by means of the procedure:


<tt><a name="index_term_2522"></a>(define&nbsp;(make-scheme-number&nbsp;n)<br>
&nbsp;&nbsp;((get&nbsp;'make&nbsp;'scheme-number)&nbsp;n))<br></tt>


Now that the framework of the generic arithmetic system is in place, we can readily include new kinds of numbers. Here is a package that performs rational arithmetic. Notice that, as a benefit of additivity, we can use without modification the rational-number code from section <a href="chapter_2_section_1.html#%_sec_2.1.1">2.1.1</a> as the internal procedures in the package: <a name="index_term_2524"></a><a name="index_term_2526"></a><a name="index_term_2528"></a>


<tt><a name="index_term_2530"></a>(define&nbsp;(install-rational-package)<br>
&nbsp;&nbsp;*;;&nbsp;internal&nbsp;procedures*<br>
&nbsp;&nbsp;(define&nbsp;(numer&nbsp;x)&nbsp;(car&nbsp;x))<br>
&nbsp;&nbsp;(define&nbsp;(denom&nbsp;x)&nbsp;(cdr&nbsp;x))<br>
&nbsp;&nbsp;(define&nbsp;(make-rat&nbsp;n&nbsp;d)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((g&nbsp;(gcd&nbsp;n&nbsp;d)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;(/&nbsp;n&nbsp;g)&nbsp;(/&nbsp;d&nbsp;g))))<br>
&nbsp;&nbsp;(define&nbsp;(add-rat&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(make-rat&nbsp;(+&nbsp;(*&nbsp;(numer&nbsp;x)&nbsp;(denom&nbsp;y))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;(numer&nbsp;y)&nbsp;(denom&nbsp;x)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;(denom&nbsp;x)&nbsp;(denom&nbsp;y))))<br>
&nbsp;&nbsp;(define&nbsp;(sub-rat&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(make-rat&nbsp;(-&nbsp;(*&nbsp;(numer&nbsp;x)&nbsp;(denom&nbsp;y))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;(numer&nbsp;y)&nbsp;(denom&nbsp;x)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;(denom&nbsp;x)&nbsp;(denom&nbsp;y))))<br>
&nbsp;&nbsp;(define&nbsp;(mul-rat&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(make-rat&nbsp;(*&nbsp;(numer&nbsp;x)&nbsp;(numer&nbsp;y))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;(denom&nbsp;x)&nbsp;(denom&nbsp;y))))<br>
&nbsp;&nbsp;(define&nbsp;(div-rat&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(make-rat&nbsp;(*&nbsp;(numer&nbsp;x)&nbsp;(denom&nbsp;y))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;(denom&nbsp;x)&nbsp;(numer&nbsp;y))))<br>
&nbsp;&nbsp;*;;&nbsp;interface&nbsp;to&nbsp;rest&nbsp;of&nbsp;the&nbsp;system*<br>
&nbsp;&nbsp;(define&nbsp;(tag&nbsp;x)&nbsp;(attach-tag&nbsp;'rational&nbsp;x))<br>
&nbsp;&nbsp;(put&nbsp;'add&nbsp;'(rational&nbsp;rational)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(x&nbsp;y)&nbsp;(tag&nbsp;(add-rat&nbsp;x&nbsp;y))))<br>
&nbsp;&nbsp;(put&nbsp;'sub&nbsp;'(rational&nbsp;rational)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(x&nbsp;y)&nbsp;(tag&nbsp;(sub-rat&nbsp;x&nbsp;y))))<br>
&nbsp;&nbsp;(put&nbsp;'mul&nbsp;'(rational&nbsp;rational)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(x&nbsp;y)&nbsp;(tag&nbsp;(mul-rat&nbsp;x&nbsp;y))))<br>
&nbsp;&nbsp;(put&nbsp;'div&nbsp;'(rational&nbsp;rational)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(x&nbsp;y)&nbsp;(tag&nbsp;(div-rat&nbsp;x&nbsp;y))))<br>
<br>
&nbsp;&nbsp;(put&nbsp;'make&nbsp;'rational<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(n&nbsp;d)&nbsp;(tag&nbsp;(make-rat&nbsp;n&nbsp;d))))<br>
&nbsp;&nbsp;'done)<br>
<a name="index_term_2532"></a>(define&nbsp;(make-rational&nbsp;n&nbsp;d)<br>
&nbsp;&nbsp;((get&nbsp;'make&nbsp;'rational)&nbsp;n&nbsp;d))<br></tt>


We can install a similar package to handle complex numbers, using the tag <tt>complex</tt>. In creating the package, we extract from the table the operations <tt>make-from-real-imag</tt> and <tt>make-from-mag-ang</tt> that were defined by the rectangular and polar packages. <a name="index_term_2534"></a>Additivity permits us to use, as the internal operations, the same <tt>add-complex</tt>, <tt>sub-complex</tt>, <tt>mul-complex</tt>, and <tt>div-complex</tt> procedures from section <a href="chapter_2_section_4.html#%_sec_2.4.1">2.4.1</a>. <a name="index_term_2536"></a><a name="index_term_2538"></a><a name="index_term_2540"></a>


<tt><a name="index_term_2542"></a>(define&nbsp;(install-complex-package)<br>
&nbsp;&nbsp;*;;&nbsp;imported&nbsp;procedures&nbsp;from&nbsp;rectangular&nbsp;and&nbsp;polar&nbsp;packages*<br>
&nbsp;&nbsp;(define&nbsp;(make-from-real-imag&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;&nbsp;&nbsp;((get&nbsp;'make-from-real-imag&nbsp;'rectangular)&nbsp;x&nbsp;y))<br>
&nbsp;&nbsp;(define&nbsp;(make-from-mag-ang&nbsp;r&nbsp;a)<br>
&nbsp;&nbsp;&nbsp;&nbsp;((get&nbsp;'make-from-mag-ang&nbsp;'polar)&nbsp;r&nbsp;a))<br>
&nbsp;&nbsp;*;;&nbsp;internal&nbsp;procedures*<br>
&nbsp;&nbsp;(define&nbsp;(add-complex&nbsp;z1&nbsp;z2)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(make-from-real-imag&nbsp;(+&nbsp;(real-part&nbsp;z1)&nbsp;(real-part&nbsp;z2))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(+&nbsp;(imag-part&nbsp;z1)&nbsp;(imag-part&nbsp;z2))))<br>
&nbsp;&nbsp;(define&nbsp;(sub-complex&nbsp;z1&nbsp;z2)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(make-from-real-imag&nbsp;(-&nbsp;(real-part&nbsp;z1)&nbsp;(real-part&nbsp;z2))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(-&nbsp;(imag-part&nbsp;z1)&nbsp;(imag-part&nbsp;z2))))<br>
&nbsp;&nbsp;(define&nbsp;(mul-complex&nbsp;z1&nbsp;z2)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(make-from-mag-ang&nbsp;(*&nbsp;(magnitude&nbsp;z1)&nbsp;(magnitude&nbsp;z2))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(+&nbsp;(angle&nbsp;z1)&nbsp;(angle&nbsp;z2))))<br>
&nbsp;&nbsp;(define&nbsp;(div-complex&nbsp;z1&nbsp;z2)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(make-from-mag-ang&nbsp;(/&nbsp;(magnitude&nbsp;z1)&nbsp;(magnitude&nbsp;z2))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(-&nbsp;(angle&nbsp;z1)&nbsp;(angle&nbsp;z2))))<br>
&nbsp;&nbsp;*;;&nbsp;interface&nbsp;to&nbsp;rest&nbsp;of&nbsp;the&nbsp;system*<br>
&nbsp;&nbsp;(define&nbsp;(tag&nbsp;z)&nbsp;(attach-tag&nbsp;'complex&nbsp;z))<br>
&nbsp;&nbsp;(put&nbsp;'add&nbsp;'(complex&nbsp;complex)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(z1&nbsp;z2)&nbsp;(tag&nbsp;(add-complex&nbsp;z1&nbsp;z2))))<br>
&nbsp;&nbsp;(put&nbsp;'sub&nbsp;'(complex&nbsp;complex)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(z1&nbsp;z2)&nbsp;(tag&nbsp;(sub-complex&nbsp;z1&nbsp;z2))))<br>
&nbsp;&nbsp;(put&nbsp;'mul&nbsp;'(complex&nbsp;complex)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(z1&nbsp;z2)&nbsp;(tag&nbsp;(mul-complex&nbsp;z1&nbsp;z2))))<br>
&nbsp;&nbsp;(put&nbsp;'div&nbsp;'(complex&nbsp;complex)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(z1&nbsp;z2)&nbsp;(tag&nbsp;(div-complex&nbsp;z1&nbsp;z2))))<br>
&nbsp;&nbsp;(put&nbsp;'make-from-real-imag&nbsp;'complex<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(x&nbsp;y)&nbsp;(tag&nbsp;(make-from-real-imag&nbsp;x&nbsp;y))))<br>
&nbsp;&nbsp;(put&nbsp;'make-from-mag-ang&nbsp;'complex<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(r&nbsp;a)&nbsp;(tag&nbsp;(make-from-mag-ang&nbsp;r&nbsp;a))))<br>
&nbsp;&nbsp;'done)<br></tt>


Programs outside the complex-number package can construct complex numbers either from real and imaginary parts or from magnitudes and angles. Notice how the underlying procedures, originally defined in the rectangular and polar packages, are exported to the complex package, and exported from there to the outside world.


<tt><a name="index_term_2544"></a>(define&nbsp;(make-complex-from-real-imag&nbsp;x&nbsp;y)<br>
&nbsp;&nbsp;((get&nbsp;'make-from-real-imag&nbsp;'complex)&nbsp;x&nbsp;y))<br>
<a name="index_term_2546"></a>(define&nbsp;(make-complex-from-mag-ang&nbsp;r&nbsp;a)<br>
&nbsp;&nbsp;((get&nbsp;'make-from-mag-ang&nbsp;'complex)&nbsp;r&nbsp;a))<br></tt>


<a name="index_term_2548"></a>What we have here is a two-level tag system. A typical complex number, such as 3 + 4*i* in rectangular form, would be represented as shown in figure&nbsp;<a href="#%_fig_2.24">2.24</a>. The outer tag (<tt>complex</tt>) is used to direct the number to the complex package. Once within the complex package, the next tag (<tt>rectangular</tt>) is used to direct the number to the rectangular package. In a large and complicated system there might be many levels, each interfaced with the next by means of generic operations. As a data object is passed "downward," the outer tag that is used to direct it to the appropriate package is stripped off (by applying <tt>contents</tt>) and the next level of tag (if any) becomes visible to be used for further dispatching.


<a name="%_fig_2.24"></a>

<div align="left">
  <div align="left">
    **Figure 2.24:**&nbsp;&nbsp;Representation of 3 + 4*i* in rectangular form.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_65.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

In the above packages, we used <tt>add-rat</tt>, <tt>add-complex</tt>, and the other arithmetic procedures exactly as originally written. Once these definitions are internal to different installation procedures, however, they no longer need names that are distinct from each other: we could simply name them <tt>add</tt>, <tt>sub</tt>, <tt>mul</tt>, and <tt>div</tt> in both packages.


<a name="exercise_2_77">**Exercise 2.77**</a> Louis Reasoner tries to evaluate the expression <tt>(magnitude z)</tt> where <tt>z</tt> is the object shown in figure&nbsp;<a href="#%_fig_2.24">2.24</a>. To his surprise, instead of the answer 5 he gets an error message from <tt>apply-generic</tt>, saying there is no method for the operation <tt>magnitude</tt> on the types <tt>(complex)</tt>. He shows this interaction to Alyssa P. Hacker, who says "The problem is that the complex-number selectors were never defined for <tt>complex</tt> numbers, just for <tt>polar</tt> and <tt>rectangular</tt> numbers. All you have to do to make this work is add the following to the <tt>complex</tt> package:"


<tt>(put&nbsp;'real-part&nbsp;'(complex)&nbsp;real-part)<br>
(put&nbsp;'imag-part&nbsp;'(complex)&nbsp;imag-part)<br>
(put&nbsp;'magnitude&nbsp;'(complex)&nbsp;magnitude)<br>
(put&nbsp;'angle&nbsp;'(complex)&nbsp;angle)<br></tt>


Describe in detail why this works. As an example, trace through all the procedures called in evaluating the expression <tt>(magnitude z)</tt> where <tt>z</tt> is the object shown in figure&nbsp;<a href="#%_fig_2.24">2.24</a>. In particular, how many times is <tt>apply-generic</tt> invoked? What procedure is dispatched to in each case?


<a name="exercise_2_78">**Exercise 2.78**</a> <a name="index_term_2550"></a><a name="index_term_2552"></a><a name="index_term_2554"></a><a name="index_term_2556"></a><a name="index_term_2558"></a><a name="index_term_2560"></a><a name="index_term_2562"></a>The internal procedures in the <tt>scheme-number</tt> package are essentially nothing more than calls to the primitive procedures <tt>+</tt>, <tt>-</tt>, etc. It was not possible to use the primitives of the language directly because our type-tag system requires that each data object have a type attached to it. In fact, however, all Lisp implementations do have a type system, which they use internally. Primitive predicates such as <tt>symbol?</tt> and <tt>number?</tt> determine whether data objects have particular types. Modify the definitions of <tt>type-tag</tt>, <tt>contents</tt>, and <tt>attach-tag</tt> from section <a href="chapter_2_section_4.html#%_sec_2.4.2">2.4.2</a> so that our generic system takes advantage of Scheme's internal type system. That is to say, the system should work as before except that ordinary numbers should be represented simply as Scheme numbers rather than as pairs whose <tt>car</tt> is the symbol <tt>scheme-number</tt>.


<a name="exercise_2_79">**Exercise 2.79**</a> <a name="index_term_2564"></a><a name="index_term_2566"></a>Define a generic equality predicate <tt>equ?</tt> that tests the equality of two numbers, and install it in the generic arithmetic package. This operation should work for ordinary numbers, rational numbers, and complex numbers.


<a name="exercise_2_80">**Exercise 2.80**</a> <a name="index_term_2568"></a><a name="index_term_2570"></a>Define a generic predicate <tt>=zero?</tt> that tests if its argument is zero, and install it in the generic arithmetic package. This operation should work for ordinary numbers, rational numbers, and complex numbers.


<a name="%_sec_2.5.2"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_2.5.2">2.5.2&nbsp;&nbsp;Combining Data of Different Types</a></h3>

We have seen how to define a unified arithmetic system that encompasses ordinary numbers, complex numbers, rational numbers, and any other type of number we might decide to invent, but we have ignored an important issue. The operations we have defined so far treat the different data types as being completely independent. Thus, there are separate packages for adding, say, two ordinary numbers, or two complex numbers. What we have not yet considered is the fact that it is meaningful to define operations that cross the type boundaries, such as the addition of a complex number to an ordinary number. We have gone to great pains to introduce barriers between parts of our programs so that they can be developed and understood separately. We would like to introduce the cross-type operations in some carefully controlled way, so that we can support them without seriously violating our module boundaries.


<a name="index_term_2572"></a><a name="index_term_2574"></a><a name="index_term_2576"></a>One way to handle cross-type operations is to design a different procedure for each possible combination of types for which the operation is valid. For example, we could extend the complex-number package so that it provides a procedure for adding complex numbers to ordinary numbers and installs this in the table using the tag <tt>(complex scheme-number)</tt>:<a name="call_footnote_Temp_283" href="#footnote_Temp_283" id="call_footnote_Temp_283"><sup><small>49</small></sup></a>


<tt>*;;&nbsp;to&nbsp;be&nbsp;included&nbsp;in&nbsp;the&nbsp;complex&nbsp;package*<br>
<a name="index_term_2578"></a>(define&nbsp;(add-complex-to-schemenum&nbsp;z&nbsp;x)<br>
&nbsp;&nbsp;(make-from-real-imag&nbsp;(+&nbsp;(real-part&nbsp;z)&nbsp;x)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(imag-part&nbsp;z)))<br>
(put&nbsp;'add&nbsp;'(complex&nbsp;scheme-number)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(z&nbsp;x)&nbsp;(tag&nbsp;(add-complex-to-schemenum&nbsp;z&nbsp;x))))<br></tt>


This technique works, but it is cumbersome. With such a system, the cost of introducing a new type is not just the construction of the package of procedures for that type but also the construction and installation of the procedures that implement the cross-type operations. This can easily be much more code than is needed to define the operations on the type itself. The method also undermines our ability to combine separate packages additively, or least to limit the extent to which the implementors of the individual packages need to take account of other packages. For instance, in the example above, it seems reasonable that handling mixed operations on complex numbers and ordinary numbers should be the responsibility of the complex-number package. Combining rational numbers and complex numbers, however, might be done by the complex package, by the rational package, or by some third package that uses operations extracted from these two packages. Formulating coherent policies on the division of responsibility among packages can be an overwhelming task in designing systems with many packages and many cross-type operations.


<a name="%_sec_Temp_284"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_284">Coercion</a></h4>

<a name="index_term_2580"></a> In the general situation of completely unrelated operations acting on completely unrelated types, implementing explicit cross-type operations, cumbersome though it may be, is the best that one can hope for. Fortunately, we can usually do better by taking advantage of additional structure that may be latent in our type system. Often the different data types are not completely independent, and there may be ways by which objects of one type may be viewed as being of another type. This process is called *coercion*. For example, if we are asked to arithmetically combine an ordinary number with a complex number, we can view the ordinary number as a complex number whose imaginary part is zero. This transforms the problem to that of combining two complex numbers, which can be handled in the ordinary way by the complex-arithmetic package.


<a name="index_term_2582"></a>In general, we can implement this idea by designing coercion procedures that transform an object of one type into an equivalent object of another type. Here is a typical coercion procedure, which transforms a given ordinary number to a complex number with that real part and zero imaginary part:


<tt><a name="index_term_2584"></a>(define&nbsp;(scheme-number-&gt;complex&nbsp;n)<br>
&nbsp;&nbsp;(make-complex-from-real-imag&nbsp;(contents&nbsp;n)&nbsp;0))<br></tt>


<a name="index_term_2586"></a><a name="index_term_2588"></a>We install these coercion procedures in a special coercion table, indexed under the names of the two types:


<tt>(put-coercion&nbsp;'scheme-number&nbsp;'complex&nbsp;scheme-number-&gt;complex)<br></tt>


(We assume that there are <tt>put-coercion</tt> and <tt>get-coercion</tt> procedures available for manipulating this table.) Generally some of the slots in the table will be empty, because it is not generally possible to coerce an arbitrary data object of each type into all other types. For example, there is no way to coerce an arbitrary complex number to an ordinary number, so there will be no general <tt>complex-&gt;scheme-number</tt> procedure included in the table.


Once the coercion table has been set up, we can handle coercion in a uniform manner by modifying the <tt>apply-generic</tt> procedure of section <a href="chapter_2_section_4.html#%_sec_2.4.3">2.4.3</a>. When asked to apply an operation, we first check whether the operation is defined for the arguments' types, just as before. If so, we dispatch to the procedure found in the operation-and-type table. Otherwise, we try coercion. For simplicity, we consider only the case where there are two arguments.<a name="call_footnote_Temp_285" href="#footnote_Temp_285" id="call_footnote_Temp_285"><sup><small>50</small></sup></a> We check the coercion table to see if objects of the first type can be coerced to the second type. If so, we coerce the first argument and try the operation again. If objects of the first type cannot in general be coerced to the second type, we try the coercion the other way around to see if there is a way to coerce the second argument to the type of the first argument. Finally, if there is no known way to coerce either type to the other type, we give up. Here is the procedure:


<tt><a name="index_term_2590"></a>(define&nbsp;(apply-generic&nbsp;op&nbsp;.&nbsp;args)<br>
&nbsp;&nbsp;(let&nbsp;((type-tags&nbsp;(map&nbsp;type-tag&nbsp;args)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((proc&nbsp;(get&nbsp;op&nbsp;type-tags)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;proc<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(apply&nbsp;proc&nbsp;(map&nbsp;contents&nbsp;args))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;(=&nbsp;(length&nbsp;args)&nbsp;2)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((type1&nbsp;(car&nbsp;type-tags))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(type2&nbsp;(cadr&nbsp;type-tags))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(a1&nbsp;(car&nbsp;args))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(a2&nbsp;(cadr&nbsp;args)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((t1-&gt;t2&nbsp;(get-coercion&nbsp;type1&nbsp;type2))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(t2-&gt;t1&nbsp;(get-coercion&nbsp;type2&nbsp;type1)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cond&nbsp;(t1-&gt;t2<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(apply-generic&nbsp;op&nbsp;(t1-&gt;t2&nbsp;a1)&nbsp;a2))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(t2-&gt;t1<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(apply-generic&nbsp;op&nbsp;a1&nbsp;(t2-&gt;t1&nbsp;a2)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(else<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(error&nbsp;&quot;No&nbsp;method&nbsp;for&nbsp;these&nbsp;types&quot;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;op&nbsp;type-tags))))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(error&nbsp;&quot;No&nbsp;method&nbsp;for&nbsp;these&nbsp;types&quot;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;op&nbsp;type-tags)))))))<br></tt>


This coercion scheme has many advantages over the method of defining explicit cross-type operations, as outlined above. Although we still need to write coercion procedures to relate the types (possibly *n*<sup>2</sup> procedures for a system with *n* types), we need to write only one procedure for each pair of types rather than a different procedure for each collection of types and each generic operation.<a name="call_footnote_Temp_286" href="#footnote_Temp_286" id="call_footnote_Temp_286"><sup><small>51</small></sup></a> What we are counting on here is the fact that the appropriate transformation between types depends only on the types themselves, not on the operation to be applied.


On the other hand, there may be applications for which our coercion scheme is not general enough. Even when neither of the objects to be combined can be converted to the type of the other it may still be possible to perform the operation by converting both objects to a third type. In order to deal with such complexity and still preserve modularity in our programs, it is usually necessary to build systems that take advantage of still further structure in the relations among types, as we discuss next.


<a name="%_sec_Temp_287"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_287">Hierarchies of types</a></h4>

<a name="index_term_2592"></a><a name="index_term_2594"></a> The coercion scheme presented above relied on the existence of natural relations between pairs of types. Often there is more "global" structure in how the different types relate to each other. For instance, suppose we are building a generic arithmetic system to handle integers, rational numbers, real numbers, and complex numbers. In such a system, it is quite natural to regard an integer as a special kind of rational number, which is in turn a special kind of real number, which is in turn a special kind of complex number. What we actually have is a so-called *hierarchy of types*, in which, for example, integers are a <a name="index_term_2596"></a><a name="index_term_2598"></a>*subtype* of rational numbers (i.e., any operation that can be applied to a rational number can automatically be applied to an integer). Conversely, we say that rational numbers form a <a name="index_term_2600"></a><a name="index_term_2602"></a>*supertype* of integers. The particular hierarchy we have here is of a very simple kind, in which each type has at most one supertype and at most one subtype. Such a structure, called a *tower*, is illustrated in figure&nbsp;<a href="#%_fig_2.25">2.25</a>.


<a name="%_fig_2.25"></a>

<div align="left">
  <div align="left">
    **Figure 2.25:**&nbsp;&nbsp;A tower of types.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_66.gif" border="0"></td>
    </tr>

    <tr>
      <td><a name="index_term_2604"></a><a name="index_term_2606"></a></td>
    </tr>
  </table>
</div>

If we have a tower structure, then we can greatly simplify the problem of adding a new type to the hierarchy, for we need only specify how the new type is embedded in the next supertype above it and how it is the supertype of the type below it. For example, if we want to add an integer to a complex number, we need not explicitly define a special coercion procedure <tt>integer-&gt;complex</tt>. Instead, we define how an integer can be transformed into a rational number, how a rational number is transformed into a real number, and how a real number is transformed into a complex number. We then allow the system to transform the integer into a complex number through these steps and then add the two complex numbers.


<a name="index_term_2608"></a><a name="index_term_2610"></a>We can redesign our <tt>apply-generic</tt> procedure in the following way: For each type, we need to supply a <tt>raise</tt> procedure, which "raises" objects of that type one level in the tower. Then when the system is required to operate on objects of different types it can successively raise the lower types until all the objects are at the same level in the tower. (Exercises&nbsp;<a href="#%_thm_2.83">2.83</a> and &nbsp;<a href="#%_thm_2.84">2.84</a> concern the details of implementing such a strategy.)


Another advantage of a tower is that we can easily implement the notion that every type "inherits" all operations defined on a supertype. For instance, if we do not supply a special procedure for finding the real part of an integer, we should nevertheless expect that <tt>real-part</tt> will be defined for integers by virtue of the fact that integers are a subtype of complex numbers. In a tower, we can arrange for this to happen in a uniform way by modifying <tt>apply-generic</tt>. If the required operation is not directly defined for the type of the object given, we raise the object to its supertype and try again. We thus crawl up the tower, transforming our argument as we go, until we either find a level at which the desired operation can be performed or hit the top (in which case we give up).


<a name="index_term_2612"></a>Yet another advantage of a tower over a more general hierarchy is that it gives us a simple way to "lower" a data object to the simplest representation. For example, if we add 2 + 3*i* to 4 - 3*i*, it would be nice to obtain the answer as the integer 6 rather than as the complex number 6 + 0*i*. Exercise&nbsp;<a href="#%_thm_2.85">2.85</a> discusses a way to implement such a lowering operation. (The trick is that we need a general way to distinguish those objects that can be lowered, such as 6 + 0*i*, from those that cannot, such as 6 + 2*i*.)


<a name="%_fig_2.26"></a>

<div align="left">
  <div align="left">
    **Figure 2.26:**&nbsp;&nbsp;Relations among types of geometric figures.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_67.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="%_sec_Temp_288"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_288">Inadequacies of hierarchies</a></h4>

<a name="index_term_2614"></a> If the data types in our system can be naturally arranged in a tower, this greatly simplifies the problems of dealing with generic operations on different types, as we have seen. Unfortunately, this is usually not the case. Figure&nbsp;<a href="#%_fig_2.26">2.26</a> illustrates a more complex arrangement of mixed types, this one showing relations among different types of geometric figures. We see that, in general, <a name="index_term_2616"></a><a name="index_term_2618"></a><a name="index_term_2620"></a>a type may have more than one subtype. Triangles and quadrilaterals, for instance, are both subtypes of polygons. In addition, a type may have more than one supertype. For example, an isosceles right triangle may be regarded either as an isosceles triangle or as a right triangle. This multiple-supertypes issue is particularly thorny, since it means that there is no unique way to "raise" a type in the hierarchy. Finding the "correct" supertype in which to apply an operation to an object may involve considerable searching through the entire type network on the part of a procedure such as <tt>apply-generic</tt>. Since there generally are multiple subtypes for a type, there is a similar problem in coercing a value "down" the type hierarchy. Dealing with large numbers of interrelated types while still preserving modularity in the design of large systems is very difficult, and is an area of much current research.<a name="call_footnote_Temp_289" href="#footnote_Temp_289" id="call_footnote_Temp_289"><sup><small>52</small></sup></a>


<a name="exercise_2_81">**Exercise 2.81**</a> <a name="index_term_2626"></a>Louis Reasoner has noticed that <tt>apply-generic</tt> may try to coerce the arguments to each other's type even if they already have the same type. Therefore, he reasons, we need to put procedures in the coercion table to &quot;coerce&quot; arguments of each type to their own type. For example, in addition to the <tt>scheme-number-&gt;complex</tt> coercion shown above, he would do:


<tt><a name="index_term_2628"></a>(define&nbsp;(scheme-number-&gt;scheme-number&nbsp;n)&nbsp;n)<br>
<a name="index_term_2630"></a>(define&nbsp;(complex-&gt;complex&nbsp;z)&nbsp;z)<br>
(put-coercion&nbsp;'scheme-number&nbsp;'scheme-number<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;scheme-number-&gt;scheme-number)<br>
(put-coercion&nbsp;'complex&nbsp;'complex&nbsp;complex-&gt;complex)<br></tt>


a. With Louis's coercion procedures installed, what happens if <tt>apply-generic</tt> is called with two arguments of type <tt>scheme-number</tt> or two arguments of type <tt>complex</tt> for an operation that is not found in the table for those types? For example, assume that we've defined a generic exponentiation operation:


<tt>(define&nbsp;(exp&nbsp;x&nbsp;y)&nbsp;(apply-generic&nbsp;'exp&nbsp;x&nbsp;y))<br></tt>


and have put a procedure for exponentiation in the Scheme-number package but not in any other package:


<tt>*;;&nbsp;following&nbsp;added&nbsp;to&nbsp;Scheme-number&nbsp;package*<br>
(put&nbsp;'exp&nbsp;'(scheme-number&nbsp;scheme-number)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(x&nbsp;y)&nbsp;(tag&nbsp;(expt&nbsp;x&nbsp;y))))&nbsp;*;&nbsp;using&nbsp;primitive&nbsp;<tt>expt</tt>*<br></tt>


What happens if we call <tt>exp</tt> with two complex numbers as arguments?


b. Is Louis correct that something had to be done about coercion with arguments of the same type, or does <tt>apply-generic</tt> work correctly as is?


c. Modify <tt>apply-generic</tt> so that it doesn't try coercion if the two arguments have the same type.


<a name="exercise_2_82">**Exercise 2.82**</a> <a name="index_term_2632"></a>Show how to generalize <tt>apply-generic</tt> to handle coercion in the general case of multiple arguments. One strategy is to attempt to coerce all the arguments to the type of the first argument, then to the type of the second argument, and so on. Give an example of a situation where this strategy (and likewise the two-argument version given above) is not sufficiently general. (Hint: Consider the case where there are some suitable mixed-type operations present in the table that will not be tried.)


<a name="exercise_2_83">**Exercise 2.83**</a> <a name="index_term_2634"></a>Suppose you are designing a generic arithmetic system for dealing with the tower of types shown in figure&nbsp;<a href="#%_fig_2.25">2.25</a>: integer, rational, real, complex. For each type (except complex), design a procedure that raises objects of that type one level in the tower. Show how to install a generic <tt>raise</tt> operation that will work for each type (except complex).


<a name="exercise_2_84">**Exercise 2.84**</a> <a name="index_term_2636"></a>Using the <tt>raise</tt> operation of exercise&nbsp;<a href="#%_thm_2.83">2.83</a>, modify the <tt>apply-generic</tt> procedure so that it coerces its arguments to have the same type by the method of successive raising, as discussed in this section. You will need to devise a way to test which of two types is higher in the tower. Do this in a manner that is "compatible" with the rest of the system and will not lead to problems in adding new levels to the tower.


<a name="exercise_2_85">**Exercise 2.85**</a> <a name="index_term_2638"></a><a name="index_term_2640"></a>This section mentioned a method for "simplifying" a data object by lowering it in the tower of types as far as possible. Design a procedure <tt>drop</tt> that accomplishes this for the tower described in exercise&nbsp;<a href="#%_thm_2.83">2.83</a>. The key is to decide, in some general way, whether an object can be lowered. For example, the complex number 1.5 + 0*i* can be lowered as far as <tt>real</tt>, the complex number 1 + 0*i* can be lowered as far as <tt>integer</tt>, and the complex number 2 + 3*i* cannot be lowered at all. Here is a plan for determining whether an object can be lowered: Begin by defining a generic operation <tt>project</tt> that "pushes" an object down in the tower. For example, projecting a complex number would involve throwing away the imaginary part. Then a number can be dropped if, when we <tt>project</tt> it and <tt>raise</tt> the result back to the type we started with, we end up with something equal to what we started with. Show how to implement this idea in detail, by writing a <tt>drop</tt> procedure that drops an object as far as possible. You will need to design the various projection operations<a name="call_footnote_Temp_295" href="#footnote_Temp_295" id="call_footnote_Temp_295"><sup><small>53</small></sup></a> and install <tt>project</tt> as a generic operation in the system. You will also need to make use of a generic equality predicate, such as described in exercise&nbsp;<a href="#%_thm_2.79">2.79</a>. Finally, use <tt>drop</tt> to rewrite <tt>apply-generic</tt> from exercise&nbsp;<a href="#%_thm_2.84">2.84</a> so that it "simplifies" its answers.


<a name="exercise_2_86">**Exercise 2.86**</a> Suppose we want to handle complex numbers whose real parts, imaginary parts, magnitudes, and angles can be either ordinary numbers, rational numbers, or other numbers we might wish to add to the system. Describe and implement the changes to the system needed to accommodate this. You will have to define operations such as <tt>sine</tt> and <tt>cosine</tt> that are generic over ordinary numbers and rational numbers.


<a name="%_sec_2.5.3"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_2.5.3">2.5.3&nbsp;&nbsp;Example: Symbolic Algebra</a></h3>

<a name="index_term_2646"></a> The manipulation of symbolic algebraic expressions is a complex process that illustrates many of the hardest problems that occur in the design of large-scale systems. An algebraic expression, in <a name="index_term_2648"></a>general, can be viewed as a hierarchical structure, a tree of operators applied to operands. We can construct algebraic expressions by starting with a set of primitive objects, such as constants and variables, and combining these by means of algebraic operators, such as addition and multiplication. As in other languages, we form abstractions that enable us to refer to compound objects in simple terms. Typical abstractions in symbolic algebra are ideas such as linear combination, polynomial, rational function, or trigonometric function. We can regard these as compound "types," which are often useful for directing the processing of expressions. For example, we could describe the expression

<div align="left">
  <img src="img/chapter_2_image_68.gif" border="0">
</div>

as a polynomial in *x* with coefficients that are trigonometric functions of polynomials in *y* whose coefficients are integers.


We will not attempt to develop a complete algebraic-manipulation system here. Such systems are exceedingly complex programs, embodying deep algebraic knowledge and elegant algorithms. What we will do is look at a simple but important part of algebraic manipulation: the arithmetic of polynomials. We will illustrate the kinds of decisions the designer of such a system faces, and how to apply the ideas of abstract data and generic operations to help organize this effort.


<a name="%_sec_Temp_297"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_297">Arithmetic on polynomials</a></h4>

<a name="index_term_2650"></a><a name="index_term_2652"></a> Our first task in designing a system for performing arithmetic on polynomials is to decide just what a polynomial is. Polynomials are normally defined relative to certain variables (the <a name="index_term_2654"></a><a name="index_term_2656"></a>*indeterminates* of the polynomial). For simplicity, we will restrict ourselves to polynomials having just one indeterminate (<a name="index_term_2658"></a><a name="index_term_2660"></a>*univariate polynomials*).<a name="call_footnote_Temp_298" href="#footnote_Temp_298" id="call_footnote_Temp_298"><sup><small>54</small></sup></a> We will define a polynomial to be a sum of terms, each of which is either a coefficient, a power of the indeterminate, or a product of a coefficient and a power of the indeterminate. A coefficient is defined as an algebraic expression that is not dependent upon the indeterminate of the polynomial. For example,

<div align="left">
  <img src="img/chapter_2_image_69.gif" border="0">
</div>

is a simple polynomial in *x*, and

<div align="left">
  <img src="img/chapter_2_image_70.gif" border="0">
</div>

is a polynomial in *x* whose coefficients are polynomials in *y*.


Already we are skirting some thorny issues. Is the first of these polynomials the same as the polynomial 5*y*<sup>2</sup> + 3*y* + 7, or not? A reasonable answer might be "yes, if we are considering a polynomial purely as a mathematical function, but no, if we are considering a polynomial to be a syntactic form." The second polynomial is algebraically equivalent to a polynomial in *y* whose coefficients are polynomials in *x*. Should our system recognize this, or not? Furthermore, there are other ways to represent a polynomial -- for example, as a product of factors, or (for a univariate polynomial) as the set of roots, or as a listing of the values of the polynomial at a specified set of points.<a name="call_footnote_Temp_299" href="#footnote_Temp_299" id="call_footnote_Temp_299"><sup><small>55</small></sup></a> We can finesse these questions by deciding that in our algebraic-manipulation system a "polynomial" will be a particular syntactic form, not its underlying mathematical meaning.


Now we must consider how to go about doing arithmetic on polynomials. In this simple system, we will consider only addition and multiplication. Moreover, we will insist that two polynomials to be combined must have the same indeterminate.


We will approach the design of our system by following the familiar discipline of data abstraction. We will represent polynomials using a data structure called a <a name="index_term_2664"></a>*poly*, which consists of a variable and a <a name="index_term_2666"></a>collection of terms. We assume that we have selectors <tt>variable</tt> and <tt>term-list</tt> that extract those parts from a poly and a constructor <tt>make-poly</tt> that assembles a poly from a given variable and a term list. A variable will be just a symbol, so we can use the <a name="index_term_2668"></a><tt>same-variable?</tt> procedure of section <a href="chapter_2_section_3.html#%_sec_2.3.2">2.3.2</a> to compare variables. <a name="index_term_2670"></a><a name="index_term_2672"></a>The following procedures define addition and multiplication of polys:


<tt><a name="index_term_2674"></a>(define&nbsp;(add-poly&nbsp;p1&nbsp;p2)<br>
&nbsp;&nbsp;(if&nbsp;(same-variable?&nbsp;(variable&nbsp;p1)&nbsp;(variable&nbsp;p2))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-poly&nbsp;(variable&nbsp;p1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(add-terms&nbsp;(term-list&nbsp;p1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(term-list&nbsp;p2)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(error&nbsp;&quot;Polys&nbsp;not&nbsp;in&nbsp;same&nbsp;var&nbsp;--&nbsp;ADD-POLY&quot;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;p1&nbsp;p2))))<br>
<a name="index_term_2676"></a>(define&nbsp;(mul-poly&nbsp;p1&nbsp;p2)<br>
&nbsp;&nbsp;(if&nbsp;(same-variable?&nbsp;(variable&nbsp;p1)&nbsp;(variable&nbsp;p2))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-poly&nbsp;(variable&nbsp;p1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(mul-terms&nbsp;(term-list&nbsp;p1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(term-list&nbsp;p2)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(error&nbsp;&quot;Polys&nbsp;not&nbsp;in&nbsp;same&nbsp;var&nbsp;--&nbsp;MUL-POLY&quot;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;p1&nbsp;p2))))<br></tt>


To incorporate polynomials into our generic arithmetic system, we need to supply them with type tags. We'll use the tag <tt>polynomial</tt>, and install appropriate operations on tagged polynomials in the operation table. We'll embed all our code in an installation procedure for the polynomial package, similar to the ones in section <a href="#%_sec_2.5.1">2.5.1</a>: <a name="index_term_2678"></a><a name="index_term_2680"></a><a name="index_term_2682"></a>


<tt><a name="index_term_2684"></a><a name="index_term_2686"></a><a name="index_term_2688"></a><a name="index_term_2690"></a>(define&nbsp;(install-polynomial-package)<br>
&nbsp;&nbsp;*;;&nbsp;internal&nbsp;procedures*<br>
&nbsp;&nbsp;*;;&nbsp;representation&nbsp;of&nbsp;poly*<br>
&nbsp;&nbsp;(define&nbsp;(make-poly&nbsp;variable&nbsp;term-list)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;variable&nbsp;term-list))<br>
&nbsp;&nbsp;(define&nbsp;(variable&nbsp;p)&nbsp;(car&nbsp;p))<br>
&nbsp;&nbsp;(define&nbsp;(term-list&nbsp;p)&nbsp;(cdr&nbsp;p))<br>
&nbsp;&nbsp;&lt;*procedures&nbsp;<tt>same-variable?</tt>&nbsp;and&nbsp;<tt>variable?</tt>&nbsp;from&nbsp;section <a href="chapter_2_section_3.html#%_sec_2.3.2">2.3.2</a>*&gt;<br>
&nbsp;&nbsp;*;;&nbsp;representation&nbsp;of&nbsp;terms&nbsp;and&nbsp;term&nbsp;lists*<br>
&nbsp;&nbsp;&lt;*procedures&nbsp;<tt>adjoin-term&nbsp;</tt>...*<tt>coeff</tt></tt>&nbsp;from&nbsp;text&nbsp;below&gt;<br>
<br>
&nbsp;&nbsp;*;;&nbsp;continued&nbsp;on&nbsp;next&nbsp;page*<br>
<br>
&nbsp;&nbsp;(define&nbsp;(add-poly&nbsp;p1&nbsp;p2)&nbsp;<tt>...</tt>)<br>
&nbsp;&nbsp;&lt;*procedures&nbsp;used&nbsp;by&nbsp;<tt>add-poly</tt>*&gt;<br>
&nbsp;&nbsp;(define&nbsp;(mul-poly&nbsp;p1&nbsp;p2)&nbsp;<tt>...</tt>)<br>
&nbsp;&nbsp;&lt;*procedures&nbsp;used&nbsp;by&nbsp;<tt>mul-poly</tt>*&gt;<br>
&nbsp;&nbsp;*;;&nbsp;interface&nbsp;to&nbsp;rest&nbsp;of&nbsp;the&nbsp;system*<br>
&nbsp;&nbsp;(define&nbsp;(tag&nbsp;p)&nbsp;(attach-tag&nbsp;'polynomial&nbsp;p))<br>
&nbsp;&nbsp;(put&nbsp;'add&nbsp;'(polynomial&nbsp;polynomial)&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(p1&nbsp;p2)&nbsp;(tag&nbsp;(add-poly&nbsp;p1&nbsp;p2))))<br>
&nbsp;&nbsp;(put&nbsp;'mul&nbsp;'(polynomial&nbsp;polynomial)&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(p1&nbsp;p2)&nbsp;(tag&nbsp;(mul-poly&nbsp;p1&nbsp;p2))))<br>
&nbsp;&nbsp;(put&nbsp;'make&nbsp;'polynomial<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(lambda&nbsp;(var&nbsp;terms)&nbsp;(tag&nbsp;(make-poly&nbsp;var&nbsp;terms))))<br>
&nbsp;&nbsp;'done)<br>


Polynomial addition is performed termwise. Terms of the same order (i.e., with the same power of the indeterminate) must be combined. This is done by forming a new term of the same order whose coefficient is the sum of the coefficients of the addends. Terms in one addend for which there are no terms of the same order in the other addend are simply accumulated into the sum polynomial being constructed.


In order to manipulate term lists, we will assume that we have a constructor <a name="index_term_2692"></a><tt>the-empty-termlist</tt> that returns an empty term list and a constructor <a name="index_term_2694"></a><tt>adjoin-term</tt> that adjoins a new term to a term list. We will also assume that we have a predicate <a name="index_term_2696"></a><tt>empty-termlist?</tt> that tells if a given term list is empty, a selector <a name="index_term_2698"></a><tt>first-term</tt> that extracts the highest-order term from a term list, and a selector <a name="index_term_2700"></a><tt>rest-terms</tt> that returns all but the highest-order term. To manipulate terms, we will suppose that we have a constructor <a name="index_term_2702"></a><tt>make-term</tt> that constructs a term with given order and coefficient, and selectors <a name="index_term_2704"></a><tt>order</tt> and <a name="index_term_2706"></a><tt>coeff</tt> that return, respectively, the order and the coefficient of the term. These operations allow us to consider both terms and term lists as data abstractions, whose concrete representations we can worry about separately.


Here is the procedure that constructs the term list for the sum of two polynomials:<a name="call_footnote_Temp_300" href="#footnote_Temp_300" id="call_footnote_Temp_300"><sup><small>56</small></sup></a>


<tt><a name="index_term_2708"></a>(define&nbsp;(add-terms&nbsp;L1&nbsp;L2)<br>
&nbsp;&nbsp;(cond&nbsp;((empty-termlist?&nbsp;L1)&nbsp;L2)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;((empty-termlist?&nbsp;L2)&nbsp;L1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(else<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((t1&nbsp;(first-term&nbsp;L1))&nbsp;(t2&nbsp;(first-term&nbsp;L2)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cond&nbsp;((&gt;&nbsp;(order&nbsp;t1)&nbsp;(order&nbsp;t2))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(adjoin-term<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;t1&nbsp;(add-terms&nbsp;(rest-terms&nbsp;L1)&nbsp;L2)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;((&lt;&nbsp;(order&nbsp;t1)&nbsp;(order&nbsp;t2))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(adjoin-term<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;t2&nbsp;(add-terms&nbsp;L1&nbsp;(rest-terms&nbsp;L2))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(else<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(adjoin-term<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-term&nbsp;(order&nbsp;t1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(add&nbsp;(coeff&nbsp;t1)&nbsp;(coeff&nbsp;t2)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(add-terms&nbsp;(rest-terms&nbsp;L1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(rest-terms&nbsp;L2)))))))))<br></tt>


The most important point to note here is that we used the generic addition procedure <a name="index_term_2710"></a><tt>add</tt> to add together the coefficients of the terms being combined. This has powerful consequences, as we will see below.


In order to multiply two term lists, we multiply each term of the first list by all the terms of the other list, repeatedly using <tt>mul-term-by-all-terms</tt>, which multiplies a given term by all terms in a given term list. The resulting term lists (one for each term of the first list) are accumulated into a sum. Multiplying two terms forms a term whose order is the sum of the orders of the factors and whose coefficient is the product of the coefficients of the factors:


<tt><a name="index_term_2712"></a>(define&nbsp;(mul-terms&nbsp;L1&nbsp;L2)<br>
&nbsp;&nbsp;(if&nbsp;(empty-termlist?&nbsp;L1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(the-empty-termlist)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(add-terms&nbsp;(mul-term-by-all-terms&nbsp;(first-term&nbsp;L1)&nbsp;L2)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(mul-terms&nbsp;(rest-terms&nbsp;L1)&nbsp;L2))))<br>
(define&nbsp;(mul-term-by-all-terms&nbsp;t1&nbsp;L)<br>
&nbsp;&nbsp;(if&nbsp;(empty-termlist?&nbsp;L)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(the-empty-termlist)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((t2&nbsp;(first-term&nbsp;L)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(adjoin-term<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(make-term&nbsp;(+&nbsp;(order&nbsp;t1)&nbsp;(order&nbsp;t2))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(mul&nbsp;(coeff&nbsp;t1)&nbsp;(coeff&nbsp;t2)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(mul-term-by-all-terms&nbsp;t1&nbsp;(rest-terms&nbsp;L))))))<br></tt>


This is really all there is to polynomial addition and multiplication. <a name="index_term_2714"></a><a name="index_term_2716"></a>Notice that, since we operate on terms using the generic procedures <tt>add</tt> and <tt>mul</tt>, our polynomial package is automatically able to handle any type of coefficient that is known about by the generic arithmetic package. If we include a <a name="index_term_2718"></a>coercion mechanism such as one of those discussed in section <a href="#%_sec_2.5.2">2.5.2</a>, then we also are automatically able to handle operations on polynomials of different coefficient types, such as

<div align="left">
  <img src="img/chapter_2_image_71.gif" border="0">
</div>

Because we installed the polynomial addition and multiplication procedures <tt>add-poly</tt> and <tt>mul-poly</tt> in the generic arithmetic system as the <tt>add</tt> and <tt>mul</tt> operations for type <tt>polynomial</tt>, our system is also automatically able to handle polynomial operations such as

<div align="left">
  <img src="img/chapter_2_image_72.gif" border="0">
</div>

The reason is that when the system tries to combine coefficients, it will dispatch through <tt>add</tt> and <tt>mul</tt>. Since the coefficients are themselves polynomials (in *y*), these will be combined using <tt>add-poly</tt> and <tt>mul-poly</tt>. The result is a kind of <a name="index_term_2720"></a><a name="index_term_2722"></a>"data-directed recursion" in which, for example, a call to <tt>mul-poly</tt> will result in recursive calls to <tt>mul-poly</tt> in order to multiply the coefficients. If the coefficients of the coefficients were themselves polynomials (as might be used to represent polynomials in three variables), the data direction would ensure that the system would follow through another level of recursive calls, and so on through as many levels as the structure of the data dictates.<a name="call_footnote_Temp_301" href="#footnote_Temp_301" id="call_footnote_Temp_301"><sup><small>57</small></sup></a> <a name="%_sec_Temp_302"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_302">Representing term lists</a></h4>

<a name="index_term_2724"></a> Finally, we must confront the job of implementing a good representation for term lists. A term list is, in effect, a set of coefficients keyed by the order of the term. Hence, any of the methods for representing sets, as discussed in section <a href="chapter_2_section_3.html#%_sec_2.3.3">2.3.3</a>, can be applied to this task. On the other hand, our procedures <tt>add-terms</tt> and <tt>mul-terms</tt> always access term lists sequentially from highest to lowest order. Thus, we will use some kind of ordered list representation.


How should we structure the list that represents a term list? One consideration is the "density" of the polynomials we intend to manipulate. A polynomial is said to be <a name="index_term_2726"></a><a name="index_term_2728"></a>*dense* if it has nonzero coefficients in terms of most orders. If it has many zero terms it is said to be <a name="index_term_2730"></a><a name="index_term_2732"></a>*sparse*. For example,

<div align="left">
  <img src="img/chapter_2_image_74.gif" border="0">
</div>

is a dense polynomial, whereas

<div align="left">
  <img src="img/chapter_2_image_75.gif" border="0">
</div>

is sparse.


The term lists of dense polynomials are most efficiently represented as lists of the coefficients. For example, *A* above would be nicely represented as <tt>(1 2 0 3 -2 -5)</tt>. The order of a term in this representation is the length of the sublist beginning with that term's coefficient, decremented by 1.<a name="call_footnote_Temp_303" href="#footnote_Temp_303" id="call_footnote_Temp_303"><sup><small>58</small></sup></a> This would be a terrible representation for a sparse polynomial such as *B*: There would be a giant list of zeros punctuated by a few lonely nonzero terms. A more reasonable representation of the term list of a sparse polynomial is as a list of the nonzero terms, where each term is a list containing the order of the term and the coefficient for that order. In such a scheme, polynomial *B* is efficiently represented as <tt>((100 1) (2 2) (0 1))</tt>. As most polynomial manipulations are performed on sparse polynomials, we will use this method. We will assume that term lists are represented as lists of terms, arranged from highest-order to lowest-order term. Once we have made this decision, implementing the selectors and constructors for terms and term lists is straightforward:<a name="call_footnote_Temp_304" href="#footnote_Temp_304" id="call_footnote_Temp_304"><sup><small>59</small></sup></a>


<tt><a name="index_term_2734"></a>(define&nbsp;(adjoin-term&nbsp;term&nbsp;term-list)<br>
&nbsp;&nbsp;(if&nbsp;(=zero?&nbsp;(coeff&nbsp;term))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;term-list<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(cons&nbsp;term&nbsp;term-list)))<br>
<a name="index_term_2736"></a>(define&nbsp;(the-empty-termlist)&nbsp;'())<br>
<a name="index_term_2738"></a>(define&nbsp;(first-term&nbsp;term-list)&nbsp;(car&nbsp;term-list))<br>
<a name="index_term_2740"></a>(define&nbsp;(rest-terms&nbsp;term-list)&nbsp;(cdr&nbsp;term-list))<br>
<a name="index_term_2742"></a>(define&nbsp;(empty-termlist?&nbsp;term-list)&nbsp;(null?&nbsp;term-list))<br>
<a name="index_term_2744"></a>(define&nbsp;(make-term&nbsp;order&nbsp;coeff)&nbsp;(list&nbsp;order&nbsp;coeff))<br>
<a name="index_term_2746"></a>(define&nbsp;(order&nbsp;term)&nbsp;(car&nbsp;term))<br>
<a name="index_term_2748"></a>(define&nbsp;(coeff&nbsp;term)&nbsp;(cadr&nbsp;term))<br></tt>


where <tt>=zero?</tt> is as defined in exercise&nbsp;<a href="#%_thm_2.80">2.80</a>. (See also exercise&nbsp;<a href="#%_thm_2.87">2.87</a> below.)


Users of the polynomial package will create (tagged) polynomials by means of the procedure:


<tt><a name="index_term_2750"></a>(define&nbsp;(make-polynomial&nbsp;var&nbsp;terms)<br>
&nbsp;&nbsp;((get&nbsp;'make&nbsp;'polynomial)&nbsp;var&nbsp;terms))<br></tt>


<a name="exercise_2_87">**Exercise 2.87**</a> <a name="index_term_2752"></a><a name="index_term_2754"></a>Install <tt>=zero?</tt> for polynomials in the generic arithmetic package. This will allow <tt>adjoin-term</tt> to work for polynomials with coefficients that are themselves polynomials.


<a name="exercise_2_88">**Exercise 2.88**</a> <a name="index_term_2756"></a>Extend the polynomial system to include subtraction of polynomials. (Hint: You may find it helpful to define a generic negation operation.)


<a name="exercise_2_89">**Exercise 2.89**</a> Define procedures that implement the term-list representation described above as appropriate for dense polynomials.


<a name="exercise_2_90">**Exercise 2.90**</a> Suppose we want to have a polynomial system that is efficient for both sparse and dense polynomials. One way to do this is to allow both kinds of term-list representations in our system. The situation is analogous to the complex-number example of section <a href="chapter_2_section_4.html#%_sec_2.4">2.4</a>, where we allowed both rectangular and polar representations. To do this we must distinguish different types of term lists and make the operations on term lists generic. Redesign the polynomial system to implement this generalization. This is a major effort, not a local change.


<a name="exercise_2_91">**Exercise 2.91**</a> <a name="index_term_2758"></a>A univariate polynomial can be divided by another one to produce a polynomial quotient and a polynomial remainder. For example,

<div align="left">
  <img src="img/chapter_2_image_76.gif" border="0">
</div>

Division can be performed via long division. That is, divide the highest-order term of the dividend by the highest-order term of the divisor. The result is the first term of the quotient. Next, multiply the result by the divisor, subtract that from the dividend, and produce the rest of the answer by recursively dividing the difference by the divisor. Stop when the order of the divisor exceeds the order of the dividend and declare the dividend to be the remainder. Also, if the dividend ever becomes zero, return zero as both quotient and remainder.


<a name="index_term_2760"></a>We can design a <tt>div-poly</tt> procedure on the model of <tt>add-poly</tt> and <tt>mul-poly</tt>. The procedure checks to see if the two polys have the same variable. If so, <tt>div-poly</tt> strips off the variable and passes the problem to <tt>div-terms</tt>, which performs the division operation on term lists. <tt>Div-poly</tt> finally reattaches the variable to the result supplied by <tt>div-terms</tt>. It is convenient to design <tt>div-terms</tt> to compute both the quotient and the remainder of a division. <tt>Div-terms</tt> can take two term lists as arguments and return a list of the quotient term list and the remainder term list.


Complete the following definition of <tt>div-terms</tt> by filling in the missing expressions. Use this to implement <tt>div-poly</tt>, which takes two polys as arguments and returns a list of the quotient and remainder polys.


<tt><a name="index_term_2762"></a>(define&nbsp;(div-terms&nbsp;L1&nbsp;L2)<br>
&nbsp;&nbsp;(if&nbsp;(empty-termlist?&nbsp;L1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;(the-empty-termlist)&nbsp;(the-empty-termlist))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((t1&nbsp;(first-term&nbsp;L1))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(t2&nbsp;(first-term&nbsp;L2)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;(&gt;&nbsp;(order&nbsp;t2)&nbsp;(order&nbsp;t1))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;(the-empty-termlist)&nbsp;L1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((new-c&nbsp;(div&nbsp;(coeff&nbsp;t1)&nbsp;(coeff&nbsp;t2)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(new-o&nbsp;(-&nbsp;(order&nbsp;t1)&nbsp;(order&nbsp;t2))))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(let&nbsp;((rest-of-result<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;*compute&nbsp;rest&nbsp;of&nbsp;result&nbsp;recursively*&gt;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;*form&nbsp;complete&nbsp;result*&gt;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;))))))<br></tt>


<a name="%_sec_Temp_310"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_310">Hierarchies of types in symbolic algebra</a></h4>

<a name="index_term_2764"></a><a name="index_term_2766"></a><a name="index_term_2768"></a> Our polynomial system illustrates how objects of one type (polynomials) may in fact be complex objects that have objects of many different types as parts. This poses no real difficulty in defining generic operations. We need only install appropriate generic operations for performing the necessary manipulations of the parts of the compound types. In fact, we saw that polynomials form a kind of "recursive data abstraction," in that parts of a polynomial may themselves be polynomials. Our generic operations and our data-directed programming style can handle this complication without much trouble.


On the other hand, polynomial algebra is a system for which the data types cannot be naturally arranged in a tower. For instance, it is possible to have polynomials in *x* whose coefficients are polynomials in *y*. It is also possible to have polynomials in *y* whose coefficients are polynomials in *x*. Neither of these types is "above" the other in any natural way, yet it is often necessary to add together elements from each set. There are several ways to do this. One possibility is to convert one polynomial to the type of the other by expanding and rearranging terms so that both polynomials have the same principal variable. One can impose a towerlike structure on this by ordering the variables and thus always converting any polynomial to a <a name="index_term_2770"></a><a name="index_term_2772"></a>"canonical form" with the highest-priority variable dominant and the lower-priority variables buried in the coefficients. This strategy works fairly well, except that the conversion may expand a polynomial unnecessarily, making it hard to read and perhaps less efficient to work with. The tower strategy is certainly not natural for this domain or for any domain where the user can invent new types dynamically using old types in various combining forms, such as trigonometric functions, power series, and integrals.


It should not be surprising that controlling <a name="index_term_2774"></a>coercion is a serious problem in the design of large-scale algebraic-manipulation systems. Much of the complexity of such systems is concerned with relationships among diverse types. Indeed, it is fair to say that we do not yet completely understand coercion. In fact, we do not yet completely understand the concept of a data type. Nevertheless, what we know provides us with powerful structuring and modularity principles to support the design of large systems.


<a name="exercise_2_92">**Exercise 2.92**</a> By imposing an ordering on variables, extend the polynomial package so that addition and multiplication of polynomials works for polynomials in different variables. (This is not easy!)


<a name="%_sec_Temp_312"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_312">Extended exercise: Rational functions</a></h4>

<a name="index_term_2776"></a><a name="index_term_2778"></a><a name="index_term_2780"></a>We can extend our generic arithmetic system to include *rational functions*. These are "fractions" whose numerator and denominator are polynomials, such as

<div align="left">
  <img src="img/chapter_2_image_77.gif" border="0">
</div>

The system should be able to add, subtract, multiply, and divide rational functions, and to perform such computations as

<div align="left">
  <img src="img/chapter_2_image_78.gif" border="0">
</div>

(Here the sum has been simplified by removing common factors. Ordinary "cross multiplication" would have produced a fourth-degree polynomial over a fifth-degree polynomial.)


If we modify our rational-arithmetic package so that it uses generic operations, then it will do what we want, except for the problem of reducing fractions to lowest terms.


<a name="exercise_2_93">**Exercise 2.93**</a> Modify the rational-arithmetic package to use generic operations, but change <tt>make-rat</tt> so that it does not attempt to reduce fractions to lowest terms. Test your system by calling <tt>make-rational</tt> on two polynomials to produce a rational function


<tt>(define&nbsp;p1&nbsp;(make-polynomial&nbsp;'x&nbsp;'((2&nbsp;1)(0&nbsp;1))))<br>
(define&nbsp;p2&nbsp;(make-polynomial&nbsp;'x&nbsp;'((3&nbsp;1)(0&nbsp;1))))<br>
(define&nbsp;rf&nbsp;(make-rational&nbsp;p2&nbsp;p1))<br></tt>


Now add <tt>rf</tt> to itself, using <tt>add</tt>. You will observe that this addition procedure does not reduce fractions to lowest terms.


We can reduce polynomial fractions to lowest terms using the same idea we used with integers: modifying <tt>make-rat</tt> to divide both the numerator and the denominator by their greatest common divisor. The notion of <a name="index_term_2782"></a><a name="index_term_2784"></a>"greatest common divisor" makes sense for polynomials. In fact, we can compute the GCD of two polynomials using essentially the same Euclid's Algorithm that works for integers.<a name="call_footnote_Temp_314" href="#footnote_Temp_314" id="call_footnote_Temp_314"><sup><small>60</small></sup></a> The integer version is


<tt>(define&nbsp;(gcd&nbsp;a&nbsp;b)<br>
&nbsp;&nbsp;(if&nbsp;(=&nbsp;b&nbsp;0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(gcd&nbsp;b&nbsp;(remainder&nbsp;a&nbsp;b))))<br></tt>


Using this, we could make the obvious modification to define a GCD operation that works on term lists:


<tt><a name="index_term_2794"></a>(define&nbsp;(gcd-terms&nbsp;a&nbsp;b)<br>
&nbsp;&nbsp;(if&nbsp;(empty-termlist?&nbsp;b)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(gcd-terms&nbsp;b&nbsp;(remainder-terms&nbsp;a&nbsp;b))))<br></tt>


where <tt>remainder-terms</tt> picks out the remainder component of the list returned by the term-list division operation <tt>div-terms</tt> that was implemented in exercise&nbsp;<a href="#%_thm_2.91">2.91</a>.


<a name="exercise_2_94">**Exercise 2.94**</a> <a name="index_term_2796"></a><a name="index_term_2798"></a>Using <tt>div-terms</tt>, implement the procedure <tt>remainder-terms</tt> and use this to define <tt>gcd-terms</tt> as above. Now write a procedure <tt>gcd-poly</tt> that computes the polynomial GCD of two polys. (The procedure should signal an error if the two polys are not in the same variable.) Install in the system a generic operation <tt>greatest-common-divisor</tt> that reduces to <tt>gcd-poly</tt> for polynomials and to ordinary <tt>gcd</tt> for ordinary numbers. As a test, try


<tt>(define&nbsp;p1&nbsp;(make-polynomial&nbsp;'x&nbsp;'((4&nbsp;1)&nbsp;(3&nbsp;-1)&nbsp;(2&nbsp;-2)&nbsp;(1&nbsp;2))))<br>
(define&nbsp;p2&nbsp;(make-polynomial&nbsp;'x&nbsp;'((3&nbsp;1)&nbsp;(1&nbsp;-1))))<br>
(greatest-common-divisor&nbsp;p1&nbsp;p2)<br></tt>


and check your result by hand.


<a name="exercise_2_95">**Exercise 2.95**</a> Define *P*<sub>1</sub>, *P*<sub>2</sub>, and *P*<sub>3</sub> to be the polynomials

<div align="left">
  <img src="img/chapter_2_image_79.gif" border="0">
</div>

<div align="left">
  <img src="img/chapter_2_image_80.gif" border="0">
</div>

<div align="left">
  <img src="img/chapter_2_image_81.gif" border="0">
</div>

Now define *Q*<sub>1</sub> to be the product of *P*<sub>1</sub> and *P*<sub>2</sub> and *Q*<sub>2</sub> to be the product of *P*<sub>1</sub> and *P*<sub>3</sub>, and use <tt>greatest-common-divisor</tt> (exercise&nbsp;<a href="#%_thm_2.94">2.94</a>) to compute the GCD of *Q*<sub>1</sub> and *Q*<sub>2</sub>. Note that the answer is not the same as *P*<sub>1</sub>. This example introduces noninteger operations into the computation, causing difficulties with the GCD algorithm.<a name="call_footnote_Temp_317" href="#footnote_Temp_317" id="call_footnote_Temp_317"><sup><small>61</small></sup></a> To understand what is happening, try tracing <tt>gcd-terms</tt> while computing the GCD or try performing the division by hand.


We can solve the problem exhibited in exercise&nbsp;<a href="#%_thm_2.95">2.95</a> if we use the following modification of the GCD algorithm (which really works only in the case of polynomials with integer coefficients). Before performing any polynomial division in the GCD computation, we multiply the dividend by an integer constant factor, chosen to guarantee that no fractions will arise during the division process. Our answer will thus differ from the actual GCD by an integer constant factor, but this does not matter in the case of reducing rational functions to lowest terms; the GCD will be used to divide both the numerator and denominator, so the integer constant factor will cancel out.


More precisely, if *P* and *Q* are polynomials, let *O*<sub>1</sub> be the order of *P* (i.e., the order of the largest term of *P*) and let *O*<sub>2</sub> be the order of *Q*. Let *c* be the leading coefficient of *Q*. Then it can be shown that, if we multiply *P* by the <a name="index_term_2800"></a>*integerizing factor* *c*<sup>1+*O*<sub>1</sub> -*O*<sub>2</sub></sup>, the resulting polynomial can be divided by *Q* by using the <tt>div-terms</tt> algorithm without introducing any fractions. The operation of multiplying the dividend by this constant and then dividing is sometimes called the <a name="index_term_2802"></a><a name="index_term_2804"></a>*pseudodivision* of *P* by *Q*. The remainder of the division is called the *pseudoremainder*.


<a name="exercise_2_96">**Exercise 2.96**</a> a.&nbsp;&nbsp;&nbsp;&nbsp;Implement the procedure <tt>pseudoremainder-terms</tt>, which is just like <tt>remainder-terms</tt> except that it multiplies the dividend by the integerizing factor described above before calling <tt>div-terms</tt>. Modify <tt>gcd-terms</tt> to use <tt>pseudoremainder-terms</tt>, and verify that <tt>greatest-common-divisor</tt> now produces an answer with integer coefficients on the example in exercise&nbsp;<a href="#%_thm_2.95">2.95</a>.


b.&nbsp;&nbsp;&nbsp;&nbsp;The GCD now has integer coefficients, but they are larger than those of *P*<sub>1</sub>. Modify <tt>gcd-terms</tt> so that it removes common factors from the coefficients of the answer by dividing all the coefficients by their (integer) greatest common divisor.


<a name="index_term_2806"></a><a name="index_term_2808"></a>Thus, here is how to reduce a rational function to lowest terms:

<ul>
  <li>Compute the GCD of the numerator and denominator, using the version of <tt>gcd-terms</tt> from exercise&nbsp;<a href="#%_thm_2.96">2.96</a>.</li>

  <li>When you obtain the GCD, multiply both numerator and denominator by the same integerizing factor before dividing through by the GCD, so that division by the GCD will not introduce any noninteger coefficients. As the factor you can use the leading coefficient of the GCD raised to the power 1 + *O*<sub>1</sub> - *O*<sub>2</sub>, where *O*<sub>2</sub> is the order of the GCD and *O*<sub>1</sub> is the maximum of the orders of the numerator and denominator. This will ensure that dividing the numerator and denominator by the GCD will not introduce any fractions.</li>

  <li>The result of this operation will be a numerator and denominator with integer coefficients. The coefficients will normally be very large because of all of the integerizing factors, so the last step is to remove the redundant factors by computing the (integer) greatest common divisor of all the coefficients of the numerator and the denominator and dividing through by this factor.</li>
</ul>

<a name="exercise_2_97">**Exercise 2.97**</a> a. Implement this algorithm as a procedure <tt>reduce-terms</tt> that takes two term lists <tt>n</tt> and <tt>d</tt> as arguments and returns a list <tt>nn</tt>, <tt>dd</tt>, which are <tt>n</tt> and <tt>d</tt> reduced to lowest terms via the algorithm given above. Also write a procedure <tt>reduce-poly</tt>, analogous to <tt>add-poly</tt>, that checks to see if the two polys have the same variable. If so, <tt>reduce-poly</tt> strips off the variable and passes the problem to <tt>reduce-terms</tt>, then reattaches the variable to the two term lists supplied by <tt>reduce-terms</tt>.


b. Define a procedure analogous to <tt>reduce-terms</tt> that does what the original <tt>make-rat</tt> did for integers:


<tt>(define&nbsp;(reduce-integers&nbsp;n&nbsp;d)<br>
&nbsp;&nbsp;(let&nbsp;((g&nbsp;(gcd&nbsp;n&nbsp;d)))<br>
&nbsp;&nbsp;&nbsp;&nbsp;(list&nbsp;(/&nbsp;n&nbsp;g)&nbsp;(/&nbsp;d&nbsp;g))))<br></tt>


and define <tt>reduce</tt> as a generic operation that calls <tt>apply-generic</tt> to dispatch to either <tt>reduce-poly</tt> (for <tt>polynomial</tt> arguments) or <tt>reduce-integers</tt> (for <tt>scheme-number</tt> arguments). You can now easily make the rational-arithmetic package reduce fractions to lowest terms by having <tt>make-rat</tt> call <tt>reduce</tt> before combining the given numerator and denominator to form a rational number. The system now handles rational expressions in either integers or polynomials. To test your program, try the example at the beginning of this extended exercise:


<tt>(define&nbsp;p1&nbsp;(make-polynomial&nbsp;'x&nbsp;'((1&nbsp;1)(0&nbsp;1))))<br>
(define&nbsp;p2&nbsp;(make-polynomial&nbsp;'x&nbsp;'((3&nbsp;1)(0&nbsp;-1))))<br>
(define&nbsp;p3&nbsp;(make-polynomial&nbsp;'x&nbsp;'((1&nbsp;1))))<br>
(define&nbsp;p4&nbsp;(make-polynomial&nbsp;'x&nbsp;'((2&nbsp;1)(0&nbsp;-1))))<br>
<br>
(define&nbsp;rf1&nbsp;(make-rational&nbsp;p1&nbsp;p2))<br>
(define&nbsp;rf2&nbsp;(make-rational&nbsp;p3&nbsp;p4))<br>
<br>
(add&nbsp;rf1&nbsp;rf2)<br></tt>


See if you get the correct answer, correctly reduced to lowest terms.


The GCD computation is at the heart of any system that does operations on rational functions. The algorithm used above, although mathematically straightforward, is extremely slow. The slowness is due partly to the large number of division operations and partly to the enormous size of the intermediate coefficients generated by the pseudodivisions. One of the active areas in the development of algebraic-manipulation systems is the design of better algorithms for computing polynomial GCDs.<a name="call_footnote_Temp_320" href="#footnote_Temp_320" id="call_footnote_Temp_320"><sup><small>62</small></sup></a>

<div class="smallprint">
  <hr>
</div>

<div class="footnote">
  
<a name="footnote_Temp_283" href="#call_footnote_Temp_283" id="footnote_Temp_283"><sup><small>49</small></sup></a> We also have to supply an almost identical procedure to handle the types <tt>(scheme-number complex)</tt>.

  
<a name="footnote_Temp_285" href="#call_footnote_Temp_285" id="footnote_Temp_285"><sup><small>50</small></sup></a> See exercise&nbsp;<a href="#%_thm_2.82">2.82</a> for generalizations.

  
<a name="footnote_Temp_286" href="#call_footnote_Temp_286" id="footnote_Temp_286"><sup><small>51</small></sup></a> If we are clever, we can usually get by with fewer than *n*<sup>2</sup> coercion procedures. For instance, if we know how to convert from type 1 to type 2 and from type 2 to type 3, then we can use this knowledge to convert from type 1 to type 3. This can greatly decrease the number of coercion procedures we need to supply explicitly when we add a new type to the system. If we are willing to build the required amount of sophistication into our system, we can have it search the "graph" of relations among types and automatically generate those coercion procedures that can be inferred from the ones that are supplied explicitly.

  
<a name="footnote_Temp_289" href="#call_footnote_Temp_289" id="footnote_Temp_289"><sup><small>52</small></sup></a> This statement, which also appears in the first edition of this book, is just as true now as it was when we wrote it twelve years ago. Developing a useful, general framework for expressing the relations among different types of entities (what philosophers call "ontology") seems intractably difficult. The main difference between the confusion that existed ten years ago and the confusion that exists now is that now a variety of inadequate ontological theories have been embodied in a plethora of correspondingly inadequate programming languages. For example, much of the complexity of <a name="index_term_2622"></a><a name="index_term_2624"></a>object-oriented programming languages -- and the subtle and confusing differences among contemporary object-oriented languages -- centers on the treatment of generic operations on interrelated types. Our own discussion of computational objects in chapter&nbsp;3 avoids these issues entirely. Readers familiar with object-oriented programming will notice that we have much to say in chapter&nbsp;3 about local state, but we do not even mention "classes" or "inheritance." In fact, we suspect that these problems cannot be adequately addressed in terms of computer-language design alone, without also drawing on work in knowledge representation and automated reasoning.

  
<a name="footnote_Temp_295" href="#call_footnote_Temp_295" id="footnote_Temp_295"><sup><small>53</small></sup></a> A real number can be projected to an integer using the <a name="index_term_2642"></a><a name="index_term_2644"></a><tt>round</tt> primitive, which returns the closest integer to its argument.

  
<a name="footnote_Temp_298" href="#call_footnote_Temp_298" id="footnote_Temp_298"><sup><small>54</small></sup></a> On the other hand, we will allow polynomials whose coefficients are themselves polynomials in other variables. This will give us essentially the same representational power as a full multivariate system, although it does lead to coercion problems, as discussed below.

  
<a name="footnote_Temp_299" href="#call_footnote_Temp_299" id="footnote_Temp_299"><sup><small>55</small></sup></a> For univariate polynomials, giving the value of a polynomial at a given set of points can be a particularly good representation. This makes polynomial arithmetic extremely simple. To obtain, for example, the sum of two polynomials represented in this way, we need only add the values of the polynomials at corresponding points. To transform back to a more familiar representation, we can use the <a name="index_term_2662"></a>Lagrange interpolation formula, which shows how to recover the coefficients of a polynomial of degree *n* given the values of the polynomial at *n* + 1 points.

  
<a name="footnote_Temp_300" href="#call_footnote_Temp_300" id="footnote_Temp_300"><sup><small>56</small></sup></a> This operation is very much like the ordered <tt>union-set</tt> operation we developed in exercise &nbsp;<a href="chapter_2_section_3.html#exercise_2_62">2.62</a>. In fact, if we think of the terms of the polynomial as a set ordered according to the power of the indeterminate, then the program that produces the term list for a sum is almost identical to <tt>union-set</tt>.

  
<a name="footnote_Temp_301" href="#call_footnote_Temp_301" id="footnote_Temp_301"><sup><small>57</small></sup></a> To make this work completely smoothly, we should also add to our generic arithmetic system the ability to coerce a "number" to a polynomial by regarding it as a polynomial of degree zero whose coefficient is the number. This is necessary if we are going to perform operations such as

  <div align="left">
    <img src="img/chapter_2_image_73.gif" border="0">
  </div>

  
which requires adding the coefficient *y* + 1 to the coefficient 2.

  
<a name="footnote_Temp_303" href="#call_footnote_Temp_303" id="footnote_Temp_303"><sup><small>58</small></sup></a> In these polynomial examples, we assume that we have implemented the generic arithmetic system using the type mechanism suggested in exercise&nbsp;<a href="#%_thm_2.78">2.78</a>. Thus, coefficients that are ordinary numbers will be represented as the numbers themselves rather than as pairs whose <tt>car</tt> is the symbol <tt>scheme-number</tt>.

  
<a name="footnote_Temp_304" href="#call_footnote_Temp_304" id="footnote_Temp_304"><sup><small>59</small></sup></a> Although we are assuming that term lists are ordered, we have implemented <tt>adjoin-term</tt> to simply <tt>cons</tt> the new term onto the existing term list. We can get away with this so long as we guarantee that the procedures (such as <tt>add-terms</tt>) that use <tt>adjoin-term</tt> always call it with a higher-order term than appears in the list. If we did not want to make such a guarantee, we could have implemented <tt>adjoin-term</tt> to be similar to the <tt>adjoin-set</tt> constructor for the ordered-list representation of sets (exercise&nbsp;<a href="chapter_2_section_3.html#exercise_2_61">2.61</a>).

  
<a name="footnote_Temp_314" href="#call_footnote_Temp_314" id="footnote_Temp_314"><sup><small>60</small></sup></a> The fact that <a name="index_term_2786"></a><a name="index_term_2788"></a>Euclid's Algorithm works for polynomials is formalized in algebra by saying that polynomials form a kind of algebraic domain called a <a name="index_term_2790"></a><a name="index_term_2792"></a>*Euclidean ring*. A Euclidean ring is a domain that admits addition, subtraction, and commutative multiplication, together with a way of assigning to each element *x* of the ring a positive integer "measure" *m*(*x*) with the properties that *m*(*x**y*)<u>&gt;</u> *m*(*x*) for any nonzero *x* and *y* and that, given any *x* and *y*, there exists a *q* such that *y* = *q**x* + *r* and either *r* = 0 or *m*(*r*)&lt; *m*(*x*). From an abstract point of view, this is what is needed to prove that Euclid's Algorithm works. For the domain of integers, the measure *m* of an integer is the absolute value of the integer itself. For the domain of polynomials, the measure of a polynomial is its degree.

  
<a name="footnote_Temp_317" href="#call_footnote_Temp_317" id="footnote_Temp_317"><sup><small>61</small></sup></a> In an implementation like MIT Scheme, this produces a polynomial that is indeed a divisor of *Q*<sub>1</sub> and *Q*<sub>2</sub>, but with rational coefficients. In many other Scheme systems, in which division of integers can produce limited-precision decimal numbers, we may fail to get a valid divisor.

  
<a name="footnote_Temp_320" href="#call_footnote_Temp_320" id="footnote_Temp_320"><sup><small>62</small></sup></a> One extremely efficient and elegant method for computing <a name="index_term_2810"></a><a name="index_term_2812"></a><a name="index_term_2814"></a><a name="index_term_2816"></a>polynomial GCDs was discovered by <a name="index_term_2818"></a>Richard Zippel (1979). The method is a probabilistic algorithm, as is the fast test for primality that we discussed in chapter&nbsp;1. Zippel's book (1993) describes this method, together with other ways to compute polynomial GCDs.

</div>