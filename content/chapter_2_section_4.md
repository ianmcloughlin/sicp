(section_24)=
# Multiple Representations for Abstract Data

<a name="index_term_2286"></a><a name="index_term_2288"></a> We have introduced data abstraction, a methodology for structuring systems in such a way that much of a program can be specified independent of the choices involved in implementing the data objects that the program manipulates. For example, we saw in section&#160;<a href="chapter_2_section_1.html#%_sec_2.1.1">2.1.1</a> how to separate the task of designing a program that uses rational numbers from the task of implementing rational numbers in terms of the computer language's primitive mechanisms for constructing compound data. The key idea was to erect an abstraction barrier -- in this case, the selectors and constructors for rational numbers (<tt>make-rat</tt>, <tt>numer</tt>, <tt>denom</tt>) -- that isolates the way rational numbers are used from their underlying representation in terms of list structure. A similar abstraction barrier isolates the details of the procedures that perform rational arithmetic (<tt>add-rat</tt>, <tt>sub-rat</tt>, <tt>mul-rat</tt>, and <tt>div-rat</tt>) from the "higher-level" procedures that use rational numbers. The resulting program has the structure shown in figure&#160;<a href="chapter_2_section_1.html#%_fig_2.1">2.1</a>.


These data-abstraction barriers are powerful tools for controlling complexity. By isolating the underlying representations of data objects, we can divide the task of designing a large program into smaller tasks that can be performed separately. But this kind of data abstraction is not yet powerful enough, because it may not always make sense to speak of "the underlying representation" for a data object.


For one thing, there might be more than one useful representation for a data object, and we might like to design systems that can deal with multiple representations. To take a simple example, complex numbers may be represented in two almost equivalent ways: in rectangular form (real and imaginary parts) and in polar form (magnitude and angle). Sometimes rectangular form is more appropriate and sometimes polar form is more appropriate. Indeed, it is perfectly plausible to imagine a system in which complex numbers are represented in both ways, and in which the procedures for manipulating complex numbers work with either representation.


More importantly, programming systems are often designed by many people working over extended periods of time, subject to requirements that change over time. In such an environment, it is simply not possible for everyone to agree in advance on choices of data representation. So in addition to the data-abstraction barriers that isolate representation from use, we need abstraction barriers that isolate different design choices from each other and permit different choices to coexist in a single program. Furthermore, since large programs are often created by combining pre-existing modules that were designed in isolation, we need conventions that permit programmers to incorporate modules into larger systems <a name="index_term_2290"></a>*additively*, that is, without having to redesign or reimplement these modules.


In this section, we will learn how to cope with data that may be represented in different ways by different parts of a program. This requires constructing <a name="index_term_2292"></a><a name="index_term_2294"></a>*generic procedures* -- procedures that can operate on data that may be represented in more than one way. Our main technique for building generic procedures will be to work in terms of data objects that have <a name="index_term_2296"></a>*type tags*, that is, data objects that include explicit information about how they are to be processed. We will also discuss <a name="index_term_2298"></a>*data-directed* programming, a powerful and convenient implementation strategy for additively assembling systems with generic operations.


We begin with the simple complex-number example. We will see how type tags and data-directed style enable us to design separate rectangular and polar representations for complex numbers while maintaining the notion of an abstract "complex-number" data object. <a name="index_term_2300"></a><a name="index_term_2302"></a>We will accomplish this by defining arithmetic procedures for complex numbers (<tt>add-complex</tt>, <tt>sub-complex</tt>, <tt>mul-complex</tt>, and <tt>div-complex</tt>) in terms of generic selectors that access parts of a complex number independent of how the number is represented. The resulting complex-number system, as shown in figure&#160;<a href="#%_fig_2.19">2.19</a>, contains two different kinds of <a name="index_term_2304"></a>abstraction barriers. The "horizontal" abstraction barriers play the same role as the ones in figure&#160;<a href="chapter_2_section_1.html#%_fig_2.1">2.1</a>. They isolate "higher-level" operations from "lower-level" representations. In addition, there is a "vertical" barrier that gives us the ability to separately design and install alternative representations.


<a name="%_fig_2.19"></a>

<div align="left">
  <div align="left">
    **Figure 2.19:**&#160;&#160;Data-abstraction barriers in the complex-number system.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_54.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

In section&#160;<a href="chapter_2_section_5.html#%_sec_2.5">2.5</a> we will show how to use type tags and data-directed style to develop a generic arithmetic package. This provides procedures (<tt>add</tt>, <tt>mul</tt>, and so on) that can be used to manipulate all sorts of "numbers" and can be easily extended when a new kind of number is needed. In section&#160;<a href="chapter_2_section_5.html#%_sec_2.5.3">2.5.3</a>, we'll show how to use generic arithmetic in a system that performs symbolic algebra.


<a name="%_sec_2.4.1"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_2.4.1">2.4.1&#160;&#160;Representations for Complex Numbers</a></h3>

<a name="index_term_2306"></a>We will develop a system that performs arithmetic operations on complex numbers as a simple but unrealistic example of a program that uses generic operations. We begin by discussing two plausible representations for complex numbers as ordered pairs: rectangular form (real part and imaginary part) and polar form (magnitude and angle).<a name="call_footnote_Temp_268" href="#footnote_Temp_268" id="call_footnote_Temp_268"><sup><small>43</small></sup></a> Section&#160;<a href="#%_sec_2.4.2">2.4.2</a> will show how both representations can be made to coexist in a single system through the use of type tags and generic operations.


Like rational numbers, complex numbers are naturally represented as ordered pairs. The set of complex numbers can be thought of as a two-dimensional space with two orthogonal axes, the "real" axis and the "imaginary" axis. (See figure&#160;<a href="#%_fig_2.20">2.20</a>.) From this point of view, the complex number *z* = *x* + *i**y* (where *i*<sup>2</sup> = - 1) can be thought of as the point in the plane whose real coordinate is *x* and whose imaginary coordinate is *y*. Addition of complex numbers reduces in this representation to addition of coordinates:

<div align="left">
  <img src="img/chapter_2_image_55.gif" border="0">
</div>

<div align="left">
  <img src="img/chapter_2_image_56.gif" border="0">
</div>

When multiplying complex numbers, it is more natural to think in terms of representing a complex number in polar form, as a magnitude and an angle (*r* and *A* in figure&#160;<a href="#%_fig_2.20">2.20</a>). The product of two complex numbers is the vector obtained by stretching one complex number by the length of the other and then rotating it through the angle of the other:

<div align="left">
  <img src="img/chapter_2_image_57.gif" border="0">
</div>

<div align="left">
  <img src="img/chapter_2_image_58.gif" border="0">
</div>

Thus, there are two different representations for complex numbers, which are appropriate for different operations. Yet, from the viewpoint of someone writing a program that uses complex numbers, the principle of data abstraction suggests that all the operations for manipulating complex numbers should be available regardless of which representation is used by the computer. For example, it is often useful to be able to find the magnitude of a complex number that is specified by rectangular coordinates. Similarly, it is often useful to be able to determine the real part of a complex number that is specified by polar coordinates.


<a name="%_fig_2.20"></a>

<div align="left">
  <div align="left">
    **Figure 2.20:**&#160;&#160;Complex numbers as points in the plane.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_59.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

To design such a system, we can follow the same <a name="index_term_2310"></a>data-abstraction strategy we followed in designing the rational-number package in section&#160;<a href="chapter_2_section_1.html#%_sec_2.1.1">2.1.1</a>. Assume that the operations on complex numbers are implemented in terms of four selectors: <tt>real-part</tt>, <tt>imag-part</tt>, <tt>magnitude</tt>, and <tt>angle</tt>. Also assume that we have two procedures for constructing complex numbers: <tt>make-from-real-imag</tt> returns a complex number with specified real and imaginary parts, and <tt>make-from-mag-ang</tt> returns a complex number with specified magnitude and angle. These procedures have the property that, for any complex number <tt>z</tt>, both


<tt>(make-from-real-imag&#160;(real-part&#160;z)&#160;(imag-part&#160;z))<br></tt>


and


<tt>(make-from-mag-ang&#160;(magnitude&#160;z)&#160;(angle&#160;z))<br></tt>


produce complex numbers that are equal to <tt>z</tt>.


Using these constructors and selectors, we can implement arithmetic on complex numbers using the "abstract data" specified by the constructors and selectors, just as we did for rational numbers in section&#160;<a href="chapter_2_section_1.html#%_sec_2.1.1">2.1.1</a>. As shown in the formulas above, we can add and subtract complex numbers in terms of real and imaginary parts while multiplying and dividing complex numbers in terms of magnitudes and angles:


<tt><a name="index_term_2312"></a>(define&#160;(add-complex&#160;z1&#160;z2)<br>
&#160;&#160;(make-from-real-imag&#160;(+&#160;(real-part&#160;z1)&#160;(real-part&#160;z2))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(+&#160;(imag-part&#160;z1)&#160;(imag-part&#160;z2))))<br>
<a name="index_term_2314"></a>(define&#160;(sub-complex&#160;z1&#160;z2)<br>
&#160;&#160;(make-from-real-imag&#160;(-&#160;(real-part&#160;z1)&#160;(real-part&#160;z2))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(-&#160;(imag-part&#160;z1)&#160;(imag-part&#160;z2))))<br>
<a name="index_term_2316"></a>(define&#160;(mul-complex&#160;z1&#160;z2)<br>
&#160;&#160;(make-from-mag-ang&#160;(*&#160;(magnitude&#160;z1)&#160;(magnitude&#160;z2))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(+&#160;(angle&#160;z1)&#160;(angle&#160;z2))))<br>
<a name="index_term_2318"></a>(define&#160;(div-complex&#160;z1&#160;z2)<br>
&#160;&#160;(make-from-mag-ang&#160;(/&#160;(magnitude&#160;z1)&#160;(magnitude&#160;z2))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(-&#160;(angle&#160;z1)&#160;(angle&#160;z2))))<br></tt>


To complete the complex-number package, we must choose a representation and we must implement the constructors and selectors in terms of primitive numbers and primitive list structure. There are two obvious ways to do this: We can represent a complex number in "rectangular form" as a pair (real part, imaginary part) or in "polar form" as a pair (magnitude, angle). Which shall we choose?


In order to make the different choices concrete, imagine that there are two programmers, Ben Bitdiddle and Alyssa P. Hacker, who are independently designing representations for the complex-number system. <a name="index_term_2320"></a>Ben chooses to represent complex numbers in rectangular form. With this choice, selecting the real and imaginary parts of a complex number is straightforward, as is constructing a complex number with given real and imaginary parts. To find the magnitude and the angle, or to construct a complex number with a given magnitude and angle, he uses the trigonometric relations

<div align="left">
  <img src="img/chapter_2_image_60.gif" border="0">
</div>

<div align="left">
  <img src="img/chapter_2_image_61.gif" border="0">
</div>

which relate the real and imaginary parts (*x*, *y*) to the magnitude and the angle (*r*, *A*).<a name="call_footnote_Temp_269" href="#footnote_Temp_269" id="call_footnote_Temp_269"><sup><small>44</small></sup></a> Ben's representation is therefore given by the following selectors and constructors:


<tt><a name="index_term_2328"></a>(define&#160;(real-part&#160;z)&#160;(car&#160;z))<br>
<a name="index_term_2330"></a>(define&#160;(imag-part&#160;z)&#160;(cdr&#160;z))<br>
<a name="index_term_2332"></a>(define&#160;(magnitude&#160;z)<br>
&#160;&#160;(sqrt&#160;(+&#160;(square&#160;(real-part&#160;z))&#160;(square&#160;(imag-part&#160;z)))))<br>
<a name="index_term_2334"></a>(define&#160;(angle&#160;z)<br>
&#160;&#160;(atan&#160;(imag-part&#160;z)&#160;(real-part&#160;z)))<br>
<a name="index_term_2336"></a>(define&#160;(make-from-real-imag&#160;x&#160;y)&#160;(cons&#160;x&#160;y))<br>
<a name="index_term_2338"></a>(define&#160;(make-from-mag-ang&#160;r&#160;a)&#160;<br>
&#160;&#160;(cons&#160;(*&#160;r&#160;(cos&#160;a))&#160;(*&#160;r&#160;(sin&#160;a))))<br></tt>


<a name="index_term_2340"></a>Alyssa, in contrast, chooses to represent complex numbers in polar form. For her, selecting the magnitude and angle is straightforward, but she has to use the <a name="index_term_2342"></a>trigonometric relations to obtain the real and imaginary parts. Alyssa's representation is:


<tt><a name="index_term_2344"></a>(define&#160;(real-part&#160;z)<br>
&#160;&#160;(*&#160;(magnitude&#160;z)&#160;(cos&#160;(angle&#160;z))))<br>
<a name="index_term_2346"></a>(define&#160;(imag-part&#160;z)<br>
&#160;&#160;(*&#160;(magnitude&#160;z)&#160;(sin&#160;(angle&#160;z))))<br>
<a name="index_term_2348"></a>(define&#160;(magnitude&#160;z)&#160;(car&#160;z))<br>
<a name="index_term_2350"></a>(define&#160;(angle&#160;z)&#160;(cdr&#160;z))<br>
<a name="index_term_2352"></a>(define&#160;(make-from-real-imag&#160;x&#160;y)&#160;<br>
&#160;&#160;(cons&#160;(sqrt&#160;(+&#160;(square&#160;x)&#160;(square&#160;y)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(atan&#160;y&#160;x)))<br>
<a name="index_term_2354"></a>(define&#160;(make-from-mag-ang&#160;r&#160;a)&#160;(cons&#160;r&#160;a))<br></tt>


The discipline of data abstraction ensures that the same implementation of <tt>add-complex</tt>, <tt>sub-complex</tt>, <tt>mul-complex</tt>, and <tt>div-complex</tt> will work with either Ben's representation or Alyssa's representation.


<a name="%_sec_2.4.2"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_2.4.2">2.4.2&#160;&#160;Tagged data</a></h3>

<a name="index_term_2356"></a><a name="index_term_2358"></a><a name="index_term_2360"></a> One way to view data abstraction is as an application of the <a name="index_term_2362"></a><a name="index_term_2364"></a>"principle of least commitment." In implementing the complex-number system in section&#160;<a href="#%_sec_2.4.1">2.4.1</a>, we can use either Ben's rectangular representation or Alyssa's polar representation. The abstraction barrier formed by the selectors and constructors permits us to defer to the last possible moment the choice of a concrete representation for our data objects and thus retain maximum flexibility in our system design.


The principle of least commitment can be carried to even further extremes. If we desire, we can maintain the ambiguity of representation even *after* we have designed the selectors and constructors, and elect to use both Ben's representation *and* Alyssa's representation. If both representations are included in a single system, however, we will need some way to distinguish data in polar form from data in rectangular form. Otherwise, if we were asked, for instance, to find the <tt>magnitude</tt> of the pair (3,4), we wouldn't know whether to answer 5 (interpreting the number in rectangular form) or 3 (interpreting the number in polar form). A straightforward way to accomplish this distinction is to include a <a name="index_term_2366"></a>*type tag* -- the symbol <tt>rectangular</tt> or <tt>polar</tt> -- as part of each complex number. Then when we need to manipulate a complex number we can use the tag to decide which selector to apply.


In order to manipulate tagged data, we will assume that we have procedures <tt>type-tag</tt> and <tt>contents</tt> that extract from a data object the tag and the actual contents (the polar or rectangular coordinates, in the case of a complex number). We will also postulate a procedure <tt>attach-tag</tt> that takes a tag and contents and produces a tagged data object. A straightforward way to implement this is to use ordinary list structure:


<tt><a name="index_term_2368"></a>(define&#160;(attach-tag&#160;type-tag&#160;contents)<br>
&#160;&#160;(cons&#160;type-tag&#160;contents))<br>
<a name="index_term_2370"></a>(define&#160;(type-tag&#160;datum)<br>
&#160;&#160;(if&#160;(pair?&#160;datum)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(car&#160;datum)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Bad&#160;tagged&#160;datum&#160;--&#160;TYPE-TAG&quot;&#160;datum)))<br>
<a name="index_term_2372"></a>(define&#160;(contents&#160;datum)<br>
&#160;&#160;(if&#160;(pair?&#160;datum)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cdr&#160;datum)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Bad&#160;tagged&#160;datum&#160;--&#160;CONTENTS&quot;&#160;datum)))<br></tt>


Using these procedures, we can define predicates <tt>rectangular?</tt> and <tt>polar?</tt>, which recognize polar and rectangular numbers, respectively:


<tt><a name="index_term_2374"></a>(define&#160;(rectangular?&#160;z)<br>
&#160;&#160;(eq?&#160;(type-tag&#160;z)&#160;'rectangular))<br>
<a name="index_term_2376"></a>(define&#160;(polar?&#160;z)<br>
&#160;&#160;(eq?&#160;(type-tag&#160;z)&#160;'polar))<br></tt>


With type tags, Ben and Alyssa can now modify their code so that their two different representations can coexist in the same system. Whenever Ben constructs a complex number, he tags it as rectangular. Whenever Alyssa constructs a complex number, she tags it as polar. In addition, Ben and Alyssa must make sure that the names of their procedures do not conflict. One way to do this is for Ben to append the suffix <tt>rectangular</tt> to the name of each of his representation procedures and for Alyssa to append <tt>polar</tt> to the names of hers. Here is Ben's revised rectangular representation from section&#160;<a href="#%_sec_2.4.1">2.4.1</a>:


<tt><a name="index_term_2378"></a>(define&#160;(real-part-rectangular&#160;z)&#160;(car&#160;z))<br>
<a name="index_term_2380"></a>(define&#160;(imag-part-rectangular&#160;z)&#160;(cdr&#160;z))<br>
<a name="index_term_2382"></a>(define&#160;(magnitude-rectangular&#160;z)<br>
&#160;&#160;(sqrt&#160;(+&#160;(square&#160;(real-part-rectangular&#160;z))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(square&#160;(imag-part-rectangular&#160;z)))))<br>
<a name="index_term_2384"></a>(define&#160;(angle-rectangular&#160;z)<br>
&#160;&#160;(atan&#160;(imag-part-rectangular&#160;z)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(real-part-rectangular&#160;z)))<br>
<a name="index_term_2386"></a>(define&#160;(make-from-real-imag-rectangular&#160;x&#160;y)<br>
&#160;&#160;(attach-tag&#160;'rectangular&#160;(cons&#160;x&#160;y)))<br>
<a name="index_term_2388"></a>(define&#160;(make-from-mag-ang-rectangular&#160;r&#160;a)&#160;<br>
&#160;&#160;(attach-tag&#160;'rectangular<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons&#160;(*&#160;r&#160;(cos&#160;a))&#160;(*&#160;r&#160;(sin&#160;a)))))<br></tt>


and here is Alyssa's revised polar representation:


<tt><a name="index_term_2390"></a>(define&#160;(real-part-polar&#160;z)<br>
&#160;&#160;(*&#160;(magnitude-polar&#160;z)&#160;(cos&#160;(angle-polar&#160;z))))<br>
<a name="index_term_2392"></a>(define&#160;(imag-part-polar&#160;z)<br>
&#160;&#160;(*&#160;(magnitude-polar&#160;z)&#160;(sin&#160;(angle-polar&#160;z))))<br>
<a name="index_term_2394"></a>(define&#160;(magnitude-polar&#160;z)&#160;(car&#160;z))<br>
<a name="index_term_2396"></a>(define&#160;(angle-polar&#160;z)&#160;(cdr&#160;z))<br>
<a name="index_term_2398"></a>(define&#160;(make-from-real-imag-polar&#160;x&#160;y)&#160;<br>
&#160;&#160;(attach-tag&#160;'polar<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons&#160;(sqrt&#160;(+&#160;(square&#160;x)&#160;(square&#160;y)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(atan&#160;y&#160;x))))<br>
<a name="index_term_2400"></a>(define&#160;(make-from-mag-ang-polar&#160;r&#160;a)<br>
&#160;&#160;(attach-tag&#160;'polar&#160;(cons&#160;r&#160;a)))<br></tt>


<a name="index_term_2402"></a><a name="index_term_2404"></a>Each generic selector is implemented as a procedure that checks the tag of its argument and calls the appropriate procedure for handling data of that type. For example, to obtain the real part of a complex number, <tt>real-part</tt> examines the tag to determine whether to use Ben's <tt>real-part-rectangular</tt> or Alyssa's <tt>real-part-polar</tt>. In either case, we use <tt>contents</tt> to extract the bare, untagged datum and send this to the rectangular or polar procedure as required:


<tt><a name="index_term_2406"></a>(define&#160;(real-part&#160;z)<br>
&#160;&#160;(cond&#160;((rectangular?&#160;z)&#160;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(real-part-rectangular&#160;(contents&#160;z)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((polar?&#160;z)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(real-part-polar&#160;(contents&#160;z)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(error&#160;&quot;Unknown&#160;type&#160;--&#160;REAL-PART&quot;&#160;z))))<br>
<a name="index_term_2408"></a>(define&#160;(imag-part&#160;z)<br>
&#160;&#160;(cond&#160;((rectangular?&#160;z)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(imag-part-rectangular&#160;(contents&#160;z)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((polar?&#160;z)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(imag-part-polar&#160;(contents&#160;z)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(error&#160;&quot;Unknown&#160;type&#160;--&#160;IMAG-PART&quot;&#160;z))))<br>
<a name="index_term_2410"></a>(define&#160;(magnitude&#160;z)<br>
&#160;&#160;(cond&#160;((rectangular?&#160;z)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(magnitude-rectangular&#160;(contents&#160;z)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((polar?&#160;z)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(magnitude-polar&#160;(contents&#160;z)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(error&#160;&quot;Unknown&#160;type&#160;--&#160;MAGNITUDE&quot;&#160;z))))<br>
<a name="index_term_2412"></a>(define&#160;(angle&#160;z)<br>
&#160;&#160;(cond&#160;((rectangular?&#160;z)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(angle-rectangular&#160;(contents&#160;z)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((polar?&#160;z)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(angle-polar&#160;(contents&#160;z)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(error&#160;&quot;Unknown&#160;type&#160;--&#160;ANGLE&quot;&#160;z))))<br></tt>


To implement the complex-number arithmetic operations, we can use the same procedures <tt>add-complex</tt>, <tt>sub-complex</tt>, <tt>mul-complex</tt>, and <tt>div-complex</tt> from section&#160;<a href="#%_sec_2.4.1">2.4.1</a>, because the selectors they call are generic, and so will work with either representation. For example, the procedure <tt>add-complex</tt> is still


<tt>(define&#160;(add-complex&#160;z1&#160;z2)<br>
&#160;&#160;(make-from-real-imag&#160;(+&#160;(real-part&#160;z1)&#160;(real-part&#160;z2))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(+&#160;(imag-part&#160;z1)&#160;(imag-part&#160;z2))))<br></tt>


Finally, we must choose whether to construct complex numbers using Ben's representation or Alyssa's representation. One reasonable choice is to construct rectangular numbers whenever we have real and imaginary parts and to construct polar numbers whenever we have magnitudes and angles:


<tt><a name="index_term_2414"></a>(define&#160;(make-from-real-imag&#160;x&#160;y)<br>
&#160;&#160;(make-from-real-imag-rectangular&#160;x&#160;y))<br>
<a name="index_term_2416"></a>(define&#160;(make-from-mag-ang&#160;r&#160;a)<br>
&#160;&#160;(make-from-mag-ang-polar&#160;r&#160;a))<br></tt>


<a name="%_fig_2.21"></a>

<div align="left">
  <div align="left">
    **Figure 2.21:**&#160;&#160;Structure of the generic complex-arithmetic system.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_62.gif" border="0"></td>
    </tr>

    <tr>
      <td><a name="index_term_2418"></a></td>
    </tr>
  </table>
</div>

The resulting complex-number system has the structure shown in figure&#160;<a href="#%_fig_2.21">2.21</a>. The system has been decomposed into three relatively independent parts: the complex-number-arithmetic operations, Alyssa's polar implementation, and Ben's rectangular implementation. The polar and rectangular implementations could have been written by Ben and Alyssa working separately, and both of these can be used as underlying representations by a third programmer implementing the complex-arithmetic procedures in terms of the abstract constructor/selector interface.


<a name="index_term_2420"></a><a name="index_term_2422"></a>Since each data object is tagged with its type, the selectors operate on the data in a generic manner. That is, each selector is defined to have a behavior that depends upon the particular type of data it is applied to. Notice the general mechanism for interfacing the separate representations: Within a given representation implementation (say, Alyssa's polar package) a complex number is an untyped pair (magnitude, angle). When a generic selector operates on a number of <tt>polar</tt> type, it strips off the tag and passes the contents on to Alyssa's code. Conversely, when Alyssa constructs a number for general use, she tags it with a type so that it can be appropriately recognized by the higher-level procedures. This discipline of stripping off and attaching tags as data objects are passed from level to level can be an important organizational strategy, as we shall see in section&#160;<a href="chapter_2_section_5.html#%_sec_2.5">2.5</a>.


<a name="%_sec_2.4.3"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_2.4.3">2.4.3&#160;&#160;Data-Directed Programming and Additivity</a></h3>

<a name="index_term_2424"></a><a name="index_term_2426"></a> <a name="index_term_2428"></a>The general strategy of checking the type of a datum and calling an appropriate procedure is called <a name="index_term_2430"></a><a name="index_term_2432"></a>*dispatching on type*. This is a powerful strategy for obtaining modularity in system design. On the other hand, implementing the dispatch as in section&#160;<a href="#%_sec_2.4.2">2.4.2</a> has two significant weaknesses. One weakness is that the generic interface procedures (<tt>real-part</tt>, <tt>imag-part</tt>, <tt>magnitude</tt>, and <tt>angle</tt>) must know about all the different representations. For instance, suppose we wanted to incorporate a new representation for complex numbers into our complex-number system. We would need to identify this new representation with a type, and then add a clause to each of the generic interface procedures to check for the new type and apply the appropriate selector for that representation.


Another weakness of the technique is that even though the individual representations can be designed separately, we must guarantee that no two procedures in the entire system have the same name. This is why Ben and Alyssa had to change the names of their original procedures from section&#160;<a href="#%_sec_2.4.1">2.4.1</a>.


The issue underlying both of these weaknesses is that the technique for implementing generic interfaces is not *additive*. The person implementing the generic selector procedures must modify those procedures each time a new representation is installed, and the people interfacing the individual representations must modify their code to avoid name conflicts. In each of these cases, the changes that must be made to the code are straightforward, but they must be made nonetheless, and this is a source of inconvenience and error. This is not much of a problem for the complex-number system as it stands, but suppose there were not two but hundreds of different representations for complex numbers. And suppose that there were many generic selectors to be maintained in the abstract-data interface. Suppose, in fact, that no one programmer knew all the interface procedures or all the representations. The problem is real and must be addressed in such programs as large-scale data-base-management systems.


What we need is a means for modularizing the system design even further. This is provided by the programming technique known as *data-directed programming*. To understand how data-directed programming works, begin with the observation that whenever we deal with a set of generic operations that are common to a set of different types we are, in effect, dealing with a two-dimensional table that contains the possible operations on one axis and the possible types on the other axis. The entries in the table are the procedures that implement each operation for each type of argument presented. In the complex-number system developed in the previous section, the correspondence between operation name, data type, and actual procedure was spread out among the various conditional clauses in the generic interface procedures. But the same information could have been organized in a table, as shown in figure&#160;<a href="#%_fig_2.22">2.22</a>.


<a name="index_term_2434"></a>Data-directed programming is the technique of designing programs to work with such a table directly. Previously, we implemented the mechanism that interfaces the complex-arithmetic code with the two representation packages as a set of procedures that each perform an explicit dispatch on type. Here we will implement the interface as a single procedure that looks up the combination of the operation name and argument type in the table to find the correct procedure to apply, and then applies it to the contents of the argument. If we do this, then to add a new representation package to the system we need not change any existing procedures; we need only add new entries to the table.


<a name="%_fig_2.22"></a>

<div align="left">
  <div align="left">
    **Figure 2.22:**&#160;&#160;Table of operations for the complex-number system.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_2_image_63.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

To implement this plan, assume that we have two procedures, <tt>put</tt> and <tt>get</tt>, for manipulating the operation-and-type table: <a name="index_term_2436"></a>

<ul>
  <li style="list-style: none"><a name="index_term_2438"></a></li>

  <li>
    <tt>(put &lt;*op*&gt; &lt;*type*&gt; &lt;*item*&gt;)</tt><br>
    installs the <tt>&lt;*item*&gt;</tt> in the table, indexed by the <tt>&lt;*op*&gt;</tt> and the <tt>&lt;*type*&gt;</tt>.

    
<a name="index_term_2440"></a>

  </li>

  <li><tt>(get &lt;*op*&gt; &lt;*type*&gt;)</tt><br>
  looks up the <tt>&lt;*op*&gt;</tt>, <tt>&lt;*type*&gt;</tt> entry in the table and returns the item found there. If no item is found, <tt>get</tt> returns false.</li>
</ul>

For now, we can assume that <tt>put</tt> and <tt>get</tt> are included in our language. In chapter&#160;3 (section&#160;<a href="chapter_3_section_3.html#%_sec_3.3.3">3.3.3</a>, exercise&#160;<a href="chapter_3_section_3.html#exercise_3_24">3.24</a>) we will see how to implement these and other operations for manipulating tables.


Here is how data-directed programming can be used in the complex-number system. Ben, who developed the rectangular representation, implements his code just as he did originally. He defines a collection of procedures, or a <a name="index_term_2442"></a>*package*, and interfaces these to the rest of the system by adding entries to the table that tell the system how to operate on rectangular numbers. This is accomplished by calling the following procedure: <a name="index_term_2444"></a><a name="index_term_2446"></a>


<tt><a name="index_term_2448"></a>(define&#160;(install-rectangular-package)<br>
&#160;&#160;*;;&#160;internal&#160;procedures*<br>
&#160;&#160;(define&#160;(real-part&#160;z)&#160;(car&#160;z))<br>
&#160;&#160;(define&#160;(imag-part&#160;z)&#160;(cdr&#160;z))<br>
&#160;&#160;(define&#160;(make-from-real-imag&#160;x&#160;y)&#160;(cons&#160;x&#160;y))<br>
&#160;&#160;(define&#160;(magnitude&#160;z)<br>
&#160;&#160;&#160;&#160;(sqrt&#160;(+&#160;(square&#160;(real-part&#160;z))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(square&#160;(imag-part&#160;z)))))<br>
&#160;&#160;(define&#160;(angle&#160;z)<br>
&#160;&#160;&#160;&#160;(atan&#160;(imag-part&#160;z)&#160;(real-part&#160;z)))<br>
&#160;&#160;(define&#160;(make-from-mag-ang&#160;r&#160;a)&#160;<br>
&#160;&#160;&#160;&#160;(cons&#160;(*&#160;r&#160;(cos&#160;a))&#160;(*&#160;r&#160;(sin&#160;a))))<br>
&#160;&#160;*;;&#160;interface&#160;to&#160;the&#160;rest&#160;of&#160;the&#160;system*<br>
&#160;&#160;(define&#160;(tag&#160;x)&#160;(attach-tag&#160;'rectangular&#160;x))<br>
&#160;&#160;(put&#160;'real-part&#160;'(rectangular)&#160;real-part)<br>
&#160;&#160;(put&#160;'imag-part&#160;'(rectangular)&#160;imag-part)<br>
&#160;&#160;(put&#160;'magnitude&#160;'(rectangular)&#160;magnitude)<br>
&#160;&#160;(put&#160;'angle&#160;'(rectangular)&#160;angle)<br>
&#160;&#160;(put&#160;'make-from-real-imag&#160;'rectangular&#160;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda&#160;(x&#160;y)&#160;(tag&#160;(make-from-real-imag&#160;x&#160;y))))<br>
&#160;&#160;(put&#160;'make-from-mag-ang&#160;'rectangular&#160;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda&#160;(r&#160;a)&#160;(tag&#160;(make-from-mag-ang&#160;r&#160;a))))<br>
&#160;&#160;'done)<br></tt>


Notice that the internal procedures here are the same procedures from section&#160;<a href="#%_sec_2.4.1">2.4.1</a> that Ben wrote when he was working in isolation. No changes are necessary in order to interface them to the rest of the system. Moreover, since these procedure definitions are internal to the installation procedure, Ben needn't worry about name conflicts with other procedures outside the rectangular package. To interface these to the rest of the system, Ben installs his <tt>real-part</tt> procedure under the operation name <tt>real-part</tt> and the type <tt>(rectangular)</tt>, and similarly for the other selectors.<a name="call_footnote_Temp_270" href="#footnote_Temp_270" id="call_footnote_Temp_270"><sup><small>45</small></sup></a> The interface also defines the constructors to be used by the external system.<a name="call_footnote_Temp_271" href="#footnote_Temp_271" id="call_footnote_Temp_271"><sup><small>46</small></sup></a> These are identical to Ben's internally defined constructors, except that they attach the tag.


<a name="index_term_2450"></a><a name="index_term_2452"></a>Alyssa's polar package is analogous:


<tt><a name="index_term_2454"></a>(define&#160;(install-polar-package)<br>
&#160;&#160;*;;&#160;internal&#160;procedures*<br>
&#160;&#160;(define&#160;(magnitude&#160;z)&#160;(car&#160;z))<br>
&#160;&#160;(define&#160;(angle&#160;z)&#160;(cdr&#160;z))<br>
&#160;&#160;(define&#160;(make-from-mag-ang&#160;r&#160;a)&#160;(cons&#160;r&#160;a))<br>
&#160;&#160;(define&#160;(real-part&#160;z)<br>
&#160;&#160;&#160;&#160;(*&#160;(magnitude&#160;z)&#160;(cos&#160;(angle&#160;z))))<br>
&#160;&#160;(define&#160;(imag-part&#160;z)<br>
&#160;&#160;&#160;&#160;(*&#160;(magnitude&#160;z)&#160;(sin&#160;(angle&#160;z))))<br>
&#160;&#160;(define&#160;(make-from-real-imag&#160;x&#160;y)&#160;<br>
&#160;&#160;&#160;&#160;(cons&#160;(sqrt&#160;(+&#160;(square&#160;x)&#160;(square&#160;y)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(atan&#160;y&#160;x)))<br>
&#160;&#160;*;;&#160;interface&#160;to&#160;the&#160;rest&#160;of&#160;the&#160;system*<br>
&#160;&#160;(define&#160;(tag&#160;x)&#160;(attach-tag&#160;'polar&#160;x))<br>
&#160;&#160;(put&#160;'real-part&#160;'(polar)&#160;real-part)<br>
&#160;&#160;(put&#160;'imag-part&#160;'(polar)&#160;imag-part)<br>
&#160;&#160;(put&#160;'magnitude&#160;'(polar)&#160;magnitude)<br>
&#160;&#160;(put&#160;'angle&#160;'(polar)&#160;angle)<br>
&#160;&#160;(put&#160;'make-from-real-imag&#160;'polar<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda&#160;(x&#160;y)&#160;(tag&#160;(make-from-real-imag&#160;x&#160;y))))<br>
&#160;&#160;(put&#160;'make-from-mag-ang&#160;'polar&#160;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda&#160;(r&#160;a)&#160;(tag&#160;(make-from-mag-ang&#160;r&#160;a))))<br>
&#160;&#160;'done)<br></tt>


Even though Ben and Alyssa both still use their original procedures defined with the same names as each other's (e.g., <tt>real-part</tt>), these definitions are now internal to different procedures (see section&#160;<a href="chapter_1_section_1.html#%_sec_1.1.8">1.1.8</a>), so there is no name conflict.


The complex-arithmetic selectors access the table by means of a general "operation" procedure called <tt>apply-generic</tt>, which applies a generic operation to some arguments. <tt>Apply-generic</tt> looks in the table under the name of the operation and the types of the arguments and applies the resulting procedure if one is present:<a name="call_footnote_Temp_272" href="#footnote_Temp_272" id="call_footnote_Temp_272"><sup><small>47</small></sup></a>


<tt><a name="index_term_2462"></a>(define&#160;(apply-generic&#160;op&#160;.&#160;args)<br>
&#160;&#160;(let&#160;((type-tags&#160;(map&#160;type-tag&#160;args)))<br>
&#160;&#160;&#160;&#160;(let&#160;((proc&#160;(get&#160;op&#160;type-tags)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;proc<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(apply&#160;proc&#160;(map&#160;contents&#160;args))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&quot;No&#160;method&#160;for&#160;these&#160;types&#160;--&#160;APPLY-GENERIC&quot;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(list&#160;op&#160;type-tags))))))<br></tt>


Using <tt>apply-generic</tt>, we can define our generic selectors as follows:


<tt><a name="index_term_2464"></a>(define&#160;(real-part&#160;z)&#160;(apply-generic&#160;'real-part&#160;z))<br>
<a name="index_term_2466"></a>(define&#160;(imag-part&#160;z)&#160;(apply-generic&#160;'imag-part&#160;z))<br>
<a name="index_term_2468"></a>(define&#160;(magnitude&#160;z)&#160;(apply-generic&#160;'magnitude&#160;z))<br>
<a name="index_term_2470"></a>(define&#160;(angle&#160;z)&#160;(apply-generic&#160;'angle&#160;z))<br></tt>


Observe that these do not change at all if a new representation is added to the system.


We can also extract from the table the constructors to be used by the programs external to the packages in making complex numbers from real and imaginary parts and from magnitudes and angles. As in section&#160;<a href="#%_sec_2.4.2">2.4.2</a>, we construct rectangular numbers whenever we have real and imaginary parts, and polar numbers whenever we have magnitudes and angles:


<tt><a name="index_term_2472"></a>(define&#160;(make-from-real-imag&#160;x&#160;y)<br>
&#160;&#160;((get&#160;'make-from-real-imag&#160;'rectangular)&#160;x&#160;y))<br>
<a name="index_term_2474"></a>(define&#160;(make-from-mag-ang&#160;r&#160;a)<br>
&#160;&#160;((get&#160;'make-from-mag-ang&#160;'polar)&#160;r&#160;a))<br></tt>


<a name="%_thm_2.73"></a> **Exercise 2.73.**&#160;&#160;Section&#160;<a href="chapter_2_section_3.html#%_sec_2.3.2">2.3.2</a> described a program that performs symbolic differentiation: <a name="index_term_2476"></a><a name="index_term_2478"></a>


<tt>(define&#160;(deriv&#160;exp&#160;var)<br>
&#160;&#160;(cond&#160;((number?&#160;exp)&#160;0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((variable?&#160;exp)&#160;(if&#160;(same-variable?&#160;exp&#160;var)&#160;1&#160;0))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((sum?&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(make-sum&#160;(deriv&#160;(addend&#160;exp)&#160;var)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(deriv&#160;(augend&#160;exp)&#160;var)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((product?&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(make-sum<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(make-product&#160;(multiplier&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(deriv&#160;(multiplicand&#160;exp)&#160;var))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(make-product&#160;(deriv&#160;(multiplier&#160;exp)&#160;var)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(multiplicand&#160;exp))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&lt;*more&#160;rules&#160;can&#160;be&#160;added&#160;here*&gt;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(error&#160;&quot;unknown&#160;expression&#160;type&#160;--&#160;DERIV&quot;&#160;exp))))<br></tt>


We can regard this program as performing a dispatch on the type of the expression to be differentiated. In this situation the "type tag" of the datum is the algebraic operator symbol (such as <tt>+</tt>) and the operation being performed is <tt>deriv</tt>. We can transform this program into data-directed style by rewriting the basic derivative procedure as


<tt><a name="index_term_2480"></a>(define&#160;(deriv&#160;exp&#160;var)<br>
&#160;&#160;&#160;(cond&#160;((number?&#160;exp)&#160;0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((variable?&#160;exp)&#160;(if&#160;(same-variable?&#160;exp&#160;var)&#160;1&#160;0))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;((get&#160;'deriv&#160;(operator&#160;exp))&#160;(operands&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;var))))<br>
(define&#160;(operator&#160;exp)&#160;(car&#160;exp))<br>
(define&#160;(operands&#160;exp)&#160;(cdr&#160;exp))<br></tt>


a.&#160;&#160;Explain what was done above. Why can't we assimilate the predicates <tt>number?</tt> and <tt>same-variable?</tt> into the data-directed dispatch?


b.&#160;&#160;Write the procedures for derivatives of sums and products, and the auxiliary code required to install them in the table used by the program above.


c.&#160;&#160;Choose any additional differentiation rule that you like, such as the one for exponents (exercise&#160;<a href="chapter_2_section_3.html#exercise_2_56">2.56</a>), and install it in this data-directed system.


d.&#160;&#160;In this simple algebraic manipulator the type of an expression is the algebraic operator that binds it together. Suppose, however, we indexed the procedures in the opposite way, so that the dispatch line in <tt>deriv</tt> looked like


<tt>((get&#160;(operator&#160;exp)&#160;'deriv)&#160;(operands&#160;exp)&#160;var)<br></tt>


What corresponding changes to the derivative system are required?


<a name="%_thm_2.74"></a> **Exercise 2.74.**&#160;&#160;<a name="index_term_2482"></a><a name="index_term_2484"></a>Insatiable Enterprises, Inc., is a highly decentralized conglomerate company consisting of a large number of independent divisions located all over the world. The company's computer facilities have just been interconnected by means of a clever network-interfacing scheme that makes the entire network appear to any user to be a single computer. Insatiable's president, in her first attempt to exploit the ability of the network to extract administrative information from division files, is dismayed to discover that, although all the division files have been implemented as data structures in Scheme, the particular data structure used varies from division to division. A meeting of division managers is hastily called to search for a strategy to integrate the files that will satisfy headquarters' needs while preserving the existing autonomy of the divisions.


Show how such a strategy can be implemented with data-directed programming. As an example, suppose that each division's personnel records consist of a single file, which contains a set of records keyed on employees' names. The structure of the set varies from division to division. Furthermore, each employee's record is itself a set (structured differently from division to division) that contains information keyed under identifiers such as <tt>address</tt> and <tt>salary</tt>. In particular:


a.&#160;&#160;Implement for headquarters a <tt>get-record</tt> procedure that retrieves a specified employee's record from a specified personnel file. The procedure should be applicable to any division's file. Explain how the individual divisions' files should be structured. In particular, what type information must be supplied?


b.&#160;&#160;Implement for headquarters a <tt>get-salary</tt> procedure that returns the salary information from a given employee's record from any division's personnel file. How should the record be structured in order to make this operation work?


c.&#160;&#160;Implement for headquarters a <tt>find-employee-record</tt> procedure. This should search all the divisions' files for the record of a given employee and return the record. Assume that this procedure takes as arguments an employee's name and a list of all the divisions' files.


d.&#160;&#160;When Insatiable takes over a new company, what changes must be made in order to incorporate the new personnel information into the central system?


<a name="%_sec_Temp_275"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_275">Message passing</a></h4>

<a name="index_term_2486"></a> The key idea of data-directed programming is to handle generic operations in programs by dealing explicitly with operation-and-type tables, such as the table in figure&#160;<a href="#%_fig_2.22">2.22</a>. The style of programming we used in section&#160;<a href="#%_sec_2.4.2">2.4.2</a> organized the required dispatching on type by having each operation take care of its own dispatching. In effect, this decomposes the operation-and-type table into rows, with each generic operation procedure representing a row of the table.


An alternative implementation strategy is to decompose the table into columns and, instead of using "intelligent operations" that dispatch on data types, to work with "intelligent data objects" that dispatch on operation names. We can do this by arranging things so that a data object, such as a rectangular number, is represented as a procedure that takes as input the required operation name and performs the operation indicated. In such a discipline, <tt>make-from-real-imag</tt> could be written as


<tt><a name="index_term_2488"></a>(define&#160;(make-from-real-imag&#160;x&#160;y)<br>
&#160;&#160;(define&#160;(dispatch&#160;op)<br>
&#160;&#160;&#160;&#160;(cond&#160;((eq?&#160;op&#160;'real-part)&#160;x)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((eq?&#160;op&#160;'imag-part)&#160;y)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((eq?&#160;op&#160;'magnitude)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(sqrt&#160;(+&#160;(square&#160;x)&#160;(square&#160;y))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((eq?&#160;op&#160;'angle)&#160;(atan&#160;y&#160;x))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Unknown&#160;op&#160;--&#160;MAKE-FROM-REAL-IMAG&quot;&#160;op))))<br>
&#160;&#160;dispatch)<br></tt>


The corresponding <tt>apply-generic</tt> procedure, which applies a generic operation to an argument, now simply feeds the operation's name to the data object and lets the object do the work:<a name="call_footnote_Temp_276" href="#footnote_Temp_276" id="call_footnote_Temp_276"><sup><small>48</small></sup></a>


<tt><a name="index_term_2490"></a>(define&#160;(apply-generic&#160;op&#160;arg)&#160;(arg&#160;op))<br></tt>


Note that the value returned by <tt>make-from-real-imag</tt> is a procedure -- the internal <tt>dispatch</tt> procedure. This is the procedure that is invoked when <tt>apply-generic</tt> requests an operation to be performed.


This style of programming is called *message passing*. The name comes from the image that a data object is an entity that receives the requested operation name as a "message." We have already seen an example of message passing in section&#160;<a href="chapter_2_section_1.html#%_sec_2.1.3">2.1.3</a>, where we saw how <tt>cons</tt>, <tt>car</tt>, and <tt>cdr</tt> could be defined with no data objects but only procedures. Here we see that message passing is not a mathematical trick but a useful technique for organizing systems with generic operations. In the remainder of this chapter we will continue to use data-directed programming, rather than message passing, to discuss generic arithmetic operations. In chapter&#160;3 we will return to message passing, and we will see that it can be a powerful tool for structuring simulation programs.


<a name="%_thm_2.75"></a> **Exercise 2.75.**&#160;&#160;<a name="index_term_2492"></a>Implement the constructor <tt>make-from-mag-ang</tt> in message-passing style. This procedure should be analogous to the <tt>make-from-real-imag</tt> procedure given above.


<a name="%_thm_2.76"></a> **Exercise 2.76.**&#160;&#160;<a name="index_term_2494"></a>As a large system with generic operations evolves, new types of data objects or new operations may be needed. For each of the three strategies -- generic operations with explicit dispatch, data-directed style, and message-passing-style -- describe the changes that must be made to a system in order to add new types or new operations. Which organization would be most appropriate for a system in which new types must often be added? Which would be most appropriate for a system in which new operations must often be added?

<div class="smallprint">
  <hr>
</div>

<div class="footnote">
  
<a name="footnote_Temp_268" href="#call_footnote_Temp_268" id="footnote_Temp_268"><sup><small>43</small></sup></a> In actual computational systems, rectangular form is preferable to polar form most of the time because of <a name="index_term_2308"></a>roundoff errors in conversion between rectangular and polar form. This is why the complex-number example is unrealistic. Nevertheless, it provides a clear illustration of the design of a system using generic operations and a good introduction to the more substantial systems to be developed later in this chapter.

  
<a name="footnote_Temp_269" href="#call_footnote_Temp_269" id="footnote_Temp_269"><sup><small>44</small></sup></a> The arctangent function referred to <a name="index_term_2322"></a><a name="index_term_2324"></a><a name="index_term_2326"></a>here, computed by Scheme's <tt>atan</tt> procedure, is defined so as to take two arguments *y*&#160;and *x* and to return the angle whose tangent is *y*/*x*. The signs of the arguments determine the quadrant of the angle.

  
<a name="footnote_Temp_270" href="#call_footnote_Temp_270" id="footnote_Temp_270"><sup><small>45</small></sup></a> We use the list <tt>(rectangular)</tt> rather than the symbol <tt>rectangular</tt> to allow for the possibility of operations with multiple arguments, not all of the same type.

  
<a name="footnote_Temp_271" href="#call_footnote_Temp_271" id="footnote_Temp_271"><sup><small>46</small></sup></a> The type the constructors are installed under needn't be a list because a constructor is always used to make an object of one particular type.

  
<a name="footnote_Temp_272" href="#call_footnote_Temp_272" id="footnote_Temp_272"><sup><small>47</small></sup></a> <tt>Apply-generic</tt> uses the <a name="index_term_2456"></a>dotted-tail notation described in exercise&#160;<a href="chapter_2_section_2.html#exercise_2_20">2.20</a>, because different generic operations may take different numbers of arguments. In <tt>apply-generic</tt>, <tt>op</tt> has as its value the first argument to <tt>apply-generic</tt> and <tt>args</tt> has as its value a list of the remaining arguments.

  
<tt>Apply-generic</tt> also uses the primitive procedure <a name="index_term_2458"></a><a name="index_term_2460"></a><tt>apply</tt>, which takes two arguments, a procedure and a list. <tt>Apply</tt> applies the procedure, using the elements in the list as arguments. For example,

  
<tt>(apply&#160;+&#160;(list&#160;1&#160;2&#160;3&#160;4))<br></tt>

  
returns 10.

  
<a name="footnote_Temp_276" href="#call_footnote_Temp_276" id="footnote_Temp_276"><sup><small>48</small></sup></a> One limitation of this organization is it permits only generic procedures of one argument.

</div>