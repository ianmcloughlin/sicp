(section_51)=
# Designing Register Machines

<a name="index_term_5462"></a><a name="index_term_5464"></a><a name="index_term_5466"></a><a name="index_term_5468"></a><a name="index_term_5470"></a><a name="index_term_5472"></a> To design a register machine, we must design its *data paths* (registers and operations) and the *controller* that sequences these operations. To illustrate the design of a simple register machine, let us examine Euclid's Algorithm, which is used to compute <a name="index_term_5474"></a>the greatest common divisor (GCD) of two integers. As we saw in <a name="index_term_5476"></a>section <a href="chapter_1_section_2.html#%_sec_1.2.5">1.2.5</a>, Euclid's Algorithm can be carried out by an iterative process, as specified by the following procedure:


<tt>(define&nbsp;(gcd&nbsp;a&nbsp;b)<br>
&nbsp;&nbsp;(if&nbsp;(=&nbsp;b&nbsp;0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(gcd&nbsp;b&nbsp;(remainder&nbsp;a&nbsp;b))))<br></tt>


A machine to carry out this algorithm must keep track of two numbers, *a* and *b*, so let us assume that these numbers are stored in two registers with those names. The basic operations required are testing whether the contents of register <tt>b</tt> is zero and computing the remainder of the contents of register <tt>a</tt> divided by the contents of register <tt>b</tt>. The remainder operation is a complex process, but assume for the moment that we have a primitive device that computes remainders. On each cycle of the GCD algorithm, the contents of register <tt>a</tt> must be replaced by the contents of register <tt>b</tt>, and the contents of <tt>b</tt> must be replaced by the remainder of the old contents of <tt>a</tt> divided by the old contents of <tt>b</tt>. It would be convenient if these replacements could be done simultaneously, but in our model of register machines we will assume that only one register can be assigned a new value at each step. To accomplish the replacements, our machine will use a third "temporary" register, which we call <tt>t</tt>. (First the remainder will be placed in <tt>t</tt>, then the contents of <tt>b</tt> will be placed in <tt>a</tt>, and finally the remainder stored in <tt>t</tt> will be placed in <tt>b</tt>.)


<a name="index_term_5478"></a><a name="index_term_5480"></a>We can illustrate the registers and operations required for this machine by using the data-path diagram shown in figure&nbsp;<a href="#%_fig_5.1">5.1</a>. In this diagram, the registers (<tt>a</tt>, <tt>b</tt>, and <tt>t</tt>) are represented by rectangles. Each way to assign a value to a register is indicated by an arrow with an <tt>X</tt> behind the head, pointing from the source of data to the register. We can think of the <tt>X</tt> as a button that, when pushed, allows the value at the source to "flow" into the designated register. The label next to each button is the name we will use to refer to the button. The names are arbitrary, and can be chosen to have mnemonic value (for example, <tt>a&lt;-b</tt> denotes pushing the button that assigns the contents of register <tt>b</tt> to register <tt>a</tt>). The source of data for a register can be another register (as in the <tt>a&lt;-b</tt> assignment), an operation result (as in the <tt>t&lt;-r</tt> assignment), or a constant (a built-in value that cannot be changed, represented in a data-path diagram by a triangle containing the constant).


An operation that computes a value from constants and the contents of registers is represented in a data-path diagram by a trapezoid containing a name for the operation. For example, the box marked <tt>rem</tt> in figure&nbsp;<a href="#%_fig_5.1">5.1</a> represents an operation that computes the remainder of the contents of the registers <tt>a</tt> and <tt>b</tt> to which it is attached. Arrows (without buttons) point from the input registers and constants to the box, and arrows connect the operation's output value to registers. A test is represented by a circle containing a name for the test. For example, our GCD machine has an operation that tests whether the contents of register <tt>b</tt> is zero. A test also has arrows from its input <a name="index_term_5482"></a><a name="index_term_5484"></a>registers and constants, but it has no output arrows; its value is used by the controller rather than by the data paths. Overall, the data-path diagram shows the registers and operations that are required for the machine and how they must be connected. If we view the arrows as wires and the <tt>X</tt> buttons as switches, the data-path diagram is very like the wiring diagram for a machine that could be constructed from electrical components.


<a name="%_fig_5.1"></a>

<div align="left">
  <div align="left">
    **Figure 5.1:**&nbsp;&nbsp;Data paths for a GCD machine.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_5_image_01.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="index_term_5486"></a><a name="index_term_5488"></a>In order for the data paths to actually compute GCDs, the buttons must be pushed in the correct sequence. We will describe this sequence in terms of a controller diagram, as illustrated in figure&nbsp;<a href="#%_fig_5.2">5.2</a>. The elements of the controller diagram indicate how the data-path components should be operated. The rectangular boxes in the controller diagram identify data-path buttons to be pushed, and the arrows describe the sequencing from one step to the next. The diamond in the diagram represents a decision. One of the two sequencing arrows will be followed, depending on the value of the data-path test identified in the diamond. We can interpret the controller in terms of a physical analogy: Think of the diagram as a maze in which a marble is rolling. When the marble rolls into a box, it pushes the data-path button that is named by the box. When the marble rolls into a decision node (such as the test for <tt>b</tt> = 0), it leaves the node on the path determined by the result of the indicated test. Taken together, the data paths and the controller completely describe a machine for computing GCDs. We start the controller (the rolling marble) at the place marked <tt>start</tt>, after placing numbers in registers <tt>a</tt> and <tt>b</tt>. When the controller reaches <tt>done</tt>, we will find the value of the GCD in register <tt>a</tt>. <a name="%_fig_5.2"></a>

<div align="left">
  <div align="left">
    **Figure 5.2:**&nbsp;&nbsp;Controller for a GCD machine.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_5_image_02.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="exercise_5_1">**Exercise 5.1**</a> <a name="index_term_5490"></a>Design a register machine to compute factorials using the iterative algorithm specified by the following procedure. Draw data-path and controller diagrams for this machine.


<tt>(define&nbsp;(factorial&nbsp;n)<br>
&nbsp;&nbsp;(define&nbsp;(iter&nbsp;product&nbsp;counter)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;(&gt;&nbsp;counter&nbsp;n)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;product<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(iter&nbsp;(*&nbsp;counter&nbsp;product)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(+&nbsp;counter&nbsp;1))))<br>
&nbsp;&nbsp;(iter&nbsp;1&nbsp;1))<br></tt>


<a name="%_sec_5.1.1"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_5.1.1">5.1.1&nbsp;&nbsp;A Language for Describing Register Machines</a></h3>

<a name="index_term_5492"></a> Data-path and controller diagrams are adequate for representing simple machines such as GCD, but they are unwieldy for describing large machines such as a Lisp interpreter. To make it possible to deal with complex machines, we will create a language that presents, in textual form, all the information given by the data-path and controller diagrams. We will start with a notation that directly mirrors the diagrams.


We define the data paths of a machine by describing the registers and the operations. To describe a register, we give it a name and specify the buttons that control assignment to it. We give each of these buttons a name and specify the source of the data that enters the register under the button's control. (The source is a register, a constant, or an operation.) To describe an operation, we give it a name and specify its inputs (registers or constants).


We define the controller of a machine as a sequence of <a name="index_term_5494"></a>*instructions* together with <a name="index_term_5496"></a><a name="index_term_5498"></a>*labels* that identify *entry points* in the sequence. An instruction is one of the following:

<ul>
  <li>The name of a data-path button to push to assign a value to a register. (This corresponds to a box in the controller diagram.)

    
<a name="index_term_5500"></a><a name="index_term_5502"></a>

  </li>

  <li>A <tt>test</tt> instruction, that performs a specified test.

    
<a name="index_term_5504"></a><a name="index_term_5506"></a><a name="index_term_5508"></a><a name="index_term_5510"></a>

  </li>

  <li>A conditional branch (<tt>branch</tt> instruction) to a location indicated by a controller label, based on the result of the previous test. (The test and branch together correspond to a diamond in the controller diagram.) If the test is false, the controller should continue with the next instruction in the sequence. Otherwise, the controller should continue with the instruction after the label.

    
<a name="index_term_5512"></a><a name="index_term_5514"></a>

  </li>

  <li>An unconditional branch (<tt>goto</tt> instruction) naming a controller label at which to continue execution.</li>
</ul>

The machine starts at the beginning of the controller instruction sequence and stops when execution reaches the end of the sequence. Except when a branch changes the flow of control, instructions are executed in the order in which they are listed.


<a name="%_fig_5.3"></a>

<div align="left">
  <div align="left">
    **Figure 5.3:**&nbsp;&nbsp;A specification of the GCD machine.
  </div>

  <table width="100%">
    <tr>
      <td>
        
<tt>(data-paths<br>
        &nbsp;(registers<br>
        &nbsp;&nbsp;((name&nbsp;a)<br>
        &nbsp;&nbsp;&nbsp;(buttons&nbsp;((name&nbsp;a&lt;-b)&nbsp;(source&nbsp;(register&nbsp;b)))))<br>
        &nbsp;&nbsp;((name&nbsp;b)<br>
        &nbsp;&nbsp;&nbsp;(buttons&nbsp;((name&nbsp;b&lt;-t)&nbsp;(source&nbsp;(register&nbsp;t)))))<br>
        &nbsp;&nbsp;((name&nbsp;t)<br>
        &nbsp;&nbsp;&nbsp;(buttons&nbsp;((name&nbsp;t&lt;-r)&nbsp;(source&nbsp;(operation&nbsp;rem))))))<br>
        <br>
        &nbsp;(operations<br>
        &nbsp;&nbsp;((name&nbsp;rem)<br>
        &nbsp;&nbsp;&nbsp;(inputs&nbsp;(register&nbsp;a)&nbsp;(register&nbsp;b)))<br>
        &nbsp;&nbsp;((name&nbsp;=)<br>
        &nbsp;&nbsp;&nbsp;(inputs&nbsp;(register&nbsp;b)&nbsp;(constant&nbsp;0)))))<br>
        <br>
        (controller<br>
        &nbsp;test-b&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;label*<br>
        &nbsp;&nbsp;&nbsp;(test&nbsp;=)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;test*<br>
        &nbsp;&nbsp;&nbsp;(branch&nbsp;(label&nbsp;gcd-done))&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;conditional&nbsp;branch*<br>
        &nbsp;&nbsp;&nbsp;(t&lt;-r)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;button&nbsp;push*<br>
        &nbsp;&nbsp;&nbsp;(a&lt;-b)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;button&nbsp;push*<br>
        &nbsp;&nbsp;&nbsp;(b&lt;-t)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;button&nbsp;push*<br>
        &nbsp;&nbsp;&nbsp;(goto&nbsp;(label&nbsp;test-b))&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;unconditional&nbsp;branch*<br>
        &nbsp;gcd-done)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;label*<br></tt>

      </td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

Figure&nbsp;<a href="#%_fig_5.3">5.3</a> shows the GCD machine described in this way. This example only hints at the generality of these descriptions, since the GCD machine is a very simple case: Each register has only one button, and each button and test is used only once in the controller.


Unfortunately, it is difficult to read such a description. In order to understand the controller instructions we must constantly refer back to the definitions of the button names and the operation names, and to understand what the buttons do we may have to refer to the definitions of the operation names. We will thus transform our notation to combine the information from the data-path and controller descriptions so that we see it all together.


To obtain this form of description, we will replace the arbitrary button and operation names by the definitions of their behavior. That is, instead of saying (in the controller) ``Push button <tt>t&lt;-r</tt>" and separately saying (in the data paths) ``Button <tt>t&lt;-r</tt> assigns the value of the <tt>rem</tt> operation to register <tt>t</tt>" and ``The <tt>rem</tt> operation's inputs are the contents of registers <a name="index_term_5516"></a><a name="index_term_5518"></a><a name="index_term_5520"></a><a name="index_term_5522"></a><a name="index_term_5524"></a><a name="index_term_5526"></a><tt>a</tt> and <tt>b</tt>," we will say (in the controller) "Push the button that assigns to register <tt>t</tt> the value of the <tt>rem</tt> operation on the contents of registers <tt>a</tt> and <tt>b</tt>." Similarly, instead of saying (in the controller) "Perform the <tt>=</tt> test" and separately saying (in the data paths) "The <tt>=</tt> test operates on the contents of register <tt>b</tt> and the constant 0," we will say "Perform the <tt>=</tt> test on the <a name="index_term_5528"></a><a name="index_term_5530"></a>contents of register <tt>b</tt> and the constant 0." We will omit the data-path description, leaving only the controller sequence. Thus, the GCD machine is described as follows:


<tt>(controller<br>
&nbsp;test-b<br>
&nbsp;&nbsp;&nbsp;(test&nbsp;(op&nbsp;=)&nbsp;(reg&nbsp;b)&nbsp;(const&nbsp;0))<br>
&nbsp;&nbsp;&nbsp;(branch&nbsp;(label&nbsp;gcd-done))<br>
&nbsp;&nbsp;&nbsp;(assign&nbsp;t&nbsp;(op&nbsp;rem)&nbsp;(reg&nbsp;a)&nbsp;(reg&nbsp;b))<br>
&nbsp;&nbsp;&nbsp;(assign&nbsp;a&nbsp;(reg&nbsp;b))<br>
&nbsp;&nbsp;&nbsp;(assign&nbsp;b&nbsp;(reg&nbsp;t))<br>
&nbsp;&nbsp;&nbsp;(goto&nbsp;(label&nbsp;test-b))<br>
&nbsp;gcd-done)<br></tt>


This form of description is easier to read than the kind illustrated in figure&nbsp;<a href="#%_fig_5.3">5.3</a>, but it also has disadvantages:

<ul>
  <li>It is more verbose for large machines, because complete descriptions of the data-path elements are repeated whenever the elements are mentioned in the controller instruction sequence. (This is not a problem in the GCD example, because each operation and button is used only once.) Moreover, repeating the data-path descriptions obscures the actual data-path structure of the machine; it is not obvious for a large machine how many registers, operations, and buttons there are and how they are interconnected.</li>

  <li>Because the controller instructions in a machine definition look like Lisp expressions, it is easy to forget that they are not arbitrary Lisp expressions. They can notate only legal machine operations. For example, operations can operate directly only on constants and the contents of registers, not on the results of other operations.</li>
</ul>

In spite of these disadvantages, we will use this register-machine language throughout this chapter, because we will be more concerned with understanding controllers than with understanding the elements and connections in data paths. We should keep in mind, however, that data-path design is crucial in designing real machines.


<a name="exercise_5_2">**Exercise 5.2**</a> <a name="index_term_5532"></a>Use the register-machine language to describe the iterative factorial machine of exercise&nbsp;<a href="#%_thm_5.1">5.1</a>.


<a name="%_sec_Temp_714"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_714">Actions</a></h4>

<a name="index_term_5534"></a><a name="index_term_5536"></a> Let us modify the GCD machine so that we can type in the numbers whose GCD we want and get the answer printed at our terminal. We will not discuss how to make a machine that can read and print, but will assume (as we do when we use <tt>read</tt> and <tt>display</tt> in Scheme) that they are available as primitive operations.<a name="call_footnote_Temp_715" href="#footnote_Temp_715" id="call_footnote_Temp_715"><sup><small>1</small></sup></a>


<a name="index_term_5538"></a><tt>Read</tt> is like the operations we have been using in that it produces a value that can be stored in a register. But <tt>read</tt> does not take inputs from any registers; its value depends on something that happens outside the parts of the machine we are designing. We will allow our machine's operations to have such behavior, and thus will draw and notate the use of <tt>read</tt> just as we do any other operation that computes a value.


<a name="index_term_5540"></a><tt>Print</tt>, on the other hand, differs from the operations we have been using in a fundamental way: It does not produce an output value to be stored in a register. Though it has an effect, this effect is not on a part of the machine we are designing. We will refer to this kind of operation as an *action*. We will represent an action in a data-path diagram just as we represent an operation that computes a value -- as a trapezoid that contains the name of the action. Arrows point to the action box from any inputs (registers or constants). We also associate a button with the action. Pushing the button makes the action happen. To make a controller push an action <a name="index_term_5542"></a><a name="index_term_5544"></a>button we use a new kind of instruction called <tt>perform</tt>. Thus, the action of printing the contents of register <tt>a</tt> is represented in a controller sequence by the instruction


<tt>(perform&nbsp;(op&nbsp;print)&nbsp;(reg&nbsp;a))<br></tt>


Figure&nbsp;<a href="#%_fig_5.4">5.4</a> shows the data paths and controller for the new GCD machine. Instead of having the machine stop after printing the answer, we have made it start over, so that it repeatedly reads a pair of numbers, computes their GCD, and prints the result. This structure is like the driver loops we used in the interpreters of chapter&nbsp;4.


<a name="%_fig_5.4"></a>

<div align="left">
  <div align="left">
    **Figure 5.4:**&nbsp;&nbsp;A GCD machine that reads inputs and prints results.
  </div>

  <table width="100%">
    <tr>
      <td>
        <img src="img/chapter_5_image_03.gif" border="0">

        
<tt>&nbsp;(controller<br>
        &nbsp;&nbsp;gcd-loop<br>
        &nbsp;&nbsp;&nbsp;&nbsp;(assign&nbsp;a&nbsp;(op&nbsp;read))<br>
        &nbsp;&nbsp;&nbsp;&nbsp;(assign&nbsp;b&nbsp;(op&nbsp;read))<br>
        &nbsp;&nbsp;test-b<br>
        &nbsp;&nbsp;&nbsp;&nbsp;(test&nbsp;(op&nbsp;=)&nbsp;(reg&nbsp;b)&nbsp;(const&nbsp;0))<br>
        &nbsp;&nbsp;&nbsp;&nbsp;(branch&nbsp;(label&nbsp;gcd-done))<br>
        &nbsp;&nbsp;&nbsp;&nbsp;(assign&nbsp;t&nbsp;(op&nbsp;rem)&nbsp;(reg&nbsp;a)&nbsp;(reg&nbsp;b))<br>
        &nbsp;&nbsp;&nbsp;&nbsp;(assign&nbsp;a&nbsp;(reg&nbsp;b))<br>
        &nbsp;&nbsp;&nbsp;&nbsp;(assign&nbsp;b&nbsp;(reg&nbsp;t))<br>
        &nbsp;&nbsp;&nbsp;&nbsp;(goto&nbsp;(label&nbsp;test-b))<br>
        &nbsp;&nbsp;gcd-done<br>
        &nbsp;&nbsp;&nbsp;&nbsp;(perform&nbsp;(op&nbsp;print)&nbsp;(reg&nbsp;a))<br>
        &nbsp;&nbsp;&nbsp;&nbsp;(goto&nbsp;(label&nbsp;gcd-loop)))<br></tt>

      </td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="%_sec_5.1.2"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_5.1.2">5.1.2&nbsp;&nbsp;Abstraction in Machine Design</a></h3>

<a name="index_term_5546"></a> We will often define a machine to include "primitive" operations that are actually very complex. For example, in sections&nbsp;<a href="chapter_5_section_4.html#%_sec_5.4">5.4</a> and <a href="chapter_5_section_5.html#%_sec_5.5">5.5</a> we will treat Scheme's environment manipulations as primitive. Such abstraction is valuable because it allows us to ignore the details of parts of a machine so that we can concentrate on other aspects of the design. The fact that we have swept a lot of complexity under the rug, however, does not mean that a machine design is unrealistic. We can always replace the complex "primitives" by simpler primitive operations.


Consider the GCD machine. The machine has an instruction that computes the remainder of the contents of registers <tt>a</tt> and <tt>b</tt> and assigns the result to register <tt>t</tt>. If we want to construct the GCD machine without using a primitive remainder operation, we must specify how to compute remainders in terms of simpler operations, such as subtraction. Indeed, we can write a Scheme procedure that finds remainders in this way:


<tt>(define&nbsp;(remainder&nbsp;n&nbsp;d)<br>
&nbsp;&nbsp;(if&nbsp;(&lt;&nbsp;n&nbsp;d)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;n<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(remainder&nbsp;(-&nbsp;n&nbsp;d)&nbsp;d)))<br></tt>


We can thus replace the remainder operation in the GCD machine's data paths with a subtraction operation and a comparison test. Figure&nbsp;<a href="#%_fig_5.5">5.5</a> shows the data paths and controller for the elaborated machine. The instruction


<a name="%_fig_5.5"></a>

<div align="left">
  <div align="left">
    **Figure 5.5:**&nbsp;&nbsp;Data paths and controller for the elaborated GCD machine.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_5_image_04.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<tt>(assign&nbsp;t&nbsp;(op&nbsp;rem)&nbsp;(reg&nbsp;a)&nbsp;(reg&nbsp;b))<br></tt>


in the GCD controller definition is replaced by a sequence of instructions that contains a loop, as shown in figure&nbsp;<a href="#%_fig_5.6">5.6</a>.


<a name="%_fig_5.6"></a>

<div align="left">
  <div align="left">
    **Figure 5.6:**&nbsp;&nbsp;Controller instruction sequence for the GCD machine in figure&nbsp;<a href="#%_fig_5.5">5.5</a>.
  </div>

  <table width="100%">
    <tr>
      <td>
        
<tt>(controller<br>
        &nbsp;test-b<br>
        &nbsp;&nbsp;&nbsp;(test&nbsp;(op&nbsp;=)&nbsp;(reg&nbsp;b)&nbsp;(const&nbsp;0))<br>
        &nbsp;&nbsp;&nbsp;(branch&nbsp;(label&nbsp;gcd-done))<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;t&nbsp;(reg&nbsp;a))<br>
        &nbsp;rem-loop<br>
        &nbsp;&nbsp;&nbsp;(test&nbsp;(op&nbsp;&lt;)&nbsp;(reg&nbsp;t)&nbsp;(reg&nbsp;b))<br>
        &nbsp;&nbsp;&nbsp;(branch&nbsp;(label&nbsp;rem-done))<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;t&nbsp;(op&nbsp;-)&nbsp;(reg&nbsp;t)&nbsp;(reg&nbsp;b))<br>
        &nbsp;&nbsp;&nbsp;(goto&nbsp;(label&nbsp;rem-loop))<br>
        &nbsp;rem-done<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;a&nbsp;(reg&nbsp;b))<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;b&nbsp;(reg&nbsp;t))<br>
        &nbsp;&nbsp;&nbsp;(goto&nbsp;(label&nbsp;test-b))<br>
        &nbsp;gcd-done)<br></tt>

      </td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="exercise_5_3">**Exercise 5.3**</a> <a name="index_term_5548"></a>Design a machine to compute square roots using Newton's method, as described in section <a href="chapter_1_section_1.html#%_sec_1.1.7">1.1.7</a>:


<tt>(define&nbsp;(sqrt&nbsp;x)<br>
&nbsp;&nbsp;(define&nbsp;(good-enough?&nbsp;guess)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(&lt;&nbsp;(abs&nbsp;(-&nbsp;(square&nbsp;guess)&nbsp;x))&nbsp;0.001))<br>
&nbsp;&nbsp;(define&nbsp;(improve&nbsp;guess)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(average&nbsp;guess&nbsp;(/&nbsp;x&nbsp;guess)))<br>
&nbsp;&nbsp;(define&nbsp;(sqrt-iter&nbsp;guess)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;(good-enough?&nbsp;guess)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;guess<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(sqrt-iter&nbsp;(improve&nbsp;guess))))<br>
&nbsp;&nbsp;(sqrt-iter&nbsp;1.0))<br></tt>


Begin by assuming that <tt>good-enough?</tt> and <tt>improve</tt> operations are available as primitives. Then show how to expand these in terms of arithmetic operations. Describe each version of the <tt>sqrt</tt> machine design by drawing a data-path diagram and writing a controller definition in the register-machine language.


<a name="%_sec_5.1.3"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_5.1.3">5.1.3&nbsp;&nbsp;Subroutines</a></h3>

<a name="index_term_5550"></a><a name="index_term_5552"></a> When designing a machine to perform a computation, we would often prefer to arrange for components to be shared by different parts of the computation rather than duplicate the components. Consider a machine that includes two GCD computations -- one that finds the GCD of the contents of registers <tt>a</tt> and <tt>b</tt> and one that finds the GCD of the contents of registers <tt>c</tt> and <tt>d</tt>. We might start by assuming we have a primitive <tt>gcd</tt> operation, then expand the two instances of <tt>gcd</tt> in terms of more primitive operations. Figure&nbsp;<a href="#%_fig_5.7">5.7</a> shows just the GCD portions of the resulting machine's data paths, without showing how they connect to the rest of the machine. The figure also shows the corresponding portions of the machine's controller sequence.


<a name="%_fig_5.7"></a>

<div align="left">
  <div align="left">
    **Figure 5.7:**&nbsp;&nbsp;Portions of the data paths and controller sequence for a machine with two GCD computations.
  </div>

  <table width="100%">
    <tr>
      <td>
        <img src="img/chapter_5_image_05.gif" border="0">

        
<tt>gcd-1<br>
        &nbsp;(test&nbsp;(op&nbsp;=)&nbsp;(reg&nbsp;b)&nbsp;(const&nbsp;0))<br>
        &nbsp;(branch&nbsp;(label&nbsp;after-gcd-1))<br>
        &nbsp;(assign&nbsp;t&nbsp;(op&nbsp;rem)&nbsp;(reg&nbsp;a)&nbsp;(reg&nbsp;b))<br>
        &nbsp;(assign&nbsp;a&nbsp;(reg&nbsp;b))<br>
        &nbsp;(assign&nbsp;b&nbsp;(reg&nbsp;t))<br>
        &nbsp;(goto&nbsp;(label&nbsp;gcd-1))<br>
        after-gcd-1<br>
        &nbsp;&nbsp;&nbsp;<img src="img/book_18.gif" border="0">&nbsp;<br>
        gcd-2<br>
        &nbsp;(test&nbsp;(op&nbsp;=)&nbsp;(reg&nbsp;d)&nbsp;(const&nbsp;0))<br>
        &nbsp;(branch&nbsp;(label&nbsp;after-gcd-2))<br>
        &nbsp;(assign&nbsp;s&nbsp;(op&nbsp;rem)&nbsp;(reg&nbsp;c)&nbsp;(reg&nbsp;d))<br>
        &nbsp;(assign&nbsp;c&nbsp;(reg&nbsp;d))<br>
        &nbsp;(assign&nbsp;d&nbsp;(reg&nbsp;s))<br>
        &nbsp;(goto&nbsp;(label&nbsp;gcd-2))<br>
        after-gcd-2<br></tt>

      </td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

This machine has two remainder operation boxes and two boxes for testing equality. If the duplicated components are complicated, as is the remainder box, this will not be an economical way to build the machine. We can avoid duplicating the data-path components by using the same components for both GCD computations, provided that doing so will not affect the rest of the larger machine's computation. If the values in registers <tt>a</tt> and <tt>b</tt> are not needed by the time the controller gets to <tt>gcd-2</tt> (or if these values can be moved to other registers for safekeeping), we can change the machine so that it uses registers <tt>a</tt> and <tt>b</tt>, rather than registers <tt>c</tt> and <tt>d</tt>, in computing the second GCD as well as the first. If we do this, we obtain the controller sequence shown in figure&nbsp;<a href="#%_fig_5.8">5.8</a>.


We have removed the duplicate data-path components (so that the data paths are again as in figure&nbsp;<a href="#%_fig_5.1">5.1</a>), but the controller now has two GCD sequences that differ only in their entry-point labels. It would be better to replace these two sequences by branches to a single sequence -- a <tt>gcd</tt> *subroutine* -- at the end of which we branch back to the correct place in the main instruction sequence. We can accomplish this as follows: Before branching to <tt>gcd</tt>, we place a distinguishing value (such as 0 or&nbsp;1) into a special register, <a name="index_term_5554"></a><tt>continue</tt>. At the end of the <tt>gcd</tt> subroutine we return either to <tt>after-gcd-1</tt> or to <tt>after-gcd-2</tt>, depending on the value of the <tt>continue</tt> register. Figure&nbsp;<a href="#%_fig_5.9">5.9</a> shows the relevant portion of the resulting controller sequence, which includes only a single copy of the <tt>gcd</tt> instructions.


<a name="%_fig_5.8"></a>

<div align="left">
  <div align="left">
    **Figure 5.8:**&nbsp;&nbsp;Portions of the controller sequence for a machine that uses the same data-path components for two different GCD computations.
  </div>

  <table width="100%">
    <tr>
      <td>
        
<tt>gcd-1<br>
        &nbsp;(test&nbsp;(op&nbsp;=)&nbsp;(reg&nbsp;b)&nbsp;(const&nbsp;0))<br>
        &nbsp;(branch&nbsp;(label&nbsp;after-gcd-1))<br>
        &nbsp;(assign&nbsp;t&nbsp;(op&nbsp;rem)&nbsp;(reg&nbsp;a)&nbsp;(reg&nbsp;b))<br>
        &nbsp;(assign&nbsp;a&nbsp;(reg&nbsp;b))<br>
        &nbsp;(assign&nbsp;b&nbsp;(reg&nbsp;t))<br>
        &nbsp;(goto&nbsp;(label&nbsp;gcd-1))<br>
        after-gcd-1<br>
        &nbsp;&nbsp;<img src="img/book_18.gif" border="0"><br>
        gcd-2<br>
        &nbsp;(test&nbsp;(op&nbsp;=)&nbsp;(reg&nbsp;b)&nbsp;(const&nbsp;0))<br>
        &nbsp;(branch&nbsp;(label&nbsp;after-gcd-2))<br>
        &nbsp;(assign&nbsp;t&nbsp;(op&nbsp;rem)&nbsp;(reg&nbsp;a)&nbsp;(reg&nbsp;b))<br>
        &nbsp;(assign&nbsp;a&nbsp;(reg&nbsp;b))<br>
        &nbsp;(assign&nbsp;b&nbsp;(reg&nbsp;t))<br>
        &nbsp;(goto&nbsp;(label&nbsp;gcd-2))<br>
        after-gcd-2<br></tt>

      </td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="%_fig_5.9"></a>

<div align="left">
  <div align="left">
    **Figure 5.9:**&nbsp;&nbsp;Using a <tt>continue</tt> register to avoid the duplicate controller sequence in figure&nbsp;<a href="#%_fig_5.8">5.8</a>.
  </div>

  <table width="100%">
    <tr>
      <td>
        
<tt>gcd<br>
        &nbsp;(test&nbsp;(op&nbsp;=)&nbsp;(reg&nbsp;b)&nbsp;(const&nbsp;0))<br>
        &nbsp;(branch&nbsp;(label&nbsp;gcd-done))<br>
        &nbsp;(assign&nbsp;t&nbsp;(op&nbsp;rem)&nbsp;(reg&nbsp;a)&nbsp;(reg&nbsp;b))<br>
        &nbsp;(assign&nbsp;a&nbsp;(reg&nbsp;b))<br>
        &nbsp;(assign&nbsp;b&nbsp;(reg&nbsp;t))<br>
        &nbsp;(goto&nbsp;(label&nbsp;gcd))<br>
        gcd-done<br>
        &nbsp;(test&nbsp;(op&nbsp;=)&nbsp;(reg&nbsp;continue)&nbsp;(const&nbsp;0))&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>
        &nbsp;(branch&nbsp;(label&nbsp;after-gcd-1))<br>
        &nbsp;(goto&nbsp;(label&nbsp;after-gcd-2))<br>
        &nbsp;&nbsp;<img src="img/book_18.gif" border="0"><br>
        *;;&nbsp;Before&nbsp;branching&nbsp;to&nbsp;<tt>gcd</tt>&nbsp;from&nbsp;the&nbsp;first&nbsp;place&nbsp;where*<br>
        *;;&nbsp;it&nbsp;is&nbsp;needed,&nbsp;we&nbsp;place&nbsp;0&nbsp;in&nbsp;the&nbsp;<tt>continue</tt>&nbsp;register*<br>
        &nbsp;(assign&nbsp;continue&nbsp;(const&nbsp;0))<br>
        &nbsp;(goto&nbsp;(label&nbsp;gcd))<br>
        after-gcd-1<br>
        &nbsp;&nbsp;<img src="img/book_18.gif" border="0"><br>
        *;;&nbsp;Before&nbsp;the&nbsp;second&nbsp;use&nbsp;of&nbsp;<tt>gcd</tt>,&nbsp;we&nbsp;place&nbsp;1&nbsp;in&nbsp;the&nbsp;<tt>continue</tt>&nbsp;register*<br>
        &nbsp;(assign&nbsp;continue&nbsp;(const&nbsp;1))<br>
        &nbsp;(goto&nbsp;(label&nbsp;gcd))<br>
        after-gcd-2<br></tt>

      </td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="%_fig_5.10"></a>

<div align="left">
  <div align="left">
    **Figure 5.10:**&nbsp;&nbsp;Assigning labels to the <tt>continue</tt> register simplifies and generalizes the strategy shown in figure&nbsp;<a href="#%_fig_5.9">5.9</a>.
  </div>

  <table width="100%">
    <tr>
      <td>
        
<tt>gcd<br>
        &nbsp;(test&nbsp;(op&nbsp;=)&nbsp;(reg&nbsp;b)&nbsp;(const&nbsp;0))<br>
        &nbsp;(branch&nbsp;(label&nbsp;gcd-done))<br>
        &nbsp;(assign&nbsp;t&nbsp;(op&nbsp;rem)&nbsp;(reg&nbsp;a)&nbsp;(reg&nbsp;b))<br>
        &nbsp;(assign&nbsp;a&nbsp;(reg&nbsp;b))<br>
        &nbsp;(assign&nbsp;b&nbsp;(reg&nbsp;t))<br>
        &nbsp;(goto&nbsp;(label&nbsp;gcd))<br>
        gcd-done<br>
        &nbsp;(goto&nbsp;(reg&nbsp;continue))<br>
        &nbsp;&nbsp;&nbsp;<img src="img/book_18.gif" border="0"><br>
        *;;&nbsp;Before&nbsp;calling&nbsp;<tt>gcd</tt>,&nbsp;we&nbsp;assign&nbsp;to&nbsp;<tt>continue</tt>*<br>
        *;;&nbsp;the&nbsp;label&nbsp;to&nbsp;which&nbsp;<tt>gcd</tt>&nbsp;should&nbsp;return.*<br>
        &nbsp;(assign&nbsp;continue&nbsp;(label&nbsp;after-gcd-1))<br>
        &nbsp;(goto&nbsp;(label&nbsp;gcd))<br>
        after-gcd-1<br>
        &nbsp;&nbsp;&nbsp;<img src="img/book_18.gif" border="0"><br>
        *;;&nbsp;Here&nbsp;is&nbsp;the&nbsp;second&nbsp;call&nbsp;to&nbsp;<tt>gcd</tt>,&nbsp;with&nbsp;a&nbsp;different&nbsp;continuation.*<br>
        &nbsp;(assign&nbsp;continue&nbsp;(label&nbsp;after-gcd-2))<br>
        &nbsp;(goto&nbsp;(label&nbsp;gcd))<br>
        after-gcd-2<br></tt>

      </td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

This is a reasonable approach for handling small problems, but it would be awkward if there were many instances of GCD computations in the controller sequence. To decide where to continue executing after the <tt>gcd</tt> subroutine, we would need tests in the data paths and branch instructions in the controller for all the places that use <tt>gcd</tt>. A more powerful method for implementing subroutines is to have the <tt>continue</tt> register hold the label of the entry point in the controller sequence at which execution should continue when the subroutine is finished. Implementing this strategy requires a new kind of connection between the data paths and the controller of a register machine: There must be a way to assign to a register a label in the controller sequence in such a way that this value can be fetched from the register and used to continue execution at the designated entry point.


<a name="index_term_5556"></a><a name="index_term_5558"></a>To reflect this ability, we will extend the <tt>assign</tt> instruction of the register-machine language to allow a register to be assigned as value a label from the controller sequence (as a special kind of constant). We will also extend the <tt>goto</tt> instruction to allow execution to continue at the entry point described by the contents of a register rather than only at an entry point described by a constant label. Using these new constructs we can terminate the <tt>gcd</tt> subroutine with a branch to the location stored in the <tt>continue</tt> register. This leads to the controller sequence shown in figure&nbsp;<a href="#%_fig_5.10">5.10</a>.


A machine with more than one subroutine could use multiple continuation registers (e.g., <tt>gcd-continue</tt>, <tt>factorial-continue</tt>) or we could have all subroutines share a single <tt>continue</tt> register. Sharing is more economical, but we must be careful if we have a subroutine (<tt>sub1</tt>) that calls another subroutine (<tt>sub2</tt>). Unless <tt>sub1</tt> saves the contents of <tt>continue</tt> in some other register before setting up <tt>continue</tt> for the call to <tt>sub2</tt>, <tt>sub1</tt> will not know where to go when it is finished. The mechanism developed in the next section to handle recursion also provides a better solution to this problem of nested subroutine calls.


<a name="%_sec_5.1.4"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_5.1.4">5.1.4&nbsp;&nbsp;Using a Stack to Implement Recursion</a></h3>

<a name="index_term_5560"></a><a name="index_term_5562"></a><a name="index_term_5564"></a> <a name="index_term_5566"></a>With the ideas illustrated so far, we can implement any iterative process by specifying a register machine that has a register corresponding to each state variable of the process. The machine repeatedly executes a controller loop, changing the contents of the registers, until some termination condition is satisfied. At each point in the controller sequence, the state of the machine (representing the state of the iterative process) is completely determined by the contents of the registers (the values of the state variables).


<a name="index_term_5568"></a><a name="index_term_5570"></a><a name="index_term_5572"></a>Implementing recursive processes, however, requires an additional mechanism. Consider the following recursive method for computing factorials, which we first examined in section <a href="chapter_1_section_2.html#%_sec_1.2.1">1.2.1</a>:


<tt>(define&nbsp;(factorial&nbsp;n)<br>
&nbsp;&nbsp;(if&nbsp;(=&nbsp;n&nbsp;1)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;(factorial&nbsp;(-&nbsp;n&nbsp;1))&nbsp;n)))<br></tt>


As we see from the procedure, computing *n*! requires computing (*n* - 1)!. Our GCD machine, modeled on the procedure


<tt>(define&nbsp;(gcd&nbsp;a&nbsp;b)<br>
&nbsp;&nbsp;(if&nbsp;(=&nbsp;b&nbsp;0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(gcd&nbsp;b&nbsp;(remainder&nbsp;a&nbsp;b))))<br></tt>


similarly had to compute another GCD. But there is an important difference between the <tt>gcd</tt> procedure, which reduces the original computation to a new GCD computation, and <tt>factorial</tt>, which requires computing another factorial as a subproblem. In GCD, the answer to the new GCD computation is the answer to the original problem. To compute the next GCD, we simply place the new arguments in the input registers of the GCD machine and reuse the machine's data paths by executing the same controller sequence. When the machine is finished solving the final GCD problem, it has completed the entire computation.


In the case of factorial (or any recursive process) the answer to the new factorial subproblem is not the answer to the original problem. The value obtained for (*n* - 1)! must be multiplied by *n* to get the final answer. If we try to imitate the GCD design, and solve the factorial subproblem by decrementing the <tt>n</tt> register and rerunning the factorial machine, we will no longer have available the old value of <tt>n</tt> by which to multiply the result. We thus need a second factorial machine to work on the subproblem. This second factorial computation itself has a factorial subproblem, which requires a third factorial machine, and so on. Since each factorial machine contains another factorial machine within it, the total machine contains an infinite nest of similar machines and hence cannot be constructed from a fixed, finite number of parts.


Nevertheless, we can implement the factorial process as a register machine if we can arrange to use the same components for each nested instance of the machine. Specifically, the machine that computes *n*! should use the same components to work on the subproblem of computing (*n* - 1)!, on the subproblem for (*n* - 2)!, and so on. This is plausible because, although the factorial process dictates that an unbounded number of copies of the same machine are needed to perform a computation, only one of these copies needs to be active at any given time. When the machine encounters a recursive subproblem, it can suspend work on the main problem, reuse the same physical parts to work on the subproblem, then continue the suspended computation.


In the subproblem, the contents of the registers will be different than they were in the main problem. (In this case the <tt>n</tt> register is decremented.) In order to be able to continue the suspended computation, the machine must save the contents of any registers that will be needed after the subproblem is solved so that these can be restored to continue the suspended computation. In the case of factorial, we will save the old value of <tt>n</tt>, to be restored when we are finished computing the factorial of the decremented <tt>n</tt> register.<a name="call_footnote_Temp_717" href="#footnote_Temp_717" id="call_footnote_Temp_717"><sup><small>2</small></sup></a>


Since there is no *a priori* limit on the depth of nested recursive calls, we may need to save an arbitrary number of register values. These values must be restored in the reverse of the order in which they were saved, since in a nest of recursions the last subproblem to be entered is the first to be finished. This dictates the use of a *stack*, or "last in, first out" data structure, to save register values. We can extend the register-machine language to include a stack by adding two kinds of instructions: Values are placed <a name="index_term_5574"></a><a name="index_term_5576"></a><a name="index_term_5578"></a><a name="index_term_5580"></a>on the stack using a <tt>save</tt> instruction and restored from the stack using a <tt>restore</tt> instruction. After a sequence of values has been <tt>save</tt>d on the stack, a sequence of <tt>restore</tt>s will retrieve these values in reverse order.<a name="call_footnote_Temp_718" href="#footnote_Temp_718" id="call_footnote_Temp_718"><sup><small>3</small></sup></a>


With the aid of the stack, we can reuse a single copy of the factorial machine's data paths for each factorial subproblem. There is a similar design issue in reusing the controller sequence that operates the data paths. To reexecute the factorial computation, the controller cannot simply loop back to the beginning, as with an iterative process, because after solving the (*n* - 1)! subproblem the machine must still multiply the result by *n*. The controller must suspend its computation of *n*!, solve the (*n* - 1)! subproblem, then continue its computation of *n*!. This view of the factorial computation suggests the use of the subroutine mechanism described in section <a href="#%_sec_5.1.3">5.1.3</a>, which has the controller use a <a name="index_term_5582"></a><tt>continue</tt> register to transfer to the part of the sequence that solves a subproblem and then continue where it left off on the main problem. We can thus make a factorial subroutine that returns to the entry point stored in the <tt>continue</tt> register. Around each subroutine call, we save and restore <tt>continue</tt> just as we do the <tt>n</tt> register, since each "level" of the factorial computation will use the same <tt>continue</tt> register. That is, the factorial subroutine must put a new value in <tt>continue</tt> when it calls itself for a subproblem, but it will need the old value in order to return to the place that called it to solve a subproblem.


Figure&nbsp;<a href="#%_fig_5.11">5.11</a> shows the data paths and controller for a machine that implements the recursive <tt>factorial</tt> procedure. The machine has a stack and three registers, called <tt>n</tt>, <tt>val</tt>, and <tt>continue</tt>. To simplify the data-path diagram, we have not named the register-assignment buttons, only the stack-operation buttons (<tt>sc</tt> and <tt>sn</tt> to save registers, <tt>rc</tt> and <tt>rn</tt> to restore registers). To operate the machine, we put in register <tt>n</tt> the number whose factorial we wish to compute and start the machine. When the machine reaches <tt>fact-done</tt>, the computation is finished and the answer will be found in the <tt>val</tt> register. In the controller sequence, <tt>n</tt> and <tt>continue</tt> are saved before each recursive call and restored upon return from the call. Returning from a call is accomplished by branching to the location stored in <tt>continue</tt>. <tt>Continue</tt> is initialized when the machine starts so that the last return will go to <tt>fact-done</tt>. The <tt>val</tt> register, which holds the result of the factorial computation, is not saved before the recursive call, because the old contents of <tt>val</tt> is not useful after the subroutine returns. Only the new value, which is the value produced by the subcomputation, is needed. Although in principle the factorial computation requires an infinite machine, the machine in figure&nbsp;<a href="#%_fig_5.11">5.11</a> is actually finite except for the stack, which is potentially unbounded. Any particular physical implementation of a stack, however, will be of finite size, and this will limit the depth of recursive calls that can be handled by the machine. This implementation of factorial illustrates the general strategy for realizing recursive algorithms as ordinary register machines augmented by stacks. When a recursive subproblem is encountered, we save on the stack the registers whose current values will be required after the subproblem is solved, solve the recursive subproblem, then restore the saved registers and continue execution on the main problem. The <tt>continue</tt> register must always be saved. Whether there are other registers that need to be saved depends on the particular machine, since not all recursive computations need the original values of registers that are modified during solution of the subproblem (see exercise&nbsp;<a href="#%_thm_5.4">5.4</a>).


<a name="%_sec_Temp_719"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_719">A double recursion</a></h4>

<a name="index_term_5584"></a>Let us examine a more complex recursive process, the tree-recursive computation of the Fibonacci numbers, which we introduced in section <a href="chapter_1_section_2.html#%_sec_1.2.2">1.2.2</a>:


<tt>(define&nbsp;(fib&nbsp;n)<br>
&nbsp;&nbsp;(if&nbsp;(&lt;&nbsp;n&nbsp;2)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;n<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(+&nbsp;(fib&nbsp;(-&nbsp;n&nbsp;1))&nbsp;(fib&nbsp;(-&nbsp;n&nbsp;2)))))<br></tt>


Just as with factorial, we can implement the recursive Fibonacci computation as a register machine with registers <tt>n</tt>, <tt>val</tt>, and <tt>continue</tt>. The machine is more complex than the one for factorial, because there are two places in the controller sequence where we need to perform recursive calls -- once to compute Fib(*n* - 1) and once to compute Fib(*n* - 2). To set up for each of these calls, we save the registers whose values will be needed later, set the <tt>n</tt> register to the number whose Fib we need to compute recursively (*n* - 1 or *n* - 2), and assign to <tt>continue</tt> the entry point in the main sequence to which to return (<tt>afterfib-n-1</tt> or <tt>afterfib-n-2</tt>, respectively). We then go to <tt>fib-loop</tt>. When we return from the recursive call, the answer is in <tt>val</tt>. Figure&nbsp;<a href="#%_fig_5.12">5.12</a> shows the controller sequence for this machine.


<a name="%_fig_5.11"></a>

<div align="left">
  <div align="left">
    **Figure 5.11:**&nbsp;&nbsp;A recursive factorial machine.
  </div>

  <table width="100%">
    <tr>
      <td>
        <img src="img/chapter_5_image_06.gif" border="0">

        
<tt>(controller<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;continue&nbsp;(label&nbsp;fact-done))&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;set&nbsp;up&nbsp;final&nbsp;return&nbsp;address*<br>
        &nbsp;fact-loop<br>
        &nbsp;&nbsp;&nbsp;(test&nbsp;(op&nbsp;=)&nbsp;(reg&nbsp;n)&nbsp;(const&nbsp;1))<br>
        &nbsp;&nbsp;&nbsp;(branch&nbsp;(label&nbsp;base-case))<br>
        &nbsp;&nbsp;&nbsp;*;;&nbsp;Set&nbsp;up&nbsp;for&nbsp;the&nbsp;recursive&nbsp;call&nbsp;by&nbsp;saving&nbsp;<tt>n</tt>&nbsp;and&nbsp;<tt>continue</tt>.*<br>
        &nbsp;&nbsp;&nbsp;*;;&nbsp;Set&nbsp;up&nbsp;<tt>continue</tt>&nbsp;so&nbsp;that&nbsp;the&nbsp;computation&nbsp;will&nbsp;continue*<br>
        &nbsp;&nbsp;&nbsp;*;;&nbsp;at&nbsp;<tt>after-fact</tt>&nbsp;when&nbsp;the&nbsp;subroutine&nbsp;returns.*<br>
        &nbsp;&nbsp;&nbsp;(save&nbsp;continue)<br>
        &nbsp;&nbsp;&nbsp;(save&nbsp;n)<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;n&nbsp;(op&nbsp;-)&nbsp;(reg&nbsp;n)&nbsp;(const&nbsp;1))<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;continue&nbsp;(label&nbsp;after-fact))<br>
        &nbsp;&nbsp;&nbsp;(goto&nbsp;(label&nbsp;fact-loop))<br>
        &nbsp;after-fact<br>
        &nbsp;&nbsp;&nbsp;(restore&nbsp;n)<br>
        &nbsp;&nbsp;&nbsp;(restore&nbsp;continue)<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;val&nbsp;(op&nbsp;*)&nbsp;(reg&nbsp;n)&nbsp;(reg&nbsp;val))&nbsp;&nbsp;&nbsp;*;&nbsp;<tt>val</tt>&nbsp;now&nbsp;contains*&nbsp;*n*(*n* - 1)!<br>
        &nbsp;&nbsp;&nbsp;(goto&nbsp;(reg&nbsp;continue))&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;return&nbsp;to&nbsp;caller*<br>
        &nbsp;base-case<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;val&nbsp;(const&nbsp;1))&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;base&nbsp;case:* 1! = 1<br>
        &nbsp;&nbsp;&nbsp;(goto&nbsp;(reg&nbsp;continue))&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;return&nbsp;to&nbsp;caller*<br>
        &nbsp;fact-done)<br></tt>

      </td>
    </tr>

    <tr>
      <td><a name="index_term_5586"></a></td>
    </tr>
  </table>
</div>

<a name="%_fig_5.12"></a>

<div align="left">
  <div align="left">
    **Figure 5.12:**&nbsp;&nbsp;Controller for a machine to compute Fibonacci numbers.
  </div>

  <table width="100%">
    <tr>
      <td>
        
<tt>(controller<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;continue&nbsp;(label&nbsp;fib-done))<br>
        &nbsp;fib-loop<br>
        &nbsp;&nbsp;&nbsp;(test&nbsp;(op&nbsp;&lt;)&nbsp;(reg&nbsp;n)&nbsp;(const&nbsp;2))<br>
        &nbsp;&nbsp;&nbsp;(branch&nbsp;(label&nbsp;immediate-answer))<br>
        &nbsp;&nbsp;&nbsp;*;;&nbsp;set&nbsp;up&nbsp;to&nbsp;compute&nbsp;*F</tt>*i**b*(*n* - 1)<br>
        &nbsp;&nbsp;&nbsp;(save&nbsp;continue)<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;continue&nbsp;(label&nbsp;afterfib-n-1))<br>
        &nbsp;&nbsp;&nbsp;(save&nbsp;n)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;save&nbsp;old&nbsp;value&nbsp;of&nbsp;<tt>n</tt>*<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;n&nbsp;(op&nbsp;-)&nbsp;(reg&nbsp;n)&nbsp;(const&nbsp;1))*;&nbsp;clobber&nbsp;<tt>n</tt>&nbsp;to&nbsp;*n - 1<br>
        &nbsp;&nbsp;&nbsp;(goto&nbsp;(label&nbsp;fib-loop))&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;perform&nbsp;recursive&nbsp;call*<br>
        &nbsp;afterfib-n-1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;upon&nbsp;return,&nbsp;<tt>val</tt>&nbsp;contains&nbsp;*F*i**b*(*n* - 1)<br>
        &nbsp;&nbsp;&nbsp;(restore&nbsp;n)<br>
        &nbsp;&nbsp;&nbsp;(restore&nbsp;continue)<br>
        &nbsp;&nbsp;&nbsp;*;;&nbsp;set&nbsp;up&nbsp;to&nbsp;compute&nbsp;*F*i**b*(*n* - 2)<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;n&nbsp;(op&nbsp;-)&nbsp;(reg&nbsp;n)&nbsp;(const&nbsp;2))<br>
        &nbsp;&nbsp;&nbsp;(save&nbsp;continue)<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;continue&nbsp;(label&nbsp;afterfib-n-2))<br>
        &nbsp;&nbsp;&nbsp;(save&nbsp;val)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;save&nbsp;*F*i**b*(*n* - 1)<br>
        &nbsp;&nbsp;&nbsp;(goto&nbsp;(label&nbsp;fib-loop))<br>
        &nbsp;afterfib-n-2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;upon&nbsp;return,&nbsp;<tt>val</tt>&nbsp;contains&nbsp;*F*i**b*(*n* - 2)<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;n&nbsp;(reg&nbsp;val))&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;<tt>n</tt>&nbsp;now&nbsp;contains&nbsp;*F*i**b*(*n* - 2)<br>
        &nbsp;&nbsp;&nbsp;(restore&nbsp;val)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;<tt>val</tt>&nbsp;now&nbsp;contains&nbsp;*F*i**b*(*n* - 1)<br>
        &nbsp;&nbsp;&nbsp;(restore&nbsp;continue)<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;val&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;&nbsp;*F*i**b*(*n* - 1) + &nbsp;*F**i**b*(*n* - 2)<br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(op&nbsp;+)&nbsp;(reg&nbsp;val)&nbsp;(reg&nbsp;n))&nbsp;<br>
        &nbsp;&nbsp;&nbsp;(goto&nbsp;(reg&nbsp;continue))&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;return&nbsp;to&nbsp;caller,&nbsp;answer&nbsp;is&nbsp;in&nbsp;<tt>val</tt>*<br>
        &nbsp;immediate-answer<br>
        &nbsp;&nbsp;&nbsp;(assign&nbsp;val&nbsp;(reg&nbsp;n))&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*;&nbsp;base&nbsp;case:&nbsp;&nbsp;*F*i**b*(*n*) = *n*<br>
        &nbsp;&nbsp;&nbsp;(goto&nbsp;(reg&nbsp;continue))<br>
        &nbsp;fib-done)<br>

      </td>
    </tr>

    <tr>
      <td><a name="index_term_5588"></a></td>
    </tr>
  </table>
</div>

<a name="exercise_5_4">**Exercise 5.4**</a> Specify register machines that implement each of the following procedures. For each machine, write a controller instruction sequence and draw a diagram showing the data paths.


a. Recursive exponentiation:


<tt><a name="index_term_5590"></a>(define&nbsp;(expt&nbsp;b&nbsp;n)<br>
&nbsp;&nbsp;(if&nbsp;(=&nbsp;n&nbsp;0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(*&nbsp;b&nbsp;(expt&nbsp;b&nbsp;(-&nbsp;n&nbsp;1)))))<br></tt>


b. Iterative exponentiation:


<tt>(define&nbsp;(expt&nbsp;b&nbsp;n)<br>
&nbsp;&nbsp;(define&nbsp;(expt-iter&nbsp;counter&nbsp;product)<br>
&nbsp;&nbsp;&nbsp;&nbsp;(if&nbsp;(=&nbsp;counter&nbsp;0)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;product<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(expt-iter&nbsp;(-&nbsp;counter&nbsp;1)&nbsp;(*&nbsp;b&nbsp;product))))<br>
&nbsp;&nbsp;(expt-iter&nbsp;n&nbsp;1))<br></tt>


<a name="exercise_5_5">**Exercise 5.5**</a> Hand-simulate the factorial and Fibonacci machines, using some nontrivial input (requiring execution of at least one recursive call). Show the contents of the stack at each significant point in the execution.


<a name="exercise_5_6">**Exercise 5.6**</a> Ben Bitdiddle observes that the Fibonacci machine's controller sequence has an extra <tt>save</tt> and an extra <tt>restore</tt>, which can be removed to make a faster machine. Where are these instructions?


<a name="%_sec_5.1.5"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_5.1.5">5.1.5&nbsp;&nbsp;Instruction Summary</a></h3>

<a name="index_term_5592"></a> <a name="index_term_5594"></a><a name="index_term_5596"></a>A controller instruction in our register-machine language has one of the following forms, where each &lt;*input<sub>*i*</sub>*&gt; is either <tt>(reg &lt;*register-name*&gt;)</tt> or <tt>(const &lt;*constant-value*&gt;)</tt>.


These instructions were introduced in section <a href="#%_sec_5.1.1">5.1.1</a>:


<tt><a name="index_term_5598"></a>(assign&nbsp;&lt;*register-name*&gt;&nbsp;(reg&nbsp;&lt;*register-name*&gt;))<br>
<br>
(assign&nbsp;&lt;*register-name*&gt;&nbsp;(const&nbsp;&lt;*constant-value*&gt;))<br>
<br>
<a name="index_term_5600"></a>(assign&nbsp;&lt;*register-name*&gt;&nbsp;(op&nbsp;&lt;*operation-name*&gt;)&nbsp;&lt;*input<sub>1</sub>*&gt;&nbsp;</tt>... &lt;*input<sub>*n*</sub>*&gt;)<br>
<br>
<a name="index_term_5602"></a>(perform&nbsp;(op&nbsp;&lt;*operation-name*&gt;)&nbsp;&lt;*input<sub>1</sub>*&gt;&nbsp;<tt>...</tt> &lt;*input<sub>*n*</sub>*&gt;)<br>
<br>
<a name="index_term_5604"></a>(test&nbsp;(op&nbsp;&lt;*operation-name*&gt;)&nbsp;&lt;*input<sub>1</sub>*&gt;&nbsp;<tt>...</tt> &lt;*input<sub>*n*</sub>*&gt;)<br>
<br>
<a name="index_term_5606"></a><a name="index_term_5608"></a>(branch&nbsp;(label&nbsp;&lt;*label-name*&gt;))<br>
<br>
<a name="index_term_5610"></a>(goto&nbsp;(label&nbsp;&lt;*label-name*&gt;))<br>


The use of registers to hold labels was introduced in section <a href="#%_sec_5.1.3">5.1.3</a>:


<tt>(assign&nbsp;&lt;*register-name*&gt;&nbsp;(label&nbsp;&lt;*label-name*&gt;))<br>
<br>
(goto&nbsp;(reg&nbsp;&lt;*register-name*&gt;))<br></tt>


Instructions to use the stack were introduced in section <a href="#%_sec_5.1.4">5.1.4</a>:


<tt><a name="index_term_5612"></a>(save&nbsp;&lt;*register-name*&gt;)<br>
<br>
<a name="index_term_5614"></a>(restore&nbsp;&lt;*register-name*&gt;)<br></tt>


<a name="index_term_5616"></a><a name="index_term_5618"></a><a name="index_term_5620"></a>The only kind of &lt;*constant-value*&gt; we have seen so far is a number, but later we will use strings, symbols, and lists. For example, <tt>(const&nbsp;&quot;abc&quot;)</tt> is the string <tt>&quot;abc&quot;</tt>, <tt>(const&nbsp;abc)</tt> is the symbol <tt>abc</tt>, <tt>(const&nbsp;(a b c))</tt> is the list <tt>(a b c)</tt>, and <tt>(const&nbsp;())</tt> is the empty list.

<div class="smallprint">
  <hr>
</div>

<div class="footnote">
  
<a name="footnote_Temp_715" href="#call_footnote_Temp_715" id="footnote_Temp_715"><sup><small>1</small></sup></a> This assumption glosses over a great deal of complexity. Usually a large portion of the implementation of a Lisp system is dedicated to making reading and printing work.

  
<a name="footnote_Temp_717" href="#call_footnote_Temp_717" id="footnote_Temp_717"><sup><small>2</small></sup></a> One might argue that we don't need to save the old <tt>n</tt>; after we decrement it and solve the subproblem, we could simply increment it to recover the old value. Although this strategy works for factorial, it cannot work in general, since the old value of a register cannot always be computed from the new one.

  
<a name="footnote_Temp_718" href="#call_footnote_Temp_718" id="footnote_Temp_718"><sup><small>3</small></sup></a> In section <a href="chapter_5_section_3.html#%_sec_5.3">5.3</a> we will see how to implement a stack in terms of more primitive operations.

</div>