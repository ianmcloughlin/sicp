(section_41)=
# The Metacircular Evaluator

<a name="index_term_4210"></a> Our evaluator for Lisp will be implemented as a Lisp program. It may seem circular to think about evaluating Lisp programs using an evaluator that is itself implemented in Lisp. However, evaluation is a process, so it is appropriate to describe the evaluation process using Lisp, which, after all, is our tool for describing processes.<a name="call_footnote_Temp_510" href="#footnote_Temp_510" id="call_footnote_Temp_510"><sup><small>3</small></sup></a> An evaluator that is written in the same language <a name="index_term_4212"></a><a name="index_term_4214"></a>that it evaluates is said to be *metacircular*.


<a name="index_term_4216"></a><a name="index_term_4218"></a>The metacircular evaluator is essentially a Scheme formulation of the environment model of evaluation described in section&#160;<a href="chapter_3_section_2.html#%_sec_3.2">3.2</a>. Recall that the model has two basic parts:

<blockquote>
  
1. To evaluate a combination (a compound expression other than a special form), evaluate the subexpressions and then apply the value of the operator subexpression to the values of the operand subexpressions.

  
2. To apply a compound procedure to a set of arguments, evaluate the body of the procedure in a new environment. To construct this environment, extend the environment part of the procedure object by a frame in which the formal parameters of the procedure are bound to the arguments to which the procedure is applied.

</blockquote>

<a name="index_term_4220"></a>These two rules describe the essence of the evaluation process, a basic cycle in which expressions to be evaluated in environments are reduced to procedures to be applied to arguments, which in turn are reduced to new expressions to be evaluated in new environments, and so on, until we get down to symbols, whose values are looked up in the environment, and to primitive procedures, which are applied directly (see figure&#160;<a href="#%_fig_4.1">4.1</a>).<a name="call_footnote_Temp_511" href="#footnote_Temp_511" id="call_footnote_Temp_511"><sup><small>4</small></sup></a> This evaluation cycle will be embodied by the interplay between the two critical procedures in the evaluator, <tt>eval</tt> and <tt>apply</tt>, which are described in section&#160;<a href="#%_sec_4.1.1">4.1.1</a> (see figure&#160;<a href="#%_fig_4.1">4.1</a>).


The implementation of the evaluator will depend upon procedures that define the *syntax* of the expressions to be evaluated. We will use <a name="index_term_4224"></a>data abstraction to make the evaluator independent of the representation of the language. For example, rather than committing to a choice that an assignment is to be represented by a list beginning with the symbol <tt>set!</tt> we use an abstract predicate <tt>assignment?</tt> to test for an assignment, and we use abstract selectors <tt>assignment-variable</tt> and <tt>assignment-value</tt> to access the parts of an assignment. Implementation of expressions will be described in detail in section&#160;<a href="#%_sec_4.1.2">4.1.2</a>. There are also operations, described in section&#160;<a href="#%_sec_4.1.3">4.1.3</a>, that specify the representation of procedures and environments. For example, <tt>make-procedure</tt> constructs compound procedures, <tt>lookup-variable-value</tt> accesses the values of variables, and <tt>apply-primitive-procedure</tt> applies a primitive procedure to a given list of arguments.


<a name="%_sec_4.1.1"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_4.1.1">4.1.1&#160;&#160;The Core of the Evaluator</a></h3>

<a name="index_term_4226"></a> <a name="%_fig_4.1"></a>

<div align="left">
  <div align="left">
    **Figure 4.1:**&#160;&#160;The <tt>eval</tt>-<tt>apply</tt> cycle exposes the essence of a computer language.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_4_image_01.gif" border="0"></td>
    </tr>

    <tr>
      <td><a name="index_term_4228"></a></td>
    </tr>
  </table>
</div>

The evaluation process can be described as the interplay between two procedures: <tt>eval</tt> and <tt>apply</tt>.


<a name="%_sec_Temp_512"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_512">Eval</a></h4>

<a name="index_term_4230"></a><tt>Eval</tt> takes as arguments an expression and an environment. It classifies the expression and directs its evaluation. <tt>Eval</tt> is structured as a case analysis of the syntactic type of the expression to be evaluated. In order to keep the procedure general, we express the determination of the type of an expression abstractly, making no commitment to any particular <a name="index_term_4232"></a>representation for the various types of expressions. Each type of expression has a predicate that tests for it and an abstract means for selecting its parts. This <a name="index_term_4234"></a><a name="index_term_4236"></a>*abstract syntax* makes it easy to see how we can change the syntax of the language by using the same evaluator, but with a different collection of syntax procedures.


<a name="%_sec_Temp_513"></a>

<h5><a href="sicp_part_04.html#%_toc_%_sec_Temp_513">Primitive expressions</a></h5>

<ul>
  <li><a name="index_term_4238"></a><a name="index_term_4240"></a>For self-evaluating expressions, such as numbers, <tt>eval</tt> returns the expression itself.</li>

  <li><tt>Eval</tt> must look up variables in the environment to find their values.</li>
</ul>

<a name="%_sec_Temp_514"></a>

<h5><a href="sicp_part_04.html#%_toc_%_sec_Temp_514">Special forms</a></h5>

<ul>
  <li>For quoted expressions, <tt>eval</tt> returns the expression that was quoted.</li>

  <li>An assignment to (or a definition of) a variable must recursively call <tt>eval</tt> to compute the new value to be associated with the variable. The environment must be modified to change (or create) the binding of the variable.</li>

  <li>An <tt>if</tt> expression requires special processing of its parts, so as to evaluate the consequent if the predicate is true, and otherwise to evaluate the alternative.</li>

  <li>A <tt>lambda</tt> expression must be transformed into an applicable procedure by packaging together the parameters and body specified by the <tt>lambda</tt> expression with the environment of the evaluation.</li>

  <li>A <tt>begin</tt> expression requires evaluating its sequence of expressions in the order in which they appear.</li>

  <li>A case analysis (<tt>cond</tt>) is transformed into a nest of <tt>if</tt> expressions and then evaluated.</li>
</ul>

<a name="%_sec_Temp_515"></a>

<h5><a href="sicp_part_04.html#%_toc_%_sec_Temp_515">Combinations</a></h5>

<ul>
  <li>For a procedure application, <tt>eval</tt> must recursively evaluate the operator part and the operands of the combination. The resulting procedure and arguments are passed to <tt>apply</tt>, which handles the actual procedure application.</li>
</ul>

<a name="index_term_4242"></a>Here is the definition of <tt>eval</tt>:


<tt>(define&#160;(eval&#160;exp&#160;env)<br>
&#160;&#160;(cond&#160;((self-evaluating?&#160;exp)&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((variable?&#160;exp)&#160;(lookup-variable-value&#160;exp&#160;env))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((quoted?&#160;exp)&#160;(text-of-quotation&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((assignment?&#160;exp)&#160;(eval-assignment&#160;exp&#160;env))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((definition?&#160;exp)&#160;(eval-definition&#160;exp&#160;env))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((if?&#160;exp)&#160;(eval-if&#160;exp&#160;env))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((lambda?&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(make-procedure&#160;(lambda-parameters&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda-body&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;env))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((begin?&#160;exp)&#160;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(eval-sequence&#160;(begin-actions&#160;exp)&#160;env))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((cond?&#160;exp)&#160;(eval&#160;(cond-&gt;if&#160;exp)&#160;env))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((application?&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(apply&#160;(eval&#160;(operator&#160;exp)&#160;env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(list-of-values&#160;(operands&#160;exp)&#160;env)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Unknown&#160;expression&#160;type&#160;--&#160;EVAL&quot;&#160;exp))))<br></tt>


<a name="index_term_4244"></a><a name="index_term_4246"></a>For clarity, <tt>eval</tt> has been implemented as a case analysis using <tt>cond</tt>. The disadvantage of this is that our procedure handles only a few distinguishable types of expressions, and no new ones can be defined without editing the definition of <tt>eval</tt>. In most Lisp implementations, dispatching on the type of an expression is done in a data-directed style. This allows a user to add new types of expressions that <tt>eval</tt> can distinguish, without modifying the definition of <tt>eval</tt> itself. (See exercise&#160;<a href="#%_thm_4.3">4.3</a>.)


<a name="%_sec_Temp_516"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_516">Apply</a></h4>

<tt>Apply</tt> takes two arguments, a procedure and a list of arguments to which the procedure should be applied. <tt>Apply</tt> classifies procedures into two kinds: It calls <a name="index_term_4248"></a><tt>apply-primitive-procedure</tt> to apply primitives; it applies compound procedures by sequentially evaluating the expressions that make up the body of the procedure. The environment for the evaluation of the body of a compound procedure is constructed by extending the base environment carried by the procedure to include a frame that binds the parameters of the procedure to the arguments to which the procedure is to be applied. Here is the definition of <tt>apply</tt>:


<tt><a name="index_term_4250"></a>(define&#160;(apply&#160;procedure&#160;arguments)<br>
&#160;&#160;(cond&#160;((primitive-procedure?&#160;procedure)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(apply-primitive-procedure&#160;procedure&#160;arguments))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((compound-procedure?&#160;procedure)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(eval-sequence<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(procedure-body&#160;procedure)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(extend-environment<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(procedure-parameters&#160;procedure)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;arguments<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(procedure-environment&#160;procedure))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&quot;Unknown&#160;procedure&#160;type&#160;--&#160;APPLY&quot;&#160;procedure))))<br></tt>


<a name="%_sec_Temp_517"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_517">Procedure arguments</a></h4>

When <tt>eval</tt> processes a procedure application, it uses <tt>list-of-values</tt> to produce the list of arguments to which the procedure is to be applied. <tt>List-of-values</tt> takes as an argument the operands of the combination. It evaluates each operand and returns a list of the corresponding values:<a name="call_footnote_Temp_518" href="#footnote_Temp_518" id="call_footnote_Temp_518"><sup><small>5</small></sup></a>


<tt><a name="index_term_4256"></a>(define&#160;(list-of-values&#160;exps&#160;env)<br>
&#160;&#160;(if&#160;(no-operands?&#160;exps)<br>
&#160;&#160;&#160;&#160;&#160;&#160;'()<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cons&#160;(eval&#160;(first-operand&#160;exps)&#160;env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(list-of-values&#160;(rest-operands&#160;exps)&#160;env))))<br></tt>


<a name="%_sec_Temp_519"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_519">Conditionals</a></h4>

<tt>Eval-if</tt> evaluates the predicate part of an <tt>if</tt> expression in the given environment. If the result is true, <tt>eval-if</tt> evaluates the consequent, otherwise it evaluates the alternative:


<tt><a name="index_term_4258"></a>(define&#160;(eval-if&#160;exp&#160;env)<br>
&#160;&#160;(if&#160;(true?&#160;(eval&#160;(if-predicate&#160;exp)&#160;env))<br>
&#160;&#160;&#160;&#160;&#160;&#160;(eval&#160;(if-consequent&#160;exp)&#160;env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(eval&#160;(if-alternative&#160;exp)&#160;env)))<br></tt>


<a name="index_term_4260"></a>The use of <tt>true?</tt> in <tt>eval-if</tt> highlights the issue of the connection between an implemented language and an implementation language. The <tt>if-predicate</tt> is evaluated in the language being implemented and thus yields a value in that language. The interpreter predicate <tt>true?</tt> translates that value into a value that can be tested by the <tt>if</tt> in the implementation language: The metacircular representation of truth might not be the same as that of the underlying Scheme.<a name="call_footnote_Temp_520" href="#footnote_Temp_520" id="call_footnote_Temp_520"><sup><small>6</small></sup></a>


<a name="%_sec_Temp_521"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_521">Sequences</a></h4>

<tt>Eval-sequence</tt> is used by <tt>apply</tt> to evaluate the sequence of expressions in a procedure body and by <tt>eval</tt> to evaluate the sequence of expressions in a <tt>begin</tt> expression. It takes as arguments a sequence of expressions and an environment, and evaluates the expressions in the order in which they occur. The value returned is the value of the final expression.


<tt><a name="index_term_4264"></a>(define&#160;(eval-sequence&#160;exps&#160;env)<br>
&#160;&#160;(cond&#160;((last-exp?&#160;exps)&#160;(eval&#160;(first-exp&#160;exps)&#160;env))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(eval&#160;(first-exp&#160;exps)&#160;env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(eval-sequence&#160;(rest-exps&#160;exps)&#160;env))))<br></tt>


<a name="%_sec_Temp_522"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_522">Assignments and definitions</a></h4>

The following procedure handles assignments to variables. It calls <tt>eval</tt> to find the value to be assigned and transmits the variable and the resulting value to <tt>set-variable-value!</tt> to be installed in the designated environment.


<tt><a name="index_term_4266"></a>(define&#160;(eval-assignment&#160;exp&#160;env)<br>
&#160;&#160;(set-variable-value!&#160;(assignment-variable&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(eval&#160;(assignment-value&#160;exp)&#160;env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;env)<br>
&#160;&#160;'ok)<br></tt>


Definitions of variables are handled in a similar manner.<a name="call_footnote_Temp_523" href="#footnote_Temp_523" id="call_footnote_Temp_523"><sup><small>7</small></sup></a>


<tt><a name="index_term_4268"></a>(define&#160;(eval-definition&#160;exp&#160;env)<br>
&#160;&#160;(define-variable!&#160;(definition-variable&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(eval&#160;(definition-value&#160;exp)&#160;env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;env)<br>
&#160;&#160;'ok)<br></tt>


We have chosen here to return the symbol <tt>ok</tt> as the value of an assignment or a definition.<a name="call_footnote_Temp_524" href="#footnote_Temp_524" id="call_footnote_Temp_524"><sup><small>8</small></sup></a>


<a name="%_thm_4.1"></a> **Exercise 4.1.**&#160;&#160;<a name="index_term_4270"></a><a name="index_term_4272"></a>Notice that we cannot tell whether the metacircular evaluator evaluates operands from left to right or from right to left. Its evaluation order is inherited from the underlying Lisp: If the arguments to <tt>cons</tt> in <tt>list-of-values</tt> are evaluated from left to right, then <tt>list-of-values</tt> will evaluate operands from left to right; and if the arguments to <tt>cons</tt> are evaluated from right to left, then <tt>list-of-values</tt> will evaluate operands from right to left.


Write a version of <tt>list-of-values</tt> that evaluates operands from left to right regardless of the order of evaluation in the underlying Lisp. Also write a version of <tt>list-of-values</tt> that evaluates operands from right to left.


<a name="%_sec_4.1.2"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_4.1.2">4.1.2&#160;&#160;Representing Expressions</a></h3>

<a name="index_term_4274"></a><a name="index_term_4276"></a> <a name="index_term_4278"></a>The evaluator is reminiscent of the symbolic differentiation program discussed in section&#160;<a href="chapter_2_section_3.html#%_sec_2.3.2">2.3.2</a>. Both programs operate on symbolic expressions. In both programs, the result of operating on a compound expression is determined by operating recursively on the pieces of the expression and combining the results in a way that depends on the type of the expression. In both programs we used <a name="index_term_4280"></a>data abstraction to decouple the general rules of operation from the details of how expressions are represented. In the differentiation program this meant that the same differentiation procedure could deal with algebraic expressions in prefix form, in infix form, or in some other form. For the evaluator, this means that the syntax of the language being evaluated is determined solely by the procedures that classify and extract pieces of expressions.


Here is the specification of the syntax of our language:


� The only self-evaluating items are numbers and strings:


<tt><a name="index_term_4282"></a>(define&#160;(self-evaluating?&#160;exp)<br>
&#160;&#160;(cond&#160;((number?&#160;exp)&#160;true)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((string?&#160;exp)&#160;true)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;false)))<br></tt>


� Variables are represented by symbols:


<tt><a name="index_term_4284"></a>(define&#160;(variable?&#160;exp)&#160;(symbol?&#160;exp))<br></tt>


� Quotations have the form <tt>(quote &lt;*text-of-quotation*&gt;)</tt>:<a name="call_footnote_Temp_526" href="#footnote_Temp_526" id="call_footnote_Temp_526"><sup><small>9</small></sup></a>


<tt><a name="index_term_4286"></a>(define&#160;(quoted?&#160;exp)<br>
&#160;&#160;(tagged-list?&#160;exp&#160;'quote))<br>
<br>
<a name="index_term_4288"></a>(define&#160;(text-of-quotation&#160;exp)&#160;(cadr&#160;exp))<br></tt>


<tt>Quoted?</tt> is defined in terms of the procedure <tt>tagged-list?</tt>, which identifies lists beginning with a designated symbol:


<tt><a name="index_term_4290"></a>(define&#160;(tagged-list?&#160;exp&#160;tag)<br>
&#160;&#160;(if&#160;(pair?&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(eq?&#160;(car&#160;exp)&#160;tag)<br>
&#160;&#160;&#160;&#160;&#160;&#160;false))<br></tt>


� Assignments have the form <tt>(set! &lt;*var*&gt; &lt;*value*&gt;)</tt>:


<tt><a name="index_term_4292"></a>(define&#160;(assignment?&#160;exp)<br>
&#160;&#160;(tagged-list?&#160;exp&#160;'set!))<br>
<a name="index_term_4294"></a>(define&#160;(assignment-variable&#160;exp)&#160;(cadr&#160;exp))<br>
<a name="index_term_4296"></a>(define&#160;(assignment-value&#160;exp)&#160;(caddr&#160;exp))<br></tt>


� Definitions have the form


<tt>(define&#160;&lt;*var*&gt;&#160;&lt;*value*&gt;)<br></tt>


or the form


<tt>(define&#160;(&lt;*var*&gt;&#160;&lt;*parameter<sub>1</sub>*&gt;</tt> ... &lt;*parameter<sub>*n*</sub>*&gt;)<br>
&#160;&#160;&lt;*body*&gt;)<br>


<a name="index_term_4298"></a><a name="index_term_4300"></a>The latter form (standard procedure definition) is syntactic sugar for


<tt>(define&#160;&lt;*var*&gt;<br>
&#160;&#160;(lambda&#160;(&lt;*parameter<sub>1</sub>*&gt;</tt> ... &lt;*parameter<sub>*n*</sub>*&gt;)<br>
&#160;&#160;&#160;&#160;&lt;*body*&gt;))<br>


The corresponding syntax procedures are the following:


<tt><a name="index_term_4302"></a>(define&#160;(definition?&#160;exp)<br>
&#160;&#160;(tagged-list?&#160;exp&#160;'define))<br>
<a name="index_term_4304"></a>(define&#160;(definition-variable&#160;exp)<br>
&#160;&#160;(if&#160;(symbol?&#160;(cadr&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cadr&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(caadr&#160;exp)))<br>
<a name="index_term_4306"></a>(define&#160;(definition-value&#160;exp)<br>
&#160;&#160;(if&#160;(symbol?&#160;(cadr&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;(caddr&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(make-lambda&#160;(cdadr&#160;exp)&#160;&#160;&#160;*;&#160;formal&#160;parameters*<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cddr&#160;exp))))&#160;*;&#160;body*<br></tt>


� <tt>Lambda</tt> expressions are lists that begin with the symbol <tt>lambda</tt>:


<tt><a name="index_term_4308"></a>(define&#160;(lambda?&#160;exp)&#160;(tagged-list?&#160;exp&#160;'lambda))<br>
<a name="index_term_4310"></a>(define&#160;(lambda-parameters&#160;exp)&#160;(cadr&#160;exp))<br>
<a name="index_term_4312"></a>(define&#160;(lambda-body&#160;exp)&#160;(cddr&#160;exp))<br></tt>


We also provide a constructor for <tt>lambda</tt> expressions, which is used by <tt>definition-value</tt>, above:


<tt><a name="index_term_4314"></a>(define&#160;(make-lambda&#160;parameters&#160;body)<br>
&#160;&#160;(cons&#160;'lambda&#160;(cons&#160;parameters&#160;body)))<br></tt>


� Conditionals begin with <tt>if</tt> and have a predicate, a consequent, and an (optional) alternative. If the expression has no alternative part, we provide <tt>false</tt> as the alternative.<a name="call_footnote_Temp_527" href="#footnote_Temp_527" id="call_footnote_Temp_527"><sup><small>10</small></sup></a>


<tt><a name="index_term_4316"></a>(define&#160;(if?&#160;exp)&#160;(tagged-list?&#160;exp&#160;'if))<br>
<a name="index_term_4318"></a>(define&#160;(if-predicate&#160;exp)&#160;(cadr&#160;exp))<br>
<a name="index_term_4320"></a>(define&#160;(if-consequent&#160;exp)&#160;(caddr&#160;exp))<br>
<a name="index_term_4322"></a>(define&#160;(if-alternative&#160;exp)<br>
&#160;&#160;(if&#160;(not&#160;(null?&#160;(cdddr&#160;exp)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cadddr&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;'false))<br></tt>


We also provide a constructor for <tt>if</tt> expressions, to be used by <tt>cond-&gt;if</tt> to transform <tt>cond</tt> expressions into <tt>if</tt> expressions:


<tt><a name="index_term_4324"></a>(define&#160;(make-if&#160;predicate&#160;consequent&#160;alternative)<br>
&#160;&#160;(list&#160;'if&#160;predicate&#160;consequent&#160;alternative))<br></tt>


� <tt>Begin</tt> packages a sequence of expressions into a single expression. We include syntax operations on <tt>begin</tt> expressions to extract the actual sequence from the <tt>begin</tt> expression, as well as selectors that return the first expression and the rest of the expressions in the sequence.<a name="call_footnote_Temp_528" href="#footnote_Temp_528" id="call_footnote_Temp_528"><sup><small>11</small></sup></a>


<tt><a name="index_term_4326"></a>(define&#160;(begin?&#160;exp)&#160;(tagged-list?&#160;exp&#160;'begin))<br>
<a name="index_term_4328"></a>(define&#160;(begin-actions&#160;exp)&#160;(cdr&#160;exp))<br>
<a name="index_term_4330"></a>(define&#160;(last-exp?&#160;seq)&#160;(null?&#160;(cdr&#160;seq)))<br>
<a name="index_term_4332"></a>(define&#160;(first-exp&#160;seq)&#160;(car&#160;seq))<br>
<a name="index_term_4334"></a>(define&#160;(rest-exps&#160;seq)&#160;(cdr&#160;seq))<br></tt>


We also include a constructor <tt>sequence-&gt;exp</tt> (for use by <tt>cond-&gt;if</tt>) that transforms a sequence into a single expression, using <tt>begin</tt> if necessary:


<tt><a name="index_term_4336"></a>(define&#160;(sequence-&gt;exp&#160;seq)<br>
&#160;&#160;(cond&#160;((null?&#160;seq)&#160;seq)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((last-exp?&#160;seq)&#160;(first-exp&#160;seq))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(make-begin&#160;seq))))<br>
<a name="index_term_4338"></a>(define&#160;(make-begin&#160;seq)&#160;(cons&#160;'begin&#160;seq))<br></tt>


� A procedure application is any compound expression that is not one of the above expression types. The <tt>car</tt> of the expression is the operator, and the <tt>cdr</tt> is the list of operands:


<tt><a name="index_term_4340"></a>(define&#160;(application?&#160;exp)&#160;(pair?&#160;exp))<br>
<a name="index_term_4342"></a>(define&#160;(operator&#160;exp)&#160;(car&#160;exp))<br>
<a name="index_term_4344"></a>(define&#160;(operands&#160;exp)&#160;(cdr&#160;exp))<br>
<a name="index_term_4346"></a>(define&#160;(no-operands?&#160;ops)&#160;(null?&#160;ops))<br>
<a name="index_term_4348"></a>(define&#160;(first-operand&#160;ops)&#160;(car&#160;ops))<br>
<a name="index_term_4350"></a>(define&#160;(rest-operands&#160;ops)&#160;(cdr&#160;ops))<br></tt>


<a name="%_sec_Temp_529"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_529">Derived expressions</a></h4>

<a name="index_term_4352"></a><a name="index_term_4354"></a><a name="index_term_4356"></a><a name="index_term_4358"></a> Some special forms in our language can be defined in terms of expressions involving other special forms, rather than being implemented directly. One example is <tt>cond</tt>, which can be implemented as a nest of <tt>if</tt> expressions. For example, we can reduce the problem of evaluating the expression


<tt>(cond&#160;((&gt;&#160;x&#160;0)&#160;x)<br>
&#160;&#160;&#160;&#160;&#160;&#160;((=&#160;x&#160;0)&#160;(display&#160;'zero)&#160;0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(-&#160;x)))<br></tt>


to the problem of evaluating the following expression involving <tt>if</tt> and <tt>begin</tt> expressions:


<tt>(if&#160;(&gt;&#160;x&#160;0)<br>
&#160;&#160;&#160;&#160;x<br>
&#160;&#160;&#160;&#160;(if&#160;(=&#160;x&#160;0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(begin&#160;(display&#160;'zero)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(-&#160;x)))<br></tt>


Implementing the evaluation of <tt>cond</tt> in this way simplifies the evaluator because it reduces the number of special forms for which the evaluation process must be explicitly specified.


We include syntax procedures that extract the parts of a <tt>cond</tt> expression, and a procedure <tt>cond-&gt;if</tt> that transforms <tt>cond</tt> expressions into <tt>if</tt> expressions. A case analysis begins with <tt>cond</tt> and has a list of predicate-action clauses. A clause is an <tt>else</tt> clause if its predicate is the symbol <tt>else</tt>.<a name="call_footnote_Temp_530" href="#footnote_Temp_530" id="call_footnote_Temp_530"><sup><small>12</small></sup></a>


<tt><a name="index_term_4360"></a>(define&#160;(cond?&#160;exp)&#160;(tagged-list?&#160;exp&#160;'cond))<br>
<a name="index_term_4362"></a>(define&#160;(cond-clauses&#160;exp)&#160;(cdr&#160;exp))<br>
<a name="index_term_4364"></a>(define&#160;(cond-else-clause?&#160;clause)<br>
&#160;&#160;(eq?&#160;(cond-predicate&#160;clause)&#160;'else))<br>
<a name="index_term_4366"></a>(define&#160;(cond-predicate&#160;clause)&#160;(car&#160;clause))<br>
<a name="index_term_4368"></a>(define&#160;(cond-actions&#160;clause)&#160;(cdr&#160;clause))<br>
<a name="index_term_4370"></a>(define&#160;(cond-&gt;if&#160;exp)<br>
&#160;&#160;(expand-clauses&#160;(cond-clauses&#160;exp)))<br>
<br>
<a name="index_term_4372"></a>(define&#160;(expand-clauses&#160;clauses)<br>
&#160;&#160;(if&#160;(null?&#160;clauses)<br>
&#160;&#160;&#160;&#160;&#160;&#160;'false&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*;&#160;no&#160;<tt>else</tt>&#160;clause*<br>
&#160;&#160;&#160;&#160;&#160;&#160;(let&#160;((first&#160;(car&#160;clauses))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(rest&#160;(cdr&#160;clauses)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;(cond-else-clause?&#160;first)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;(null?&#160;rest)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(sequence-&gt;exp&#160;(cond-actions&#160;first))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;ELSE&#160;clause&#160;isn't&#160;last&#160;--&#160;COND-&gt;IF&quot;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;clauses))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(make-if&#160;(cond-predicate&#160;first)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(sequence-&gt;exp&#160;(cond-actions&#160;first))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(expand-clauses&#160;rest))))))<br></tt>


Expressions (such as <tt>cond</tt>) that we choose to implement as syntactic transformations are called *derived expressions*. <tt>Let</tt> expressions are also derived expressions (see exercise&#160;<a href="#%_thm_4.6">4.6</a>).<a name="call_footnote_Temp_531" href="#footnote_Temp_531" id="call_footnote_Temp_531"><sup><small>13</small></sup></a>


<a name="%_thm_4.2"></a> **Exercise 4.2.**&#160;&#160;<a name="index_term_4384"></a>Louis Reasoner plans to reorder the <tt>cond</tt> clauses in <tt>eval</tt> so that the clause for procedure applications appears before the clause for assignments. He argues that this will make the interpreter more efficient: Since programs usually contain more applications than assignments, definitions, and so on, his modified <tt>eval</tt> will usually check fewer clauses than the original <tt>eval</tt> before identifying the type of an expression.


a. What is wrong with Louis's plan? (Hint: What will Louis's evaluator do with the expression <tt>(define x 3)</tt>?)


<a name="index_term_4386"></a>b. Louis is upset that his plan didn't work. He is willing to go to any lengths to make his evaluator recognize procedure applications before it checks for most other kinds of expressions. Help him by changing the syntax of the evaluated language so that procedure applications start with <tt>call</tt>. For example, instead of <tt>(factorial 3)</tt> we will now have to write <tt>(call factorial 3)</tt> and instead of <tt>(+ 1 2)</tt> we will have to write <tt>(call + 1 2)</tt>.


<a name="%_thm_4.3"></a> **Exercise 4.3.**&#160;&#160;<a name="index_term_4388"></a><a name="index_term_4390"></a><a name="index_term_4392"></a>Rewrite <tt>eval</tt> so that the dispatch is done in data-directed style. Compare this with the data-directed differentiation procedure of exercise&#160;<a href="chapter_2_section_4.html#exercise_2_73">2.73</a>. (You may use the <tt>car</tt> of a compound expression as the type of the expression, as is appropriate for the syntax implemented in this section.) .


<a name="%_thm_4.4"></a> **Exercise 4.4.**&#160;&#160;<a name="index_term_4394"></a><a name="index_term_4396"></a><a name="index_term_4398"></a>Recall the definitions of the special forms <tt>and</tt> and <tt>or</tt> from chapter&#160;1:

<ul>
  <li><tt>and</tt>: The expressions are evaluated from left to right. If any expression evaluates to false, false is returned; any remaining expressions are not evaluated. If all the expressions evaluate to true values, the value of the last expression is returned. If there are no expressions then true is returned.</li>

  <li><tt>or</tt>: The expressions are evaluated from left to right. If any expression evaluates to a true value, that value is returned; any remaining expressions are not evaluated. If all expressions evaluate to false, or if there are no expressions, then false is returned.</li>
</ul>

Install <tt>and</tt> and <tt>or</tt> as new special forms for the evaluator by defining appropriate syntax procedures and evaluation procedures <tt>eval-and</tt> and <tt>eval-or</tt>. Alternatively, show how to implement <tt>and</tt> and <tt>or</tt> as derived expressions.


<a name="%_thm_4.5"></a> **Exercise 4.5.**&#160;&#160;<a name="index_term_4400"></a><a name="index_term_4402"></a><a name="index_term_4404"></a>Scheme allows an additional syntax for <tt>cond</tt> clauses, <tt>(&lt;*test*&gt; =&gt; &lt;*recipient*&gt;)</tt>. If &lt;*test*&gt; evaluates to a true value, then &lt;*recipient*&gt; is evaluated. Its value must be a procedure of one argument; this procedure is then invoked on the value of the &lt;*test*&gt;, and the result is returned as the value of the <tt>cond</tt> expression. For example


<tt>(cond&#160;((assoc&#160;'b&#160;'((a&#160;1)&#160;(b&#160;2)))&#160;=&gt;&#160;cadr)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;false))<br></tt>


returns 2. Modify the handling of <tt>cond</tt> so that it supports this extended syntax.


<a name="%_thm_4.6"></a> **Exercise 4.6.**&#160;&#160;<a name="index_term_4406"></a><tt>Let</tt> expressions are derived expressions, because


<tt>(let&#160;((&lt;*var<sub>1</sub>*&gt;&#160;&lt;*exp<sub>1</sub>*&gt;)&#160;</tt>... (&lt;*var<sub>*n*</sub>*&gt;&#160;&lt;*exp<sub>*n*</sub>*&gt;))<br>
&#160;&#160;&lt;*body*&gt;)<br>


is equivalent to


<tt>((lambda&#160;(&lt;*var<sub>1</sub>*&gt;&#160;</tt>... &lt;*var<sub>*n*</sub>*&gt;)<br>
&#160;&#160;&#160;&lt;*body*&gt;)<br>
&#160;&lt;*exp<sub>1</sub>*&gt;<br>
&#160;<img src="img/book_18.gif" border="0"><br>
&#160;&lt;*exp<sub>*n*</sub>*&gt;)<br>


Implement a syntactic transformation <tt>let-&gt;combination</tt> that reduces evaluating <tt>let</tt> expressions to evaluating combinations of the type shown above, and add the appropriate clause to <tt>eval</tt> to handle <tt>let</tt> expressions.


<a name="%_thm_4.7"></a> **Exercise 4.7.**&#160;&#160;<a name="index_term_4408"></a><a name="index_term_4410"></a><a name="index_term_4412"></a><tt>Let*</tt> is similar to <tt>let</tt>, except that the bindings of the <tt>let</tt> variables are performed sequentially from left to right, and each binding is made in an environment in which all of the preceding bindings are visible. For example


<tt>(let*&#160;((x&#160;3)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(y&#160;(+&#160;x&#160;2))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(z&#160;(+&#160;x&#160;y&#160;5)))<br>
&#160;&#160;(*&#160;x&#160;z))<br></tt>


returns 39. Explain how a <tt>let*</tt> expression can be rewritten as a set of nested <tt>let</tt> expressions, and write a procedure <tt>let*-&gt;nested-lets</tt> that performs this transformation. If we have already implemented <tt>let</tt> (exercise&#160;<a href="#%_thm_4.6">4.6</a>) and we want to extend the evaluator to handle <tt>let*</tt>, is it sufficient to add a clause to <tt>eval</tt> whose action is


<tt>(eval&#160;(let*-&gt;nested-lets&#160;exp)&#160;env)<br></tt>


or must we explicitly expand <tt>let*</tt> in terms of non-derived expressions?


<a name="%_thm_4.8"></a> **Exercise 4.8.**&#160;&#160;<a name="index_term_4414"></a><a name="index_term_4416"></a><a name="index_term_4418"></a><a name="index_term_4420"></a>"Named <tt>let</tt>" is a variant of <tt>let</tt> that has the form


<tt>(let&#160;&lt;*var*&gt;&#160;&lt;*bindings*&gt;&#160;&lt;*body*&gt;)<br></tt>


The &lt;*bindings*&gt; and &lt;*body*&gt; are just as in ordinary <tt>let</tt>, except that &lt;*var*&gt; is bound within &lt;*body*&gt; to a procedure whose body is &lt;*body*&gt; and whose parameters are the variables in the &lt;*bindings*&gt;. Thus, one can repeatedly execute the &lt;*body*&gt; by invoking the procedure named &lt;*var*&gt;. For example, the iterative Fibonacci procedure (section&#160;<a href="chapter_1_section_2.html#%_sec_1.2.2">1.2.2</a>) can be rewritten using named <tt>let</tt> as follows:


<tt><a name="index_term_4422"></a>(define&#160;(fib&#160;n)<br>
&#160;&#160;(let&#160;fib-iter&#160;((a&#160;1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(b&#160;0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(count&#160;n))<br>
&#160;&#160;&#160;&#160;(if&#160;(=&#160;count&#160;0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;b<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(fib-iter&#160;(+&#160;a&#160;b)&#160;a&#160;(-&#160;count&#160;1)))))<br></tt>


Modify <tt>let-&gt;combination</tt> of exercise&#160;<a href="#%_thm_4.6">4.6</a> to also support named <tt>let</tt>.


<a name="%_thm_4.9"></a> **Exercise 4.9.**&#160;&#160;<a name="index_term_4424"></a><a name="index_term_4426"></a>Many languages support a variety of iteration constructs, such as <tt>do</tt>, <tt>for</tt>, <tt>while</tt>, and <tt>until</tt>. In Scheme, iterative processes can be expressed in terms of ordinary procedure calls, so special iteration constructs provide no essential gain in computational power. On the other hand, such constructs are often convenient. Design some iteration constructs, give examples of their use, and show how to implement them as derived expressions.


<a name="%_thm_4.10"></a> **Exercise 4.10.**&#160;&#160;<a name="index_term_4428"></a><a name="index_term_4430"></a>By using data abstraction, we were able to write an <tt>eval</tt> procedure that is independent of the particular syntax of the language to be evaluated. To illustrate this, design and implement a new syntax for Scheme by modifying the procedures in this section, without changing <tt>eval</tt> or <tt>apply</tt>.


<a name="%_sec_4.1.3"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_4.1.3">4.1.3&#160;&#160;Evaluator Data Structures</a></h3>

In addition to defining the external syntax of expressions, the evaluator implementation must also define the data structures that the evaluator manipulates internally, as part of the execution of a program, such as the representation of procedures and environments and the representation of true and false.


<a name="%_sec_Temp_541"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_541">Testing of predicates</a></h4>

<a name="index_term_4432"></a>For conditionals, we accept anything to be true that is not the explicit <tt>false</tt> object.


<tt><a name="index_term_4434"></a>(define&#160;(true?&#160;x)<br>
&#160;&#160;(not&#160;(eq?&#160;x&#160;false)))<br>
<a name="index_term_4436"></a>(define&#160;(false?&#160;x)<br>
&#160;&#160;(eq?&#160;x&#160;false))<br></tt>


<a name="%_sec_Temp_542"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_542">Representing procedures</a></h4>

<a name="index_term_4438"></a> To handle primitives, we assume that we have available the following procedures:

<ul>
  <li><a name="index_term_4440"></a><tt>(apply-primitive-procedure &lt;*proc*&gt; &lt;*args*&gt;)</tt><br>
  applies the given primitive procedure to the argument values in the list &lt;*args*&gt; and returns the result of the application.</li>

  <li><a name="index_term_4442"></a><tt>(primitive-procedure? &lt;*proc*&gt;)</tt><br>
  tests whether &lt;*proc*&gt; is a primitive procedure.</li>
</ul>

These mechanisms for handling primitives are further described in section&#160;<a href="#%_sec_4.1.4">4.1.4</a>.


Compound procedures are constructed from parameters, procedure bodies, and environments using the constructor <tt>make-procedure</tt>:


<tt><a name="index_term_4444"></a>(define&#160;(make-procedure&#160;parameters&#160;body&#160;env)<br>
&#160;&#160;(list&#160;'procedure&#160;parameters&#160;body&#160;env))<br>
<a name="index_term_4446"></a>(define&#160;(compound-procedure?&#160;p)<br>
&#160;&#160;(tagged-list?&#160;p&#160;'procedure))<br>
<a name="index_term_4448"></a>(define&#160;(procedure-parameters&#160;p)&#160;(cadr&#160;p))<br>
<a name="index_term_4450"></a>(define&#160;(procedure-body&#160;p)&#160;(caddr&#160;p))<br>
<a name="index_term_4452"></a>(define&#160;(procedure-environment&#160;p)&#160;(cadddr&#160;p))<br></tt>


<a name="%_sec_Temp_543"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_543">Operations on Environments</a></h4>

<a name="index_term_4454"></a> The evaluator needs operations for manipulating environments. As explained in section&#160;<a href="chapter_3_section_2.html#%_sec_3.2">3.2</a>, an environment is a sequence of frames, where each frame is a table of bindings that associate variables with their corresponding values. We use the following operations for manipulating environments:

<ul>
  <li style="list-style: none"><a name="index_term_4456"></a></li>

  <li><tt>(lookup-variable-value &lt;*var*&gt; &lt;*env*&gt;)</tt><br>
  returns the value that is bound to the symbol &lt;*var*&gt; in the environment &lt;*env*&gt;, or signals an error if the variable is unbound.</li>

  <li><a name="index_term_4458"></a><tt>(extend-environment &lt;*variables*&gt; &lt;*values*&gt; &lt;*base-env*&gt;)</tt><br>
  returns a new environment, consisting of a new frame in which the symbols in the list &lt;*variables*&gt; are bound to the corresponding elements in the list &lt;*values*&gt;, where the enclosing environment is the environment &lt;*base-env*&gt;.</li>

  <li><a name="index_term_4460"></a><tt>(define-variable! &lt;*var*&gt; &lt;*value*&gt; &lt;*env*&gt;)</tt><br>
  adds to the first frame in the environment &lt;*env*&gt; a new binding that associates the variable &lt;*var*&gt; with the value &lt;*value*&gt;.</li>

  <li><a name="index_term_4462"></a><tt>(set-variable-value! &lt;*var*&gt; &lt;*value*&gt; &lt;*env*&gt;)</tt><br>
  changes the binding of the variable &lt;*var*&gt; in the environment &lt;*env*&gt; so that the variable is now bound to the value &lt;*value*&gt;, or signals an error if the variable is unbound.</li>
</ul>

<a name="index_term_4464"></a>To implement these operations we represent an environment as a list of frames. The enclosing environment of an environment is the <tt>cdr</tt> of the list. The empty environment is simply the empty list.


<tt><a name="index_term_4466"></a>(define&#160;(enclosing-environment&#160;env)&#160;(cdr&#160;env))<br>
<a name="index_term_4468"></a>(define&#160;(first-frame&#160;env)&#160;(car&#160;env))<br>
(define&#160;the-empty-environment&#160;'())<br></tt>


Each frame of an environment is represented as a pair of lists: a list of the variables bound in that frame and a list of the associated values.<a name="call_footnote_Temp_544" href="#footnote_Temp_544" id="call_footnote_Temp_544"><sup><small>14</small></sup></a>


<tt><a name="index_term_4470"></a>(define&#160;(make-frame&#160;variables&#160;values)<br>
&#160;&#160;(cons&#160;variables&#160;values))<br>
<a name="index_term_4472"></a>(define&#160;(frame-variables&#160;frame)&#160;(car&#160;frame))<br>
<a name="index_term_4474"></a>(define&#160;(frame-values&#160;frame)&#160;(cdr&#160;frame))<br>
<a name="index_term_4476"></a>(define&#160;(add-binding-to-frame!&#160;var&#160;val&#160;frame)<br>
&#160;&#160;(set-car!&#160;frame&#160;(cons&#160;var&#160;(car&#160;frame)))<br>
&#160;&#160;(set-cdr!&#160;frame&#160;(cons&#160;val&#160;(cdr&#160;frame))))<br></tt>


To extend an environment by a new frame that associates variables with values, we make a frame consisting of the list of variables and the list of values, and we adjoin this to the environment. We signal an error if the number of variables does not match the number of values.


<tt><a name="index_term_4478"></a>(define&#160;(extend-environment&#160;vars&#160;vals&#160;base-env)<br>
&#160;&#160;(if&#160;(=&#160;(length&#160;vars)&#160;(length&#160;vals))<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cons&#160;(make-frame&#160;vars&#160;vals)&#160;base-env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;(&lt;&#160;(length&#160;vars)&#160;(length&#160;vals))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Too&#160;many&#160;arguments&#160;supplied&quot;&#160;vars&#160;vals)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Too&#160;few&#160;arguments&#160;supplied&quot;&#160;vars&#160;vals))))<br></tt>


To look up a variable in an environment, we scan the list of variables in the first frame. If we find the desired variable, we return the corresponding element in the list of values. If we do not find the variable in the current frame, we search the enclosing environment, and so on. If we reach the empty environment, we signal an "unbound variable" error.


<tt><a name="index_term_4480"></a>(define&#160;(lookup-variable-value&#160;var&#160;env)<br>
&#160;&#160;(define&#160;(env-loop&#160;env)<br>
&#160;&#160;&#160;&#160;(define&#160;(scan&#160;vars&#160;vals)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cond&#160;((null?&#160;vars)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(env-loop&#160;(enclosing-environment&#160;env)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((eq?&#160;var&#160;(car&#160;vars))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(car&#160;vals))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(scan&#160;(cdr&#160;vars)&#160;(cdr&#160;vals)))))<br>
&#160;&#160;&#160;&#160;(if&#160;(eq?&#160;env&#160;the-empty-environment)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Unbound&#160;variable&quot;&#160;var)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(let&#160;((frame&#160;(first-frame&#160;env)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(scan&#160;(frame-variables&#160;frame)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(frame-values&#160;frame)))))<br>
&#160;&#160;(env-loop&#160;env))<br></tt>


To set a variable to a new value in a specified environment, we scan for the variable, just as in <tt>lookup-variable-value</tt>, and change the corresponding value when we find it.


<tt><a name="index_term_4482"></a>(define&#160;(set-variable-value!&#160;var&#160;val&#160;env)<br>
&#160;&#160;(define&#160;(env-loop&#160;env)<br>
&#160;&#160;&#160;&#160;(define&#160;(scan&#160;vars&#160;vals)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cond&#160;((null?&#160;vars)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(env-loop&#160;(enclosing-environment&#160;env)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((eq?&#160;var&#160;(car&#160;vars))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(set-car!&#160;vals&#160;val))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(scan&#160;(cdr&#160;vars)&#160;(cdr&#160;vals)))))<br>
&#160;&#160;&#160;&#160;(if&#160;(eq?&#160;env&#160;the-empty-environment)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Unbound&#160;variable&#160;--&#160;SET!&quot;&#160;var)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(let&#160;((frame&#160;(first-frame&#160;env)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(scan&#160;(frame-variables&#160;frame)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(frame-values&#160;frame)))))<br>
&#160;&#160;(env-loop&#160;env))<br></tt>


To define a variable, we search the first frame for a binding for the variable, and change the binding if it exists (just as in <tt>set-variable-value!</tt>). If no such binding exists, we adjoin one to the first frame.


<tt><a name="index_term_4484"></a>(define&#160;(define-variable!&#160;var&#160;val&#160;env)<br>
&#160;&#160;(let&#160;((frame&#160;(first-frame&#160;env)))<br>
&#160;&#160;&#160;&#160;(define&#160;(scan&#160;vars&#160;vals)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cond&#160;((null?&#160;vars)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(add-binding-to-frame!&#160;var&#160;val&#160;frame))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((eq?&#160;var&#160;(car&#160;vars))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(set-car!&#160;vals&#160;val))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(scan&#160;(cdr&#160;vars)&#160;(cdr&#160;vals)))))<br>
&#160;&#160;&#160;&#160;(scan&#160;(frame-variables&#160;frame)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(frame-values&#160;frame))))<br></tt>


<a name="index_term_4486"></a>The method described here is only one of many plausible ways to represent environments. Since we used data abstraction to isolate the rest of the evaluator from the detailed choice of representation, we could change the environment representation if we wanted to. (See exercise&#160;<a href="#%_thm_4.11">4.11</a>.) In a production-quality Lisp system, the speed of the evaluator's environment operations -- especially that of variable lookup -- has a major impact on the performance of the system. The representation described here, although conceptually simple, is not efficient and would not ordinarily be used in a production system.<a name="call_footnote_Temp_545" href="#footnote_Temp_545" id="call_footnote_Temp_545"><sup><small>15</small></sup></a>


<a name="%_thm_4.11"></a> **Exercise 4.11.**&#160;&#160;Instead of representing a frame as a pair of lists, we can represent a frame as a list of bindings, where each binding is a name-value pair. Rewrite the environment operations to use this alternative representation.


<a name="%_thm_4.12"></a> **Exercise 4.12.**&#160;&#160;The procedures <tt>set-variable-value!</tt>, <tt>define-variable!</tt>, and <tt>lookup-variable-value</tt> can be expressed in terms of more abstract procedures for traversing the environment structure. Define abstractions that capture the common patterns and redefine the three procedures in terms of these abstractions.


<a name="%_thm_4.13"></a> **Exercise 4.13.**&#160;&#160;Scheme allows us to create new bindings for variables by means of <tt>define</tt>, but provides no way to get rid of bindings. Implement for the evaluator a special form <tt>make-unbound!</tt> that removes the binding of a given symbol from the environment in which the <tt>make-unbound!</tt> expression is evaluated. This problem is not completely specified. For example, should we remove only the binding in the first frame of the environment? Complete the specification and justify any choices you make.


<a name="%_sec_4.1.4"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_4.1.4">4.1.4&#160;&#160;Running the Evaluator as a Program</a></h3>

<a name="index_term_4492"></a> Given the evaluator, we have in our hands a description (expressed in Lisp) of the process by which Lisp expressions are evaluated. One advantage of expressing the evaluator as a program is that we can run the program. This gives us, running within Lisp, a working model of how Lisp itself evaluates expressions. This can serve as a framework for experimenting with evaluation rules, as we shall do later in this chapter.


<a name="index_term_4494"></a>Our evaluator program reduces expressions ultimately to the application of primitive procedures. Therefore, all that we need to run the evaluator is to create a mechanism that calls on the underlying Lisp system to model the application of primitive procedures.


There must be a binding for each primitive procedure name, so that when <tt>eval</tt> evaluates the operator of an application of a primitive, it will find an object to pass to <tt>apply</tt>. We thus set up a <a name="index_term_4496"></a><a name="index_term_4498"></a>global environment that associates unique objects with the names of the primitive procedures that can appear in the expressions we will be evaluating. The global environment also includes bindings for the symbols <a name="index_term_4500"></a><tt>true</tt> and <tt>false</tt>, so that they can be used as variables in expressions to be evaluated.


<tt><a name="index_term_4502"></a>(define&#160;(setup-environment)<br>
&#160;&#160;(let&#160;((initial-env<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(extend-environment&#160;(primitive-procedure-names)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(primitive-procedure-objects)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;the-empty-environment)))<br>
&#160;&#160;&#160;&#160;(define-variable!&#160;'true&#160;true&#160;initial-env)<br>
&#160;&#160;&#160;&#160;(define-variable!&#160;'false&#160;false&#160;initial-env)<br>
&#160;&#160;&#160;&#160;initial-env))<br>
<a name="index_term_4504"></a>(define&#160;the-global-environment&#160;(setup-environment))<br></tt>


It does not matter how we represent the primitive procedure objects, so long as <tt>apply</tt> can identify and apply them by using the procedures <tt>primitive-procedure?</tt> and <tt>apply-primitive-procedure</tt>. We have chosen to represent a primitive procedure as a list beginning with the symbol <tt>primitive</tt> and containing a procedure in the underlying Lisp that implements that primitive.


<tt><a name="index_term_4506"></a>(define&#160;(primitive-procedure?&#160;proc)<br>
&#160;&#160;(tagged-list?&#160;proc&#160;'primitive))<br>
<br>
<a name="index_term_4508"></a>(define&#160;(primitive-implementation&#160;proc)&#160;(cadr&#160;proc))<br></tt>


<tt>Setup-environment</tt> will get the primitive names and implementation procedures from a list:<a name="call_footnote_Temp_549" href="#footnote_Temp_549" id="call_footnote_Temp_549"><sup><small>16</small></sup></a>


<tt>(define&#160;primitive-procedures<br>
&#160;&#160;(list&#160;(list&#160;'car&#160;car)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(list&#160;'cdr&#160;cdr)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(list&#160;'cons&#160;cons)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(list&#160;'null?&#160;null?)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&lt;*more&#160;primitives*&gt;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;))<br>
<a name="index_term_4510"></a>(define&#160;(primitive-procedure-names)<br>
&#160;&#160;(map&#160;car<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;primitive-procedures))<br>
<br>
(define&#160;(primitive-procedure-objects)<br>
<a name="index_term_4512"></a>&#160;&#160;(map&#160;(lambda&#160;(proc)&#160;(list&#160;'primitive&#160;(cadr&#160;proc)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;primitive-procedures))<br></tt>


To apply a primitive procedure, we simply apply the implementation procedure to the arguments, using the underlying Lisp system:<a name="call_footnote_Temp_550" href="#footnote_Temp_550" id="call_footnote_Temp_550"><sup><small>17</small></sup></a>


<tt><a name="index_term_4514"></a>(define&#160;(apply-primitive-procedure&#160;proc&#160;args)<br>
&#160;&#160;(apply-in-underlying-scheme<br>
&#160;&#160;&#160;(primitive-implementation&#160;proc)&#160;args))<br></tt>


<a name="index_term_4516"></a><a name="index_term_4518"></a>For convenience in running the metacircular evaluator, we provide a *driver loop* that models the read-eval-print loop of the underlying Lisp system. It prints a <a name="index_term_4520"></a>*prompt*, reads an input expression, evaluates this expression in the global environment, and prints the result. We precede each printed result by an *output prompt* so as to distinguish the value of the expression from other output that may be printed.<a name="call_footnote_Temp_551" href="#footnote_Temp_551" id="call_footnote_Temp_551"><sup><small>18</small></sup></a>


<tt><a name="index_term_4530"></a>(define&#160;input-prompt&#160;&quot;;;;&#160;M-Eval&#160;input:&quot;)<br>
(define&#160;output-prompt&#160;&quot;;;;&#160;M-Eval&#160;value:&quot;)<br>
<a name="index_term_4532"></a>(define&#160;(driver-loop)<br>
&#160;&#160;(prompt-for-input&#160;input-prompt)<br>
&#160;&#160;(let&#160;((input&#160;(read)))<br>
&#160;&#160;&#160;&#160;(let&#160;((output&#160;(eval&#160;input&#160;the-global-environment)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;(announce-output&#160;output-prompt)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(user-print&#160;output)))<br>
&#160;&#160;(driver-loop))<br>
<a name="index_term_4534"></a>(define&#160;(prompt-for-input&#160;string)<br>
&#160;&#160;(newline)&#160;(newline)&#160;(display&#160;string)&#160;(newline))<br>
<br>
<a name="index_term_4536"></a>(define&#160;(announce-output&#160;string)<br>
&#160;&#160;(newline)&#160;(display&#160;string)&#160;(newline))<br></tt>


We use a special printing procedure, <tt>user-print</tt>, to avoid printing the environment part of a compound procedure, which may be a very long list (or may even contain cycles).


<tt><a name="index_term_4538"></a>(define&#160;(user-print&#160;object)<br>
&#160;&#160;(if&#160;(compound-procedure?&#160;object)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(display&#160;(list&#160;'compound-procedure<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(procedure-parameters&#160;object)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(procedure-body&#160;object)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;'&lt;procedure-env&gt;))<br>
&#160;&#160;&#160;&#160;&#160;&#160;(display&#160;object)))<br></tt>


Now all we need to do to run the evaluator is to initialize the global environment and start the driver loop. Here is a sample interaction:


<tt>(define&#160;the-global-environment&#160;(setup-environment))<br>
(driver-loop)<br>
<i>;;;&#160;M-Eval&#160;input:</i><br>
(define&#160;(append&#160;x&#160;y)<br>
&#160;&#160;(if&#160;(null?&#160;x)<br>
&#160;&#160;&#160;&#160;&#160;&#160;y<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cons&#160;(car&#160;x)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(append&#160;(cdr&#160;x)&#160;y))))<br>
<i>;;;&#160;M-Eval&#160;value:</i><br>
<i>ok</i><br>
<i>;;;&#160;M-Eval&#160;input:</i><br>
(append&#160;'(a&#160;b&#160;c)&#160;'(d&#160;e&#160;f))<br>
<i>;;;&#160;M-Eval&#160;value:</i><br>
<i>(a&#160;b&#160;c&#160;d&#160;e&#160;f)</i><br></tt>


<a name="%_thm_4.14"></a> **Exercise 4.14.**&#160;&#160;Eva Lu Ator and Louis Reasoner are each experimenting with the metacircular evaluator. Eva types in the definition of <tt>map</tt>, and runs some test programs that use it. They work fine. Louis, in contrast, has installed the system version of <tt>map</tt> as a primitive for the metacircular evaluator. When he tries it, things go terribly wrong. Explain why Louis's <tt>map</tt> fails even though Eva's works.


<a name="%_sec_4.1.5"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_4.1.5">4.1.5&#160;&#160;Data as Programs</a></h3>

<a name="index_term_4540"></a><a name="index_term_4542"></a> In thinking about a Lisp program that evaluates Lisp expressions, an analogy might be helpful. One operational view of the meaning of a program is that a <a name="index_term_4544"></a>program is a description of an abstract (perhaps infinitely large) machine. For example, consider the familiar program to compute factorials:


<tt>(define&#160;(factorial&#160;n)<br>
&#160;&#160;(if&#160;(=&#160;n&#160;1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;1<br>
&#160;&#160;&#160;&#160;&#160;&#160;(*&#160;(factorial&#160;(-&#160;n&#160;1))&#160;n)))<br></tt>


<a name="index_term_4546"></a>We may regard this program as the description of a machine containing parts that decrement, multiply, and test for equality, together with a two-position switch and another factorial machine. (The factorial machine is infinite because it contains another factorial machine within it.) Figure&#160;<a href="#%_fig_4.2">4.2</a> is a flow diagram for the factorial machine, showing how the parts are wired together.


<a name="%_fig_4.2"></a>

<div align="left">
  <div align="left">
    **Figure 4.2:**&#160;&#160;The factorial program, viewed as an abstract machine.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_4_image_02.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="index_term_4548"></a>In a similar way, we can regard the evaluator as a very special machine that takes as input a description of a machine. Given this input, the evaluator configures itself to emulate the machine described. For example, if we feed our evaluator the definition of <tt>factorial</tt>, as shown in figure&#160;<a href="#%_fig_4.3">4.3</a>, the evaluator will be able to compute factorials.


<a name="%_fig_4.3"></a>

<div align="left">
  <div align="left">
    **Figure 4.3:**&#160;&#160;The evaluator emulating a factorial machine.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_4_image_03.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="index_term_4550"></a><a name="index_term_4552"></a>From this perspective, our evaluator is seen to be a *universal machine*. It mimics other machines when these are described as Lisp programs.<a name="call_footnote_Temp_553" href="#footnote_Temp_553" id="call_footnote_Temp_553"><sup><small>19</small></sup></a> This is striking. Try to imagine an analogous evaluator for electrical circuits. This would be a circuit that takes as input a signal encoding the plans for some other circuit, such as a filter. Given this input, the circuit evaluator would then behave like a filter with the same description. Such a universal electrical circuit is almost unimaginably complex. It is remarkable that the program evaluator is a rather simple program.<a name="call_footnote_Temp_554" href="#footnote_Temp_554" id="call_footnote_Temp_554"><sup><small>20</small></sup></a>


Another striking aspect of the evaluator is that it acts as a bridge between the data objects that are manipulated by our programming language and the programming language itself. Imagine that the evaluator program (implemented in Lisp) is running, and that a user is typing expressions to the evaluator and observing the results. From the perspective of the user, an input expression such as <tt>(* x x)</tt> is an expression in the programming language, which the evaluator should execute. From the perspective of the evaluator, however, the expression is simply a list (in this case, a list of three symbols: <tt>*</tt>, <tt>x</tt>, and <tt>x</tt>) that is to be manipulated according to a well-defined set of rules.


That the user's programs are the evaluator's data need not be a source of confusion. In fact, it is sometimes convenient to ignore this distinction, and to give the user the ability to explicitly evaluate a data object as a Lisp expression, by making <tt>eval</tt> available for use in programs. Many Lisp dialects provide a <a name="index_term_4572"></a><a name="index_term_4574"></a>primitive <tt>eval</tt> procedure that takes as arguments an expression and an environment and evaluates the expression relative to the environment.<a name="call_footnote_Temp_555" href="#footnote_Temp_555" id="call_footnote_Temp_555"><sup><small>21</small></sup></a> Thus,


<tt>(eval&#160;'(*&#160;5&#160;5)&#160;user-initial-environment)<br></tt>


and


<tt>(eval&#160;(cons&#160;'*&#160;(list&#160;5&#160;5))&#160;user-initial-environment)<br></tt>


will both return 25.<a name="call_footnote_Temp_556" href="#footnote_Temp_556" id="call_footnote_Temp_556"><sup><small>22</small></sup></a>


<a name="%_thm_4.15"></a> **Exercise 4.15.**&#160;&#160;<a name="index_term_4588"></a>Given a one-argument procedure <tt>p</tt> and an object <tt>a</tt>, <tt>p</tt> is said to "halt" on <tt>a</tt> if evaluating the expression <tt>(p a)</tt> returns a value (as opposed to terminating with an error message or running forever). Show that it is impossible to write a procedure <tt>halts?</tt> that correctly determines whether <tt>p</tt> halts on <tt>a</tt> for any procedure <tt>p</tt> and object <tt>a</tt>. Use the following reasoning: If you had such a procedure <tt>halts?</tt>, you could implement the following program:


<tt>(define&#160;(run-forever)&#160;(run-forever))<br>
<br>
(define&#160;(try&#160;p)<br>
&#160;&#160;(if&#160;(halts?&#160;p&#160;p)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(run-forever)<br>
&#160;&#160;&#160;&#160;&#160;&#160;'halted))<br></tt>


Now consider evaluating the expression <tt>(try try)</tt> and show that any possible outcome (either halting or running forever) violates the intended behavior of <tt>halts?</tt>.<a name="call_footnote_Temp_558" href="#footnote_Temp_558" id="call_footnote_Temp_558"><sup><small>23</small></sup></a>


<a name="%_sec_4.1.6"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_4.1.6">4.1.6&#160;&#160;Internal Definitions</a></h3>

<a name="index_term_4598"></a><a name="index_term_4600"></a> <a name="index_term_4602"></a>Our environment model of evaluation and our metacircular evaluator execute definitions in sequence, extending the environment frame one definition at a time. This is particularly convenient for interactive program development, in which the programmer needs to freely mix the application of procedures with the definition of new procedures. However, if we think carefully about the internal definitions used to implement block structure (introduced in section&#160;<a href="chapter_1_section_1.html#%_sec_1.1.8">1.1.8</a>), we will find that name-by-name extension of the environment may not be the best way to define local variables.


Consider a procedure with internal definitions, such as


<tt>(define&#160;(f&#160;x)<br>
&#160;&#160;(define&#160;(even?&#160;n)<br>
&#160;&#160;&#160;&#160;(if&#160;(=&#160;n&#160;0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;true<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(odd?&#160;(-&#160;n&#160;1))))<br>
&#160;&#160;(define&#160;(odd?&#160;n)<br>
&#160;&#160;&#160;&#160;(if&#160;(=&#160;n&#160;0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;false<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(even?&#160;(-&#160;n&#160;1))))<br>
&#160;&#160;&lt;*rest&#160;of&#160;body&#160;of&#160;<tt>f</tt>*&gt;)<br></tt>


Our intention here is that the name <tt>odd?</tt> in the body of the procedure <tt>even?</tt> should refer to the procedure <tt>odd?</tt> that is defined after <tt>even?</tt>. The scope of the name <tt>odd?</tt> is the entire body of <tt>f</tt>, not just the portion of the body of <tt>f</tt> starting at the point where the <tt>define</tt> for <tt>odd?</tt> occurs. Indeed, when we consider that <tt>odd?</tt> is itself defined in terms of <tt>even?</tt> -- so that <tt>even?</tt> and <tt>odd?</tt> are mutually recursive procedures -- we see that the only satisfactory interpretation of the two <tt>define</tt>s is to regard them as if the names <tt>even?</tt> and <tt>odd?</tt> were being added to the environment simultaneously. More generally, in block structure, the scope of a local name is the entire procedure body in which the <tt>define</tt> is evaluated.


As it happens, our interpreter will evaluate calls to <tt>f</tt> correctly, but for an "accidental" reason: Since the definitions of the internal procedures come first, no calls to these procedures will be evaluated until all of them have been defined. Hence, <tt>odd?</tt> will have been defined by the time <tt>even?</tt> is executed. In fact, our sequential evaluation mechanism will give the same result as a mechanism that directly implements simultaneous definition for any procedure in which the <a name="index_term_4604"></a>internal definitions come first in a body and evaluation of the value expressions for the defined variables doesn't actually use any of the defined variables. (For an example of a procedure that doesn't obey these restrictions, so that sequential definition isn't equivalent to simultaneous definition, see exercise&#160;<a href="#%_thm_4.19">4.19</a>.)<a name="call_footnote_Temp_559" href="#footnote_Temp_559" id="call_footnote_Temp_559"><sup><small>24</small></sup></a>


There is, however, a simple way to treat definitions so that internally defined names have truly simultaneous scope -- just create all local variables that will be in the current environment before evaluating any of the value expressions. One way to do this is by a syntax transformation on <tt>lambda</tt> expressions. Before evaluating the body of a <tt>lambda</tt> expression, we <a name="index_term_4606"></a><a name="index_term_4608"></a>"scan out" and eliminate all the internal definitions in the body. The internally defined variables will be created with a <tt>let</tt> and then set to their values by assignment. For example, the procedure


<tt>(lambda&#160;&lt;*vars*&gt;<br>
&#160;&#160;(define&#160;u&#160;&lt;*e1*&gt;)<br>
&#160;&#160;(define&#160;v&#160;&lt;*e2*&gt;)<br>
&#160;&#160;&lt;*e3*&gt;)<br></tt>


would be transformed into


<tt>(lambda&#160;&lt;*vars*&gt;<br>
&#160;&#160;(let&#160;((u&#160;'*unassigned*)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(v&#160;'*unassigned*))<br>
&#160;&#160;&#160;&#160;(set!&#160;u&#160;&lt;*e1*&gt;)<br>
&#160;&#160;&#160;&#160;(set!&#160;v&#160;&lt;*e2*&gt;)<br>
&#160;&#160;&#160;&#160;&lt;*e3*&gt;))<br></tt>


where <tt>*unassigned*</tt> is a special symbol that causes looking up a variable to signal an error if an attempt is made to use the value of the not-yet-assigned variable.


An alternative strategy for scanning out internal definitions is shown in exercise&#160;<a href="#%_thm_4.18">4.18</a>. Unlike the transformation shown above, this enforces the restriction that the defined variables' values can be evaluated without using any of the variables' values.<a name="call_footnote_Temp_560" href="#footnote_Temp_560" id="call_footnote_Temp_560"><sup><small>25</small></sup></a>


<a name="%_thm_4.16"></a> **Exercise 4.16.**&#160;&#160;In this exercise we implement the method just described for interpreting internal definitions. We assume that the evaluator supports <tt>let</tt> (see exercise&#160;<a href="#%_thm_4.6">4.6</a>).


<a name="index_term_4612"></a>a.&#160;&#160;Change <tt>lookup-variable-value</tt> (section&#160;<a href="#%_sec_4.1.3">4.1.3</a>) to signal an error if the value it finds is the symbol <tt>*unassigned*</tt>.


<a name="index_term_4614"></a>b.&#160;&#160;Write a procedure <tt>scan-out-defines</tt> that takes a procedure body and returns an equivalent one that has no internal definitions, by making the transformation described above.


c.&#160;&#160;Install <tt>scan-out-defines</tt> in the interpreter, either in <tt>make-procedure</tt> or in <tt>procedure-body</tt> (see section&#160;<a href="#%_sec_4.1.3">4.1.3</a>). Which place is better? Why?


<a name="%_thm_4.17"></a> **Exercise 4.17.**&#160;&#160;Draw diagrams of the environment in effect when evaluating the expression &lt;*e3*&gt; in the procedure in the text, comparing how this will be structured when definitions are interpreted sequentially with how it will be structured if definitions are scanned out as described. Why is there an extra frame in the transformed program? Explain why this difference in environment structure can never make a difference in the behavior of a correct program. Design a way to make the interpreter implement the "simultaneous" scope rule for internal definitions without constructing the extra frame.


<a name="%_thm_4.18"></a> **Exercise 4.18.**&#160;&#160;Consider an alternative strategy for scanning out definitions that translates the example in the text to


<tt>(lambda&#160;&lt;*vars*&gt;<br>
&#160;&#160;(let&#160;((u&#160;'*unassigned*)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(v&#160;'*unassigned*))<br>
&#160;&#160;&#160;&#160;(let&#160;((a&#160;&lt;*e1*&gt;)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(b&#160;&lt;*e2*&gt;))<br>
&#160;&#160;&#160;&#160;&#160;&#160;(set!&#160;u&#160;a)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(set!&#160;v&#160;b))<br>
&#160;&#160;&#160;&#160;&lt;*e3*&gt;))<br></tt>


Here <tt>a</tt> and <tt>b</tt> are meant to represent new variable names, created by the interpreter, that do not appear in the user's program. Consider the <tt>solve</tt> procedure from section&#160;<a href="chapter_3_section_5.html#%_sec_3.5.4">3.5.4</a>:


<tt><a name="index_term_4616"></a>(define&#160;(solve&#160;f&#160;y0&#160;dt)<br>
&#160;&#160;(define&#160;y&#160;(integral&#160;(delay&#160;dy)&#160;y0&#160;dt))<br>
&#160;&#160;(define&#160;dy&#160;(stream-map&#160;f&#160;y))<br>
&#160;&#160;y)<br></tt>


Will this procedure work if internal definitions are scanned out as shown in this exercise? What if they are scanned out as shown in the text? Explain.


<a name="%_thm_4.19"></a> **Exercise 4.19.**&#160;&#160;Ben Bitdiddle, Alyssa P. Hacker, and Eva Lu Ator are arguing about the desired result of evaluating the expression


<tt>(let&#160;((a&#160;1))<br>
&#160;&#160;(define&#160;(f&#160;x)<br>
&#160;&#160;&#160;&#160;(define&#160;b&#160;(+&#160;a&#160;x))<br>
&#160;&#160;&#160;&#160;(define&#160;a&#160;5)<br>
&#160;&#160;&#160;&#160;(+&#160;a&#160;b))<br>
&#160;&#160;(f&#160;10))<br></tt>


Ben asserts that the result should be obtained using the sequential rule for <tt>define</tt>: <tt>b</tt> is defined to be 11, then <tt>a</tt> is defined to be 5, so the result is 16. Alyssa objects that mutual recursion requires the simultaneous scope rule for internal procedure definitions, and that it is unreasonable to treat procedure names differently from other names. Thus, she argues for the mechanism implemented in exercise&#160;<a href="#%_thm_4.16">4.16</a>. This would lead to <tt>a</tt> being unassigned at the time that the value for <tt>b</tt> is to be computed. Hence, in Alyssa's view the procedure should produce an error. Eva has a third opinion. She says that if the definitions of <tt>a</tt> and <tt>b</tt> are truly meant to be simultaneous, then the value 5 for <tt>a</tt> should be used in evaluating <tt>b</tt>. Hence, in Eva's view <tt>a</tt> should be 5, <tt>b</tt> should be 15, and the result should be 20. Which (if any) of these viewpoints do you support? Can you devise a way to implement internal definitions so that they behave as Eva prefers?<a name="call_footnote_Temp_565" href="#footnote_Temp_565" id="call_footnote_Temp_565"><sup><small>26</small></sup></a>


<a name="%_thm_4.20"></a> **Exercise 4.20.**&#160;&#160;<a name="index_term_4618"></a><a name="index_term_4620"></a>Because internal definitions look sequential but are actually simultaneous, some people prefer to avoid them entirely, and use the special form <tt>letrec</tt> instead. <tt>Letrec</tt> looks like <tt>let</tt>, so it is not surprising that the variables it binds are bound simultaneously and have the same scope as each other. The sample procedure <tt>f</tt> above can be written without internal definitions, but with exactly the same meaning, as


<tt>(define&#160;(f&#160;x)<br>
&#160;&#160;(letrec&#160;((even?<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda&#160;(n)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;(=&#160;n&#160;0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;true<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(odd?&#160;(-&#160;n&#160;1)))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(odd?<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda&#160;(n)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;(=&#160;n&#160;0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;false<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(even?&#160;(-&#160;n&#160;1))))))<br>
&#160;&#160;&#160;&#160;&lt;*rest&#160;of&#160;body&#160;of&#160;<tt>f</tt>*&gt;))<br></tt>


<tt>Letrec</tt> expressions, which have the form


<tt>(letrec&#160;((&lt;*var<sub>1</sub>*&gt;&#160;&lt;*exp<sub>1</sub>*&gt;)&#160;</tt>... (&lt;*var<sub>*n*</sub>*&gt;&#160;&lt;*exp<sub>*n*</sub>*&gt;))<br>
&#160;&#160;&lt;*body*&gt;)<br>


are a variation on <tt>let</tt> in which the expressions &lt;*exp<sub>*k*</sub>*&gt; that provide the initial values for the variables &lt;*var<sub>*k*</sub>*&gt; are evaluated in an environment that includes all the <tt>letrec</tt> bindings. This permits recursion in the bindings, such as the mutual recursion of <tt>even?</tt> and <tt>odd?</tt> in the example above, or <a name="index_term_4622"></a>the evaluation of 10 factorial with


<tt>(letrec&#160;((fact<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda&#160;(n)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;(=&#160;n&#160;1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;1<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(*&#160;n&#160;(fact&#160;(-&#160;n&#160;1)))))))<br>
&#160;&#160;(fact&#160;10))<br></tt>


a. Implement <tt>letrec</tt> as a derived expression, by transforming a <tt>letrec</tt> expression into a <tt>let</tt> expression as shown in the text above or in exercise&#160;<a href="#%_thm_4.18">4.18</a>. That is, the <tt>letrec</tt> variables should be created with a <tt>let</tt> and then be assigned their values with <tt>set!</tt>.


b. Louis Reasoner is confused by all this fuss about internal definitions. The way he sees it, if you don't like to use <tt>define</tt> inside a procedure, you can just use <tt>let</tt>. Illustrate what is loose about his reasoning by drawing an environment diagram that shows the environment in which the &lt;*rest of body of <tt>f</tt>*&gt; is evaluated during evaluation of the expression <tt>(f 5)</tt>, with <tt>f</tt> defined as in this exercise. Draw an environment diagram for the same evaluation, but with <tt>let</tt> in place of <tt>letrec</tt> in the definition of <tt>f</tt>.


<a name="%_thm_4.21"></a> **Exercise 4.21.**&#160;&#160;<a name="index_term_4624"></a>Amazingly, Louis's intuition in exercise&#160;<a href="#%_thm_4.20">4.20</a> is correct. It is indeed possible to specify recursive procedures without using <tt>letrec</tt> (or even <tt>define</tt>), although the method for accomplishing this is much more subtle than Louis imagined. The following expression computes 10 factorial by applying a recursive <a name="index_term_4626"></a>factorial procedure:<a name="call_footnote_Temp_568" href="#footnote_Temp_568" id="call_footnote_Temp_568"><sup><small>27</small></sup></a>


<tt>((lambda&#160;(n)<br>
&#160;&#160;&#160;((lambda&#160;(fact)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(fact&#160;fact&#160;n))<br>
&#160;&#160;&#160;&#160;(lambda&#160;(ft&#160;k)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;(=&#160;k&#160;1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;1<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(*&#160;k&#160;(ft&#160;ft&#160;(-&#160;k&#160;1)))))))<br>
&#160;10)<br></tt>


a. Check (by evaluating the expression) that this really does compute factorials. Devise an analogous expression for computing Fibonacci numbers.


b. Consider the following procedure, which includes mutually recursive internal definitions:


<tt>(define&#160;(f&#160;x)<br>
&#160;&#160;(define&#160;(even?&#160;n)<br>
&#160;&#160;&#160;&#160;(if&#160;(=&#160;n&#160;0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;true<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(odd?&#160;(-&#160;n&#160;1))))<br>
&#160;&#160;(define&#160;(odd?&#160;n)<br>
&#160;&#160;&#160;&#160;(if&#160;(=&#160;n&#160;0)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;false<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(even?&#160;(-&#160;n&#160;1))))<br>
&#160;&#160;(even?&#160;x))<br></tt>


Fill in the missing expressions to complete an alternative definition of <tt>f</tt>, which uses neither internal definitions nor <tt>letrec</tt>:


<tt>(define&#160;(f&#160;x)<br>
&#160;&#160;((lambda&#160;(even?&#160;odd?)<br>
&#160;&#160;&#160;&#160;&#160;(even?&#160;even?&#160;odd?&#160;x))<br>
&#160;&#160;&#160;(lambda&#160;(ev?&#160;od?&#160;n)<br>
&#160;&#160;&#160;&#160;&#160;(if&#160;(=&#160;n&#160;0)&#160;true&#160;(od?&#160;&lt;??&gt;&#160;&lt;??&gt;&#160;&lt;??&gt;)))<br>
&#160;&#160;&#160;(lambda&#160;(ev?&#160;od?&#160;n)<br>
&#160;&#160;&#160;&#160;&#160;(if&#160;(=&#160;n&#160;0)&#160;false&#160;(ev?&#160;&lt;??&gt;&#160;&lt;??&gt;&#160;&lt;??&gt;)))))<br></tt>


<a name="%_sec_4.1.7"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_4.1.7">4.1.7&#160;&#160;Separating Syntactic Analysis from Execution</a></h3>

<a name="index_term_4634"></a><a name="index_term_4636"></a><a name="index_term_4638"></a> <a name="index_term_4640"></a><a name="index_term_4642"></a>The evaluator implemented above is simple, but it is very inefficient, because the syntactic analysis of expressions is interleaved with their execution. Thus if a program is executed many times, its syntax is analyzed many times. Consider, for example, evaluating <tt>(factorial 4)</tt> using the following definition of <tt>factorial</tt>:


<tt>(define&#160;(factorial&#160;n)<br>
&#160;&#160;(if&#160;(=&#160;n&#160;1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;1<br>
&#160;&#160;&#160;&#160;&#160;&#160;(*&#160;(factorial&#160;(-&#160;n&#160;1))&#160;n)))<br></tt>


Each time <tt>factorial</tt> is called, the evaluator must determine that the body is an <tt>if</tt> expression and extract the predicate. Only then can it evaluate the predicate and dispatch on its value. Each time it evaluates the expression <tt>(* (factorial (- n 1)) n)</tt>, or the subexpressions <tt>(factorial (- n 1))</tt> and <tt>(- n 1)</tt>, the evaluator must perform the case analysis in <tt>eval</tt> to determine that the expression is an application, and must extract its operator and operands. This analysis is expensive. Performing it repeatedly is wasteful.


We can transform the evaluator to be significantly more efficient by arranging things so that syntactic analysis is performed only once.<a name="call_footnote_Temp_569" href="#footnote_Temp_569" id="call_footnote_Temp_569"><sup><small>28</small></sup></a> We split <tt>eval</tt>, which takes an expression and an environment, into two parts. The procedure <tt>analyze</tt> takes only the expression. It performs the syntactic analysis and returns a new procedure, the <a name="index_term_4652"></a>*execution procedure*, that encapsulates the work to be done in executing the analyzed expression. The execution procedure takes an environment as its argument and completes the evaluation. This saves work because <tt>analyze</tt> will be called only once on an expression, while the execution procedure may be called many times.


With the separation into analysis and execution, <tt>eval</tt> now becomes


<tt><a name="index_term_4654"></a>(define&#160;(eval&#160;exp&#160;env)<br>
&#160;&#160;((analyze&#160;exp)&#160;env))<br></tt>


The result of calling <tt>analyze</tt> is the execution procedure to be applied to the environment. The <tt>analyze</tt> procedure is the same case analysis as performed by the original <tt>eval</tt> of section&#160;<a href="#%_sec_4.1.1">4.1.1</a>, except that the procedures to which we dispatch perform only analysis, not full evaluation:


<tt><a name="index_term_4656"></a>(define&#160;(analyze&#160;exp)<br>
&#160;&#160;(cond&#160;((self-evaluating?&#160;exp)&#160;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(analyze-self-evaluating&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((quoted?&#160;exp)&#160;(analyze-quoted&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((variable?&#160;exp)&#160;(analyze-variable&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((assignment?&#160;exp)&#160;(analyze-assignment&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((definition?&#160;exp)&#160;(analyze-definition&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((if?&#160;exp)&#160;(analyze-if&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((lambda?&#160;exp)&#160;(analyze-lambda&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((begin?&#160;exp)&#160;(analyze-sequence&#160;(begin-actions&#160;exp)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((cond?&#160;exp)&#160;(analyze&#160;(cond-&gt;if&#160;exp)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((application?&#160;exp)&#160;(analyze-application&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Unknown&#160;expression&#160;type&#160;--&#160;ANALYZE&quot;&#160;exp))))<br></tt>


Here is the simplest syntactic analysis procedure, which handles self-evaluating expressions. It returns an execution procedure that ignores its environment argument and just returns the expression:


<a name="index_term_4658"></a>


<tt>(define&#160;(analyze-self-evaluating&#160;exp)<br>
&#160;&#160;(lambda&#160;(env)&#160;exp))<br></tt>


For a quoted expression, we can gain a little efficiency by extracting the text of the quotation only once, in the analysis phase, rather than in the execution phase.


<tt>(define&#160;(analyze-quoted&#160;exp)<br>
&#160;&#160;(let&#160;((qval&#160;(text-of-quotation&#160;exp)))<br>
&#160;&#160;&#160;&#160;(lambda&#160;(env)&#160;qval)))<br></tt>


Looking up a variable value must still be done in the execution phase, since this depends upon knowing the environment.<a name="call_footnote_Temp_570" href="#footnote_Temp_570" id="call_footnote_Temp_570"><sup><small>29</small></sup></a>


<tt>(define&#160;(analyze-variable&#160;exp)<br>
&#160;&#160;(lambda&#160;(env)&#160;(lookup-variable-value&#160;exp&#160;env)))<br></tt>


<tt>Analyze-assignment</tt> also must defer actually setting the variable until the execution, when the environment has been supplied. However, the fact that the <tt>assignment-value</tt> expression can be analyzed (recursively) during analysis is a major gain in efficiency, because the <tt>assignment-value</tt> expression will now be analyzed only once. The same holds true for definitions.


<tt>(define&#160;(analyze-assignment&#160;exp)<br>
&#160;&#160;(let&#160;((var&#160;(assignment-variable&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(vproc&#160;(analyze&#160;(assignment-value&#160;exp))))<br>
&#160;&#160;&#160;&#160;(lambda&#160;(env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(set-variable-value!&#160;var&#160;(vproc&#160;env)&#160;env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;'ok)))<br>
(define&#160;(analyze-definition&#160;exp)<br>
&#160;&#160;(let&#160;((var&#160;(definition-variable&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(vproc&#160;(analyze&#160;(definition-value&#160;exp))))<br>
&#160;&#160;&#160;&#160;(lambda&#160;(env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(define-variable!&#160;var&#160;(vproc&#160;env)&#160;env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;'ok)))<br></tt>


For <tt>if</tt> expressions, we extract and analyze the predicate, consequent, and alternative at analysis time.


<tt>(define&#160;(analyze-if&#160;exp)<br>
&#160;&#160;(let&#160;((pproc&#160;(analyze&#160;(if-predicate&#160;exp)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cproc&#160;(analyze&#160;(if-consequent&#160;exp)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(aproc&#160;(analyze&#160;(if-alternative&#160;exp))))<br>
&#160;&#160;&#160;&#160;(lambda&#160;(env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;(true?&#160;(pproc&#160;env))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cproc&#160;env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(aproc&#160;env)))))<br></tt>


Analyzing a <tt>lambda</tt> expression also achieves a major gain in efficiency: We analyze the <tt>lambda</tt> body only once, even though procedures resulting from evaluation of the <tt>lambda</tt> may be applied many times.


<tt>(define&#160;(analyze-lambda&#160;exp)<br>
&#160;&#160;(let&#160;((vars&#160;(lambda-parameters&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(bproc&#160;(analyze-sequence&#160;(lambda-body&#160;exp))))<br>
&#160;&#160;&#160;&#160;(lambda&#160;(env)&#160;(make-procedure&#160;vars&#160;bproc&#160;env))))<br></tt>


Analysis of a sequence of expressions (as in a <tt>begin</tt> or the body of a <tt>lambda</tt> expression) is more involved.<a name="call_footnote_Temp_571" href="#footnote_Temp_571" id="call_footnote_Temp_571"><sup><small>30</small></sup></a> Each expression in the sequence is analyzed, yielding an execution procedure. These execution procedures are combined to produce an execution procedure that takes an environment as argument and sequentially calls each individual execution procedure with the environment as argument.


<tt>(define&#160;(analyze-sequence&#160;exps)<br>
&#160;&#160;(define&#160;(sequentially&#160;proc1&#160;proc2)<br>
&#160;&#160;&#160;&#160;(lambda&#160;(env)&#160;(proc1&#160;env)&#160;(proc2&#160;env)))<br>
&#160;&#160;(define&#160;(loop&#160;first-proc&#160;rest-procs)<br>
&#160;&#160;&#160;&#160;(if&#160;(null?&#160;rest-procs)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;first-proc<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(loop&#160;(sequentially&#160;first-proc&#160;(car&#160;rest-procs))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cdr&#160;rest-procs))))<br>
&#160;&#160;(let&#160;((procs&#160;(map&#160;analyze&#160;exps)))<br>
&#160;&#160;&#160;&#160;(if&#160;(null?&#160;procs)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Empty&#160;sequence&#160;--&#160;ANALYZE&quot;))<br>
&#160;&#160;&#160;&#160;(loop&#160;(car&#160;procs)&#160;(cdr&#160;procs))))<br></tt>


To analyze an application, we analyze the operator and operands and construct an execution procedure that calls the operator execution procedure (to obtain the actual procedure to be applied) and the operand execution procedures (to obtain the actual arguments). We then pass these to <tt>execute-application</tt>, which is the analog of <tt>apply</tt> in section&#160;<a href="#%_sec_4.1.1">4.1.1</a>. <tt>Execute-application</tt> differs from <tt>apply</tt> in that the procedure body for a compound procedure has already been analyzed, so there is no need to do further analysis. Instead, we just call the execution procedure for the body on the extended environment.


<tt>(define&#160;(analyze-application&#160;exp)<br>
&#160;&#160;(let&#160;((fproc&#160;(analyze&#160;(operator&#160;exp)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(aprocs&#160;(map&#160;analyze&#160;(operands&#160;exp))))<br>
&#160;&#160;&#160;&#160;(lambda&#160;(env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(execute-application&#160;(fproc&#160;env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(map&#160;(lambda&#160;(aproc)&#160;(aproc&#160;env))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;aprocs)))))<br>
<a name="index_term_4660"></a>(define&#160;(execute-application&#160;proc&#160;args)<br>
&#160;&#160;(cond&#160;((primitive-procedure?&#160;proc)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(apply-primitive-procedure&#160;proc&#160;args))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((compound-procedure?&#160;proc)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((procedure-body&#160;proc)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(extend-environment&#160;(procedure-parameters&#160;proc)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;args<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(procedure-environment&#160;proc))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&quot;Unknown&#160;procedure&#160;type&#160;--&#160;EXECUTE-APPLICATION&quot;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;proc))))<br></tt>


Our new evaluator uses the same data structures, syntax procedures, and run-time support procedures as in sections&#160;<a href="#%_sec_4.1.2">4.1.2</a>, &#160;<a href="#%_sec_4.1.3">4.1.3</a>, and&#160;<a href="#%_sec_4.1.4">4.1.4</a>.


<a name="%_thm_4.22"></a> **Exercise 4.22.**&#160;&#160;<a name="index_term_4662"></a>Extend the evaluator in this section to support the special form <tt>let</tt>. (See exercise&#160;<a href="#%_thm_4.6">4.6</a>.)


<a name="%_thm_4.23"></a> **Exercise 4.23.**&#160;&#160;<a name="index_term_4664"></a>Alyssa P. Hacker doesn't understand why <tt>analyze-sequence</tt> needs to be so complicated. All the other analysis procedures are straightforward transformations of the corresponding evaluation procedures (or <tt>eval</tt> clauses) in section&#160;<a href="#%_sec_4.1.1">4.1.1</a>. She expected <tt>analyze-sequence</tt> to look like this:


<tt>(define&#160;(analyze-sequence&#160;exps)<br>
&#160;&#160;(define&#160;(execute-sequence&#160;procs&#160;env)<br>
&#160;&#160;&#160;&#160;(cond&#160;((null?&#160;(cdr&#160;procs))&#160;((car&#160;procs)&#160;env))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;((car&#160;procs)&#160;env)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(execute-sequence&#160;(cdr&#160;procs)&#160;env))))<br>
&#160;&#160;(let&#160;((procs&#160;(map&#160;analyze&#160;exps)))<br>
&#160;&#160;&#160;&#160;(if&#160;(null?&#160;procs)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Empty&#160;sequence&#160;--&#160;ANALYZE&quot;))<br>
&#160;&#160;&#160;&#160;(lambda&#160;(env)&#160;(execute-sequence&#160;procs&#160;env))))<br></tt>


Eva Lu Ator explains to Alyssa that the version in the text does more of the work of evaluating a sequence at analysis time. Alyssa's sequence-execution procedure, rather than having the calls to the individual execution procedures built in, loops through the procedures in order to call them: In effect, although the individual expressions in the sequence have been analyzed, the sequence itself has not been.


Compare the two versions of <tt>analyze-sequence</tt>. For example, consider the common case (typical of procedure bodies) where the sequence has just one expression. What work will the execution procedure produced by Alyssa's program do? What about the execution procedure produced by the program in the text above? How do the two versions compare for a sequence with two expressions?


<a name="%_thm_4.24"></a> **Exercise 4.24.**&#160;&#160;Design and carry out some experiments to compare the speed of the original metacircular evaluator with the version in this section. Use your results to estimate the fraction of time that is spent in analysis versus execution for various procedures.

<div class="smallprint">
  <hr>
</div>

<div class="footnote">
  
<a name="footnote_Temp_510" href="#call_footnote_Temp_510" id="footnote_Temp_510"><sup><small>3</small></sup></a> Even so, there will remain important aspects of the evaluation process that are not elucidated by our evaluator. The most important of these are the detailed mechanisms by which procedures call other procedures and return values to their callers. We will address these issues in chapter&#160;5, where we take a closer look at the evaluation process by implementing the evaluator as a simple register machine.

  
<a name="footnote_Temp_511" href="#call_footnote_Temp_511" id="footnote_Temp_511"><sup><small>4</small></sup></a> If we grant ourselves the ability to apply primitives, <a name="index_term_4222"></a>then what remains for us to implement in the evaluator? The job of the evaluator is not to specify the primitives of the language, but rather to provide the connective tissue -- the means of combination and the means of abstraction -- that binds a collection of primitives to form a language. Specifically:

  <ul>
    <li>The evaluator enables us to deal with nested expressions. For example, although simply applying primitives would suffice for evaluating the expression <tt>(+ 1 6)</tt>, it is not adequate for handling <tt>(+ 1 (* 2 3))</tt>. As far as the primitive procedure <tt>+</tt> is concerned, its arguments must be numbers, and it would choke if we passed it the expression <tt>(* 2 3)</tt> as an argument. One important role of the evaluator is to choreograph procedure composition so that <tt>(* 2 3)</tt> is reduced to 6 before being passed as an argument to <tt>+</tt>.</li>

    <li>The evaluator allows us to use variables. For example, the primitive procedure for addition has no way to deal with expressions such as <tt>(+ x 1)</tt>. We need an evaluator to keep track of variables and obtain their values before invoking the primitive procedures.</li>

    <li>The evaluator allows us to define compound procedures. This involves keeping track of procedure definitions, knowing how to use these definitions in evaluating expressions, and providing a mechanism that enables procedures to accept arguments.</li>

    <li>The evaluator provides the special forms, which must be evaluated differently from procedure calls.</li>
  </ul>

  
<a name="footnote_Temp_518" href="#call_footnote_Temp_518" id="footnote_Temp_518"><sup><small>5</small></sup></a> We could have simplified the <tt>application?</tt> clause in <tt>eval</tt> by using <tt>map</tt> (and stipulating that <tt>operands</tt> returns a list) rather than writing an explicit <tt>list-of-values</tt> procedure. We chose not to use <tt>map</tt> here to emphasize the fact that the <a name="index_term_4252"></a><a name="index_term_4254"></a>evaluator can be implemented without any use of higher-order procedures (and thus could be written in a language that doesn't have higher-order procedures), even though the language that it supports will include higher-order procedures.

  
<a name="footnote_Temp_520" href="#call_footnote_Temp_520" id="footnote_Temp_520"><sup><small>6</small></sup></a> In this case, the language being implemented and the implementation language are the same. Contemplation of the meaning of <a name="index_term_4262"></a><tt>true?</tt> here yields expansion of consciousness without the abuse of substance.

  
<a name="footnote_Temp_523" href="#call_footnote_Temp_523" id="footnote_Temp_523"><sup><small>7</small></sup></a> This implementation of <tt>define</tt> ignores a subtle issue in the handling of internal definitions, although it works correctly in most cases. We will see what the problem is and how to solve it in section&#160;<a href="#%_sec_4.1.6">4.1.6</a>.

  
<a name="footnote_Temp_524" href="#call_footnote_Temp_524" id="footnote_Temp_524"><sup><small>8</small></sup></a> As we said when we introduced <tt>define</tt> and <tt>set!</tt>, these values are implementation-dependent in Scheme -- that is, the implementor can choose what value to return.

  
<a name="footnote_Temp_526" href="#call_footnote_Temp_526" id="footnote_Temp_526"><sup><small>9</small></sup></a> As mentioned in section&#160;<a href="chapter_2_section_3.html#%_sec_2.3.1">2.3.1</a>, the evaluator sees a quoted expression as a list beginning with <tt>quote</tt>, even if the expression is typed with the quotation mark. For example, the expression <tt>'a</tt> would be seen by the evaluator as <tt>(quote a)</tt>. See exercise&#160;<a href="chapter_2_section_3.html#exercise_2_55">2.55</a>.

  
<a name="footnote_Temp_527" href="#call_footnote_Temp_527" id="footnote_Temp_527"><sup><small>10</small></sup></a> The value of an <tt>if</tt> expression when the predicate is false and there is no alternative is unspecified in Scheme; we have chosen here to make it false. We will support the use of the variables <tt>true</tt> and <tt>false</tt> in expressions to be evaluated by binding them in the global environment. See section&#160;<a href="#%_sec_4.1.4">4.1.4</a>.

  
<a name="footnote_Temp_528" href="#call_footnote_Temp_528" id="footnote_Temp_528"><sup><small>11</small></sup></a> These selectors for a list of expressions -- and the corresponding ones for a list of operands -- are not intended as a data abstraction. They are introduced as mnemonic names for the basic list operations in order to make it easier to understand the explicit-control evaluator in section&#160;<a href="chapter_5_section_4.html#%_sec_5.4">5.4</a>.

  
<a name="footnote_Temp_530" href="#call_footnote_Temp_530" id="footnote_Temp_530"><sup><small>12</small></sup></a> The value of a <tt>cond</tt> expression when all the predicates are false and there is no <tt>else</tt> clause is unspecified in Scheme; we have chosen here to make it false.

  
<a name="footnote_Temp_531" href="#call_footnote_Temp_531" id="footnote_Temp_531"><sup><small>13</small></sup></a> Practical Lisp systems provide a mechanism that allows a user to add new derived expressions and specify their implementation as syntactic transformations without modifying the evaluator. Such a user-defined transformation is called a <a name="index_term_4374"></a>*macro*. Although it is easy to add an elementary mechanism for defining macros, the resulting language has subtle name-conflict problems. There has been much research on mechanisms for macro definition that do not cause these difficulties. See, <a name="index_term_4376"></a><a name="index_term_4378"></a><a name="index_term_4380"></a><a name="index_term_4382"></a>for example, Kohlbecker 1986, Clinger and Rees 1991, and Hanson 1991.

  
<a name="footnote_Temp_544" href="#call_footnote_Temp_544" id="footnote_Temp_544"><sup><small>14</small></sup></a> Frames are not really a data abstraction in the following code: <tt>Set-variable-value!</tt> and <tt>define-variable!</tt> use <tt>set-car!</tt> to directly modify the values in a frame. The purpose of the frame procedures is to make the environment-manipulation procedures easy to read.

  
<a name="footnote_Temp_545" href="#call_footnote_Temp_545" id="footnote_Temp_545"><sup><small>15</small></sup></a> The drawback of this representation (as well as the variant in exercise&#160;<a href="#%_thm_4.11">4.11</a>) is that the evaluator may have to search through many frames in order to find the binding for a given variable. <a name="index_term_4488"></a><a name="index_term_4490"></a>(Such an approach is referred to as *deep binding*.) One way to avoid this inefficiency is to make use of a strategy called *lexical addressing*, which will be discussed in section&#160;<a href="chapter_5_section_5.html#%_sec_5.5.6">5.5.6</a>.

  
<a name="footnote_Temp_549" href="#call_footnote_Temp_549" id="footnote_Temp_549"><sup><small>16</small></sup></a> Any procedure defined in the underlying Lisp can be used as a primitive for the metacircular evaluator. The name of a primitive installed in the evaluator need not be the same as the name of its implementation in the underlying Lisp; the names are the same here because the metacircular evaluator implements Scheme itself. Thus, for example, we could put <tt>(list 'first car)</tt> or <tt>(list 'square (lambda (x) (* x x)))</tt> in the list of <tt>primitive-procedures</tt>.

  
<a name="footnote_Temp_550" href="#call_footnote_Temp_550" id="footnote_Temp_550"><sup><small>17</small></sup></a> <tt>Apply-in-underlying-scheme</tt> is the <tt>apply</tt> procedure we have used in earlier chapters. The metacircular evaluator's <tt>apply</tt> procedure (section&#160;<a href="#%_sec_4.1.1">4.1.1</a>) models the working of this primitive. Having two different things called <tt>apply</tt> leads to a technical problem in running the metacircular evaluator, because defining the metacircular evaluator's <tt>apply</tt> will mask the definition of the primitive. One way around this is to rename the metacircular <tt>apply</tt> to avoid conflict with the name of the primitive procedure. We have assumed instead that we have saved a reference to the underlying <tt>apply</tt> by doing

  
<tt>(define&#160;apply-in-underlying-scheme&#160;apply)<br></tt>

  
before defining the metacircular <tt>apply</tt>. This allows us to access the original version of <tt>apply</tt> under a different name.

  
<a name="footnote_Temp_551" href="#call_footnote_Temp_551" id="footnote_Temp_551"><sup><small>18</small></sup></a> The primitive procedure <a name="index_term_4522"></a><a name="index_term_4524"></a><tt>read</tt> waits for input from the user, and returns the next complete expression that is typed. For example, if the user types <tt>(+ 23 x)</tt>, <tt>read</tt> returns a three-element list containing the symbol <tt>+</tt>, the number 23, and the symbol <tt>x</tt>. <a name="index_term_4526"></a><a name="index_term_4528"></a>If the user types <tt>'x</tt>, <tt>read</tt> returns a two-element list containing the symbol <tt>quote</tt> and the symbol <tt>x</tt>.

  
<a name="footnote_Temp_553" href="#call_footnote_Temp_553" id="footnote_Temp_553"><sup><small>19</small></sup></a> The fact that the machines are described in Lisp is inessential. If we give our evaluator a Lisp program that behaves as an evaluator for some other language, say C, the Lisp evaluator will emulate the C evaluator, which in turn can emulate any machine described as a C program. Similarly, writing a Lisp evaluator in C produces a C program that can execute any Lisp program. The deep idea here is that any evaluator can emulate any other. Thus, the notion of "what can in principle be computed" (ignoring practicalities of time and memory required) is independent of the language or the computer, and instead reflects an underlying notion of <a name="index_term_4554"></a>*computability*. This was first demonstrated in a clear way by <a name="index_term_4556"></a>Alan M. Turing (1912-1954), whose 1936 paper laid the foundations for theoretical <a name="index_term_4558"></a>computer science. In the paper, Turing presented a simple computational model -- now known as a <a name="index_term_4560"></a>*Turing machine* -- and argued that any "effective process" can be formulated as a program for such a machine. (This argument is known as the <a name="index_term_4562"></a>*Church-Turing thesis*.) Turing then implemented a universal machine, i.e., a Turing machine that behaves as an evaluator for Turing-machine programs. He used this framework to demonstrate that there are well-posed problems that cannot be computed by Turing machines (see exercise&#160;<a href="#%_thm_4.15">4.15</a>), and so by implication cannot be formulated as "effective processes." Turing went on to make fundamental contributions to practical computer science as well. For example, he invented the idea of <a name="index_term_4564"></a>structuring programs using general-purpose subroutines. See <a name="index_term_4566"></a>Hodges 1983 for a biography of Turing.

  
<a name="footnote_Temp_554" href="#call_footnote_Temp_554" id="footnote_Temp_554"><sup><small>20</small></sup></a> Some people find it counterintuitive that an evaluator, which is implemented by a relatively simple procedure, can emulate programs that are more complex than the evaluator itself. The existence of a universal evaluator machine is a deep and wonderful property of computation. <a name="index_term_4568"></a>*Recursion theory*, a branch of mathematical logic, is concerned with logical limits of computation. <a name="index_term_4570"></a>Douglas Hofstadter's beautiful book *G�del, Escher, Bach* (1979) explores some of these ideas.

  
<a name="footnote_Temp_555" href="#call_footnote_Temp_555" id="footnote_Temp_555"><sup><small>21</small></sup></a> Warning: <a name="index_term_4576"></a>This <tt>eval</tt> primitive is not identical to the <tt>eval</tt> procedure we implemented in section&#160;<a href="#%_sec_4.1.1">4.1.1</a>, because it uses *actual* Scheme environments rather than the sample environment structures we built in section&#160;<a href="#%_sec_4.1.3">4.1.3</a>. These actual environments cannot be manipulated by the user as ordinary lists; they must be accessed via <tt>eval</tt> or other special operations. <a name="index_term_4578"></a>Similarly, the <tt>apply</tt> primitive we saw earlier is not identical to the metacircular <tt>apply</tt>, because it uses actual Scheme procedures rather than the procedure objects we constructed in sections&#160;<a href="#%_sec_4.1.3">4.1.3</a> and&#160;<a href="#%_sec_4.1.4">4.1.4</a>.

  
<a name="footnote_Temp_556" href="#call_footnote_Temp_556" id="footnote_Temp_556"><sup><small>22</small></sup></a> The MIT <a name="index_term_4580"></a><a name="index_term_4582"></a><a name="index_term_4584"></a><a name="index_term_4586"></a>implementation of Scheme includes <tt>eval</tt>, as well as a symbol <tt>user-initial-environment</tt> that is bound to the initial environment in which the user's input expressions are evaluated.

  
<a name="footnote_Temp_558" href="#call_footnote_Temp_558" id="footnote_Temp_558"><sup><small>23</small></sup></a> Although we stipulated that <tt>halts?</tt> is given a procedure object, notice that this reasoning still applies even if <tt>halts?</tt> can gain access to the procedure's text and its environment. <a name="index_term_4590"></a><a name="index_term_4592"></a><a name="index_term_4594"></a><a name="index_term_4596"></a>This is Turing's celebrated *Halting Theorem*, which gave the first clear example of a *non-computable* problem, i.e., a well-posed task that cannot be carried out as a computational procedure.

  
<a name="footnote_Temp_559" href="#call_footnote_Temp_559" id="footnote_Temp_559"><sup><small>24</small></sup></a> Wanting programs to not depend on this evaluation mechanism is the reason for the "management is not responsible" remark in footnote&#160;<a href="chapter_1_section_1.html#footnote_Temp_45">28</a> of chapter&#160;1. By insisting that internal definitions come first and do not use each other while the definitions are being evaluated, the IEEE standard for Scheme leaves implementors some choice in the mechanism used to evaluate these definitions. The choice of one evaluation rule rather than another here may seem like a small issue, affecting only the interpretation of "badly formed" programs. However, we will see in section&#160;<a href="chapter_5_section_5.html#%_sec_5.5.6">5.5.6</a> that moving to a model of simultaneous scoping for internal definitions avoids some nasty difficulties that would otherwise arise in implementing a compiler.

  
<a name="footnote_Temp_560" href="#call_footnote_Temp_560" id="footnote_Temp_560"><sup><small>25</small></sup></a> The IEEE standard for Scheme allows for different implementation strategies by specifying that it is up to the programmer to obey this restriction, not up to the implementation to enforce it. Some Scheme implementations, including <a name="index_term_4610"></a>MIT Scheme, use the transformation shown above. Thus, some programs that don't obey this restriction will in fact run in such implementations.

  
<a name="footnote_Temp_565" href="#call_footnote_Temp_565" id="footnote_Temp_565"><sup><small>26</small></sup></a> The MIT implementors of Scheme support Alyssa on the following grounds: Eva is in principle correct -- the definitions should be regarded as simultaneous. But it seems difficult to implement a general, efficient mechanism that does what Eva requires. In the absence of such a mechanism, it is better to generate an error in the difficult cases of simultaneous definitions (Alyssa's notion) than to produce an incorrect answer (as Ben would have it).

  
<a name="footnote_Temp_568" href="#call_footnote_Temp_568" id="footnote_Temp_568"><sup><small>27</small></sup></a> This example illustrates a programming trick for formulating recursive procedures without using <tt>define</tt>. The <a name="index_term_4628"></a>most general trick of this sort is the *Y* *operator*, which can be used to give a "pure <img src="img/book_06.gif" border="0">-calculus" implementation of <a name="index_term_4630"></a><a name="index_term_4632"></a>recursion. (See Stoy 1977 for details on the lambda calculus, and Gabriel 1988 for an exposition of the *Y* operator in Scheme.)

  
<a name="footnote_Temp_569" href="#call_footnote_Temp_569" id="footnote_Temp_569"><sup><small>28</small></sup></a> This technique is an integral part of the compilation process, which we shall discuss in chapter&#160;5. Jonathan Rees wrote a Scheme <a name="index_term_4644"></a><a name="index_term_4646"></a><a name="index_term_4648"></a><a name="index_term_4650"></a>interpreter like this in about 1982 for the T project (Rees and Adams 1982). Marc Feeley (1986) (see also Feeley and Lapalme 1987) independently invented this technique in his master's thesis.

  
<a name="footnote_Temp_570" href="#call_footnote_Temp_570" id="footnote_Temp_570"><sup><small>29</small></sup></a> There is, however, an important part of the variable search that *can* be done as part of the syntactic analysis. As we will show in section&#160;<a href="chapter_5_section_5.html#%_sec_5.5.6">5.5.6</a>, one can determine the position in the environment structure where the value of the variable will be found, thus obviating the need to scan the environment for the entry that matches the variable.

  
<a name="footnote_Temp_571" href="#call_footnote_Temp_571" id="footnote_Temp_571"><sup><small>30</small></sup></a> See exercise&#160;<a href="#%_thm_4.23">4.23</a> for some insight into the processing of sequences.

</div>