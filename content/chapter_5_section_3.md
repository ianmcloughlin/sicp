(section_53)=
# Storage Allocation and Garbage Collection

<a name="index_term_5828"></a><a name="index_term_5830"></a> In section <a href="chapter_5_section_4.html#%_sec_5.4">5.4</a>, we will show how to implement a Scheme evaluator as a register machine. In order to simplify the discussion, we will assume that our register machines can be equipped with a *list-structured memory*, in which the basic operations for manipulating list-structured data are primitive. Postulating the existence of such a memory is a useful abstraction when one is focusing on the mechanisms of control in a Scheme interpreter, but this does not reflect a realistic view of the actual primitive data operations of contemporary computers. To obtain a more complete picture of how a Lisp system operates, we must investigate how list structure can be represented in a way that is compatible with conventional computer memories.


There are two considerations in implementing list structure. The first is purely an issue of representation: how to represent the "box-and-pointer" structure of Lisp pairs, using only the storage and addressing capabilities of typical computer memories. The second issue concerns the management of memory as a computation proceeds. The operation of a Lisp system depends crucially on the ability to continually create new data objects. These include objects that are explicitly created by the Lisp procedures being interpreted as well as structures created by the interpreter itself, such as environments and argument lists. Although the constant creation of new data objects would pose no problem on a computer with an infinite amount of rapidly addressable memory, computer memories are available only in finite sizes (more's the pity). Lisp systems thus provide an <a name="index_term_5832"></a>*automatic storage allocation* facility to support the illusion of an infinite memory. When a data object is no longer needed, the memory allocated to it is automatically recycled and used to construct new data objects. There are various techniques for providing such automatic storage allocation. The method we shall discuss in this section is called *garbage collection*.


<a name="%_sec_5.3.1"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_5.3.1">5.3.1&nbsp;&nbsp;Memory as Vectors</a></h3>

A conventional computer memory can be thought of as an array of cubbyholes, each of which can contain a piece of information. Each cubbyhole has a unique name, called its <a name="index_term_5834"></a>*address* or <a name="index_term_5836"></a>*location*. Typical memory systems provide two primitive operations: one that fetches the data stored in a specified location and one that assigns new data to a specified location. Memory addresses can be incremented to support sequential access to some set of the cubbyholes. More generally, many important data operations require that memory addresses be treated as data, which can be stored in memory locations and manipulated in machine registers. The representation of list structure is one application of such <a name="index_term_5838"></a><a name="index_term_5840"></a>*address arithmetic*.


To model computer memory, we use a new kind of data structure called a <a name="index_term_5842"></a>*vector*. Abstractly, a vector is a compound data object whose individual elements can be accessed by means of an integer index in an amount of time that is independent of the index.<a name="call_footnote_Temp_744" href="#footnote_Temp_744" id="call_footnote_Temp_744"><sup><small>5</small></sup></a> In order to describe memory operations, we use two primitive Scheme procedures for manipulating vectors:

<ul>
  <li style="list-style: none"><a name="index_term_5844"></a><a name="index_term_5846"></a></li>

  <li>
    <tt>(vector-ref &lt;*vector*&gt; &lt;*n*&gt;)</tt> returns the *n*th element of the vector.

    
<a name="index_term_5848"></a><a name="index_term_5850"></a>

  </li>

  <li><tt>(vector-set! &lt;*vector*&gt; &lt;*n*&gt; &lt;*value*&gt;)</tt> sets the *n*th element of the vector to the designated value.</li>
</ul>

For example, if <tt>v</tt> is a vector, then <tt>(vector-ref v 5)</tt> gets the fifth entry in the vector <tt>v</tt> and <tt>(vector-set! v 5 7)</tt> changes the value of the fifth entry of the vector <tt>v</tt> to 7.<a name="call_footnote_Temp_745" href="#footnote_Temp_745" id="call_footnote_Temp_745"><sup><small>6</small></sup></a> For computer memory, this access can be implemented through the use of address arithmetic to combine a *base address* that specifies the beginning location of a vector in memory with an *index* that specifies the offset of a particular element of the vector.


<a name="%_sec_Temp_746"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_746">Representing Lisp data</a></h4>

<a name="index_term_5852"></a><a name="index_term_5854"></a> We can use vectors to implement the basic pair structures required for a list-structured memory. Let us imagine that computer memory is divided into two vectors: <a name="index_term_5856"></a><tt>the-cars</tt> and <a name="index_term_5858"></a><tt>the-cdrs</tt>. We will represent list structure as follows: A pointer to a pair is an index into the two vectors. The <tt>car</tt> of the pair is the entry in <tt>the-cars</tt> with the designated index, and the <tt>cdr</tt> of the pair is the entry in <tt>the-cdrs</tt> with the designated index. We also need a representation for objects other than pairs (such as numbers and symbols) and a way to distinguish one kind of data from another. There are many methods of accomplishing this, but they all reduce to using <a name="index_term_5860"></a><a name="index_term_5862"></a>*typed pointers*, that is, to extending the notion of "pointer" to include information on data type.<a name="call_footnote_Temp_747" href="#footnote_Temp_747" id="call_footnote_Temp_747"><sup><small>7</small></sup></a> The data type enables the system to distinguish a pointer to a pair (which consists of the "pair" data type and an index into the memory vectors) from pointers to other kinds of data (which consist of some other data type and whatever is being used to represent data of that type). Two data objects are <a name="index_term_5868"></a>considered to be the same (<tt>eq?</tt>) if their pointers are identical.<a name="call_footnote_Temp_748" href="#footnote_Temp_748" id="call_footnote_Temp_748"><sup><small>8</small></sup></a> Figure&nbsp;<a href="#%_fig_5.14">5.14</a> illustrates the use of this method to represent the list <tt>((1 2) 3 4)</tt>, whose box-and-pointer diagram is also shown. We use letter prefixes to denote the data-type information. Thus, a pointer to the pair with index 5 is denoted <tt>p5</tt>, the empty list is denoted by the pointer <tt>e0</tt>, and a pointer to the number 4 is denoted <tt>n4</tt>. In the box-and-pointer diagram, we have indicated at the lower left of each pair the vector index that specifies where the <tt>car</tt> and <tt>cdr</tt> of the pair are stored. The blank locations in <tt>the-cars</tt> and <tt>the-cdrs</tt> may contain parts of other list structures (not of interest here).


<a name="%_fig_5.14"></a>

<div align="left">
  <div align="left">
    **Figure 5.14:**&nbsp;&nbsp;Box-and-pointer and memory-vector representations of the list <tt>((1 2) 3 4)</tt>.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_5_image_07.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

A pointer to a number, such as <tt>n4</tt>, might consist of a type indicating numeric data together with the actual representation of the number 4.<a name="call_footnote_Temp_749" href="#footnote_Temp_749" id="call_footnote_Temp_749"><sup><small>9</small></sup></a> To deal with numbers that are too large to be represented in the fixed amount of space allocated for a single pointer, we could use a distinct <a name="index_term_5880"></a>*bignum* data type, for which the pointer designates a list in which the parts of the number are stored.<a name="call_footnote_Temp_750" href="#footnote_Temp_750" id="call_footnote_Temp_750"><sup><small>10</small></sup></a>


<a name="index_term_5882"></a>A symbol might be represented as a typed pointer that designates a sequence of the characters that form the symbol's printed representation. This sequence is constructed by the Lisp reader when the character string is initially encountered in input. Since we want two instances of a symbol to be recognized as the "same" symbol by <tt>eq?</tt> and we <a name="index_term_5884"></a>want <tt>eq?</tt> to be a simple test for equality of pointers, we must ensure that if the reader sees the same character string twice, it will use the same pointer (to the same sequence of characters) to represent both occurrences. To accomplish this, the reader maintains a table, traditionally called the <a name="index_term_5886"></a>*obarray*, of all the symbols it has ever encountered. When the reader encounters a character string and is about to construct a symbol, it checks the obarray to see if it has ever before seen the same character string. If it has not, it uses the characters to construct a new symbol (a typed pointer to a new character sequence) and enters this pointer in the obarray. If the reader has seen the string before, it returns the symbol pointer stored in the obarray. This process of replacing character strings by unique pointers is called <a name="index_term_5888"></a><a name="index_term_5890"></a>*interning* symbols.


<a name="%_sec_Temp_751"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_751">Implementing the primitive list operations</a></h4>

<a name="index_term_5892"></a><a name="index_term_5894"></a>Given the above representation scheme, we can replace each "primitive" list operation of a register machine with one or more primitive vector operations. We will use two registers, <tt>the-cars</tt> and <tt>the-cdrs</tt>, to identify the memory vectors, and will assume that <tt>vector-ref</tt> and <tt>vector-set!</tt> are available as primitive operations. We also assume that numeric operations on pointers (such as incrementing a pointer, using a pair pointer to index a vector, or adding two numbers) use only the index portion of the typed pointer.


For example, we can make a register machine support the instructions


<a name="index_term_5896"></a><a name="index_term_5898"></a>


<tt>(assign&nbsp;&lt;*reg<sub>1</sub>*&gt;&nbsp;(op&nbsp;car)&nbsp;(reg&nbsp;&lt;*reg<sub>2</sub>*&gt;))<br>
<br>
(assign&nbsp;&lt;*reg<sub>1</sub>*&gt;&nbsp;(op&nbsp;cdr)&nbsp;(reg&nbsp;&lt;*reg<sub>2</sub>*&gt;))<br></tt>


if we implement these, respectively, as


<tt>(assign&nbsp;&lt;*reg<sub>1</sub>*&gt;&nbsp;(op&nbsp;vector-ref)&nbsp;(reg&nbsp;the-cars)&nbsp;(reg&nbsp;&lt;*reg<sub>2</sub>*&gt;))<br>
<br>
(assign&nbsp;&lt;*reg<sub>1</sub>*&gt;&nbsp;(op&nbsp;vector-ref)&nbsp;(reg&nbsp;the-cdrs)&nbsp;(reg&nbsp;&lt;*reg<sub>2</sub>*&gt;))<br></tt>


The instructions


<a name="index_term_5900"></a><a name="index_term_5902"></a>


<tt>(perform&nbsp;(op&nbsp;set-car!)&nbsp;(reg&nbsp;&lt;*reg<sub>1</sub>*&gt;)&nbsp;(reg&nbsp;&lt;*reg<sub>2</sub>*&gt;))<br>
<br>
(perform&nbsp;(op&nbsp;set-cdr!)&nbsp;(reg&nbsp;&lt;*reg<sub>1</sub>*&gt;)&nbsp;(reg&nbsp;&lt;*reg<sub>2</sub>*&gt;))<br></tt>


are implemented as


<tt>(perform<br>
&nbsp;(op&nbsp;vector-set!)&nbsp;(reg&nbsp;the-cars)&nbsp;(reg&nbsp;&lt;*reg<sub>1</sub>*&gt;)&nbsp;(reg&nbsp;&lt;*reg<sub>2</sub>*&gt;))<br>
<br>
(perform<br>
&nbsp;(op&nbsp;vector-set!)&nbsp;(reg&nbsp;the-cdrs)&nbsp;(reg&nbsp;&lt;*reg<sub>1</sub>*&gt;)&nbsp;(reg&nbsp;&lt;*reg<sub>2</sub>*&gt;))<br></tt>


<a name="index_term_5904"></a><tt>Cons</tt> is performed by allocating an unused index and storing the arguments to <tt>cons</tt> in <tt>the-cars</tt> and <tt>the-cdrs</tt> at that indexed vector position. We presume that there is a special register, <a name="index_term_5906"></a><tt>free</tt>, that always holds a pair pointer containing the next available index, and that we can increment the index part of that pointer to find the next free location.<a name="call_footnote_Temp_752" href="#footnote_Temp_752" id="call_footnote_Temp_752"><sup><small>11</small></sup></a> For example, the instruction


<tt>(assign&nbsp;&lt;*reg<sub>1</sub>*&gt;&nbsp;(op&nbsp;cons)&nbsp;(reg&nbsp;&lt;*reg<sub>2</sub>*&gt;)&nbsp;(reg&nbsp;&lt;*reg<sub>3</sub>*&gt;))<br></tt>


is implemented as the following sequence of vector operations:<a name="call_footnote_Temp_753" href="#footnote_Temp_753" id="call_footnote_Temp_753"><sup><small>12</small></sup></a>


<tt>(perform<br>
&nbsp;(op&nbsp;vector-set!)&nbsp;(reg&nbsp;the-cars)&nbsp;(reg&nbsp;free)&nbsp;(reg&nbsp;&lt;*reg<sub>2</sub>*&gt;))<br>
(perform<br>
&nbsp;(op&nbsp;vector-set!)&nbsp;(reg&nbsp;the-cdrs)&nbsp;(reg&nbsp;free)&nbsp;(reg&nbsp;&lt;*reg<sub>3</sub>*&gt;))<br>
(assign&nbsp;&lt;*reg<sub>1</sub>*&gt;&nbsp;(reg&nbsp;free))<br>
(assign&nbsp;free&nbsp;(op&nbsp;+)&nbsp;(reg&nbsp;free)&nbsp;(const&nbsp;1))<br></tt>


The <tt>eq?</tt> operation


<tt>(op&nbsp;eq?)&nbsp;(reg&nbsp;&lt;*reg<sub>1</sub>*&gt;)&nbsp;(reg&nbsp;&lt;*reg<sub>2</sub>*&gt;)<br></tt>


simply tests the equality of all fields in the registers, and <a name="index_term_5910"></a><a name="index_term_5912"></a><a name="index_term_5914"></a><a name="index_term_5916"></a>predicates such as <tt>pair?</tt>, <tt>null?</tt>, <tt>symbol?</tt>, and <tt>number?</tt> need only check the type field.


<a name="%_sec_Temp_754"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_754">Implementing stacks</a></h4>

<a name="index_term_5918"></a> Although our register machines use stacks, we need do nothing special here, since stacks can be modeled in terms of lists. The stack can be a list of the saved values, pointed to by a special register <tt>the-stack</tt>. Thus, <tt>(save &lt;*reg*&gt;)</tt> can be implemented as


<a name="index_term_5920"></a>


<tt>(assign&nbsp;the-stack&nbsp;(op&nbsp;cons)&nbsp;(reg&nbsp;&lt;*reg*&gt;)&nbsp;(reg&nbsp;the-stack))<br></tt>


<a name="index_term_5922"></a>Similarly, <tt>(restore &lt;*reg*&gt;)</tt> can be implemented as


<tt>(assign&nbsp;&lt;*reg*&gt;&nbsp;(op&nbsp;car)&nbsp;(reg&nbsp;the-stack))<br>
(assign&nbsp;the-stack&nbsp;(op&nbsp;cdr)&nbsp;(reg&nbsp;the-stack))<br></tt>


and <tt>(perform (op initialize-stack))</tt> can be implemented as


<tt>(assign&nbsp;the-stack&nbsp;(const&nbsp;()))<br></tt>


These operations can be further expanded in terms of the vector operations given above. In conventional computer architectures, however, it is usually advantageous to allocate the stack as a separate vector. Then pushing and popping the stack can be accomplished by incrementing or decrementing an index into that vector.


<a name="exercise_5_20">**Exercise 5.20**</a> Draw the box-and-pointer representation and the memory-vector representation (as in figure&nbsp;<a href="#%_fig_5.14">5.14</a>) of the list structure produced by


<tt>(define&nbsp;x&nbsp;(cons&nbsp;1&nbsp;2))<br>
(define&nbsp;y&nbsp;(list&nbsp;x&nbsp;x))<br></tt>


with the <tt>free</tt> pointer initially <tt>p1</tt>. What is the final value of <tt>free</tt> ? What pointers represent the values of <tt>x</tt> and <tt>y</tt> ?


<a name="exercise_5_21">**Exercise 5.21**</a> <a name="index_term_5924"></a>Implement register machines for the following procedures. Assume that the list-structure memory operations are available as machine primitives.


a. Recursive <tt>count-leaves</tt>:


<tt>(define&nbsp;(count-leaves&nbsp;tree)<br>
&nbsp;&nbsp;(cond&nbsp;((null?&nbsp;tree)&nbsp;0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;((not&nbsp;(pair?&nbsp;tree))&nbsp;1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(else&nbsp;(+&nbsp;(count-leaves&nbsp;(car&nbsp;tree))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(count-leaves&nbsp;(cdr&nbsp;tree))))))<br></tt>


b. Recursive <tt>count-leaves</tt> with explicit counter:


<tt>(define&nbsp;(count-leaves&nbsp;tree)<br>
&nbsp;&nbsp;(define&nbsp;(count-iter&nbsp;tree&nbsp;n)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(cond&nbsp;((null?&nbsp;tree)&nbsp;n)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;((not&nbsp;(pair?&nbsp;tree))&nbsp;(+&nbsp;n&nbsp;1))<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(else&nbsp;(count-iter&nbsp;(cdr&nbsp;tree)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(count-iter&nbsp;(car&nbsp;tree)&nbsp;n)))))<br>
&nbsp;&nbsp;(count-iter&nbsp;tree&nbsp;0))<br></tt>


<a name="exercise_5_22">**Exercise 5.22**</a> <a name="index_term_5926"></a><a name="index_term_5928"></a>Exercise&nbsp;<a href="chapter_3_section_3.html#exercise_3_12">3.12</a> of section <a href="chapter_3_section_3.html#%_sec_3.3.1">3.3.1</a> presented an <tt>append</tt> procedure that appends two lists to form a new list and an <tt>append!</tt> procedure that splices two lists together. Design a register machine to implement each of these procedures. Assume that the list-structure memory operations are available as primitive operations.


<a name="%_sec_5.3.2"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_5.3.2">5.3.2&nbsp;&nbsp;Maintaining the Illusion of Infinite Memory</a></h3>

<a name="index_term_5930"></a>


The representation method outlined in section <a href="#%_sec_5.3.1">5.3.1</a> solves the problem of implementing list structure, provided that we have an infinite amount of memory. With a real computer we will eventually run out of free space in which to construct new pairs.<a name="call_footnote_Temp_758" href="#footnote_Temp_758" id="call_footnote_Temp_758"><sup><small>13</small></sup></a> However, most of the pairs generated in a typical computation are used only to hold intermediate results. After these results are accessed, the pairs are no longer needed -- they are *garbage*. For instance, the computation


<tt>(accumulate&nbsp;+&nbsp;0&nbsp;(filter&nbsp;odd?&nbsp;(enumerate-interval&nbsp;0&nbsp;n)))<br></tt>


constructs two lists: the enumeration and the result of filtering the enumeration. When the accumulation is complete, these lists are no longer needed, and the allocated memory can be reclaimed. If we can arrange to collect all the garbage periodically, and if this turns out to recycle memory at about the same rate at which we construct new pairs, we will have preserved the illusion that there is an infinite amount of memory.


In order to recycle pairs, we must have a way to determine which allocated pairs are not needed (in the sense that their contents can no longer influence the future of the computation). The method we shall examine for accomplishing this is known as *garbage collection*. Garbage collection is based on the observation that, at any moment in a Lisp interpretation, the only objects that can affect the future of the computation are those that can be reached by some succession of <tt>car</tt> and <tt>cdr</tt> operations starting from the pointers that are currently in the machine registers.<a name="call_footnote_Temp_759" href="#footnote_Temp_759" id="call_footnote_Temp_759"><sup><small>14</small></sup></a> Any memory cell that is not so accessible may be recycled.


There are many ways to perform garbage collection. The method we shall examine here is called <a name="index_term_5932"></a><a name="index_term_5934"></a>*stop-and-copy*. The basic idea is to divide memory into two halves: "working memory" and "free memory." When <tt>cons</tt> constructs pairs, it allocates these in working memory. When working memory is full, we perform garbage collection by locating all the useful pairs in working memory and copying these into consecutive locations in free memory. (The useful pairs are located by tracing all the <tt>car</tt> and <tt>cdr</tt> pointers, starting with the machine registers.) Since we do not copy the garbage, there will presumably be additional free memory that we can use to allocate new pairs. In addition, nothing in the working memory is needed, since all the useful pairs in it have been copied. Thus, if we interchange the roles of working memory and free memory, we can continue processing; new pairs will be allocated in the new working memory (which was the old free memory). When this is full, we can copy the useful pairs into the new free memory (which was the old working memory).<a name="call_footnote_Temp_760" href="#footnote_Temp_760" id="call_footnote_Temp_760"><sup><small>15</small></sup></a>


<a name="%_sec_Temp_761"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_761">Implementation of a stop-and-copy garbage collector</a></h4>

We now use our register-machine language to describe the stop-and-copy algorithm in more detail. We will assume that there is a register called <a name="index_term_5966"></a><tt>root</tt> that contains a pointer to a structure that eventually points at all accessible data. This can be arranged by storing the contents of all the machine registers in a pre-allocated list pointed at by <tt>root</tt> just before starting garbage collection.<a name="call_footnote_Temp_762" href="#footnote_Temp_762" id="call_footnote_Temp_762"><sup><small>16</small></sup></a> We also assume that, in addition to the current working memory, there is free memory available into which we can copy the useful data. The current working memory consists of vectors whose base addresses are in <a name="index_term_5968"></a><a name="index_term_5970"></a>registers called <tt>the-cars</tt> and <tt>the-cdrs</tt>, and the free memory is in registers called <a name="index_term_5972"></a><a name="index_term_5974"></a><tt>new-cars</tt> and <tt>new-cdrs</tt>.


Garbage collection is triggered when we exhaust the free cells in the current working memory, that is, when a <tt>cons</tt> operation attempts to increment the <tt>free</tt> pointer beyond the end of the memory vector. When the garbage-collection process is complete, the <tt>root</tt> pointer will point into the new memory, all objects accessible from the <tt>root</tt> will have been moved to the new memory, and the <tt>free</tt> pointer will indicate the next place in the new memory where a new pair can be allocated. In addition, the roles of working memory and new memory will have been interchanged -- new pairs will be constructed in the new memory, beginning at the place indicated by <tt>free</tt>, and the (previous) working memory will be available as the new memory for the next garbage collection. Figure&nbsp;<a href="#%_fig_5.15">5.15</a> shows the arrangement of memory just before and just after garbage collection.


<a name="%_fig_5.15"></a>

<div align="left">
  <div align="left">
    **Figure 5.15:**&nbsp;&nbsp;Reconfiguration of memory by the garbage-collection process.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_5_image_08.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="index_term_5976"></a><a name="index_term_5978"></a>The state of the garbage-collection process is controlled by maintaining two pointers: <tt>free</tt> and <tt>scan</tt>. These are initialized to point to the beginning of the new memory. The algorithm begins by relocating the pair pointed at by <tt>root</tt> to the beginning of the new memory. The pair is copied, the <tt>root</tt> pointer is adjusted to point to the new location, and the <tt>free</tt> pointer is incremented. In addition, the old location of the pair is marked to show that its contents have been moved. This marking is done as follows: In the <tt>car</tt> position, we place a special tag that signals that this is an already-moved object. (Such an object is traditionally called a <a name="index_term_5980"></a>*broken heart*.)<a name="call_footnote_Temp_763" href="#footnote_Temp_763" id="call_footnote_Temp_763"><sup><small>17</small></sup></a> In the <tt>cdr</tt> position we place a <a name="index_term_5988"></a>*forwarding address* that points at the location to which the object has been moved.


After relocating the root, the garbage collector enters its basic cycle. At each step in the algorithm, the <tt>scan</tt> pointer (initially pointing at the relocated root) points at a pair that has been moved to the new memory but whose <tt>car</tt> and <tt>cdr</tt> pointers still refer to objects in the old memory. These objects are each relocated, and the <tt>scan</tt> pointer is incremented. To relocate an object (for example, the object indicated by the <tt>car</tt> pointer of the pair we are scanning) we check to see if the object has already been moved (as indicated by the presence of a broken-heart tag in the <tt>car</tt> position of the object). If the object has not already been moved, we copy it to the place indicated by <tt>free</tt>, update <tt>free</tt>, set up a broken heart at the object's old location, and update the pointer to the object (in this example, the <tt>car</tt> pointer of the pair we are scanning) to point to the new location. If the object has already been moved, its forwarding address (found in the <tt>cdr</tt> position of the broken heart) is substituted for the pointer in the pair being scanned. Eventually, all accessible objects will have been moved and scanned, at which point the <tt>scan</tt> pointer will overtake the <tt>free</tt> pointer and the process will terminate.


We can specify the stop-and-copy algorithm as a sequence of instructions for a register machine. The basic step of relocating an object is accomplished by a subroutine called <tt>relocate-old-result-in-new</tt>. This subroutine gets its argument, a pointer to the object to be relocated, from a register named <a name="index_term_5990"></a><tt>old</tt>. It relocates the designated object (incrementing <tt>free</tt> in the process), puts a pointer to the relocated object into a register called <a name="index_term_5992"></a><tt>new</tt>, and returns by branching to the entry point stored in the register <tt>relocate-continue</tt>. To begin garbage collection, we invoke this subroutine to relocate the <tt>root</tt> pointer, after initializing <tt>free</tt> and <tt>scan</tt>. When the relocation of <tt>root</tt> has been accomplished, we install the new pointer as the new <tt>root</tt> and enter the main loop of the garbage collector.


<tt>begin-garbage-collection<br>
&nbsp;&nbsp;(assign&nbsp;free&nbsp;(const&nbsp;0))<br>
&nbsp;&nbsp;(assign&nbsp;scan&nbsp;(const&nbsp;0))<br>
&nbsp;&nbsp;(assign&nbsp;old&nbsp;(reg&nbsp;root))<br>
&nbsp;&nbsp;(assign&nbsp;relocate-continue&nbsp;(label&nbsp;reassign-root))<br>
&nbsp;&nbsp;(goto&nbsp;(label&nbsp;relocate-old-result-in-new))<br>
reassign-root<br>
&nbsp;&nbsp;(assign&nbsp;root&nbsp;(reg&nbsp;new))<br>
&nbsp;&nbsp;(goto&nbsp;(label&nbsp;gc-loop))<br></tt>


In the main loop of the garbage collector we must determine whether there are any more objects to be scanned. We do this by testing whether the <tt>scan</tt> pointer is coincident with the <tt>free</tt> pointer. If the pointers are equal, then all accessible objects have been relocated, and we branch to <tt>gc-flip</tt>, which cleans things up so that we can continue the interrupted computation. If there are still pairs to be scanned, we call the relocate subroutine to relocate the <tt>car</tt> of the next pair (by placing the <tt>car</tt> pointer in <tt>old</tt>). The <tt>relocate-continue</tt> register is set up so that the subroutine will return to update the <tt>car</tt> pointer.


<tt>gc-loop<br>
&nbsp;&nbsp;(test&nbsp;(op&nbsp;=)&nbsp;(reg&nbsp;scan)&nbsp;(reg&nbsp;free))<br>
&nbsp;&nbsp;(branch&nbsp;(label&nbsp;gc-flip))<br>
&nbsp;&nbsp;(assign&nbsp;old&nbsp;(op&nbsp;vector-ref)&nbsp;(reg&nbsp;new-cars)&nbsp;(reg&nbsp;scan))<br>
&nbsp;&nbsp;(assign&nbsp;relocate-continue&nbsp;(label&nbsp;update-car))<br>
&nbsp;&nbsp;(goto&nbsp;(label&nbsp;relocate-old-result-in-new))<br></tt>


At <tt>update-car</tt>, we modify the <tt>car</tt> pointer of the pair being scanned, then proceed to relocate the <tt>cdr</tt> of the pair. We return to <tt>update-cdr</tt> when that relocation has been accomplished. After relocating and updating the <tt>cdr</tt>, we are finished scanning that pair, so we continue with the main loop.


<tt>update-car<br>
&nbsp;&nbsp;(perform<br>
&nbsp;&nbsp;&nbsp;(op&nbsp;vector-set!)&nbsp;(reg&nbsp;new-cars)&nbsp;(reg&nbsp;scan)&nbsp;(reg&nbsp;new))<br>
&nbsp;&nbsp;(assign&nbsp;old&nbsp;(op&nbsp;vector-ref)&nbsp;(reg&nbsp;new-cdrs)&nbsp;(reg&nbsp;scan))<br>
&nbsp;&nbsp;(assign&nbsp;relocate-continue&nbsp;(label&nbsp;update-cdr))<br>
&nbsp;&nbsp;(goto&nbsp;(label&nbsp;relocate-old-result-in-new))<br>
<br>
update-cdr<br>
&nbsp;&nbsp;(perform<br>
&nbsp;&nbsp;&nbsp;(op&nbsp;vector-set!)&nbsp;(reg&nbsp;new-cdrs)&nbsp;(reg&nbsp;scan)&nbsp;(reg&nbsp;new))<br>
&nbsp;&nbsp;(assign&nbsp;scan&nbsp;(op&nbsp;+)&nbsp;(reg&nbsp;scan)&nbsp;(const&nbsp;1))<br>
&nbsp;&nbsp;(goto&nbsp;(label&nbsp;gc-loop))<br></tt>


The subroutine <tt>relocate-old-result-in-new</tt> relocates objects as follows: If the object to be relocated (pointed at by <tt>old</tt>) is not a pair, then we return the same pointer to the object unchanged (in <tt>new</tt>). (For example, we may be scanning a pair whose <tt>car</tt> is the number 4. If we represent the <tt>car</tt> by <tt>n4</tt>, as described in section <a href="#%_sec_5.3.1">5.3.1</a>, then we want the "relocated" <tt>car</tt> pointer to still be <tt>n4</tt>.) Otherwise, we must perform the relocation. If the <tt>car</tt> position of the pair to be relocated contains a broken-heart tag, then the pair has in fact already been moved, so we retrieve the forwarding address (from the <tt>cdr</tt> position of the broken heart) and return this in <tt>new</tt>. If the pointer in <tt>old</tt> points at a yet-unmoved pair, then we move the pair to the first free cell in new memory (pointed at by <tt>free</tt>) and set up the broken heart by storing a broken-heart tag and forwarding address at the old location. <tt>Relocate-old-result-in-new</tt> uses a register <a name="index_term_5994"></a><tt>oldcr</tt> to hold the <tt>car</tt> or the <tt>cdr</tt> of the object pointed at by <tt>old</tt>.<a name="call_footnote_Temp_764" href="#footnote_Temp_764" id="call_footnote_Temp_764"><sup><small>18</small></sup></a>


<tt>relocate-old-result-in-new<br>
&nbsp;&nbsp;(test&nbsp;(op&nbsp;pointer-to-pair?)&nbsp;(reg&nbsp;old))<br>
&nbsp;&nbsp;(branch&nbsp;(label&nbsp;pair))<br>
&nbsp;&nbsp;(assign&nbsp;new&nbsp;(reg&nbsp;old))<br>
&nbsp;&nbsp;(goto&nbsp;(reg&nbsp;relocate-continue))<br>
pair<br>
&nbsp;&nbsp;(assign&nbsp;oldcr&nbsp;(op&nbsp;vector-ref)&nbsp;(reg&nbsp;the-cars)&nbsp;(reg&nbsp;old))<br>
&nbsp;&nbsp;(test&nbsp;(op&nbsp;broken-heart?)&nbsp;(reg&nbsp;oldcr))<br>
&nbsp;&nbsp;(branch&nbsp;(label&nbsp;already-moved))<br>
&nbsp;&nbsp;(assign&nbsp;new&nbsp;(reg&nbsp;free))&nbsp;*;&nbsp;new&nbsp;location&nbsp;for&nbsp;pair*<br>
&nbsp;&nbsp;*;;&nbsp;Update&nbsp;<tt>free</tt>&nbsp;pointer.*<br>
&nbsp;&nbsp;(assign&nbsp;free&nbsp;(op&nbsp;+)&nbsp;(reg&nbsp;free)&nbsp;(const&nbsp;1))<br>
&nbsp;&nbsp;*;;&nbsp;Copy&nbsp;the&nbsp;<tt>car</tt>&nbsp;and&nbsp;<tt>cdr</tt>&nbsp;to&nbsp;new&nbsp;memory.*<br>
&nbsp;&nbsp;(perform&nbsp;(op&nbsp;vector-set!)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(reg&nbsp;new-cars)&nbsp;(reg&nbsp;new)&nbsp;(reg&nbsp;oldcr))<br>
&nbsp;&nbsp;(assign&nbsp;oldcr&nbsp;(op&nbsp;vector-ref)&nbsp;(reg&nbsp;the-cdrs)&nbsp;(reg&nbsp;old))<br>
&nbsp;&nbsp;(perform&nbsp;(op&nbsp;vector-set!)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(reg&nbsp;new-cdrs)&nbsp;(reg&nbsp;new)&nbsp;(reg&nbsp;oldcr))<br>
&nbsp;&nbsp;*;;&nbsp;Construct&nbsp;the&nbsp;broken&nbsp;heart.*<br>
&nbsp;&nbsp;(perform&nbsp;(op&nbsp;vector-set!)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(reg&nbsp;the-cars)&nbsp;(reg&nbsp;old)&nbsp;(const&nbsp;broken-heart))<br>
&nbsp;&nbsp;(perform<br>
&nbsp;&nbsp;&nbsp;(op&nbsp;vector-set!)&nbsp;(reg&nbsp;the-cdrs)&nbsp;(reg&nbsp;old)&nbsp;(reg&nbsp;new))<br>
&nbsp;&nbsp;(goto&nbsp;(reg&nbsp;relocate-continue))<br>
already-moved<br>
&nbsp;&nbsp;(assign&nbsp;new&nbsp;(op&nbsp;vector-ref)&nbsp;(reg&nbsp;the-cdrs)&nbsp;(reg&nbsp;old))<br>
&nbsp;&nbsp;(goto&nbsp;(reg&nbsp;relocate-continue))<br></tt>


At the very end of the garbage-collection process, we interchange the role of old and new memories by interchanging pointers: interchanging <tt>the-cars</tt> with <tt>new-cars</tt>, and <tt>the-cdrs</tt> with <tt>new-cdrs</tt>. We will then be ready to perform another garbage collection the next time memory runs out.


<tt>gc-flip<br>
&nbsp;&nbsp;(assign&nbsp;temp&nbsp;(reg&nbsp;the-cdrs))<br>
&nbsp;&nbsp;(assign&nbsp;the-cdrs&nbsp;(reg&nbsp;new-cdrs))<br>
&nbsp;&nbsp;(assign&nbsp;new-cdrs&nbsp;(reg&nbsp;temp))<br>
&nbsp;&nbsp;(assign&nbsp;temp&nbsp;(reg&nbsp;the-cars))<br>
&nbsp;&nbsp;(assign&nbsp;the-cars&nbsp;(reg&nbsp;new-cars))<br>
&nbsp;&nbsp;(assign&nbsp;new-cars&nbsp;(reg&nbsp;temp))<br></tt>

<div class="smallprint">
  <hr>
</div>

<div class="footnote">
  
<a name="footnote_Temp_744" href="#call_footnote_Temp_744" id="footnote_Temp_744"><sup><small>5</small></sup></a> We could represent memory as lists of items. However, the access time would then not be independent of the index, since accessing the *n*th element of a list requires *n* - 1 <tt>cdr</tt> operations.

  
<a name="footnote_Temp_745" href="#call_footnote_Temp_745" id="footnote_Temp_745"><sup><small>6</small></sup></a> For completeness, we should specify a <tt>make-vector</tt> operation that constructs vectors. However, in the present application we will use vectors only to model fixed divisions of the computer memory.

  
<a name="footnote_Temp_747" href="#call_footnote_Temp_747" id="footnote_Temp_747"><sup><small>7</small></sup></a> This is precisely the same <a name="index_term_5864"></a><a name="index_term_5866"></a>"tagged data" idea we introduced in chapter&nbsp;2 for dealing with generic operations. Here, however, the data types are included at the primitive machine level rather than constructed through the use of lists.

  
<a name="footnote_Temp_748" href="#call_footnote_Temp_748" id="footnote_Temp_748"><sup><small>8</small></sup></a> Type information may be encoded in a variety of ways, depending on the details of the machine on which the Lisp system is to be implemented. The execution efficiency of Lisp programs will be strongly dependent on how cleverly this choice is made, but it is difficult to formulate general design rules for good choices. The most straightforward way to implement typed pointers is to allocate a fixed set of bits in each pointer to be a <a name="index_term_5870"></a>*type field* that encodes the data type. Important questions to be addressed in designing such a representation include the following: How many type bits are required? How large must the vector indices be? How efficiently can the primitive machine instructions be used to manipulate the type fields of pointers? Machines that include special hardware for the efficient handling of type fields are said to have <a name="index_term_5872"></a>*tagged architectures*.

  
<a name="footnote_Temp_749" href="#call_footnote_Temp_749" id="footnote_Temp_749"><sup><small>9</small></sup></a> This decision on the <a name="index_term_5874"></a><a name="index_term_5876"></a><a name="index_term_5878"></a>representation of numbers determines whether <tt>eq?</tt>, which tests equality of pointers, can be used to test for equality of numbers. If the pointer contains the number itself, then equal numbers will have the same pointer. But if the pointer contains the index of a location where the number is stored, equal numbers will be guaranteed to have equal pointers only if we are careful never to store the same number in more than one location.

  
<a name="footnote_Temp_750" href="#call_footnote_Temp_750" id="footnote_Temp_750"><sup><small>10</small></sup></a> This is just like writing a number as a sequence of digits, except that each "digit" is a number between 0 and the largest number that can be stored in a single pointer.

  
<a name="footnote_Temp_752" href="#call_footnote_Temp_752" id="footnote_Temp_752"><sup><small>11</small></sup></a> There are other ways of finding free storage. For example, we could link together all the unused pairs into a <a name="index_term_5908"></a>*free list*. Our free locations are consecutive (and hence can be accessed by incrementing a pointer) because we are using a compacting garbage collector, as we will see in section <a href="#%_sec_5.3.2">5.3.2</a>.

  
<a name="footnote_Temp_753" href="#call_footnote_Temp_753" id="footnote_Temp_753"><sup><small>12</small></sup></a> This is essentially the implementation of <tt>cons</tt> in terms of <tt>set-car!</tt> and <tt>set-cdr!</tt>, as described in section <a href="chapter_3_section_3.html#%_sec_3.3.1">3.3.1</a>. The operation <tt>get-new-pair</tt> used in that implementation is realized here by the <tt>free</tt> pointer.

  
<a name="footnote_Temp_758" href="#call_footnote_Temp_758" id="footnote_Temp_758"><sup><small>13</small></sup></a> This may not be true eventually, because memories may get large enough so that it would be impossible to run out of free memory in the lifetime of the computer. For example, there are about 3� 10<sup>13</sup>, microseconds in a year, so if we were to <tt>cons</tt> once per microsecond we would need about 10<sup>15</sup> cells of memory to build a machine that could operate for 30 years without running out of memory. That much memory seems absurdly large by today's standards, but it is not physically impossible. On the other hand, processors are getting faster and a future computer may have large numbers of processors operating in parallel on a single memory, so it may be possible to use up memory much faster than we have postulated.

  
<a name="footnote_Temp_759" href="#call_footnote_Temp_759" id="footnote_Temp_759"><sup><small>14</small></sup></a> We assume here that the stack is represented as a list as described in section <a href="#%_sec_5.3.1">5.3.1</a>, so that items on the stack are accessible via the pointer in the stack register.

  
<a name="footnote_Temp_760" href="#call_footnote_Temp_760" id="footnote_Temp_760"><sup><small>15</small></sup></a> This idea was invented and first implemented <a name="index_term_5936"></a>by Minsky, as part of the implementation of <a name="index_term_5938"></a>Lisp for the PDP-1 at the <a name="index_term_5940"></a>MIT Research Laboratory of Electronics. It was further developed by <a name="index_term_5942"></a><a name="index_term_5944"></a>Fenichel and Yochelson (1969) for use in the Lisp implementation for <a name="index_term_5946"></a>the Multics time-sharing system. Later, <a name="index_term_5948"></a>Baker (1978) developed a "real-time" version of the method, which does not require the computation to stop during garbage collection. Baker's idea was extended by <a name="index_term_5950"></a><a name="index_term_5952"></a><a name="index_term_5954"></a>Hewitt, Lieberman, and Moon (see Lieberman and Hewitt 1983) to take advantage of the fact that some structure is more volatile and other structure is more permanent.

  
An alternative commonly used garbage-collection technique is the <a name="index_term_5956"></a><a name="index_term_5958"></a>*mark-sweep* method. This consists of tracing all the structure accessible from the machine registers and marking each pair we reach. We then scan all of memory, and any location that is unmarked is "swept up" as garbage and made available for reuse. A full <a name="index_term_5960"></a>discussion of the mark-sweep method can be found in Allen 1978.

  
The Minsky-Fenichel-Yochelson algorithm is the dominant algorithm in use for large-memory systems because it examines only the useful part of memory. This is in contrast to mark-sweep, in which the sweep phase must check all of memory. A second advantage of stop-and-copy is that it is a <a name="index_term_5962"></a><a name="index_term_5964"></a>*compacting* garbage collector. That is, at the end of the garbage-collection phase the useful data will have been moved to consecutive memory locations, with all garbage pairs compressed out. This can be an extremely important performance consideration in machines with virtual memory, in which accesses to widely separated memory addresses may require extra paging operations.

  
<a name="footnote_Temp_762" href="#call_footnote_Temp_762" id="footnote_Temp_762"><sup><small>16</small></sup></a> This list of registers does not include the registers used by the storage-allocation system -- <tt>root</tt>, <tt>the-cars</tt>, <tt>the-cdrs</tt>, and the other registers that will be introduced in this section.

  
<a name="footnote_Temp_763" href="#call_footnote_Temp_763" id="footnote_Temp_763"><sup><small>17</small></sup></a> The term *<a name="index_term_5982"></a>broken heart* was coined by David Cressey, who wrote a garbage collector for <a name="index_term_5984"></a><a name="index_term_5986"></a>MDL, a dialect of Lisp developed at MIT during the early 1970s.

  
<a name="footnote_Temp_764" href="#call_footnote_Temp_764" id="footnote_Temp_764"><sup><small>18</small></sup></a> The garbage collector uses the low-level predicate <tt>pointer-to-pair?</tt> instead of the list-structure <tt>pair?</tt> operation because in a real system there might be various things that are treated as pairs for garbage-collection purposes. For example, in a Scheme system that conforms to the IEEE standard a procedure object may be implemented as a special kind of "pair" that doesn't satisfy the <tt>pair?</tt> predicate. For simulation purposes, <tt>pointer-to-pair?</tt> can be implemented as <tt>pair?</tt>.

</div>