(section_35)=
# Streams

<a name="index_term_3726"></a> We've gained a good understanding of assignment as a tool in modeling, as well as an appreciation of the complex problems that assignment raises. It is time to ask whether we could have gone about things in a different way, so as to avoid some of these problems. In this section, we explore an alternative approach to modeling state, based on data structures called *streams*. As we shall see, streams can mitigate some of the complexity of modeling state.


Let's step back and review where this complexity comes from. In an attempt to model real-world phenomena, we made some apparently reasonable decisions: We modeled real-world objects with local state by computational objects with local variables. We identified time variation in the real world with time variation in the computer. We implemented the time variation of the states of the model objects in the computer with assignments to the local variables of the model objects.


Is there another approach? Can we avoid identifying time in the computer with time in the modeled world? Must we make the model change with time in order to model phenomena in a changing world? Think about the issue in terms of mathematical functions. We can describe the time-varying behavior of a quantity *x* as a function of time *x*(*t*). If we concentrate on *x* instant by instant, we think of it as a changing quantity. Yet if we concentrate on the entire time history of values, we do not emphasize change -- the function itself does not change.<a name="call_footnote_Temp_442" href="#footnote_Temp_442" id="call_footnote_Temp_442"><sup><small>52</small></sup></a>


If time is measured in discrete steps, then we can model a time function as a (possibly infinite) sequence. In this section, we will see how to model change in terms of sequences that represent the time histories of the systems being modeled. To accomplish this, we introduce new data structures called *streams*. From an abstract point of view, a stream is simply a sequence. However, we will find that the straightforward implementation of streams as lists (as in section&#160;<a href="chapter_2_section_2.html#%_sec_2.2.1">2.2.1</a>) doesn't fully reveal the power of stream processing. As an alternative, we introduce the technique of <a name="index_term_3730"></a>*delayed evaluation*, which enables us to represent very large (even infinite) sequences as streams.


Stream processing lets us model systems that have state without ever using assignment or mutable data. This has important implications, both theoretical and practical, because we can build models that avoid the drawbacks inherent in introducing assignment. On the other hand, the stream framework raises difficulties of its own, and the question of which modeling technique leads to more modular and more easily maintained systems remains open.


<a name="%_sec_3.5.1"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_3.5.1">3.5.1&#160;&#160;Streams Are Delayed Lists</a></h3>

<a name="index_term_3732"></a> As we saw in section&#160;<a href="chapter_2_section_2.html#%_sec_2.2.3">2.2.3</a>, sequences can serve as standard interfaces for combining program modules. We formulated powerful abstractions for manipulating sequences, such as <tt>map</tt>, <tt>filter</tt>, and <tt>accumulate</tt>, that capture a wide variety of operations in a manner that is both succinct and elegant.


Unfortunately, if we represent sequences as lists, this elegance is bought at the price of severe inefficiency with respect to both the time and space required by our computations. When we represent manipulations on sequences as transformations of lists, our programs must construct and copy data structures (which may be huge) at every step of a process.


To see why this is true, let us compare two programs for computing the sum of all the prime numbers in an interval. The first program is written in standard iterative style:<a name="call_footnote_Temp_443" href="#footnote_Temp_443" id="call_footnote_Temp_443"><sup><small>53</small></sup></a>


<tt><a name="index_term_3734"></a>(define&#160;(sum-primes&#160;a&#160;b)<br>
&#160;&#160;(define&#160;(iter&#160;count&#160;accum)<br>
&#160;&#160;&#160;&#160;(cond&#160;((&gt;&#160;count&#160;b)&#160;accum)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((prime?&#160;count)&#160;(iter&#160;(+&#160;count&#160;1)&#160;(+&#160;count&#160;accum)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(iter&#160;(+&#160;count&#160;1)&#160;accum))))<br>
&#160;&#160;(iter&#160;a&#160;0))<br></tt>


The second program performs the same computation using the sequence operations of section&#160;<a href="chapter_2_section_2.html#%_sec_2.2.3">2.2.3</a>:


<tt><a name="index_term_3736"></a>(define&#160;(sum-primes&#160;a&#160;b)<br>
&#160;&#160;(accumulate&#160;+<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;0<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(filter&#160;prime?&#160;(enumerate-interval&#160;a&#160;b))))<br></tt>


In carrying out the computation, the first program needs to store only the sum being accumulated. In contrast, the filter in the second program cannot do any testing until <tt>enumerate-interval</tt> has constructed a complete list of the numbers in the interval. The filter generates another list, which in turn is passed to <tt>accumulate</tt> before being collapsed to form a sum. Such large intermediate storage is not needed by the first program, which we can think of as enumerating the interval incrementally, adding each prime to the sum as it is generated.


The inefficiency in using lists becomes painfully apparent if we use the sequence paradigm to compute the second prime in the interval from 10,000 to 1,000,000 by evaluating the expression


<tt>(car&#160;(cdr&#160;(filter&#160;prime?<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(enumerate-interval&#160;10000&#160;1000000))))<br></tt>


This expression does find the second prime, but the computational overhead is outrageous. We construct a list of almost a million integers, filter this list by testing each element for primality, and then ignore almost all of the result. In a more traditional programming style, we would interleave the enumeration and the filtering, and stop when we reached the second prime.


Streams are a clever idea that allows one to use sequence manipulations without incurring the costs of manipulating sequences as lists. With streams we can achieve the best of both worlds: We can formulate programs elegantly as sequence manipulations, while attaining the efficiency of incremental computation. The basic idea is to arrange to construct a stream only partially, and to pass the partial construction to the program that consumes the stream. If the consumer attempts to access a part of the stream that has not yet been constructed, the stream will automatically construct just enough more of itself to produce the required part, thus preserving the illusion that the entire stream exists. In other words, although we will write programs as if we were processing complete sequences, we design our stream implementation to automatically and transparently interleave the construction of the stream with its use.


On the surface, streams are just lists with different names for the procedures that manipulate them. There is a constructor, <a name="index_term_3738"></a><tt>cons-stream</tt>, and two selectors, <a name="index_term_3740"></a><tt>stream-car</tt> and <a name="index_term_3742"></a><tt>stream-cdr</tt>, which satisfy the constraints

<div align="left">
  <img src="img/chapter_3_image_34.gif" border="0">
</div>

There is a distinguishable object, <a name="index_term_3744"></a><a name="index_term_3746"></a><a name="index_term_3748"></a><tt>the-empty-stream</tt>, which cannot be the result of any <tt>cons-stream</tt> operation, and which can be identified with the predicate <a name="index_term_3750"></a><tt>stream-null?</tt>.<a name="call_footnote_Temp_444" href="#footnote_Temp_444" id="call_footnote_Temp_444"><sup><small>54</small></sup></a> Thus we can make and use streams, in just the same way as we can make and use lists, to represent aggregate data arranged in a sequence. In particular, we can build stream analogs of the list operations from chapter&#160;2, such as <tt>list-ref</tt>, <tt>map</tt>, and <tt>for-each</tt>:<a name="call_footnote_Temp_445" href="#footnote_Temp_445" id="call_footnote_Temp_445"><sup><small>55</small></sup></a>


<tt><a name="index_term_3758"></a>(define&#160;(stream-ref&#160;s&#160;n)<br>
&#160;&#160;(if&#160;(=&#160;n&#160;0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(stream-car&#160;s)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(stream-ref&#160;(stream-cdr&#160;s)&#160;(-&#160;n&#160;1))))<br>
<a name="index_term_3760"></a>(define&#160;(stream-map&#160;proc&#160;s)<br>
&#160;&#160;(if&#160;(stream-null?&#160;s)<br>
&#160;&#160;&#160;&#160;&#160;&#160;the-empty-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream&#160;(proc&#160;(stream-car&#160;s))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-map&#160;proc&#160;(stream-cdr&#160;s)))))<br>
<a name="index_term_3762"></a>(define&#160;(stream-for-each&#160;proc&#160;s)<br>
&#160;&#160;(if&#160;(stream-null?&#160;s)<br>
&#160;&#160;&#160;&#160;&#160;&#160;'done<br>
&#160;&#160;&#160;&#160;&#160;&#160;(begin&#160;(proc&#160;(stream-car&#160;s))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-for-each&#160;proc&#160;(stream-cdr&#160;s)))))<br></tt>


<tt>Stream-for-each</tt> is useful for viewing streams:


<tt><a name="index_term_3764"></a>(define&#160;(display-stream&#160;s)<br>
&#160;&#160;(stream-for-each&#160;display-line&#160;s))<br>
<br>
<a name="index_term_3766"></a>(define&#160;(display-line&#160;x)<br>
&#160;&#160;(newline)<br>
&#160;&#160;(display&#160;x))<br></tt>


To make the stream implementation automatically and transparently interleave the construction of a stream with its use, we will arrange for the <tt>cdr</tt> of a stream to be evaluated when it is accessed by the <tt>stream-cdr</tt> procedure rather than when the stream is constructed by <tt>cons-stream</tt>. This implementation choice is reminiscent of our discussion of rational numbers in section&#160;<a href="chapter_2_section_1.html#%_sec_2.1.2">2.1.2</a>, where we saw that we can choose to implement rational numbers so that the reduction of numerator and denominator to lowest terms is performed either at construction time or at selection time. The two rational-number implementations produce the same data abstraction, but the choice has an effect on efficiency. There is a similar relationship between streams and ordinary lists. As a data abstraction, streams are the same as lists. The difference is the time at which the elements are evaluated. With ordinary lists, both the <tt>car</tt> and the <tt>cdr</tt> are evaluated at construction time. With streams, the <tt>cdr</tt> is evaluated at selection time.


<a name="index_term_3768"></a><a name="index_term_3770"></a>Our implementation of streams will be based on a special form called <tt>delay</tt>. Evaluating <tt>(delay &lt;*exp*&gt;)</tt> does not evaluate the expression &lt;*exp*&gt;, but rather returns a so-called <a name="index_term_3772"></a>*delayed object*, which we can think of as a "promise" to evaluate &lt;*exp*&gt; at some future time. As a companion to <tt>delay</tt>, there is a procedure called <a name="index_term_3774"></a><tt>force</tt> that takes a delayed object as argument and performs the evaluation -- in effect, forcing the <tt>delay</tt> to fulfill its promise. We will see below how <tt>delay</tt> and <tt>force</tt> can be implemented, but first let us use these to construct streams.


<a name="index_term_3776"></a><a name="index_term_3778"></a><tt>Cons-stream</tt> is a special form defined so that


<tt>(cons-stream&#160;&lt;*a*&gt;&#160;&lt;*b*&gt;)<br></tt>


is equivalent to


<tt>(cons&#160;&lt;*a*&gt;&#160;(delay&#160;&lt;*b*&gt;))<br></tt>


What this means is that we will construct streams using pairs. However, rather than placing the value of the rest of the stream into the <tt>cdr</tt> of the pair we will put there a promise to compute the rest if it is ever requested. <tt>Stream-car</tt> and <tt>stream-cdr</tt> can now be defined as procedures:


<tt><a name="index_term_3780"></a>(define&#160;(stream-car&#160;stream)&#160;(car&#160;stream))<br>
<br>
<a name="index_term_3782"></a>(define&#160;(stream-cdr&#160;stream)&#160;(force&#160;(cdr&#160;stream)))<br></tt>


<tt>Stream-car</tt> selects the <tt>car</tt> of the pair; <tt>stream-cdr</tt> selects the <tt>cdr</tt> of the pair and evaluates the delayed expression found there to obtain the rest of the stream.<a name="call_footnote_Temp_446" href="#footnote_Temp_446" id="call_footnote_Temp_446"><sup><small>56</small></sup></a> <a name="%_sec_Temp_447"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_447">The stream implementation in action</a></h4>

To see how this implementation behaves, let us analyze the "outrageous" prime computation we saw above, reformulated in terms of streams:


<tt>(stream-car<br>
&#160;(stream-cdr<br>
&#160;&#160;(stream-filter&#160;prime?<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-enumerate-interval&#160;10000&#160;1000000))))<br></tt>


We will see that it does indeed work efficiently.


We begin by calling <tt>stream-enumerate-interval</tt> with the arguments 10,000 and 1,000,000. <tt>Stream-enumerate-interval</tt> is the stream analog of <tt>enumerate-interval</tt> (section&#160;<a href="chapter_2_section_2.html#%_sec_2.2.3">2.2.3</a>):


<tt><a name="index_term_3788"></a>(define&#160;(stream-enumerate-interval&#160;low&#160;high)<br>
&#160;&#160;(if&#160;(&gt;&#160;low&#160;high)<br>
&#160;&#160;&#160;&#160;&#160;&#160;the-empty-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;low<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-enumerate-interval&#160;(+&#160;low&#160;1)&#160;high))))<br></tt>


and thus the result returned by <tt>stream-enumerate-interval</tt>, formed by the <tt>cons-stream</tt>, is<a name="call_footnote_Temp_448" href="#footnote_Temp_448" id="call_footnote_Temp_448"><sup><small>57</small></sup></a>


<tt>(cons&#160;10000<br>
&#160;&#160;&#160;&#160;&#160;&#160;(delay&#160;(stream-enumerate-interval&#160;10001&#160;1000000)))<br></tt>


That is, <tt>stream-enumerate-interval</tt> returns a stream represented as a pair whose <tt>car</tt> is 10,000 and whose <tt>cdr</tt> is a promise to enumerate more of the interval if so requested. This stream is now filtered for primes, using the stream analog of the <tt>filter</tt> procedure (section&#160;<a href="chapter_2_section_2.html#%_sec_2.2.3">2.2.3</a>):


<tt><a name="index_term_3790"></a>(define&#160;(stream-filter&#160;pred&#160;stream)<br>
&#160;&#160;(cond&#160;((stream-null?&#160;stream)&#160;the-empty-stream)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((pred&#160;(stream-car&#160;stream))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream&#160;(stream-car&#160;stream)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-filter&#160;pred<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-cdr&#160;stream))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(stream-filter&#160;pred&#160;(stream-cdr&#160;stream)))))<br></tt>


<tt>Stream-filter</tt> tests the <tt>stream-car</tt> of the stream (the <tt>car</tt> of the pair, which is 10,000). Since this is not prime, <tt>stream-filter</tt> examines the <tt>stream-cdr</tt> of its input stream. The call to <tt>stream-cdr</tt> forces evaluation of the delayed <tt>stream-enumerate-interval</tt>, which now returns


<tt>(cons&#160;10001<br>
&#160;&#160;&#160;&#160;&#160;&#160;(delay&#160;(stream-enumerate-interval&#160;10002&#160;1000000)))<br></tt>


<tt>Stream-filter</tt> now looks at the <tt>stream-car</tt> of this stream, 10,001, sees that this is not prime either, forces another <tt>stream-cdr</tt>, and so on, until <tt>stream-enumerate-interval</tt> yields the prime 10,007, whereupon <tt>stream-filter</tt>, according to its definition, returns


<tt>(cons-stream&#160;(stream-car&#160;stream)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-filter&#160;pred&#160;(stream-cdr&#160;stream)))<br></tt>


which in this case is


<tt>(cons&#160;10007<br>
&#160;&#160;&#160;&#160;&#160;&#160;(delay<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-filter<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;prime?<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons&#160;10008<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(delay<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-enumerate-interval&#160;10009<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;1000000))))))<br></tt>


This result is now passed to <tt>stream-cdr</tt> in our original expression. This forces the delayed <tt>stream-filter</tt>, which in turn keeps forcing the delayed <tt>stream-enumerate-interval</tt> until it finds the next prime, which is 10,009. Finally, the result passed to <tt>stream-car</tt> in our original expression is


<tt>(cons&#160;10009<br>
&#160;&#160;&#160;&#160;&#160;&#160;(delay<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-filter<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;prime?<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons&#160;10010<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(delay<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-enumerate-interval&#160;10011<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;1000000))))))<br></tt>


<tt>Stream-car</tt> returns 10,009, and the computation is complete. Only as many integers were tested for primality as were necessary to find the second prime, and the interval was enumerated only as far as was necessary to feed the prime filter.


In general, we can think of delayed evaluation as <a name="index_term_3792"></a>"demand-driven" programming, whereby each stage in the stream process is activated only enough to satisfy the next stage. What we have done is to <a name="index_term_3794"></a>decouple the actual order of events in the computation from the apparent structure of our procedures. We write procedures as if the streams existed "all at once" when, in reality, the computation is performed incrementally, as in traditional programming styles.


<a name="%_sec_Temp_449"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_449">Implementing <tt>delay</tt> and <tt>force</tt></a></h4>

<a name="index_term_3796"></a>Although <tt>delay</tt> and <tt>force</tt> may seem like mysterious operations, their implementation is really quite straightforward. <tt>Delay</tt> must package an expression so that it can be evaluated later on demand, and we can accomplish this simply by treating the expression as the body of a procedure. <tt>Delay</tt> can be a special form such that


<tt>(delay&#160;&lt;*exp*&gt;)<br></tt>


is syntactic sugar for


<tt>(lambda&#160;()&#160;&lt;*exp*&gt;)<br></tt>


<tt>Force</tt> simply calls the procedure (of no arguments) produced by <tt>delay</tt>, so we can implement <tt>force</tt> as a procedure:


<tt><a name="index_term_3798"></a>(define&#160;(force&#160;delayed-object)<br>
&#160;&#160;(delayed-object))<br></tt>


<a name="index_term_3800"></a><a name="index_term_3802"></a>This implementation suffices for <tt>delay</tt> and <tt>force</tt> to work as advertised, but there is an important optimization that we can include. In many applications, we end up forcing the same delayed object many times. This can lead to serious inefficiency in recursive programs involving streams. (See exercise&#160;<a href="#%_thm_3.57">3.57</a>.) The solution is to build delayed objects so that the first time they are forced, they store the value that is computed. Subsequent forcings will simply return the stored value without repeating the computation. In other words, we implement <tt>delay</tt> as a special-purpose memoized procedure similar to the one described in exercise&#160;<a href="chapter_3_section_3.html#exercise_3_27">3.27</a>. One way to accomplish this is to use the following procedure, which takes as argument a procedure (of no arguments) and returns a memoized version of the procedure. The first time the memoized procedure is run, it saves the computed result. On subsequent evaluations, it simply returns the result.


<tt><a name="index_term_3804"></a>(define&#160;(memo-proc&#160;proc)<br>
&#160;&#160;(let&#160;((already-run?&#160;false)&#160;(result&#160;false))<br>
&#160;&#160;&#160;&#160;(lambda&#160;()<br>
&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;(not&#160;already-run?)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(begin&#160;(set!&#160;result&#160;(proc))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(set!&#160;already-run?&#160;true)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;result)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;result))))<br></tt>


<tt>Delay</tt> is then defined so that <tt>(delay &lt;*exp*&gt;)</tt> is equivalent to


<tt>(memo-proc&#160;(lambda&#160;()&#160;&lt;*exp*&gt;))<br></tt>


and <tt>force</tt> is as defined previously.<a name="call_footnote_Temp_450" href="#footnote_Temp_450" id="call_footnote_Temp_450"><sup><small>58</small></sup></a>


<a name="%_thm_3.50"></a> **Exercise 3.50.**&#160;&#160;Complete the following definition, which generalizes <tt>stream-map</tt> to allow procedures that take multiple arguments, analogous to <tt>map</tt> in section&#160;<a href="chapter_2_section_2.html#%_sec_2.2.3">2.2.3</a>, footnote&#160;<a href="chapter_2_section_2.html#footnote_Temp_166">12</a>.


<tt><a name="index_term_3824"></a>(define&#160;(stream-map&#160;proc&#160;.&#160;argstreams)<br>
&#160;&#160;(if&#160;(&lt;*??*&gt;&#160;(car&#160;argstreams))<br>
&#160;&#160;&#160;&#160;&#160;&#160;the-empty-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;(&lt;*??*&gt;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(apply&#160;proc&#160;(map&#160;&lt;*??*&gt;&#160;argstreams))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(apply&#160;stream-map<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons&#160;proc&#160;(map&#160;&lt;*??*&gt;&#160;argstreams))))))<br></tt>


<a name="%_thm_3.51"></a> **Exercise 3.51.**&#160;&#160;<a name="index_term_3826"></a>In order to take a closer look at delayed evaluation, we will use the following procedure, which simply returns its argument after printing it:


<tt>(define&#160;(show&#160;x)<br>
&#160;&#160;(display-line&#160;x)<br>
&#160;&#160;x)<br></tt>


What does the interpreter print in response to evaluating each expression in the following sequence?<a name="call_footnote_Temp_453" href="#footnote_Temp_453" id="call_footnote_Temp_453"><sup><small>59</small></sup></a>


<tt>(define&#160;x&#160;(stream-map&#160;show&#160;(stream-enumerate-interval&#160;0&#160;10)))<br>
(stream-ref&#160;x&#160;5)<br>
(stream-ref&#160;x&#160;7)<br></tt>


<a name="%_thm_3.52"></a> **Exercise 3.52.**&#160;&#160;<a name="index_term_3830"></a>Consider the sequence of expressions


<tt>(define&#160;sum&#160;0)<br>
(define&#160;(accum&#160;x)<br>
&#160;&#160;(set!&#160;sum&#160;(+&#160;x&#160;sum))<br>
&#160;&#160;sum)<br>
(define&#160;seq&#160;(stream-map&#160;accum&#160;(stream-enumerate-interval&#160;1&#160;20)))<br>
(define&#160;y&#160;(stream-filter&#160;even?&#160;seq))<br>
(define&#160;z&#160;(stream-filter&#160;(lambda&#160;(x)&#160;(=&#160;(remainder&#160;x&#160;5)&#160;0))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;seq))<br>
(stream-ref&#160;y&#160;7)<br>
(display-stream&#160;z)<br></tt>


What is the value of <tt>sum</tt> after each of the above expressions is evaluated? What is the printed response to evaluating the <tt>stream-ref</tt> and <tt>display-stream</tt> expressions? Would these responses differ if we had implemented <tt>(delay&#160;&lt;*exp*&gt;)</tt> simply as <tt>(lambda () &lt;*exp*&gt;)</tt> without using the optimization provided by <tt>memo-proc</tt> ? Explain.


<a name="%_sec_3.5.2"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_3.5.2">3.5.2&#160;&#160;Infinite Streams</a></h3>

<a name="index_term_3832"></a> We have seen how to support the illusion of manipulating streams as complete entities even though, in actuality, we compute only as much of the stream as we need to access. We can exploit this technique to represent sequences efficiently as streams, even if the sequences are very long. What is more striking, we can use streams to represent sequences that are infinitely long. For instance, consider the following definition of the stream of positive integers:


<tt><a name="index_term_3834"></a>(define&#160;(integers-starting-from&#160;n)<br>
&#160;&#160;(cons-stream&#160;n&#160;(integers-starting-from&#160;(+&#160;n&#160;1))))<br>
<br>
<a name="index_term_3836"></a>(define&#160;integers&#160;(integers-starting-from&#160;1))<br></tt>


This makes sense because <tt>integers</tt> will be a pair whose <tt>car</tt> is 1 and whose <tt>cdr</tt> is a promise to produce the integers beginning with 2. This is an infinitely long stream, but in any given time we can examine only a finite portion of it. Thus, our programs will never know that the entire infinite stream is not there.


Using <tt>integers</tt> we can define other infinite streams, such as the stream of integers that are not divisible by 7:


<tt><a name="index_term_3838"></a>(define&#160;(divisible?&#160;x&#160;y)&#160;(=&#160;(remainder&#160;x&#160;y)&#160;0))<br>
(define&#160;no-sevens<br>
&#160;&#160;(stream-filter&#160;(lambda&#160;(x)&#160;(not&#160;(divisible?&#160;x&#160;7)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;integers))<br></tt>


Then we can find integers not divisible by 7 simply by accessing elements of this stream:


<tt>(stream-ref&#160;no-sevens&#160;100)<br>
<i>117</i><br></tt>


In analogy with <tt>integers</tt>, we can define the infinite stream of Fibonacci numbers:


<tt>(define&#160;(fibgen&#160;a&#160;b)<br>
&#160;&#160;(cons-stream&#160;a&#160;(fibgen&#160;b&#160;(+&#160;a&#160;b))))<br>
<a name="index_term_3840"></a>(define&#160;fibs&#160;(fibgen&#160;0&#160;1))<br></tt>


<tt>Fibs</tt> is a pair whose <tt>car</tt> is 0 and whose <tt>cdr</tt> is a promise to evaluate <tt>(fibgen 1 1)</tt>. When we evaluate this delayed <tt>(fibgen 1 1)</tt>, it will produce a pair whose <tt>car</tt> is 1 and whose <tt>cdr</tt> is a promise to evaluate <tt>(fibgen&#160;1&#160;2)</tt>, and so on.


<a name="index_term_3842"></a>For a look at a more exciting infinite stream, we can generalize the <tt>no-sevens</tt> example to construct the infinite stream of prime numbers, using a method known as the <a name="index_term_3844"></a>*sieve of Eratosthenes*.<a name="call_footnote_Temp_455" href="#footnote_Temp_455" id="call_footnote_Temp_455"><sup><small>60</small></sup></a> We start with the integers beginning with 2, which is the first prime. To get the rest of the primes, we start by filtering the multiples of 2 from the rest of the integers. This leaves a stream beginning with 3, which is the next prime. Now we filter the multiples of 3 from the rest of this stream. This leaves a stream beginning with 5, which is the next prime, and so on. In other words, we construct the primes by a sieving process, described as follows: To sieve a stream <tt>S</tt>, form a stream whose first element is the first element of <tt>S</tt> and the rest of which is obtained by filtering all multiples of the first element of <tt>S</tt> out of the rest of <tt>S</tt> and sieving the result. This process is readily described in terms of stream operations:


<tt><a name="index_term_3852"></a>(define&#160;(sieve&#160;stream)<br>
&#160;&#160;(cons-stream<br>
&#160;&#160;&#160;(stream-car&#160;stream)<br>
&#160;&#160;&#160;(sieve&#160;(stream-filter<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda&#160;(x)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(not&#160;(divisible?&#160;x&#160;(stream-car&#160;stream))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-cdr&#160;stream)))))<br>
<br>
<a name="index_term_3854"></a>(define&#160;primes&#160;(sieve&#160;(integers-starting-from&#160;2)))<br></tt>


Now to find a particular prime we need only ask for it:


<tt>(stream-ref&#160;primes&#160;50)<br>
<i>233</i><br></tt>


It is interesting to contemplate the signal-processing system set up by <tt>sieve</tt>, shown in the <a name="index_term_3856"></a>"Henderson diagram" in figure&#160;<a href="#%_fig_3.31">3.31</a>.<a name="call_footnote_Temp_456" href="#footnote_Temp_456" id="call_footnote_Temp_456"><sup><small>61</small></sup></a> The input stream feeds into an "un<tt>cons</tt>er" that separates the first element of the stream from the rest of the stream. The first element is used to construct a divisibility filter, through which the rest is passed, and the output of the filter is fed to another sieve box. Then the original first element is <tt>cons</tt>ed onto the output of the internal sieve to form the output stream. Thus, not only is the stream infinite, but the signal processor is also infinite, because the sieve contains a sieve within it.


<a name="%_fig_3.31"></a>

<div align="left">
  <div align="left">
    **Figure 3.31:**&#160;&#160;The prime sieve viewed as a signal-processing system.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_3_image_35.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="%_sec_Temp_457"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_457">Defining streams implicitly</a></h4>

<a name="index_term_3860"></a> The <tt>integers</tt> and <tt>fibs</tt> streams above were defined by specifying "generating" procedures that explicitly compute the stream elements one by one. An alternative way to specify streams is to take advantage of delayed evaluation to define streams implicitly. For example, the following expression defines the stream <tt>ones</tt> to be an infinite stream of ones:


<tt><a name="index_term_3862"></a>(define&#160;ones&#160;(cons-stream&#160;1&#160;ones))<br></tt>


This works much like the definition of a recursive procedure: <tt>ones</tt> is a pair whose <tt>car</tt> is 1 and whose <tt>cdr</tt> is a promise to evaluate <tt>ones</tt>. Evaluating the <tt>cdr</tt> gives us again a 1 and a promise to evaluate <tt>ones</tt>, and so on.


We can do more interesting things by manipulating streams with operations such as <tt>add-streams</tt>, which produces the elementwise sum of two given streams:<a name="call_footnote_Temp_458" href="#footnote_Temp_458" id="call_footnote_Temp_458"><sup><small>62</small></sup></a>


<tt><a name="index_term_3864"></a>(define&#160;(add-streams&#160;s1&#160;s2)<br>
&#160;&#160;(stream-map&#160;+&#160;s1&#160;s2))<br></tt>


Now we can define the integers as follows:


<tt><a name="index_term_3866"></a>(define&#160;integers&#160;(cons-stream&#160;1&#160;(add-streams&#160;ones&#160;integers)))<br></tt>


This defines <tt>integers</tt> to be a stream whose first element is 1 and the rest of which is the sum of <tt>ones</tt> and <tt>integers</tt>. Thus, the second element of <tt>integers</tt> is 1 plus the first element of <tt>integers</tt>, or 2; the third element of <tt>integers</tt> is 1 plus the second element of <tt>integers</tt>, or 3; and so on. This definition works because, at any point, enough of the <tt>integers</tt> stream has been generated so that we can feed it back into the definition to produce the next integer.


We can define the Fibonacci numbers in the same style:


<tt><a name="index_term_3868"></a>(define&#160;fibs<br>
&#160;&#160;(cons-stream&#160;0<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream&#160;1<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(add-streams&#160;(stream-cdr&#160;fibs)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;fibs))))<br></tt>


This definition says that <tt>fibs</tt> is a stream beginning with 0 and 1, such that the rest of the stream can be generated by adding <tt>fibs</tt> to itself shifted by one place:

<table border="0">
  <tr>
    <td valign="top"></td>

    <td valign="top"></td>

    <td valign="top">1</td>

    <td valign="top">1</td>

    <td valign="top">2</td>

    <td valign="top">3</td>

    <td valign="top">5</td>

    <td valign="top">8</td>

    <td valign="top">13</td>

    <td valign="top">21</td>

    <td valign="top"><tt>...</tt> = <tt>(stream-cdr fibs)</tt></td>
  </tr>

  <tr>
    <td valign="top"></td>

    <td valign="top"></td>

    <td valign="top">0</td>

    <td valign="top">1</td>

    <td valign="top">1</td>

    <td valign="top">2</td>

    <td valign="top">3</td>

    <td valign="top">5</td>

    <td valign="top">8</td>

    <td valign="top">13</td>

    <td valign="top"><tt>...</tt> = <tt>fibs</tt></td>
  </tr>

  <tr>
    <td valign="top">0</td>

    <td valign="top">1</td>

    <td valign="top">1</td>

    <td valign="top">2</td>

    <td valign="top">3</td>

    <td valign="top">5</td>

    <td valign="top">8</td>

    <td valign="top">13</td>

    <td valign="top">21</td>

    <td valign="top">34</td>

    <td valign="top"><tt>...</tt> = <tt>fibs</tt></td>
  </tr>

  <tr>
    <td valign="top"></td>
  </tr>
</table>

<tt>Scale-stream</tt> is another useful procedure in formulating such stream definitions. This multiplies each item in a stream by a given constant:


<tt><a name="index_term_3870"></a>(define&#160;(scale-stream&#160;stream&#160;factor)<br>
&#160;&#160;(stream-map&#160;(lambda&#160;(x)&#160;(*&#160;x&#160;factor))&#160;stream))<br></tt>


For example,


<tt>(define&#160;double&#160;(cons-stream&#160;1&#160;(scale-stream&#160;double&#160;2)))<br></tt>


produces the stream of powers of 2: 1, 2, 4, 8, 16, 32, <tt>...</tt>.


An alternate definition of the stream of primes can be given by starting with the integers and filtering them by testing for primality. We will need the first prime, 2, to get started:


<tt><a name="index_term_3872"></a>(define&#160;primes<br>
&#160;&#160;(cons-stream<br>
&#160;&#160;&#160;2<br>
&#160;&#160;&#160;(stream-filter&#160;prime?&#160;(integers-starting-from&#160;3))))<br></tt>


This definition is not so straightforward as it appears, because we will test whether a number *n* is prime by checking whether *n* is divisible by a prime (not by just any integer) less than or equal to <img src="img/book_13.gif" border="0">*n*:


<tt><a name="index_term_3874"></a>(define&#160;(prime?&#160;n)<br>
&#160;&#160;(define&#160;(iter&#160;ps)<br>
&#160;&#160;&#160;&#160;(cond&#160;((&gt;&#160;(square&#160;(stream-car&#160;ps))&#160;n)&#160;true)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((divisible?&#160;n&#160;(stream-car&#160;ps))&#160;false)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(iter&#160;(stream-cdr&#160;ps)))))<br>
&#160;&#160;(iter&#160;primes))<br></tt>


This is a recursive definition, since <tt>primes</tt> is defined in terms of the <tt>prime?</tt> predicate, which itself uses the <tt>primes</tt> stream. The reason this procedure works is that, at any point, enough of the <tt>primes</tt> stream has been generated to test the primality of the numbers we need to check next. That is, for every *n* we test for primality, either *n* is not prime (in which case there is a prime already generated that divides it) or *n* is prime (in which case there is a prime already generated -- i.e., a prime less than *n* -- that is greater than <img src="img/book_13.gif" border="0">*n*).<a name="call_footnote_Temp_459" href="#footnote_Temp_459" id="call_footnote_Temp_459"><sup><small>63</small></sup></a>


<a name="%_thm_3.53"></a> **Exercise 3.53.**&#160;&#160;Without running the program, describe the elements of the stream defined by


<tt>(define&#160;s&#160;(cons-stream&#160;1&#160;(add-streams&#160;s&#160;s)))<br></tt>


<a name="%_thm_3.54"></a> **Exercise 3.54.**&#160;&#160;Define a procedure <a name="index_term_3886"></a><a name="index_term_3888"></a><a name="index_term_3890"></a><tt>mul-streams</tt>, analogous to <tt>add-streams</tt>, that produces the elementwise product of its two input streams. Use this together with the stream of <tt>integers</tt> to complete the following definition of the stream whose *n*th element (counting from 0) is *n* + 1 factorial:


<tt>(define&#160;factorials&#160;(cons-stream&#160;1&#160;(mul-streams&#160;&lt;*??*&gt;&#160;&lt;*??*&gt;)))<br></tt>


<a name="%_thm_3.55"></a> **Exercise 3.55.**&#160;&#160;Define a procedure <a name="index_term_3892"></a><tt>partial-sums</tt> that takes as argument a stream *S* and returns the stream whose elements are *S*<sub>0</sub>, *S*<sub>0</sub> + *S*<sub>1</sub>, *S*<sub>0</sub> + *S*<sub>1</sub> + *S*<sub>2</sub>, <tt>...</tt>. For example, <tt>(partial-sums integers)</tt> should be the stream 1, 3, 6, 10, 15, <tt>...</tt>.


<a name="%_thm_3.56"></a> **Exercise 3.56.**&#160;&#160;A famous problem, first raised by <a name="index_term_3894"></a>R. Hamming, is to enumerate, in ascending order with no repetitions, all positive integers with no prime factors other than 2, 3, or 5. One obvious way to do this is to simply test each integer in turn to see whether it has any factors other than 2, 3, and 5. But this is very inefficient, since, as the integers get larger, fewer and fewer of them fit the requirement. As an alternative, let us call the required stream of numbers <tt>S</tt> and notice the following facts about it.

<ul>
  <li><tt>S</tt> begins with 1.</li>

  <li>The elements of <tt>(scale-stream S 2)</tt> are also elements of <tt>S</tt>.</li>

  <li>The same is true for <tt>(scale-stream S 3)</tt> and <tt>(scale-stream 5 S)</tt>.</li>

  <li>These are all the elements of <tt>S</tt>.</li>
</ul>

<a name="index_term_3896"></a>Now all we have to do is combine elements from these sources. For this we define a procedure <tt>merge</tt> that combines two ordered streams into one ordered result stream, eliminating repetitions:


<tt><a name="index_term_3898"></a>(define&#160;(merge&#160;s1&#160;s2)<br>
&#160;&#160;(cond&#160;((stream-null?&#160;s1)&#160;s2)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((stream-null?&#160;s2)&#160;s1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(let&#160;((s1car&#160;(stream-car&#160;s1))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(s2car&#160;(stream-car&#160;s2)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cond&#160;((&lt;&#160;s1car&#160;s2car)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream&#160;s1car&#160;(merge&#160;(stream-cdr&#160;s1)&#160;s2)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((&gt;&#160;s1car&#160;s2car)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream&#160;s2car&#160;(merge&#160;s1&#160;(stream-cdr&#160;s2))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream&#160;s1car<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(merge&#160;(stream-cdr&#160;s1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-cdr&#160;s2)))))))))<br></tt>


Then the required stream may be constructed with <tt>merge</tt>, as follows:


<tt>(define&#160;S&#160;(cons-stream&#160;1&#160;(merge&#160;&lt;*??*&gt;&#160;&lt;*??*&gt;)))<br></tt>


Fill in the missing expressions in the places marked &lt;*??*&gt; above.


<a name="%_thm_3.57"></a> **Exercise 3.57.**&#160;&#160;<a name="index_term_3900"></a>How many additions are performed when we compute the *n*th Fibonacci number using the definition of <tt>fibs</tt> based on the <tt>add-streams</tt> procedure? Show that the number of additions would be exponentially greater if we had implemented <tt>(delay &lt;*exp*&gt;)</tt> simply as <tt>(lambda () &lt;*exp*&gt;)</tt>, without using the optimization provided by the <tt>memo-proc</tt> procedure described in section&#160;<a href="#%_sec_3.5.1">3.5.1</a>.<a name="call_footnote_Temp_465" href="#footnote_Temp_465" id="call_footnote_Temp_465"><sup><small>64</small></sup></a>


<a name="%_thm_3.58"></a> **Exercise 3.58.**&#160;&#160;Give an interpretation of the stream computed by the following procedure:


<tt>(define&#160;(expand&#160;num&#160;den&#160;radix)<br>
&#160;&#160;(cons-stream<br>
&#160;&#160;&#160;(quotient&#160;(*&#160;num&#160;radix)&#160;den)<br>
&#160;&#160;&#160;(expand&#160;(remainder&#160;(*&#160;num&#160;radix)&#160;den)&#160;den&#160;radix)))<br></tt>


<a name="index_term_3906"></a><a name="index_term_3908"></a>(<tt>Quotient</tt> is a primitive that returns the integer quotient of two integers.) What are the successive elements produced by <tt>(expand 1 7 10)</tt> ? What is produced by <tt>(expand 3 8 10)</tt> ?


<a name="%_thm_3.59"></a> **Exercise 3.59.**&#160;&#160;<a name="index_term_3910"></a><a name="index_term_3912"></a>In section&#160;<a href="chapter_2_section_5.html#%_sec_2.5.3">2.5.3</a> we saw how to implement a polynomial arithmetic system representing polynomials as lists of terms. In a similar way, we can work with *power series*, such as


<a name="index_term_3914"></a>

<div align="left">
  <img src="img/chapter_3_image_36.gif" border="0">
</div>

<a name="index_term_3916"></a>

<div align="left">
  <img src="img/chapter_3_image_37.gif" border="0">
</div>

<a name="index_term_3918"></a>

<div align="left">
  <img src="img/chapter_3_image_38.gif" border="0">
</div>

represented as infinite streams. We will represent the series *a*<sub>0</sub> + *a*<sub>1</sub> *x* + *a*<sub>2</sub> *x*<sup>2</sup> + *a*<sub>3</sub> *x*<sup>3</sup> + <tt>���</tt> as the stream whose elements are the coefficients *a*<sub>0</sub>, *a*<sub>1</sub>, *a*<sub>2</sub>, *a*<sub>3</sub>, <tt>...</tt>.


<a name="index_term_3920"></a><a name="index_term_3922"></a>a. The integral of the series *a*<sub>0</sub> + *a*<sub>1</sub> *x* + *a*<sub>2</sub> *x*<sup>2</sup> + *a*<sub>3</sub> *x*<sup>3</sup> + <tt>���</tt> is the series

<div align="left">
  <img src="img/chapter_3_image_39.gif" border="0">
</div>

where *c* is any constant. Define a procedure <a name="index_term_3924"></a><tt>integrate-series</tt> that takes as input a stream *a*<sub>0</sub>, *a*<sub>1</sub>, *a*<sub>2</sub>, <tt>...</tt> representing a power series and returns the stream *a*<sub>0</sub>, (1/2)*a*<sub>1</sub>, (1/3)*a*<sub>2</sub>, <tt>...</tt> of coefficients of the non-constant terms of the integral of the series. (Since the result has no constant term, it doesn't represent a power series; when we use <tt>integrate-series</tt>, we will <tt>cons</tt> on the appropriate constant.)


b. The function *x* <img src="img/book_17.gif" border="0"> *e*<sup>*x*</sup> is its own derivative. This implies that *e*<sup>*x*</sup> and the integral of *e*<sup>*x*</sup> are the same series, except for the constant term, which is *e*<sup>0</sup> = 1. Accordingly, we can generate the series for *e*<sup>*x*</sup> as


<tt>(define&#160;exp-series<br>
&#160;&#160;(cons-stream&#160;1&#160;(integrate-series&#160;exp-series)))<br></tt>


Show how to generate the series for sine and cosine, starting from the facts that the derivative of sine is cosine and the derivative of cosine is the negative of sine:


<tt>(define&#160;cosine-series<br>
&#160;&#160;(cons-stream&#160;1&#160;&lt;*??*&gt;))<br>
(define&#160;sine-series<br>
&#160;&#160;(cons-stream&#160;0&#160;&lt;*??*&gt;))<br></tt>


<a name="%_thm_3.60"></a> **Exercise 3.60.**&#160;&#160;<a name="index_term_3926"></a><a name="index_term_3928"></a><a name="index_term_3930"></a><a name="index_term_3932"></a>With power series represented as streams of coefficients as in exercise&#160;<a href="#%_thm_3.59">3.59</a>, adding series is implemented by <tt>add-streams</tt>. Complete the definition of the following procedure for multiplying series:


<tt>(define&#160;(mul-series&#160;s1&#160;s2)<br>
&#160;&#160;(cons-stream&#160;&lt;*??*&gt;&#160;(add-streams&#160;&lt;*??*&gt;&#160;&lt;*??*&gt;)))<br></tt>


You can test your procedure by verifying that *s**i**n*<sup>2</sup> *x* + *c**o**s*<sup>2</sup> *x* = 1, using the series from exercise&#160;<a href="#%_thm_3.59">3.59</a>.


<a name="%_thm_3.61"></a> **Exercise 3.61.**&#160;&#160;Let *S* be a power series (exercise&#160;<a href="#%_thm_3.59">3.59</a>) whose constant term is 1. Suppose we want to find the power series 1/*S*, that is, the series *X* such that *S* � *X* = 1. Write *S* = 1 + *S*<sub>*R*</sub> where *S*<sub>*R*</sub> is the part of *S* after the constant term. Then we can solve for *X* as follows:

<div align="left">
  <img src="img/chapter_3_image_40.gif" border="0">
</div>

In other words, *X* is the power series whose constant term is 1 and whose higher-order terms are given by the negative of *S*<sub>*R*</sub> times *X*. Use this idea to write a procedure <tt>invert-unit-series</tt> that computes 1/*S* for a power series *S* with constant term 1. You will need to use <tt>mul-series</tt> from exercise&#160;<a href="#%_thm_3.60">3.60</a>.


<a name="%_thm_3.62"></a> **Exercise 3.62.**&#160;&#160;<a name="index_term_3934"></a><a name="index_term_3936"></a><a name="index_term_3938"></a>Use the results of exercises&#160;<a href="#%_thm_3.60">3.60</a> and&#160;<a href="#%_thm_3.61">3.61</a> to define a procedure <tt>div-series</tt> that divides two power series. <tt>Div-series</tt> should work for any two series, provided that the denominator series begins with a nonzero constant term. (If the denominator has a zero constant term, then <tt>div-series</tt> should signal an error.) Show how to use <tt>div-series</tt> together with the result of exercise&#160;<a href="#%_thm_3.59">3.59</a> to generate <a name="index_term_3940"></a>the power series for tangent.


<a name="%_sec_3.5.3"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_3.5.3">3.5.3&#160;&#160;Exploiting the Stream Paradigm</a></h3>

Streams with delayed evaluation can be a powerful modeling tool, providing many of the benefits of local state and assignment. Moreover, they avoid some of the theoretical tangles that accompany the introduction of assignment into a programming language.


<a name="index_term_3942"></a>The stream approach can be illuminating because it allows us to build systems with different module boundaries than systems organized around assignment to state variables. For example, we can think of an entire time series (or signal) as a focus of interest, rather than the values of the state variables at individual moments. This makes it convenient to combine and compare components of state from different moments.


<a name="%_sec_Temp_471"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_471">Formulating iterations as stream processes</a></h4>

<a name="index_term_3944"></a> In section&#160;<a href="chapter_1_section_2.html#%_sec_1.2.1">1.2.1</a>, we introduced iterative processes, which proceed by updating state variables. We know now that we can represent state as a "timeless" stream of values rather than as a set of variables to be updated. Let's adopt this perspective in revisiting the square-root procedure from section&#160;<a href="chapter_1_section_1.html#%_sec_1.1.7">1.1.7</a>. Recall that the idea is to generate a sequence of better and better guesses for the square root of *x* by applying over and over again the procedure that improves guesses:


<tt>(define&#160;(sqrt-improve&#160;guess&#160;x)<br>
&#160;&#160;(average&#160;guess&#160;(/&#160;x&#160;guess)))<br></tt>


<a name="index_term_3946"></a>In our original <tt>sqrt</tt> procedure, we made these guesses be the successive values of a state variable. Instead we can generate the infinite stream of guesses, starting with an initial guess of 1:<a name="call_footnote_Temp_472" href="#footnote_Temp_472" id="call_footnote_Temp_472"><sup><small>65</small></sup></a>


<tt><a name="index_term_3948"></a>(define&#160;(sqrt-stream&#160;x)<br>
&#160;&#160;(define&#160;guesses<br>
&#160;&#160;&#160;&#160;(cons-stream&#160;1.0<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-map&#160;(lambda&#160;(guess)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(sqrt-improve&#160;guess&#160;x))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;guesses)))<br>
&#160;&#160;guesses)<br>
(display-stream&#160;(sqrt-stream&#160;2))<br>
<i>1.</i><br>
<i>1.5</i><br>
<i>1.4166666666666665</i><br>
<i>1.4142156862745097</i><br>
<i>1.4142135623746899</i><br></tt>...


We can generate more and more terms of the stream to get better and better guesses. If we like, we can write a procedure that keeps generating terms until the answer is good enough. (See exercise&#160;<a href="#%_thm_3.64">3.64</a>.)


<a name="index_term_3950"></a><a name="index_term_3952"></a><a name="index_term_3954"></a><a name="index_term_3956"></a><a name="index_term_3958"></a><a name="index_term_3960"></a>Another iteration that we can treat in the same way is to generate an approximation to <img src="img/book_09.gif" border="0">, based upon the alternating series that we saw in section&#160;<a href="chapter_1_section_3.html#%_sec_1.3.1">1.3.1</a>:

<div align="left">
  <img src="img/chapter_3_image_41.gif" border="0">
</div>

We first generate the stream of summands of the series (the reciprocals of the odd integers, with alternating signs). Then we take the stream of sums of more and more terms (using the <tt>partial-sums</tt> procedure of exercise&#160;<a href="#%_thm_3.55">3.55</a>) and scale the result by 4:


<tt>(define&#160;(pi-summands&#160;n)<br>
&#160;&#160;(cons-stream&#160;(/&#160;1.0&#160;n)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-map&#160;-&#160;(pi-summands&#160;(+&#160;n&#160;2)))))<br>
<a name="index_term_3962"></a>(define&#160;pi-stream<br>
&#160;&#160;(scale-stream&#160;(partial-sums&#160;(pi-summands&#160;1))&#160;4))<br>
(display-stream&#160;pi-stream)<br>
<i>4.</i><br>
<i>2.666666666666667</i><br>
<i>3.466666666666667</i><br>
<i>2.8952380952380956</i><br>
<i>3.3396825396825403</i><br>
<i>2.9760461760461765</i><br>
<i>3.2837384837384844</i><br>
<i>3.017071817071818</i><br></tt>...


This gives us a stream of better and better approximations to <img src="img/book_09.gif" border="0">, although the approximations converge rather slowly. Eight terms of the sequence bound the value of <img src="img/book_09.gif" border="0"> between 3.284 and 3.017.


<a name="index_term_3964"></a>So far, our use of the stream of states approach is not much different from updating state variables. But streams give us an opportunity to do some interesting tricks. For example, we can transform a stream with a <a name="index_term_3966"></a>*sequence accelerator* that converts a sequence of approximations to a new sequence that converges to the same value as the original, only faster.


One such accelerator, due to the eighteenth-century Swiss mathematician <a name="index_term_3968"></a>Leonhard Euler, works well with sequences that are partial sums of alternating series (series of terms with alternating signs). In Euler's technique, if *S*<sub>*n*</sub> is the *n*th term of the original sum sequence, then the accelerated sequence has terms

<div align="left">
  <img src="img/chapter_3_image_42.gif" border="0">
</div>

Thus, if the original sequence is represented as a stream of values, the transformed sequence is given by


<tt><a name="index_term_3970"></a>(define&#160;(euler-transform&#160;s)<br>
&#160;&#160;(let&#160;((s0&#160;(stream-ref&#160;s&#160;0))&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*;&#160;*S</tt><sub>*n*-1</sub><br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(s1&#160;(stream-ref&#160;s&#160;1))&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*;&#160;*S<sub>*n*</sub><br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(s2&#160;(stream-ref&#160;s&#160;2)))&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*;&#160;*S<sub>*n*+1</sub><br>
&#160;&#160;&#160;&#160;(cons-stream&#160;(-&#160;s2&#160;(/&#160;(square&#160;(-&#160;s2&#160;s1))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(+&#160;s0&#160;(*&#160;-2&#160;s1)&#160;s2)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(euler-transform&#160;(stream-cdr&#160;s)))))<br>


We can demonstrate Euler acceleration with our sequence of approximations to <img src="img/book_09.gif" border="0">:


<tt>(display-stream&#160;(euler-transform&#160;pi-stream))<br>
<i>3.166666666666667</i><br>
<i>3.1333333333333337</i><br>
<i>3.1452380952380956</i><br>
<i>3.13968253968254</i><br>
<i>3.1427128427128435</i><br>
<i>3.1408813408813416</i><br>
<i>3.142071817071818</i><br>
<i>3.1412548236077655</i><br></tt>...


Even better, we can accelerate the accelerated sequence, and recursively accelerate that, and so on. Namely, we create a stream of streams (a structure we'll call a <a name="index_term_3972"></a>*tableau*) in which each stream is the transform of the preceding one:


<tt><a name="index_term_3974"></a>(define&#160;(make-tableau&#160;transform&#160;s)<br>
&#160;&#160;(cons-stream&#160;s<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(make-tableau&#160;transform<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(transform&#160;s))))<br></tt>


The tableau has the form

<div align="left">
  <img src="img/chapter_3_image_43.gif" border="0">
</div>

Finally, we form a sequence by taking the first term in each row of the tableau:


<tt><a name="index_term_3976"></a>(define&#160;(accelerated-sequence&#160;transform&#160;s)<br>
&#160;&#160;(stream-map&#160;stream-car<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(make-tableau&#160;transform&#160;s)))<br></tt>


We can demonstrate this kind of "super-acceleration" of the <img src="img/book_09.gif" border="0"> sequence:


<tt>(display-stream&#160;(accelerated-sequence&#160;euler-transform<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;pi-stream))<br>
<i>4.</i><br>
<i>3.166666666666667</i><br>
<i>3.142105263157895</i><br>
<i>3.141599357319005</i><br>
<i>3.1415927140337785</i><br>
<i>3.1415926539752927</i><br>
<i>3.1415926535911765</i><br>
<i>3.141592653589778</i><br></tt>...


The result is impressive. Taking eight terms of the sequence yields the correct value of <img src="img/book_09.gif" border="0"> to 14 decimal places. If we had used only the original <img src="img/book_09.gif" border="0"> sequence, we would need to compute on the order of 10<sup>13</sup> terms (i.e., expanding the series far enough so that the individual terms are less then 10<sup>-13</sup>) to get that much accuracy! We could have implemented these acceleration techniques without using streams. But the stream formulation is particularly elegant and convenient because the entire sequence of states is available to us as a data structure that can be manipulated with a uniform set of operations.


<a name="%_thm_3.63"></a> **Exercise 3.63.**&#160;&#160;Louis Reasoner asks why the <tt>sqrt-stream</tt> procedure was not written in the following more straightforward way, without the local variable <tt>guesses</tt>:


<tt>(define&#160;(sqrt-stream&#160;x)<br>
&#160;&#160;(cons-stream&#160;1.0<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-map&#160;(lambda&#160;(guess)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(sqrt-improve&#160;guess&#160;x))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(sqrt-stream&#160;x))))<br></tt>


Alyssa P. Hacker replies that this version of the procedure is considerably less efficient because it performs redundant computation. Explain Alyssa's answer. Would the two versions still differ in efficiency if our implementation of <tt>delay</tt> used only <tt>(lambda () &lt;*exp*&gt;)</tt> without using the optimization provided by <tt>memo-proc</tt> (section&#160;<a href="#%_sec_3.5.1">3.5.1</a>)?


<a name="%_thm_3.64"></a> **Exercise 3.64.**&#160;&#160;Write a procedure <a name="index_term_3978"></a><tt>stream-limit</tt> that takes as arguments a stream and a number (the tolerance). It should examine the stream until it finds two successive elements that differ in absolute value by less than the tolerance, and return the second of the two elements. Using this, we could compute square roots up to a given tolerance by


<tt><a name="index_term_3980"></a>(define&#160;(sqrt&#160;x&#160;tolerance)<br>
&#160;&#160;(stream-limit&#160;(sqrt-stream&#160;x)&#160;tolerance))<br></tt>


<a name="%_thm_3.65"></a> **Exercise 3.65.**&#160;&#160;<a name="index_term_3982"></a>Use the series

<div align="left">
  <img src="img/chapter_3_image_44.gif" border="0">
</div>

to compute three sequences of approximations to the natural logarithm of 2, in the same way we did above for <img src="img/book_09.gif" border="0">. How rapidly do these sequences converge?


<a name="%_sec_Temp_476"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_476">Infinite streams of pairs</a></h4>

<a name="index_term_3984"></a><a name="index_term_3986"></a><a name="index_term_3988"></a> In section&#160;<a href="chapter_2_section_2.html#%_sec_2.2.3">2.2.3</a>, we saw how the sequence paradigm handles traditional nested loops as processes defined on sequences of pairs. If we generalize this technique to infinite streams, then we can write programs that are not easily represented as loops, because the "looping" must range over an infinite set.


<a name="index_term_3990"></a>For example, suppose we want to generalize the <tt>prime-sum-pairs</tt> procedure of section&#160;<a href="chapter_2_section_2.html#%_sec_2.2.3">2.2.3</a> to produce the stream of pairs of *all* integers (*i*,*j*) with *i* <u>&lt;</u> *j* such that *i* + *j* is prime. If <tt>int-pairs</tt> is the sequence of all pairs of integers (*i*,*j*) with *i* <u>&lt;</u> *j*, then our required stream is simply<a name="call_footnote_Temp_477" href="#footnote_Temp_477" id="call_footnote_Temp_477"><sup><small>66</small></sup></a>


<tt>(stream-filter&#160;(lambda&#160;(pair)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(prime?&#160;(+&#160;(car&#160;pair)&#160;(cadr&#160;pair))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;int-pairs)<br></tt>


Our problem, then, is to produce the stream <tt>int-pairs</tt>. More generally, suppose we have two streams *S* = (*S*<sub>*i*</sub>) and *T* = (*T*<sub>*j*</sub>), and imagine the infinite rectangular array

<div align="left">
  <img src="img/chapter_3_image_45.gif" border="0">
</div>

We wish to generate a stream that contains all the pairs in the array that lie on or above the diagonal, i.e., the pairs

<div align="left">
  <img src="img/chapter_3_image_46.gif" border="0">
</div>

(If we take both *S* and *T* to be the stream of integers, then this will be our desired stream <tt>int-pairs</tt>.)


Call the general stream of pairs <tt>(pairs S T)</tt>, and consider it to be composed of three parts: the pair (*S*<sub>0</sub>,*T*<sub>0</sub>), the rest of the pairs in the first row, and the remaining pairs:<a name="call_footnote_Temp_478" href="#footnote_Temp_478" id="call_footnote_Temp_478"><sup><small>67</small></sup></a>

<div align="left">
  <img src="img/chapter_3_image_47.gif" border="0">
</div>

Observe that the third piece in this decomposition (pairs that are not in the first row) is (recursively) the pairs formed from <tt>(stream-cdr S)</tt> and <tt>(stream-cdr T)</tt>. Also note that the second piece (the rest of the first row) is


<tt>(stream-map&#160;(lambda&#160;(x)&#160;(list&#160;(stream-car&#160;s)&#160;x))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-cdr&#160;t))<br></tt>


Thus we can form our stream of pairs as follows:


<tt>(define&#160;(pairs&#160;s&#160;t)<br>
&#160;&#160;(cons-stream<br>
&#160;&#160;&#160;(list&#160;(stream-car&#160;s)&#160;(stream-car&#160;t))<br>
&#160;&#160;&#160;(&lt;*combine-in-some-way*&gt;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-map&#160;(lambda&#160;(x)&#160;(list&#160;(stream-car&#160;s)&#160;x))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-cdr&#160;t))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(pairs&#160;(stream-cdr&#160;s)&#160;(stream-cdr&#160;t)))))<br></tt>


<a name="index_term_3992"></a>In order to complete the procedure, we must choose some way to combine the two inner streams. One idea is to use the stream analog of the <tt>append</tt> procedure from section&#160;<a href="chapter_2_section_2.html#%_sec_2.2.1">2.2.1</a>:


<tt><a name="index_term_3994"></a>(define&#160;(stream-append&#160;s1&#160;s2)<br>
&#160;&#160;(if&#160;(stream-null?&#160;s1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;s2<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream&#160;(stream-car&#160;s1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-append&#160;(stream-cdr&#160;s1)&#160;s2))))<br></tt>


This is unsuitable for infinite streams, however, because it takes all the elements from the first stream before incorporating the second stream. In particular, if we try to generate all pairs of positive integers using


<tt>(pairs&#160;integers&#160;integers)<br></tt>


our stream of results will first try to run through all pairs with the first integer equal to 1, and hence will never produce pairs with any other value of the first integer.


To handle infinite streams, we need to devise an order of combination that ensures that every element will eventually be reached if we let our program run long enough. An elegant way to accomplish this is with the following <tt>interleave</tt> procedure:<a name="call_footnote_Temp_479" href="#footnote_Temp_479" id="call_footnote_Temp_479"><sup><small>68</small></sup></a>


<tt><a name="index_term_4000"></a>(define&#160;(interleave&#160;s1&#160;s2)<br>
&#160;&#160;(if&#160;(stream-null?&#160;s1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;s2<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream&#160;(stream-car&#160;s1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(interleave&#160;s2&#160;(stream-cdr&#160;s1)))))<br></tt>


Since <tt>interleave</tt> takes elements alternately from the two streams, every element of the second stream will eventually find its way into the interleaved stream, even if the first stream is infinite.


We can thus generate the required stream of pairs as


<tt><a name="index_term_4002"></a>(define&#160;(pairs&#160;s&#160;t)<br>
&#160;&#160;(cons-stream<br>
&#160;&#160;&#160;(list&#160;(stream-car&#160;s)&#160;(stream-car&#160;t))<br>
&#160;&#160;&#160;(interleave<br>
&#160;&#160;&#160;&#160;(stream-map&#160;(lambda&#160;(x)&#160;(list&#160;(stream-car&#160;s)&#160;x))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-cdr&#160;t))<br>
&#160;&#160;&#160;&#160;(pairs&#160;(stream-cdr&#160;s)&#160;(stream-cdr&#160;t)))))<br></tt>


<a name="%_thm_3.66"></a> **Exercise 3.66.**&#160;&#160;Examine the stream <tt>(pairs integers integers)</tt>. Can you make any general comments about the order in which the pairs are placed into the stream? For example, about how many pairs precede the pair (1,100)? the pair (99,100)? the pair (100,100)? (If you can make precise mathematical statements here, all the better. But feel free to give more qualitative answers if you find yourself getting bogged down.)


<a name="%_thm_3.67"></a> **Exercise 3.67.**&#160;&#160;Modify the <tt>pairs</tt> procedure so that <tt>(pairs integers integers)</tt> will produce the stream of *all* pairs of integers (*i*,*j*) (without the condition *i* <u>&lt;</u> *j*). Hint: You will need to mix in an additional stream.


<a name="%_thm_3.68"></a> **Exercise 3.68.**&#160;&#160;Louis Reasoner thinks that building a stream of pairs from three parts is unnecessarily complicated. Instead of separating the pair (*S*<sub>0</sub>,*T*<sub>0</sub>) from the rest of the pairs in the first row, he proposes to work with the whole first row, as follows:


<tt>(define&#160;(pairs&#160;s&#160;t)<br>
&#160;&#160;(interleave<br>
&#160;&#160;&#160;(stream-map&#160;(lambda&#160;(x)&#160;(list&#160;(stream-car&#160;s)&#160;x))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;t)<br>
&#160;&#160;&#160;(pairs&#160;(stream-cdr&#160;s)&#160;(stream-cdr&#160;t))))<br></tt>


Does this work? Consider what happens if we evaluate <tt>(pairs integers integers)</tt> using Louis's definition of <tt>pairs</tt>.


<a name="%_thm_3.69"></a> **Exercise 3.69.**&#160;&#160;Write a procedure <tt>triples</tt> that takes three infinite streams, *S*, *T*, and *U*, and produces the stream of triples (*S*<sub>*i*</sub>,*T*<sub>*j*</sub>,*U*<sub>*k*</sub>) such that *i* <u>&lt;</u> *j* <u>&lt;</u> *k*. Use <tt>triples</tt> to generate the stream of all <a name="index_term_4004"></a>Pythagorean triples of positive integers, i.e., the triples (*i*,*j*,*k*) such that *i* <u>&lt;</u> *j* and *i*<sup>2</sup> + *j*<sup>2</sup> = *k*<sup>2</sup>.


<a name="%_thm_3.70"></a> **Exercise 3.70.**&#160;&#160;<a name="index_term_4006"></a><a name="index_term_4008"></a>It would be nice to be able to generate streams in which the pairs appear in some useful order, rather than in the order that results from an *ad hoc* interleaving process. We can use a technique similar to the <tt>merge</tt> procedure of exercise&#160;<a href="#%_thm_3.56">3.56</a>, if we define a way to say that one pair of integers is "less than" another. One way to do this is to define a "weighting function" *W*(*i*,*j*) and stipulate that (*i*<sub>1</sub>,*j*<sub>1</sub>) is less than (*i*<sub>2</sub>,*j*<sub>2</sub>) if *W*(*i*<sub>1</sub>,*j*<sub>1</sub>) &lt; *W*(*i*<sub>2</sub>,*j*<sub>2</sub>). Write a procedure <tt>merge-weighted</tt> that is like <tt>merge</tt>, except that <tt>merge-weighted</tt> takes an additional argument <tt>weight</tt>, which is a procedure that computes the weight of a pair, and is used to determine the order in which elements should appear in the resulting merged stream.<a name="call_footnote_Temp_485" href="#footnote_Temp_485" id="call_footnote_Temp_485"><sup><small>69</small></sup></a> Using this, generalize <tt>pairs</tt> to a procedure <tt>weighted-pairs</tt> that takes two streams, together with a procedure that computes a weighting function, and generates the stream of pairs, ordered according to weight. Use your procedure to generate


a. the stream of all pairs of positive integers (*i*,*j*) with *i* <u>&lt;</u> *j* ordered according to the sum *i* + *j*


b. the stream of all pairs of positive integers (*i*,*j*) with *i* <u>&lt;</u> *j*, where neither *i* nor *j* is divisible by 2, 3, or 5, and the pairs are ordered according to the sum 2 *i* + 3 *j* + 5 *i* *j*.


<a name="%_thm_3.71"></a> **Exercise 3.71.**&#160;&#160;<a name="index_term_4010"></a>Numbers that can be expressed as the sum of two cubes in more than one way are sometimes called *Ramanujan numbers*, in honor of the mathematician Srinivasa Ramanujan.<a name="call_footnote_Temp_487" href="#footnote_Temp_487" id="call_footnote_Temp_487"><sup><small>70</small></sup></a> Ordered streams of pairs provide an elegant solution to the problem of computing these numbers. To find a number that can be written as the sum of two cubes in two different ways, we need only generate the stream of pairs of integers (*i*,*j*) weighted according to the sum *i*<sup>3</sup> + *j*<sup>3</sup> (see exercise&#160;<a href="#%_thm_3.70">3.70</a>), then search the stream for two consecutive pairs with the same weight. Write a procedure to generate the Ramanujan numbers. The first such number is 1,729. What are the next five?


<a name="%_thm_3.72"></a> **Exercise 3.72.**&#160;&#160;In a similar way to exercise&#160;<a href="#%_thm_3.71">3.71</a> generate a stream of all numbers that can be written as the sum of two squares in three different ways (showing how they can be so written).


<a name="%_sec_Temp_489"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_489">Streams as signals</a></h4>

<a name="index_term_4018"></a><a name="index_term_4020"></a> We began our discussion of streams by describing them as computational analogs of the "signals" in signal-processing systems. In fact, we can use streams to model signal-processing systems in a very direct way, representing the values of a signal at successive time intervals as consecutive elements of a stream. For instance, we can implement an <a name="index_term_4022"></a>*integrator* or *summer* that, for an input stream *x* = (*x*<sub>*i*</sub>), an initial value *C*, and a small increment *d**t*, accumulates the sum

<div align="left">
  <img src="img/chapter_3_image_48.gif" border="0">
</div>

and returns the stream of values *S* = (*S*<sub>*i*</sub>). The following <tt>integral</tt> procedure is reminiscent of the "implicit style" definition of the stream of integers (section&#160;<a href="#%_sec_3.5.2">3.5.2</a>):


<tt><a name="index_term_4024"></a>(define&#160;(integral&#160;integrand&#160;initial-value&#160;dt)<br>
&#160;&#160;(define&#160;int<br>
&#160;&#160;&#160;&#160;(cons-stream&#160;initial-value<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(add-streams&#160;(scale-stream&#160;integrand&#160;dt)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;int)))<br>
&#160;&#160;int)<br></tt>


<a name="%_fig_3.32"></a>

<div align="left">
  <div align="left">
    **Figure 3.32:**&#160;&#160;The <tt>integral</tt> procedure viewed as a signal-processing system.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_3_image_49.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

Figure&#160;<a href="#%_fig_3.32">3.32</a> is a picture of a signal-processing system that corresponds to the <tt>integral</tt> procedure. The input stream is scaled by *d**t* and passed through an adder, whose output is passed back through the same adder. The self-reference in the definition of <tt>int</tt> is reflected in the figure by the feedback loop that connects the output of the adder to one of the inputs.


<a name="%_thm_3.73"></a> **Exercise 3.73.**&#160;&#160;<a name="%_fig_3.33"></a>

<div align="left">
  <div align="left">
    **Figure 3.33:**&#160;&#160;An RC circuit and the associated signal-flow diagram.
  </div>

  <table width="100%">
    <tr>
      <td>
        <div align="center">
          &#160;<img src="img/chapter_3_image_50.gif" border="0"> &#160;&#160;&#160;&#160;&#160;&#160; *v* = *v*<sub>0</sub> + (1/*C*)<img src="img/book_19.gif" border="0"><sub>0</sub><sup>*t*</sup>*i* *d**t* + *R* *i* &#160;&#160;&#160;&#160;&#160;
        </div>

        
<img src="img/chapter_3_image_51.gif" border="0">

      </td>
    </tr>

    <tr>
      <td><a name="index_term_4026"></a></td>
    </tr>
  </table>
</div>

<a name="index_term_4028"></a><a name="index_term_4030"></a><a name="index_term_4032"></a>We can model electrical circuits using streams to represent the values of currents or voltages at a sequence of times. For instance, suppose we have an *RC circuit* consisting of a resistor of resistance *R* and a capacitor of capacitance *C* in series. The voltage response *v* of the circuit to an injected current *i* is determined by the formula in figure&#160;<a href="#%_fig_3.33">3.33</a>, whose structure is shown by the accompanying signal-flow diagram.


Write a procedure <tt>RC</tt> that models this circuit. <tt>RC</tt> should take as inputs the values of *R*, *C*, and *d**t* and should return a procedure that takes as inputs a stream representing the current *i* and an initial value for the capacitor voltage *v*<sub>0</sub> and produces as output the stream of voltages *v*. For example, you should be able to use <tt>RC</tt> to model an RC circuit with *R* = 5 ohms, *C* = 1 farad, and a 0.5-second time step by evaluating <tt>(define RC1 (RC 5 1 0.5))</tt>. This defines <tt>RC1</tt> as a procedure that takes a stream representing the time sequence of currents and an initial capacitor voltage and produces the output stream of voltages.


<a name="%_thm_3.74"></a> **Exercise 3.74.**&#160;&#160;<a name="index_term_4034"></a><a name="index_term_4036"></a>Alyssa P. Hacker is designing a system to process signals coming from physical sensors. One important feature she wishes to produce is a signal that describes the *zero crossings* of the input signal. That is, the resulting signal should be + 1 whenever the input signal changes from negative to positive, - 1 whenever the input signal changes from positive to negative, and 0 otherwise. (Assume that the sign of a 0 input is positive.) For example, a typical input signal with its associated zero-crossing signal would be


<tt><tt>...</tt>1&#160;&#160;2&#160;&#160;1.5&#160;&#160;1&#160;&#160;0.5&#160;&#160;-0.1&#160;&#160;-2&#160;&#160;-3&#160;&#160;-2&#160;&#160;-0.5&#160;&#160;0.2&#160;&#160;3&#160;&#160;4&#160;<tt>...</tt><tt>...</tt>&#160;0&#160;&#160;0&#160;&#160;&#160;&#160;0&#160;&#160;0&#160;&#160;&#160;&#160;0&#160;&#160;&#160;&#160;&#160;-1&#160;&#160;0&#160;&#160;&#160;0&#160;&#160;&#160;0&#160;&#160;&#160;&#160;&#160;0&#160;&#160;&#160;&#160;1&#160;&#160;0&#160;&#160;0&#160;<tt>...</tt></tt>


In Alyssa's system, the signal from the sensor is represented as a stream <tt>sense-data</tt> and the stream <tt>zero-crossings</tt> is the corresponding stream of zero crossings. Alyssa first writes a procedure <tt>sign-change-detector</tt> that takes two values as arguments and compares the signs of the values to produce an appropriate 0, 1, or - 1. She then constructs her zero-crossing stream as follows:


<tt>(define&#160;(make-zero-crossings&#160;input-stream&#160;last-value)<br>
&#160;&#160;(cons-stream<br>
&#160;&#160;&#160;(sign-change-detector&#160;(stream-car&#160;input-stream)&#160;last-value)<br>
&#160;&#160;&#160;(make-zero-crossings&#160;(stream-cdr&#160;input-stream)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-car&#160;input-stream))))<br>
<br>
(define&#160;zero-crossings&#160;(make-zero-crossings&#160;sense-data&#160;0))<br></tt>


Alyssa's boss, Eva Lu Ator, walks by and suggests that this program is approximately equivalent to the following one, which uses the generalized version of <tt>stream-map</tt> from exercise&#160;<a href="#%_thm_3.50">3.50</a>:


<tt>(define&#160;zero-crossings<br>
&#160;&#160;(stream-map&#160;sign-change-detector&#160;sense-data&#160;&lt;*expression*&gt;))<br></tt>


Complete the program by supplying the indicated &lt;*expression*&gt;.


<a name="%_thm_3.75"></a> **Exercise 3.75.**&#160;&#160;<a name="index_term_4038"></a><a name="index_term_4040"></a><a name="index_term_4042"></a><a name="index_term_4044"></a>Unfortunately, Alyssa's zero-crossing detector in exercise&#160;<a href="#%_thm_3.74">3.74</a> proves to be insufficient, because the noisy signal from the sensor leads to spurious zero crossings. Lem E. Tweakit, a hardware specialist, suggests that Alyssa smooth the signal to filter out the noise before extracting the zero crossings. Alyssa takes his advice and decides to extract the zero crossings from the signal constructed by averaging each value of the sense data with the previous value. She explains the problem to her assistant, Louis Reasoner, who attempts to implement the idea, altering Alyssa's program as follows:


<tt>(define&#160;(make-zero-crossings&#160;input-stream&#160;last-value)<br>
&#160;&#160;(let&#160;((avpt&#160;(/&#160;(+&#160;(stream-car&#160;input-stream)&#160;last-value)&#160;2)))<br>
&#160;&#160;&#160;&#160;(cons-stream&#160;(sign-change-detector&#160;avpt&#160;last-value)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(make-zero-crossings&#160;(stream-cdr&#160;input-stream)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;avpt))))<br></tt>


This does not correctly implement Alyssa's plan. Find the bug that Louis has installed and fix it without changing the structure of the program. (Hint: You will need to increase the number of arguments to <tt>make-zero-crossings</tt>.)


<a name="%_thm_3.76"></a> **Exercise 3.76.**&#160;&#160;<a name="index_term_4046"></a><a name="index_term_4048"></a><a name="index_term_4050"></a><a name="index_term_4052"></a>Eva Lu Ator has a criticism of Louis's approach in exercise&#160;<a href="#%_thm_3.75">3.75</a>. The program he wrote is not modular, because it intermixes the operation of smoothing with the zero-crossing extraction. For example, the extractor should not have to be changed if Alyssa finds a better way to condition her input signal. Help Louis by writing a procedure <tt>smooth</tt> that takes a stream as input and produces a stream in which each element is the average of two successive input stream elements. Then use <tt>smooth</tt> as a component to implement the zero-crossing detector in a more modular style.


<a name="%_sec_3.5.4"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_3.5.4">3.5.4&#160;&#160;Streams and Delayed Evaluation</a></h3>

<a name="index_term_4054"></a><a name="index_term_4056"></a> The <tt>integral</tt> procedure at the end of the preceding section shows how we can use streams to model signal-processing systems that contain <a name="index_term_4058"></a>feedback loops. The feedback loop for the adder shown in figure&#160;<a href="#%_fig_3.32">3.32</a> is modeled by the fact that <tt>integral</tt>'s <a name="index_term_4060"></a>internal stream <tt>int</tt> is defined in terms of itself:


<tt>(define&#160;int<br>
&#160;&#160;(cons-stream&#160;initial-value<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(add-streams&#160;(scale-stream&#160;integrand&#160;dt)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;int)))<br></tt>


The interpreter's ability to deal with such an implicit definition depends on the <tt>delay</tt> that is incorporated into <tt>cons-stream</tt>. Without this <tt>delay</tt>, the interpreter could not construct <tt>int</tt> before evaluating both arguments to <tt>cons-stream</tt>, which would require that <tt>int</tt> already be defined. In general, <tt>delay</tt> is crucial for using streams to model signal-processing systems that contain loops. Without <tt>delay</tt>, our models would have to be formulated so that the inputs to any signal-processing component would be fully evaluated before the output could be produced. This would outlaw loops.


Unfortunately, stream models of systems with loops may require uses of <tt>delay</tt> beyond the "hidden" <tt>delay</tt> supplied by <tt>cons-stream</tt>. For instance, figure&#160;<a href="#%_fig_3.34">3.34</a> shows a signal-processing system for solving the <a name="index_term_4062"></a>differential equation *d**y*/*d**t* = *f*(*y*) where *f* is a given function. The figure shows a mapping component, which applies *f* to its input signal, linked in a feedback loop to an integrator in a manner very similar to that of the analog computer circuits that are actually used to solve such equations.


<a name="%_fig_3.34"></a>

<div align="left">
  <div align="left">
    **Figure 3.34:**&#160;&#160;An "analog computer circuit" that solves the equation *d**y*/*d**t* = *f*(*y*).
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_3_image_52.gif" border="0"></td>
    </tr>

    <tr>
      <td><a name="index_term_4064"></a></td>
    </tr>
  </table>
</div>

Assuming we are given an initial value *y*<sub>0</sub> for *y*, we could try to model this system using the procedure


<tt><a name="index_term_4066"></a>(define&#160;(solve&#160;f&#160;y0&#160;dt)<br>
&#160;&#160;(define&#160;y&#160;(integral&#160;dy&#160;y0&#160;dt))<br>
&#160;&#160;(define&#160;dy&#160;(stream-map&#160;f&#160;y))<br>
&#160;&#160;y)<br></tt>


This procedure does not work, because in the first line of <tt>solve</tt> the call to <tt>integral</tt> requires that the input <tt>dy</tt> be defined, which does not happen until the second line of <tt>solve</tt>.


On the other hand, the intent of our definition does make sense, because we can, in principle, begin to generate the <tt>y</tt> stream without knowing <tt>dy</tt>. Indeed, <tt>integral</tt> and many other stream operations have properties similar to those of <tt>cons-stream</tt>, in that we can generate part of the answer given only partial information about the arguments. For <tt>integral</tt>, the first element of the output stream is the specified <tt>initial-value</tt>. Thus, we can generate the first element of the output stream without evaluating the integrand <tt>dy</tt>. Once we know the first element of <tt>y</tt>, the <tt>stream-map</tt> in the second line of <tt>solve</tt> can begin working to generate the first element of <tt>dy</tt>, which will produce the next element of <tt>y</tt>, and so on.


To take advantage of this idea, we will redefine <tt>integral</tt> to expect the integrand stream to be a <a name="index_term_4068"></a><a name="index_term_4070"></a><a name="index_term_4072"></a>*delayed argument*. <tt>Integral</tt> will <tt>force</tt> the integrand to be evaluated only when it is required to generate more than the first element of the output stream:


<tt><a name="index_term_4074"></a>(define&#160;(integral&#160;delayed-integrand&#160;initial-value&#160;dt)<br>
&#160;&#160;(define&#160;int<br>
&#160;&#160;&#160;&#160;(cons-stream&#160;initial-value<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(let&#160;((integrand&#160;(force&#160;delayed-integrand)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(add-streams&#160;(scale-stream&#160;integrand&#160;dt)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;int))))<br>
&#160;&#160;int)<br></tt>


Now we can implement our <tt>solve</tt> procedure by delaying the evaluation of <tt>dy</tt> in the definition of <tt>y</tt>:<a name="call_footnote_Temp_494" href="#footnote_Temp_494" id="call_footnote_Temp_494"><sup><small>71</small></sup></a>


<tt><a name="index_term_4076"></a>(define&#160;(solve&#160;f&#160;y0&#160;dt)<br>
&#160;&#160;(define&#160;y&#160;(integral&#160;(delay&#160;dy)&#160;y0&#160;dt))<br>
&#160;&#160;(define&#160;dy&#160;(stream-map&#160;f&#160;y))<br>
&#160;&#160;y)<br></tt>


In general, every caller of <tt>integral</tt> must now <tt>delay</tt> the integrand argument. We can demonstrate that the <tt>solve</tt> procedure works by approximating <a name="index_term_4078"></a>*e* <img src="img/book_20.gif" border="0"> 2.718 by computing the value at *y* = 1 of the solution to the differential equation *d**y*/*d**t* = *y* with initial condition *y*(0) = 1:


<tt>(stream-ref&#160;(solve&#160;(lambda&#160;(y)&#160;y)&#160;1&#160;0.001)&#160;1000)<br>
<i>2.716924</i></tt>


<a name="%_thm_3.77"></a> **Exercise 3.77.**&#160;&#160;The <tt>integral</tt> procedure used above was analogous to the "implicit" definition of the infinite stream of integers in section&#160;<a href="#%_sec_3.5.2">3.5.2</a>. Alternatively, we can give a definition of <tt>integral</tt> that is more like <tt>integers-starting-from</tt> (also in section&#160;<a href="#%_sec_3.5.2">3.5.2</a>):


<tt><a name="index_term_4080"></a>(define&#160;(integral&#160;integrand&#160;initial-value&#160;dt)<br>
&#160;&#160;(cons-stream&#160;initial-value<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;(stream-null?&#160;integrand)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;the-empty-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(integral&#160;(stream-cdr&#160;integrand)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(+&#160;(*&#160;dt&#160;(stream-car&#160;integrand))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;initial-value)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;dt))))<br></tt>


When used in systems with loops, this procedure has the same problem as does our original version of <tt>integral</tt>. Modify the procedure so that it expects the <tt>integrand</tt> as a delayed argument and hence can be used in the <tt>solve</tt> procedure shown above.


<a name="%_thm_3.78"></a> **Exercise 3.78.**&#160;&#160;<a name="%_fig_3.35"></a>

<div align="left">
  <div align="left">
    **Figure 3.35:**&#160;&#160;Signal-flow diagram for the solution to a second-order linear differential equation.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_3_image_53.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="index_term_4082"></a>Consider the problem of designing a signal-processing system to study the homogeneous second-order linear differential equation

<div align="left">
  <img src="img/chapter_3_image_54.gif" border="0">
</div>

The output stream, modeling *y*, is generated by a network that contains a loop. This is because the value of *d*<sup>2</sup>*y*/*d**t*<sup>2</sup> depends upon the values of *y* and *d**y*/*d**t* and both of these are determined by integrating *d*<sup>2</sup>*y*/*d**t*<sup>2</sup>. The diagram we would like to encode is shown in figure&#160;<a href="#%_fig_3.35">3.35</a>. Write a procedure <tt>solve-2nd</tt> that takes as arguments the constants *a*, *b*, and *d**t* and the initial values *y*<sub>0</sub> and *d**y*<sub>0</sub> for *y* and *d**y*/*d**t* and generates the stream of successive values of *y*.


<a name="%_thm_3.79"></a> **Exercise 3.79.**&#160;&#160;<a name="index_term_4084"></a>Generalize the <tt>solve-2nd</tt> procedure of exercise&#160;<a href="#%_thm_3.78">3.78</a> so that it can be used to solve general second-order differential equations *d*<sup>2</sup> *y*/*d**t*<sup>2</sup> = *f*(*d**y*/*d**t*, *y*).


<a name="%_thm_3.80"></a> **Exercise 3.80.**&#160;&#160;<a name="index_term_4086"></a><a name="index_term_4088"></a><a name="index_term_4090"></a>A *series RLC circuit* consists of a resistor, a capacitor, and an inductor connected in series, as shown in figure&#160;<a href="#%_fig_3.36">3.36</a>. If *R*, *L*, and *C* are the resistance, inductance, and capacitance, then the relations between voltage (*v*) and current (*i*) for the three components are described by the equations

<div align="left">
  <img src="img/chapter_3_image_55.gif" border="0">
</div>

and the circuit connections dictate the relations

<div align="left">
  <img src="img/chapter_3_image_56.gif" border="0">
</div>

Combining these equations shows that the state of the circuit (summarized by *v*<sub>*C*</sub>, the voltage across the capacitor, and *i*<sub>*L*</sub>, the current in the inductor) is described by the pair of differential equations

<div align="left">
  <img src="img/chapter_3_image_57.gif" border="0">
</div>

The signal-flow diagram representing this system of differential equations is shown in figure&#160;<a href="#%_fig_3.37">3.37</a>. <a name="%_fig_3.36"></a>

<div align="left">
  <div align="left">
    **Figure 3.36:**&#160;&#160;A series RLC circuit.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_3_image_58.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="%_fig_3.37"></a>

<div align="left">
  <div align="left">
    **Figure 3.37:**&#160;&#160;A signal-flow diagram for the solution to a series RLC circuit.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_3_image_59.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

Write a procedure <tt>RLC</tt> that takes as arguments the parameters *R*, *L*, and *C* of the circuit and the time increment *d**t*. In a manner similar to that of the <tt>RC</tt> procedure of exercise&#160;<a href="#%_thm_3.73">3.73</a>, <tt>RLC</tt> should produce a procedure that takes the initial values of the state variables, *v*<sub>*C*<sub>0</sub></sub> and *i*<sub>*L*<sub>0</sub></sub>, and produces a pair (using <tt>cons</tt>) of the streams of states *v*<sub>*C*</sub> and *i*<sub>*L*</sub>. Using <tt>RLC</tt>, generate the pair of streams that models the behavior of a series RLC circuit with *R* = 1 ohm, *C* = 0.2 farad, *L* = 1 henry, *d**t* = 0.1 second, and initial values *i*<sub>*L*<sub>0</sub></sub> = 0 amps and *v*<sub>*C*<sub>0</sub></sub> = 10 volts.


<a name="%_sec_Temp_499"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_499">Normal-order evaluation</a></h4>

<a name="index_term_4092"></a><a name="index_term_4094"></a> The examples in this section illustrate how the explicit use of <tt>delay</tt> and <tt>force</tt> provides great programming flexibility, but the same examples also show how this can make our programs more complex. Our new <tt>integral</tt> procedure, for instance, gives us the power to model systems with loops, but we must now remember that <tt>integral</tt> should be called with a delayed integrand, and every procedure that uses <tt>integral</tt> must be aware of this. In effect, we have created two classes of procedures: ordinary procedures and procedures that take delayed arguments. In general, creating separate classes of procedures forces us to create separate classes of higher-order procedures as well.<a name="call_footnote_Temp_500" href="#footnote_Temp_500" id="call_footnote_Temp_500"><sup><small>72</small></sup></a>


One way to avoid the need for two different classes of procedures is to make all procedures take delayed arguments. We could adopt a model of evaluation in which all arguments to procedures are automatically delayed and arguments are forced only when they are actually needed (for example, when they are required by a primitive operation). This would transform our language to use normal-order evaluation, which we first described when we introduced the substitution model for evaluation in section&#160;<a href="chapter_1_section_1.html#%_sec_1.1.5">1.1.5</a>. Converting to normal-order evaluation provides a uniform and elegant way to simplify the use of delayed evaluation, and this would be a natural strategy to adopt if we were concerned only with stream processing. In section&#160;<a href="chapter_4_section_2.html#%_sec_4.2">4.2</a>, after we have studied the evaluator, we will see how to transform our language in just this way. Unfortunately, including delays in procedure calls wreaks havoc with our ability to design programs that depend on the order of events, such as programs that use assignment, mutate data, or perform input or output. Even the single <tt>delay</tt> in <tt>cons-stream</tt> can cause great confusion, as illustrated by exercises&#160;<a href="#%_thm_3.51">3.51</a> and&#160;<a href="#%_thm_3.52">3.52</a>. As far as anyone knows, mutability and delayed evaluation do not mix well in programming languages, and devising ways to deal with both of these at once is an active area of research. <a name="%_sec_3.5.5"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_3.5.5">3.5.5&#160;&#160;Modularity of Functional Programs and Modularity of Objects</a></h3>

<a name="index_term_4116"></a><a name="index_term_4118"></a> As we saw in section&#160;<a href="chapter_3_section_1.html#%_sec_3.1.2">3.1.2</a>, one of the major benefits of introducing assignment is that we can increase the modularity of our systems by encapsulating, or "hiding," parts of the state of a large system within local variables. Stream models can provide an equivalent modularity without the use of assignment. As an <a name="index_term_4120"></a><a name="index_term_4122"></a>illustration, we can reimplement the Monte Carlo estimation of <img src="img/book_09.gif" border="0">, which we examined in section&#160;<a href="chapter_3_section_1.html#%_sec_3.1.2">3.1.2</a>, from a stream-processing point of view.


The key modularity issue was that we wished to hide the internal state of a random-number generator from programs that used random numbers. We began with a procedure <tt>rand-update</tt>, whose successive values furnished our supply of random numbers, and used this to produce a random-number generator:


<tt>(define&#160;rand<br>
&#160;&#160;(let&#160;((x&#160;random-init))<br>
&#160;&#160;&#160;&#160;(lambda&#160;()<br>
&#160;&#160;&#160;&#160;&#160;&#160;(set!&#160;x&#160;(rand-update&#160;x))<br>
&#160;&#160;&#160;&#160;&#160;&#160;x)))<br></tt>


In the stream formulation there is no random-number generator *per se*, just a stream of random numbers produced by successive calls to <tt>rand-update</tt>:


<tt><a name="index_term_4124"></a><a name="index_term_4126"></a>(define&#160;random-numbers<br>
&#160;&#160;(cons-stream&#160;random-init<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-map&#160;rand-update&#160;random-numbers)))<br></tt>


We use this to construct the stream of outcomes of the Ces�ro experiment performed on consecutive pairs in the <tt>random-numbers</tt> stream:


<tt><a name="index_term_4128"></a>(define&#160;cesaro-stream<br>
&#160;&#160;(map-successive-pairs&#160;(lambda&#160;(r1&#160;r2)&#160;(=&#160;(gcd&#160;r1&#160;r2)&#160;1))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;random-numbers))<br>
<br>
<a name="index_term_4130"></a>(define&#160;(map-successive-pairs&#160;f&#160;s)<br>
&#160;&#160;(cons-stream<br>
&#160;&#160;&#160;(f&#160;(stream-car&#160;s)&#160;(stream-car&#160;(stream-cdr&#160;s)))<br>
&#160;&#160;&#160;(map-successive-pairs&#160;f&#160;(stream-cdr&#160;(stream-cdr&#160;s)))))<br></tt>


The <tt>cesaro-stream</tt> is now fed to a <tt>monte-carlo</tt> procedure, which produces a stream of estimates of probabilities. The results are then converted into a stream of estimates of <img src="img/book_09.gif" border="0">. This version of the program doesn't need a parameter telling how many trials to perform. Better estimates of <img src="img/book_09.gif" border="0"> (from performing more experiments) are obtained by looking farther into the <tt>pi</tt> stream:


<tt><a name="index_term_4132"></a>(define&#160;(monte-carlo&#160;experiment-stream&#160;passed&#160;failed)<br>
&#160;&#160;(define&#160;(next&#160;passed&#160;failed)<br>
&#160;&#160;&#160;&#160;(cons-stream<br>
&#160;&#160;&#160;&#160;&#160;(/&#160;passed&#160;(+&#160;passed&#160;failed))<br>
&#160;&#160;&#160;&#160;&#160;(monte-carlo<br>
&#160;&#160;&#160;&#160;&#160;&#160;(stream-cdr&#160;experiment-stream)&#160;passed&#160;failed)))<br>
&#160;&#160;(if&#160;(stream-car&#160;experiment-stream)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(next&#160;(+&#160;passed&#160;1)&#160;failed)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(next&#160;passed&#160;(+&#160;failed&#160;1))))<br>
<br>
(define&#160;pi<br>
&#160;&#160;(stream-map&#160;(lambda&#160;(p)&#160;(sqrt&#160;(/&#160;6&#160;p)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(monte-carlo&#160;cesaro-stream&#160;0&#160;0)))<br></tt>


<a name="index_term_4134"></a>There is considerable modularity in this approach, because we still can formulate a general <tt>monte-carlo</tt> procedure that can deal with arbitrary experiments. Yet there is no assignment or local state.


<a name="%_thm_3.81"></a> **Exercise 3.81.**&#160;&#160;<a name="index_term_4136"></a>Exercise&#160;<a href="chapter_3_section_1.html#exercise_3_6">3.6</a> discussed generalizing the random-number generator to allow one to reset the random-number sequence so as to produce repeatable sequences of "random" numbers. Produce a stream formulation of this same generator that operates on an input stream of requests to <tt>generate</tt> a new random number or to <tt>reset</tt> the sequence to a specified value and that produces the desired stream of random numbers. Don't use assignment in your solution.


<a name="%_thm_3.82"></a> **Exercise 3.82.**&#160;&#160;<a name="index_term_4138"></a><a name="index_term_4140"></a><a name="index_term_4142"></a>Redo exercise&#160;<a href="chapter_3_section_1.html#exercise_3_5">3.5</a> on Monte Carlo integration in terms of streams. The stream version of <tt>estimate-integral</tt> will not have an argument telling how many trials to perform. Instead, it will produce a stream of estimates based on successively more trials.


<a name="%_sec_Temp_503"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_503">A functional-programming view of time</a></h4>

<a name="index_term_4144"></a><a name="index_term_4146"></a> Let us now return to the issues of objects and state that were raised at the beginning of this chapter and examine them in a new light. We introduced assignment and mutable objects to provide a mechanism for modular construction of programs that model systems with state. We constructed computational objects with local state variables and used assignment to modify these variables. We modeled the temporal behavior of the objects in the world by the temporal behavior of the corresponding computational objects.


Now we have seen that streams provide an alternative way to model objects with local state. We can model a changing quantity, such as the local state of some object, using a stream that represents the time history of successive states. In essence, we represent time explicitly, using streams, so that we decouple time in our simulated world from the sequence of events that take place during evaluation. Indeed, because of the presence of <tt>delay</tt> there may be little relation between simulated time in the model and the order of events during the evaluation.


In order to contrast these two approaches to modeling, let us reconsider the implementation of a "withdrawal processor" that <a name="index_term_4148"></a>monitors the balance in a bank account. In section&#160;<a href="chapter_3_section_1.html#%_sec_3.1.3">3.1.3</a> we implemented a simplified version of such a processor:


<tt><a name="index_term_4150"></a>(define&#160;(make-simplified-withdraw&#160;balance)<br>
&#160;&#160;(lambda&#160;(amount)<br>
&#160;&#160;&#160;&#160;(set!&#160;balance&#160;(-&#160;balance&#160;amount))<br>
&#160;&#160;&#160;&#160;balance))<br></tt>


Calls to <tt>make-simplified-withdraw</tt> produce computational objects, each with a local state variable <tt>balance</tt> that is decremented by successive calls to the object. The object takes an <tt>amount</tt> as an argument and returns the new balance. We can imagine the user of a bank account typing a sequence of inputs to such an object and observing the sequence of returned values shown on a display screen.


Alternatively, we can model a withdrawal processor as a procedure that takes as input a balance and a stream of amounts to withdraw and produces the stream of successive balances in the account:


<tt><a name="index_term_4152"></a>(define&#160;(stream-withdraw&#160;balance&#160;amount-stream)<br>
&#160;&#160;(cons-stream<br>
&#160;&#160;&#160;balance<br>
&#160;&#160;&#160;(stream-withdraw&#160;(-&#160;balance&#160;(stream-car&#160;amount-stream))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-cdr&#160;amount-stream))))<br></tt>


<tt>Stream-withdraw</tt> implements a well-defined mathematical function whose output is fully determined by its input. Suppose, however, that the input <tt>amount-stream</tt> is the stream of successive values typed by the user and that the resulting stream of balances is displayed. Then, from the perspective of the user who is typing values and watching results, the stream process has the same behavior as the object created by <tt>make-simplified-withdraw</tt>. However, with the stream version, there is no assignment, no local state variable, and consequently none of the theoretical difficulties that we encountered <a name="index_term_4154"></a>in section&#160;<a href="chapter_3_section_1.html#%_sec_3.1.3">3.1.3</a>. Yet the system has state!


This is really remarkable. Even though <tt>stream-withdraw</tt> implements a well-defined mathematical function whose behavior does not change, the user's perception here is one of interacting with a system that has a changing state. One way to resolve this paradox is to realize that it is the user's temporal existence that imposes state on the system. If the user could step back from the interaction and think in terms of streams of balances rather than individual transactions, the system would appear stateless.<a name="call_footnote_Temp_504" href="#footnote_Temp_504" id="call_footnote_Temp_504"><sup><small>73</small></sup></a>


From the point of view of one part of a complex process, the other parts appear to change with time. They have hidden time-varying local state. If we wish to write programs that model this kind of natural decomposition in our world (as we see it from our viewpoint as a part of that world) with structures in our computer, we make computational objects that are not functional -- they must change with time. We model state with local state variables, and we model the changes of state with assignments to those variables. By doing this we make the time of execution of a computation model time in the world that we are part of, and thus we get "objects" in our computer.


Modeling with objects is powerful and intuitive, largely because this matches the perception of interacting with a world of which we are part. However, as we've seen repeatedly throughout this chapter, these models raise thorny problems of constraining the order of events and of synchronizing multiple processes. The possibility of avoiding these problems has stimulated the development of <a name="index_term_4158"></a><a name="index_term_4160"></a>*functional programming languages*, which do not include any provision for assignment or mutable data. In such a language, all procedures implement well-defined mathematical functions of their arguments, whose behavior does not change. The functional approach is extremely <a name="index_term_4162"></a><a name="index_term_4164"></a>attractive for dealing with concurrent systems.<a name="call_footnote_Temp_505" href="#footnote_Temp_505" id="call_footnote_Temp_505"><sup><small>74</small></sup></a>


On the other hand, if we look closely, we can see time-related problems creeping into functional models as well. One particularly troublesome area arises when we wish to design interactive systems, especially ones that model interactions between independent entities. For instance, consider once more the implementation a banking system that permits joint bank accounts. In a conventional system using assignment and objects, we would model the fact that Peter and Paul share an account by having both Peter and Paul send their transaction requests to the same bank-account object, as we saw in section&#160;<a href="chapter_3_section_1.html#%_sec_3.1.3">3.1.3</a>. From the stream point of view, where there are no "objects" *per se*, we have already indicated that a bank account can be modeled as a process that operates on a stream of transaction requests to produce a stream of responses. Accordingly, we could model the fact that Peter and Paul have a joint bank account by merging Peter's stream of transaction requests with Paul's stream of requests and feeding the result to the bank-account stream process, as shown in figure&#160;<a href="#%_fig_3.38">3.38</a>.


<a name="%_fig_3.38"></a>

<div align="left">
  <div align="left">
    **Figure 3.38:**&#160;&#160;A joint bank account, modeled by merging two streams of transaction requests.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_3_image_60.gif" border="0"></td>
    </tr>

    <tr>
      <td><a name="index_term_4176"></a></td>
    </tr>
  </table>
</div>

<a name="index_term_4178"></a>The trouble with this formulation is in the notion of *merge*. It will not do to merge the two streams by simply taking alternately one request from Peter and one request from Paul. Suppose Paul accesses the account only very rarely. We could hardly force Peter to wait for Paul to access the account before he could issue a second transaction. However such a merge is implemented, it must interleave the two transaction streams in some way that is constrained by "real time" as perceived by Peter and Paul, in the sense that, if Peter and Paul meet, they can agree that certain transactions were processed before the meeting, and other transactions were processed after the meeting.<a name="call_footnote_Temp_506" href="#footnote_Temp_506" id="call_footnote_Temp_506"><sup><small>75</small></sup></a> This is precisely the same constraint that we had to deal with in section&#160;<a href="chapter_3_section_4.html#%_sec_3.4.1">3.4.1</a>, where we found the need to introduce explicit synchronization to ensure a "correct" order of events in concurrent processing of objects with state. Thus, in an attempt to support the functional style, the need to merge inputs from different agents reintroduces the same problems that the functional style was meant to eliminate.


We began this chapter with the goal of building computational models whose structure matches our perception of the real world we are trying to model. We can model the world as a collection of separate, time-bound, interacting objects with state, or we can model the world as a single, timeless, stateless unity. Each view has powerful advantages, but neither view alone is completely satisfactory. A grand unification has yet to emerge.<a name="call_footnote_Temp_507" href="#footnote_Temp_507" id="call_footnote_Temp_507"><sup><small>76</small></sup></a>

<div class="smallprint">
  <hr>
</div>

<div class="footnote">
  
<a name="footnote_Temp_442" href="#call_footnote_Temp_442" id="footnote_Temp_442"><sup><small>52</small></sup></a> Physicists sometimes adopt this view by introducing the <a name="index_term_3728"></a>"world lines" of particles as a device for reasoning about motion. We've also already mentioned (section&#160;<a href="chapter_2_section_2.html#%_sec_2.2.3">2.2.3</a>) that this is the natural way to think about signal-processing systems. We will explore applications of streams to signal processing in section&#160;<a href="#%_sec_3.5.3">3.5.3</a>.

  
<a name="footnote_Temp_443" href="#call_footnote_Temp_443" id="footnote_Temp_443"><sup><small>53</small></sup></a> Assume that we have a predicate <tt>prime?</tt> (e.g., as in section&#160;<a href="chapter_1_section_2.html#%_sec_1.2.6">1.2.6</a>) that tests for primality.

  
<a name="footnote_Temp_444" href="#call_footnote_Temp_444" id="footnote_Temp_444"><sup><small>54</small></sup></a> In the MIT implementation, <a name="index_term_3752"></a><a name="index_term_3754"></a><a name="index_term_3756"></a><tt>the-empty-stream</tt> is the same as the empty list <tt>'()</tt>, and <tt>stream-null?</tt> is the same as <tt>null?</tt>.

  
<a name="footnote_Temp_445" href="#call_footnote_Temp_445" id="footnote_Temp_445"><sup><small>55</small></sup></a> This should bother you. The fact that we are defining such similar procedures for streams and lists indicates that we are missing some underlying abstraction. Unfortunately, in order to exploit this abstraction, we will need to exert finer control over the process of evaluation than we can at present. We will discuss this point further at the end of section&#160;<a href="#%_sec_3.5.4">3.5.4</a>. In section&#160;<a href="chapter_4_section_2.html#%_sec_4.2">4.2</a>, we'll develop a framework that unifies lists and streams.

  
<a name="footnote_Temp_446" href="#call_footnote_Temp_446" id="footnote_Temp_446"><sup><small>56</small></sup></a> Although <tt>stream-car</tt> and <a name="index_term_3784"></a><a name="index_term_3786"></a><tt>stream-cdr</tt> can be defined as procedures, <tt>cons-stream</tt> must be a special form. If <tt>cons-stream</tt> were a procedure, then, according to our model of evaluation, evaluating <tt>(cons-stream &lt;*a*&gt; &lt;*b*&gt;)</tt> would automatically cause &lt;*b*&gt; to be evaluated, which is precisely what we do not want to happen. For the same reason, <tt>delay</tt> must be a special form, though <tt>force</tt> can be an ordinary procedure.

  
<a name="footnote_Temp_448" href="#call_footnote_Temp_448" id="footnote_Temp_448"><sup><small>57</small></sup></a> The numbers shown here do not really appear in the delayed expression. What actually appears is the original expression, in an environment in which the variables are bound to the appropriate numbers. For example, <tt>(+ low 1)</tt> with <tt>low</tt> bound to 10,000 actually appears where <tt>10001</tt> is shown.

  
<a name="footnote_Temp_450" href="#call_footnote_Temp_450" id="footnote_Temp_450"><sup><small>58</small></sup></a> There are many possible implementations of streams other than the one described in this section. Delayed evaluation, which is the key to making streams practical, was inherent in <a name="index_term_3806"></a><a name="index_term_3808"></a>Algol 60's *call-by-name* parameter-passing method. The use of this mechanism to implement streams was first described by <a name="index_term_3810"></a>Landin (1965). Delayed evaluation for streams was introduced into Lisp by <a name="index_term_3812"></a><a name="index_term_3814"></a>Friedman and Wise (1976). In their implementation, <tt>cons</tt> always delays evaluating its arguments, so that lists automatically behave as streams. The memoizing optimization is also known as <a name="index_term_3816"></a><a name="index_term_3818"></a><a name="index_term_3820"></a><a name="index_term_3822"></a>*call-by-need*. The Algol community would refer to our original delayed objects as *call-by-name thunks* and to the optimized versions as *call-by-need thunks*.

  
<a name="footnote_Temp_453" href="#call_footnote_Temp_453" id="footnote_Temp_453"><sup><small>59</small></sup></a> Exercises such as&#160;<a href="#%_thm_3.51">3.51</a> and&#160;<a href="#%_thm_3.52">3.52</a> are valuable for testing our understanding of how <tt>delay</tt> works. On the other hand, intermixing delayed evaluation with printing -- and, even worse, with assignment -- is extremely confusing, and instructors of courses on computer languages have traditionally tormented their students with examination questions such as the ones in this section. Needless to say, writing programs that depend on such subtleties is <a name="index_term_3828"></a>odious programming style. Part of the power of stream processing is that it lets us ignore the order in which events actually happen in our programs. Unfortunately, this is precisely what we cannot afford to do in the presence of assignment, which forces us to be concerned with time and change.

  
<a name="footnote_Temp_455" href="#call_footnote_Temp_455" id="footnote_Temp_455"><sup><small>60</small></sup></a> Eratosthenes, a third-century <font size="-2">B</font>.<font size="-2">C</font>. <a name="index_term_3846"></a><a name="index_term_3848"></a>Alexandrian Greek philosopher, is famous for giving the first accurate estimate of the circumference of the Earth, which he computed by observing shadows cast at noon on the day of the summer solstice. Eratosthenes's sieve method, although ancient, has formed the basis for special-purpose hardware "sieves" that, until recently, were the most powerful tools in existence for locating large primes. Since the 70s, however, these methods have been superseded by outgrowths of the <a name="index_term_3850"></a>probabilistic techniques discussed in section&#160;<a href="chapter_1_section_2.html#%_sec_1.2.6">1.2.6</a>.

  
<a name="footnote_Temp_456" href="#call_footnote_Temp_456" id="footnote_Temp_456"><sup><small>61</small></sup></a> We have named these figures after <a name="index_term_3858"></a>Peter Henderson, who was the first person to show us diagrams of this sort as a way of thinking about stream processing. Each solid line represents a stream of values being transmitted. The dashed line from the <tt>car</tt> to the <tt>cons</tt> and the <tt>filter</tt> indicates that this is a single value rather than a stream.

  
<a name="footnote_Temp_458" href="#call_footnote_Temp_458" id="footnote_Temp_458"><sup><small>62</small></sup></a> This uses the generalized version of <tt>stream-map</tt> from exercise&#160;<a href="#%_thm_3.50">3.50</a>.

  
<a name="footnote_Temp_459" href="#call_footnote_Temp_459" id="footnote_Temp_459"><sup><small>63</small></sup></a> This last point is very subtle and relies on the fact that *p*<sub>*n*+1</sub> <u>&lt;</u> *p*<sub>*n*</sub><sup>2</sup>. (Here, *p*<sub>*k*</sub> denotes the *k*th prime.) Estimates such as these are very difficult to establish. The ancient proof by <a name="index_term_3876"></a>Euclid that there are an infinite number of primes shows that *p*<sub>*n*+1</sub><u>&lt;</u> *p*<sub>1</sub> *p*<sub>2</sub> <tt>���</tt> *p*<sub>*n*</sub> + 1, and no substantially better result was proved until 1851, when the Russian mathematician P. L. Chebyshev established <a name="index_term_3878"></a><a name="index_term_3880"></a><a name="index_term_3882"></a><a name="index_term_3884"></a>that *p*<sub>*n*+1</sub><u>&lt;</u> 2*p*<sub>*n*</sub> for all *n*. This result, originally conjectured in 1845, is known as *Bertrand's hypothesis*. A proof can be found in section 22.3 of Hardy and Wright 1960.

  
<a name="footnote_Temp_465" href="#call_footnote_Temp_465" id="footnote_Temp_465"><sup><small>64</small></sup></a> This exercise shows how call-by-need is closely related to <a name="index_term_3902"></a><a name="index_term_3904"></a>ordinary memoization as described in exercise&#160;<a href="chapter_3_section_3.html#exercise_3_27">3.27</a>. In that exercise, we used assignment to explicitly construct a local table. Our call-by-need stream optimization effectively constructs such a table automatically, storing values in the previously forced parts of the stream.

  
<a name="footnote_Temp_472" href="#call_footnote_Temp_472" id="footnote_Temp_472"><sup><small>65</small></sup></a> We can't use <tt>let</tt> to bind the local variable <tt>guesses</tt>, because the value of <tt>guesses</tt> depends on <tt>guesses</tt> itself. Exercise&#160;<a href="#%_thm_3.63">3.63</a> addresses why we want a local variable here.

  
<a name="footnote_Temp_477" href="#call_footnote_Temp_477" id="footnote_Temp_477"><sup><small>66</small></sup></a> As in section&#160;<a href="chapter_2_section_2.html#%_sec_2.2.3">2.2.3</a>, we represent a pair of integers as a list rather than a Lisp pair.

  
<a name="footnote_Temp_478" href="#call_footnote_Temp_478" id="footnote_Temp_478"><sup><small>67</small></sup></a> See exercise&#160;<a href="#%_thm_3.68">3.68</a> for some insight into why we chose this decomposition.

  
<a name="footnote_Temp_479" href="#call_footnote_Temp_479" id="footnote_Temp_479"><sup><small>68</small></sup></a> The precise statement of the required property on the order of combination is as follows: There should be a function *f* of two arguments such that the pair corresponding to element&#160;*i* of the first stream and element&#160;*j* of the second stream will appear as element number *f*(*i*,*j*) of the output stream. The trick of using <tt>interleave</tt> to accomplish this was shown to us by <a name="index_term_3996"></a>David Turner, who employed it in the language <a name="index_term_3998"></a>KRC (Turner 1981).

  
<a name="footnote_Temp_485" href="#call_footnote_Temp_485" id="footnote_Temp_485"><sup><small>69</small></sup></a> We will require that the weighting function be such that the weight of a pair increases as we move out along a row or down along a column of the array of pairs.

  
<a name="footnote_Temp_487" href="#call_footnote_Temp_487" id="footnote_Temp_487"><sup><small>70</small></sup></a> To quote from G. H. Hardy's obituary of <a name="index_term_4012"></a><a name="index_term_4014"></a><a name="index_term_4016"></a>Ramanujan (Hardy 1921): ``It was Mr. Littlewood (I believe) who remarked that `every positive integer was one of his friends.' I remember once going to see him when he was lying ill at Putney. I had ridden in taxi-cab No. 1729, and remarked that the number seemed to me a rather dull one, and that I hoped it was not an unfavorable omen. `No,' he replied, `it is a very interesting number; it is the smallest number expressible as the sum of two cubes in two different ways.' " The trick of using weighted pairs to generate the Ramanujan numbers was shown to us by Charles Leiserson.

  
<a name="footnote_Temp_494" href="#call_footnote_Temp_494" id="footnote_Temp_494"><sup><small>71</small></sup></a> This procedure is not guaranteed to work in all Scheme implementations, although for any implementation there is a simple variation that will work. The problem has to do with subtle differences in the ways that Scheme implementations handle internal definitions. (See section&#160;<a href="chapter_4_section_1.html#%_sec_4.1.6">4.1.6</a>.)

  
<a name="footnote_Temp_500" href="#call_footnote_Temp_500" id="footnote_Temp_500"><sup><small>72</small></sup></a> This is a small reflection, in Lisp, of the difficulties that conventional strongly typed languages such as <a name="index_term_4096"></a><a name="index_term_4098"></a><a name="index_term_4100"></a><a name="index_term_4102"></a><a name="index_term_4104"></a>Pascal have in coping with higher-order procedures. In such languages, the programmer must specify the data types of the arguments and the result of each procedure: number, logical value, sequence, and so on. Consequently, we could not express an abstraction such as "map a given procedure <tt>proc</tt> over all the elements in a sequence" by a single higher-order procedure such as <tt>stream-map</tt>. Rather, we would need a different mapping procedure for each different combination of argument and result data types that might be specified for a <tt>proc</tt>. Maintaining a practical notion of "data type" in the presence of higher-order procedures raises many difficult issues. One way of dealing with this problem is illustrated by the language ML <a name="index_term_4106"></a><a name="index_term_4108"></a><a name="index_term_4110"></a><a name="index_term_4112"></a>(Gordon, Milner, and Wadsworth 1979), whose "polymorphic data types" include templates for higher-order transformations between data types. Moreover, data types for most procedures in ML are never explicitly declared by the programmer. Instead, ML includes a <a name="index_term_4114"></a>*type-inferencing* mechanism that uses information in the environment to deduce the data types for newly defined procedures.

  
<a name="footnote_Temp_504" href="#call_footnote_Temp_504" id="footnote_Temp_504"><sup><small>73</small></sup></a> Similarly in physics, when we observe a moving particle, we say that the position (state) of the particle is changing. However, from the perspective of the particle's <a name="index_term_4156"></a>world line in space-time there is no change involved.

  
<a name="footnote_Temp_505" href="#call_footnote_Temp_505" id="footnote_Temp_505"><sup><small>74</small></sup></a> John Backus, the inventor of Fortran, gave high <a name="index_term_4166"></a><a name="index_term_4168"></a><a name="index_term_4170"></a><a name="index_term_4172"></a><a name="index_term_4174"></a>visibility to functional programming when he was awarded the ACM Turing award in 1978. His acceptance speech (Backus 1978) strongly advocated the functional approach. A good overview of functional programming is given in Henderson 1980 and in Darlington, Henderson, and Turner 1982.

  
<a name="footnote_Temp_506" href="#call_footnote_Temp_506" id="footnote_Temp_506"><sup><small>75</small></sup></a> Observe that, for any two streams, there is in general more than one <a name="index_term_4180"></a><a name="index_term_4182"></a>acceptable order of interleaving. Thus, technically, "merge" is a relation rather than a function -- the answer is not a deterministic function of the inputs. We already mentioned (footnote&#160;<a href="chapter_3_section_4.html#footnote_Temp_411">39</a>) that nondeterminism is essential when dealing with concurrency. The merge relation illustrates the same essential nondeterminism, from the functional perspective. In section&#160;<a href="chapter_4_section_3.html#%_sec_4.3">4.3</a>, we will look at nondeterminism from yet another point of view.

  
<a name="footnote_Temp_507" href="#call_footnote_Temp_507" id="footnote_Temp_507"><sup><small>76</small></sup></a> The object model approximates the world by dividing it into separate pieces. The functional model does not modularize along object boundaries. The object model is useful when <a name="index_term_4184"></a>the unshared state of the "objects" is much larger than the state that they share. An example of a place where the object viewpoint fails is <a name="index_term_4186"></a>quantum mechanics, where thinking of things as individual particles leads to paradoxes and confusions. Unifying the object view with the functional view may have little to do with programming, but rather with fundamental epistemological issues.

</div>