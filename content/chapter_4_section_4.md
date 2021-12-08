(section_44)=
# Logic Programming

<a name="index_term_5028"></a> <a name="index_term_5030"></a><a name="index_term_5032"></a><a name="index_term_5034"></a><a name="index_term_5036"></a>In chapter&#160;1 we stressed that computer science deals with imperative (how to) knowledge, whereas mathematics deals with declarative (what is) knowledge. Indeed, programming languages require that the programmer express knowledge in a form that indicates the step-by-step methods for solving particular problems. On the other hand, high-level languages provide, as part of the language implementation, a substantial amount of methodological knowledge that frees the user from concern with numerous details of how a specified computation will progress.


Most programming languages, including Lisp, are organized around computing the values of mathematical functions. Expression-oriented languages (such as Lisp, Fortran, and Algol) capitalize on the "pun" that an expression that describes the value of a function may also be interpreted as a means of computing that value. Because of this, most programming languages are strongly biased toward unidirectional computations (computations with well-defined inputs and outputs). There are, however, radically different programming languages that relax this bias. We saw one such example in section&#160;<a href="chapter_3_section_3.html#%_sec_3.3.5">3.3.5</a>, where the objects of computation were arithmetic constraints. In a constraint system the direction and the order of computation are not so well specified; in carrying out a computation the system must therefore provide more detailed "how to" knowledge than would be the case with an ordinary arithmetic computation. This does not mean, however, that the user is released altogether from the responsibility of providing imperative knowledge. There are many constraint networks that implement the same set of constraints, and the user must choose from the set of mathematically equivalent networks a suitable network to specify a particular computation.


The nondeterministic program evaluator of section&#160;<a href="chapter_4_section_3.html#%_sec_4.3">4.3</a> also moves away from the view that programming is about constructing algorithms for computing unidirectional functions. In a nondeterministic language, expressions can have more than one value, and, as a result, the computation is <a name="index_term_5038"></a>dealing with relations rather than with single-valued functions. Logic programming extends this idea by combining a relational vision of programming with a powerful kind of symbolic pattern matching called *unification*.<a name="call_footnote_Temp_645" href="#footnote_Temp_645" id="call_footnote_Temp_645"><sup><small>58</small></sup></a>


<a name="index_term_5070"></a><a name="index_term_5072"></a>This approach, when it works, can be a very powerful way to write programs. Part of the power comes from the fact that a single "what is" fact can be used to solve a number of different problems that would have different "how to" components. As an example, consider the <a name="index_term_5074"></a><tt>append</tt> operation, which takes two lists as arguments and combines their elements to form a single list. In a procedural language such as Lisp, we could define <tt>append</tt> in terms of the basic list constructor <tt>cons</tt>, as we did in section&#160;<a href="chapter_2_section_2.html#%_sec_2.2.1">2.2.1</a>:


<tt>(define&#160;(append&#160;x&#160;y)<br>
&#160;&#160;(if&#160;(null?&#160;x)<br>
&#160;&#160;&#160;&#160;&#160;&#160;y<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cons&#160;(car&#160;x)&#160;(append&#160;(cdr&#160;x)&#160;y))))<br></tt>


This procedure can be regarded as a translation into Lisp of the following two rules, the first of which covers the case where the first list is empty and the second of which handles the case of a nonempty list, which is a <tt>cons</tt> of two parts:

<ul>
  <li>For any list <tt>y</tt>, the empty list and <tt>y</tt> <tt>append</tt> to form <tt>y</tt>.</li>

  <li>For any <tt>u</tt>, <tt>v</tt>, <tt>y</tt>, and <tt>z</tt>, <tt>(cons u v)</tt> and <tt>y</tt> <tt>append</tt> to form <tt>(cons&#160;u&#160;z)</tt> if <tt>v</tt> and <tt>y</tt> <tt>append</tt> to form <tt>z</tt>.<a name="call_footnote_Temp_646" href="#footnote_Temp_646" id="call_footnote_Temp_646"><sup><small>59</small></sup></a></li>
</ul>

Using the <tt>append</tt> procedure, we can answer questions such as

<blockquote>
  Find the <tt>append</tt> of <tt>(a b)</tt> and <tt>(c d)</tt>.
</blockquote>

But the same two rules are also sufficient for answering the following sorts of questions, which the procedure can't answer:

<blockquote>
  Find a list <tt>y</tt> that <tt>append</tt>s with <tt>(a b)</tt> to produce <tt>(a b c d)</tt>.

  
Find all <tt>x</tt> and <tt>y</tt> that <tt>append</tt> to form <tt>(a b c d)</tt>.

</blockquote>

<a name="index_term_5076"></a><a name="index_term_5078"></a>In a logic programming language, the programmer writes an <tt>append</tt> "procedure" by stating the two rules about <tt>append</tt> given above. "How to" knowledge is provided automatically by the interpreter to allow this single pair of rules to be used to answer all three types of questions about <tt>append</tt>.<a name="call_footnote_Temp_647" href="#footnote_Temp_647" id="call_footnote_Temp_647"><sup><small>60</small></sup></a>


Contemporary logic programming languages (including the one we implement here) have substantial deficiencies, in that their general "how to" methods can lead them into spurious infinite loops or other undesirable behavior. Logic programming is an active field of research in computer science.<a name="call_footnote_Temp_648" href="#footnote_Temp_648" id="call_footnote_Temp_648"><sup><small>61</small></sup></a>


Earlier in this chapter we explored the technology of implementing interpreters and described the elements that are essential to an interpreter for a Lisp-like language (indeed, to an interpreter for any conventional language). Now we will apply these ideas to discuss an interpreter for a logic programming language. We call this <a name="index_term_5090"></a>language the *query language*, because it is very useful for retrieving information from data bases by formulating <a name="index_term_5092"></a>*queries*, or questions, expressed in the language. Even though the query language is very different from Lisp, we will find it convenient to describe the language in terms of the same general framework we have been using all along: as a collection of primitive elements, together with means of combination that enable us to combine simple elements to create more complex elements and means of abstraction that enable us to regard complex elements as single conceptual units. An interpreter for a logic programming language is considerably more complex than an interpreter for a language like Lisp. Nevertheless, we will see <a name="index_term_5094"></a>that our query-language interpreter contains many of the same elements found in the interpreter of section&#160;<a href="chapter_4_section_1.html#%_sec_4.1">4.1</a>. In particular, there will be an "eval" part that classifies expressions according to type and an "apply" part that implements the language's abstraction mechanism (procedures in the case of Lisp, and *rules* in the case of logic programming). Also, a central role is played in the implementation by a frame data structure, which determines the correspondence between symbols and their associated values. One additional interesting aspect of our query-language implementation is that we make substantial use of streams, which were introduced in chapter&#160;3. <a name="%_sec_4.4.1"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_4.4.1">4.4.1&#160;&#160;Deductive Information Retrieval</a></h3>

<a name="index_term_5096"></a> <a name="index_term_5098"></a>Logic programming excels in providing interfaces to data bases for information retrieval. The query language we shall implement in this chapter is designed to be used in this way.


In order to illustrate what the query system does, we will show how it can be used to manage the data base of personnel records for <a name="index_term_5100"></a>Microshaft, a thriving high-technology company in the Boston area. The language provides pattern-directed access to personnel information and can also take advantage of general rules in order to make logical deductions.


<a name="%_sec_Temp_649"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_649">A sample data base</a></h4>

<a name="index_term_5102"></a> <a name="index_term_5104"></a><a name="index_term_5106"></a>The personnel data base for Microshaft contains *assertions* about company personnel. Here is the information about Ben Bitdiddle, the resident computer wizard:


<tt>(address&#160;(Bitdiddle&#160;Ben)&#160;(Slumerville&#160;(Ridge&#160;Road)&#160;10))<br>
(job&#160;(Bitdiddle&#160;Ben)&#160;(computer&#160;wizard))<br>
(salary&#160;(Bitdiddle&#160;Ben)&#160;60000)<br></tt>


Each assertion is a list (in this case a triple) whose elements can themselves be lists.


As resident wizard, Ben is in charge of the company's computer division, and he supervises two programmers and one technician. Here is the information about them:


<tt>(address&#160;(Hacker&#160;Alyssa&#160;P)&#160;(Cambridge&#160;(Mass&#160;Ave)&#160;78))<br>
(job&#160;(Hacker&#160;Alyssa&#160;P)&#160;(computer&#160;programmer))<br>
(salary&#160;(Hacker&#160;Alyssa&#160;P)&#160;40000)<br>
(supervisor&#160;(Hacker&#160;Alyssa&#160;P)&#160;(Bitdiddle&#160;Ben))<br>
(address&#160;(Fect&#160;Cy&#160;D)&#160;(Cambridge&#160;(Ames&#160;Street)&#160;3))<br>
(job&#160;(Fect&#160;Cy&#160;D)&#160;(computer&#160;programmer))<br>
(salary&#160;(Fect&#160;Cy&#160;D)&#160;35000)<br>
(supervisor&#160;(Fect&#160;Cy&#160;D)&#160;(Bitdiddle&#160;Ben))<br>
(address&#160;(Tweakit&#160;Lem&#160;E)&#160;(Boston&#160;(Bay&#160;State&#160;Road)&#160;22))<br>
(job&#160;(Tweakit&#160;Lem&#160;E)&#160;(computer&#160;technician))<br>
(salary&#160;(Tweakit&#160;Lem&#160;E)&#160;25000)<br>
(supervisor&#160;(Tweakit&#160;Lem&#160;E)&#160;(Bitdiddle&#160;Ben))<br></tt>


There is also a programmer trainee, who is supervised by Alyssa:


<tt>(address&#160;(Reasoner&#160;Louis)&#160;(Slumerville&#160;(Pine&#160;Tree&#160;Road)&#160;80))<br>
(job&#160;(Reasoner&#160;Louis)&#160;(computer&#160;programmer&#160;trainee))<br>
(salary&#160;(Reasoner&#160;Louis)&#160;30000)<br>
(supervisor&#160;(Reasoner&#160;Louis)&#160;(Hacker&#160;Alyssa&#160;P))<br></tt>


All of these people are in the computer division, as indicated by the word <tt>computer</tt> as the first item in their job descriptions.


Ben is a high-level employee. His supervisor is the company's big wheel himself:


<tt>(supervisor&#160;(Bitdiddle&#160;Ben)&#160;(Warbucks&#160;Oliver))<br>
(address&#160;(Warbucks&#160;Oliver)&#160;(Swellesley&#160;(Top&#160;Heap&#160;Road)))<br>
(job&#160;(Warbucks&#160;Oliver)&#160;(administration&#160;big&#160;wheel))<br>
(salary&#160;(Warbucks&#160;Oliver)&#160;150000)<br></tt>


Besides the computer division supervised by Ben, the company has an accounting division, consisting of a chief accountant and his assistant:


<tt>(address&#160;(Scrooge&#160;Eben)&#160;(Weston&#160;(Shady&#160;Lane)&#160;10))<br>
(job&#160;(Scrooge&#160;Eben)&#160;(accounting&#160;chief&#160;accountant))<br>
(salary&#160;(Scrooge&#160;Eben)&#160;75000)<br>
(supervisor&#160;(Scrooge&#160;Eben)&#160;(Warbucks&#160;Oliver))<br>
(address&#160;(Cratchet&#160;Robert)&#160;(Allston&#160;(N&#160;Harvard&#160;Street)&#160;16))<br>
(job&#160;(Cratchet&#160;Robert)&#160;(accounting&#160;scrivener))<br>
(salary&#160;(Cratchet&#160;Robert)&#160;18000)<br>
(supervisor&#160;(Cratchet&#160;Robert)&#160;(Scrooge&#160;Eben))<br></tt>


There is also a secretary for the big wheel:


<tt>(address&#160;(Aull&#160;DeWitt)&#160;(Slumerville&#160;(Onion&#160;Square)&#160;5))<br>
(job&#160;(Aull&#160;DeWitt)&#160;(administration&#160;secretary))<br>
(salary&#160;(Aull&#160;DeWitt)&#160;25000)<br>
(supervisor&#160;(Aull&#160;DeWitt)&#160;(Warbucks&#160;Oliver))<br></tt>


The data base also contains assertions about which kinds of jobs can be done by people holding other kinds of jobs. For instance, a computer wizard can do the jobs of both a computer programmer and a computer technician:


<tt>(can-do-job&#160;(computer&#160;wizard)&#160;(computer&#160;programmer))<br>
(can-do-job&#160;(computer&#160;wizard)&#160;(computer&#160;technician))<br></tt>


A computer programmer could fill in for a trainee:


<tt>(can-do-job&#160;(computer&#160;programmer)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(computer&#160;programmer&#160;trainee))<br></tt>


<a name="index_term_5108"></a>Also, as is well known,


<tt>(can-do-job&#160;(administration&#160;secretary)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(administration&#160;big&#160;wheel))<br></tt>


<a name="%_sec_Temp_650"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_650">Simple queries</a></h4>

<a name="index_term_5110"></a> The query language allows users to retrieve information from the data base by posing queries in response to the system's prompt. For example, to find all computer programmers one can say


<tt><i>;;;&#160;Query&#160;input:</i><br>
(job&#160;?x&#160;(computer&#160;programmer))<br></tt>


The system will respond with the following items:


<tt><i>;;;&#160;Query&#160;results:</i><br>
(job&#160;(Hacker&#160;Alyssa&#160;P)&#160;(computer&#160;programmer))<br>
(job&#160;(Fect&#160;Cy&#160;D)&#160;(computer&#160;programmer))<br></tt>


<a name="index_term_5112"></a>The input query specifies that we are looking for entries in the data base that match a certain *pattern*. In this example, the pattern specifies entries consisting of three items, of which the first is the literal symbol <tt>job</tt>, the second can be anything, and the third is the literal list <tt>(computer programmer)</tt>. The "anything" that can be the second item in the matching list is specified by a <a name="index_term_5114"></a>*pattern variable*, <tt>?x</tt>. The general form of a pattern variable is a symbol, taken to be the name of the variable, preceded by a question mark. We will see below why it is useful to specify names for pattern variables rather than just putting <tt>?</tt> into patterns to represent "anything." The system responds to a simple query by showing all entries in the data base that match the specified pattern.


A pattern can have more than one variable. For example, the query


<tt>(address&#160;?x&#160;?y)<br></tt>


will list all the employees' addresses.


A pattern can have no variables, in which case the query simply determines whether that pattern is an entry in the data base. If so, there will be one match; if not, there will be no matches.


The same pattern variable can appear more than once in a query, specifying that the same "anything" must appear in each position. This is why variables have names. For example,


<tt>(supervisor&#160;?x&#160;?x)<br></tt>


finds all people who supervise themselves (though there are no such assertions in our sample data base).


The query


<tt>(job&#160;?x&#160;(computer&#160;?type))<br></tt>


matches all job entries whose third item is a two-element list whose first item is <tt>computer</tt>:


<tt>(job&#160;(Bitdiddle&#160;Ben)&#160;(computer&#160;wizard))<br>
(job&#160;(Hacker&#160;Alyssa&#160;P)&#160;(computer&#160;programmer))<br>
(job&#160;(Fect&#160;Cy&#160;D)&#160;(computer&#160;programmer))<br>
(job&#160;(Tweakit&#160;Lem&#160;E)&#160;(computer&#160;technician))<br></tt>


This same pattern does *not* match


<tt>(job&#160;(Reasoner&#160;Louis)&#160;(computer&#160;programmer&#160;trainee))<br></tt>


because the third item in the entry is a list of three elements, and the pattern's third item specifies that there should be two elements. If we wanted to change the pattern so that the third item could be any list beginning with <tt>computer</tt>, we could specify<a name="index_term_5116"></a><a name="call_footnote_Temp_651" href="#footnote_Temp_651" id="call_footnote_Temp_651"><sup><small>62</small></sup></a>


<tt>(job&#160;?x&#160;(computer&#160;.&#160;?type))<br></tt>


For example,


<tt>(computer&#160;.&#160;?type)<br></tt>


matches the data


<tt>(computer&#160;programmer&#160;trainee)<br></tt>


with <tt>?type</tt> as the list <tt>(programmer trainee)</tt>. It also matches the data


<tt>(computer&#160;programmer)<br></tt>


with <tt>?type</tt> as the list <tt>(programmer)</tt>, and matches the data


<tt>(computer)<br></tt>


with <tt>?type</tt> as the empty list <tt>()</tt>.


We can describe the query language's processing of simple queries as follows:

<ul>
  <li>The system finds all assignments to variables in the query <a name="index_term_5118"></a>pattern that *satisfy* the pattern -- that is, all sets of values for the variables such that if the pattern variables are <a name="index_term_5120"></a>*instantiated with* (replaced by) the values, the result is in the data base.</li>

  <li>The system responds to the query by listing all instantiations of the query pattern with the variable assignments that satisfy it.</li>
</ul>

Note that if the pattern has no variables, the query reduces to a determination of whether that pattern is in the data base. If so, the empty assignment, which assigns no values to variables, satisfies that pattern for that data base.


<a name="%_thm_4.55"></a> **Exercise 4.55.**&#160;&#160;Give simple queries that retrieve the following information from the data base:


a. all people supervised by Ben Bitdiddle;


b. the names and jobs of all people in the accounting division;


c. the names and addresses of all people who live in Slumerville.


<a name="%_sec_Temp_653"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_653">Compound queries</a></h4>

<a name="index_term_5122"></a> Simple queries form the primitive operations of the query language. In order to form compound operations, the query language provides means of combination. One thing that makes the query language a logic programming language is that the means of combination mirror the means of combination used in forming logical expressions: <tt>and</tt>, <tt>or</tt>, and <tt>not</tt>. (Here <tt>and</tt>, <tt>or</tt>, and <tt>not</tt> are not the Lisp primitives, but rather operations built into the query language.)


<a name="index_term_5124"></a>We can use <tt>and</tt> as follows to find the addresses of all the computer programmers:


<tt>(and&#160;(job&#160;?person&#160;(computer&#160;programmer))<br>
&#160;&#160;&#160;&#160;&#160;(address&#160;?person&#160;?where))<br></tt>


The resulting output is


<tt>(and&#160;(job&#160;(Hacker&#160;Alyssa&#160;P)&#160;(computer&#160;programmer))<br>
&#160;&#160;&#160;&#160;&#160;(address&#160;(Hacker&#160;Alyssa&#160;P)&#160;(Cambridge&#160;(Mass&#160;Ave)&#160;78)))<br>
(and&#160;(job&#160;(Fect&#160;Cy&#160;D)&#160;(computer&#160;programmer))<br>
&#160;&#160;&#160;&#160;&#160;(address&#160;(Fect&#160;Cy&#160;D)&#160;(Cambridge&#160;(Ames&#160;Street)&#160;3)))<br></tt>


<a name="index_term_5126"></a>In general,


<tt>(and&#160;&lt;*query<sub>1</sub>*&gt;&#160;&lt;*query<sub>2</sub>*&gt;&#160;</tt>... &lt;*query<sub>*n*</sub>*&gt;)<br>


is satisfied by all sets of values for the pattern variables that simultaneously satisfy &lt;*query<sub>1</sub>*&gt; <tt>...</tt> &lt;*query<sub>*n*</sub>*&gt;.


As for simple queries, the system processes a compound query by finding all assignments to the pattern variables that satisfy the query, then displaying instantiations of the query with those values.


<a name="index_term_5128"></a>Another means of constructing compound queries is through <tt>or</tt>. For example,


<tt>(or&#160;(supervisor&#160;?x&#160;(Bitdiddle&#160;Ben))<br>
&#160;&#160;&#160;&#160;(supervisor&#160;?x&#160;(Hacker&#160;Alyssa&#160;P)))<br></tt>


will find all employees supervised by Ben Bitdiddle or Alyssa P. Hacker:


<tt>(or&#160;(supervisor&#160;(Hacker&#160;Alyssa&#160;P)&#160;(Bitdiddle&#160;Ben))<br>
&#160;&#160;&#160;&#160;(supervisor&#160;(Hacker&#160;Alyssa&#160;P)&#160;(Hacker&#160;Alyssa&#160;P)))<br>
(or&#160;(supervisor&#160;(Fect&#160;Cy&#160;D)&#160;(Bitdiddle&#160;Ben))<br>
&#160;&#160;&#160;&#160;(supervisor&#160;(Fect&#160;Cy&#160;D)&#160;(Hacker&#160;Alyssa&#160;P)))<br>
(or&#160;(supervisor&#160;(Tweakit&#160;Lem&#160;E)&#160;(Bitdiddle&#160;Ben))<br>
&#160;&#160;&#160;&#160;(supervisor&#160;(Tweakit&#160;Lem&#160;E)&#160;(Hacker&#160;Alyssa&#160;P)))<br>
(or&#160;(supervisor&#160;(Reasoner&#160;Louis)&#160;(Bitdiddle&#160;Ben))<br>
&#160;&#160;&#160;&#160;(supervisor&#160;(Reasoner&#160;Louis)&#160;(Hacker&#160;Alyssa&#160;P)))<br></tt>


In general,


<tt>(or&#160;&lt;*query<sub>1</sub>*&gt;&#160;&lt;*query<sub>2</sub>*&gt;&#160;</tt>... &lt;*query<sub>*n*</sub>*&gt;)<br>


is satisfied by all sets of values for the pattern variables that satisfy at least one of &lt;*query<sub>1</sub>*&gt; <tt>...</tt> &lt;*query<sub>*n*</sub>*&gt;.


<a name="index_term_5130"></a>Compound queries can also be formed with <tt>not</tt>. For example,


<tt>(and&#160;(supervisor&#160;?x&#160;(Bitdiddle&#160;Ben))<br>
&#160;&#160;&#160;&#160;&#160;(not&#160;(job&#160;?x&#160;(computer&#160;programmer))))<br></tt>


finds all people supervised by Ben Bitdiddle who are not computer programmers. In general,


<tt>(not&#160;&lt;*query<sub>1</sub>*&gt;)<br></tt>


is satisfied by all assignments to the pattern variables that do not satisfy &lt;*query<sub>1</sub>*&gt;.<a name="call_footnote_Temp_654" href="#footnote_Temp_654" id="call_footnote_Temp_654"><sup><small>63</small></sup></a>


<a name="index_term_5132"></a>The final combining form is called <tt>lisp-value</tt>. When <tt>lisp-value</tt> is the first element of a pattern, it specifies that the next element is a Lisp predicate to be applied to the rest of the (instantiated) elements as arguments. In general,


<tt>(lisp-value&#160;&lt;*predicate*&gt;&#160;&lt;*arg<sub>1</sub>*&gt;&#160;</tt>... &lt;*arg<sub>*n*</sub>*&gt;)<br>


will be satisfied by assignments to the pattern variables for which the &lt;*predicate*&gt; applied to the instantiated &lt;*arg<sub>1</sub>*&gt; <tt>...</tt> &lt;*arg<sub>*n*</sub>*&gt; is true. For example, to find all people whose salary is greater than $30,000 we could write<a name="call_footnote_Temp_655" href="#footnote_Temp_655" id="call_footnote_Temp_655"><sup><small>64</small></sup></a>


<tt>(and&#160;(salary&#160;?person&#160;?amount)<br>
&#160;&#160;&#160;&#160;&#160;(lisp-value&#160;&gt;&#160;?amount&#160;30000))<br></tt>


<a name="%_thm_4.56"></a> **Exercise 4.56.**&#160;&#160;Formulate compound queries that retrieve the following information:


a. the names of all people who are supervised by Ben Bitdiddle, together with their addresses;


b. all people whose salary is less than Ben Bitdiddle's, together with their salary and Ben Bitdiddle's salary;


c. all people who are supervised by someone who is not in the computer division, together with the supervisor's name and job.


<a name="%_sec_Temp_657"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_657">Rules</a></h4>

<a name="index_term_5136"></a> <a name="index_term_5138"></a>In addition to primitive queries and compound queries, the query language provides means for abstracting queries. These are given by *rules*. The rule


<tt><a name="index_term_5140"></a>(rule&#160;(lives-near&#160;?person-1&#160;?person-2)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(and&#160;(address&#160;?person-1&#160;(?town&#160;.&#160;?rest-1))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(address&#160;?person-2&#160;(?town&#160;.&#160;?rest-2))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(not&#160;(same&#160;?person-1&#160;?person-2))))<br></tt>


specifies that two people live near each other if they live in the same town. The final <tt>not</tt> clause prevents the rule from saying that all people live near themselves. The <tt>same</tt> relation is defined by a very simple rule:<a name="call_footnote_Temp_658" href="#footnote_Temp_658" id="call_footnote_Temp_658"><sup><small>65</small></sup></a>


<tt><a name="index_term_5142"></a>(rule&#160;(same&#160;?x&#160;?x))<br></tt>


The following rule declares that a person is a "wheel" in an organization if he supervises someone who is in turn a supervisor:


<tt><a name="index_term_5144"></a>(rule&#160;(wheel&#160;?person)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(and&#160;(supervisor&#160;?middle-manager&#160;?person)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(supervisor&#160;?x&#160;?middle-manager)))<br></tt>


The general form of a rule is


<tt>(rule&#160;&lt;*conclusion*&gt;&#160;&lt;*body*&gt;)<br></tt>


where &lt;*conclusion*&gt; is a pattern and &lt;*body*&gt; is any query.<a name="call_footnote_Temp_659" href="#footnote_Temp_659" id="call_footnote_Temp_659"><sup><small>66</small></sup></a> We can think of a rule as representing a large (even infinite) set of assertions, namely all instantiations of the rule conclusion with variable assignments that satisfy the rule body. When we described simple queries (patterns), we said that an assignment to variables satisfies a pattern if the instantiated pattern is in the data base. But the pattern needn't be explicitly in the data base as an assertion. It <a name="index_term_5148"></a>can be an implicit assertion implied by a rule. For example, the query


<tt>(lives-near&#160;?x&#160;(Bitdiddle&#160;Ben))<br></tt>


results in


<tt>(lives-near&#160;(Reasoner&#160;Louis)&#160;(Bitdiddle&#160;Ben))<br>
(lives-near&#160;(Aull&#160;DeWitt)&#160;(Bitdiddle&#160;Ben))<br></tt>


To find all computer programmers who live near Ben Bitdiddle, we can ask


<tt>(and&#160;(job&#160;?x&#160;(computer&#160;programmer))<br>
&#160;&#160;&#160;&#160;&#160;(lives-near&#160;?x&#160;(Bitdiddle&#160;Ben)))<br></tt>


<a name="index_term_5150"></a>As in the case of compound procedures, rules can be used as parts of other rules (as we saw with the <tt>lives-near</tt> rule above) or even be defined recursively. For instance, the rule


<tt><a name="index_term_5152"></a>(rule&#160;(outranked-by&#160;?staff-person&#160;?boss)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(or&#160;(supervisor&#160;?staff-person&#160;?boss)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(and&#160;(supervisor&#160;?staff-person&#160;?middle-manager)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(outranked-by&#160;?middle-manager&#160;?boss))))<br></tt>


says that a staff person is outranked by a boss in the organization if the boss is the person's supervisor or (recursively) if the person's supervisor is outranked by the boss.


<a name="%_thm_4.57"></a> **Exercise 4.57.**&#160;&#160;Define a rule that says that person 1 can replace person 2 if either person 1 does the same job as person 2 or someone who does person 1's job can also do person&#160;2's job, and if person 1 and person 2 are not the same person. Using your rule, give queries that find the following:


a.&#160;&#160;all people who can replace Cy D. Fect;


b.&#160;&#160;all people who can replace someone who is being paid more than they are, together with the two salaries.


<a name="%_thm_4.58"></a> **Exercise 4.58.**&#160;&#160;Define a rule that says that a person is a "big shot" in a division if the person works in the division but does not have a supervisor who works in the division.


<a name="%_thm_4.59"></a> **Exercise 4.59.**&#160;&#160;Ben Bitdiddle has missed one meeting too many. Fearing that his habit of forgetting meetings could cost him his job, Ben decides to do something about it. He adds all the weekly meetings of the firm to the Microshaft data base by asserting the following:


<tt>(meeting&#160;accounting&#160;(Monday&#160;9am))<br>
(meeting&#160;administration&#160;(Monday&#160;10am))<br>
(meeting&#160;computer&#160;(Wednesday&#160;3pm))<br>
(meeting&#160;administration&#160;(Friday&#160;1pm))<br></tt>


Each of the above assertions is for a meeting of an entire division. Ben also adds an entry for the company-wide meeting that spans all the divisions. All of the company's employees attend this meeting.


<tt>(meeting&#160;whole-company&#160;(Wednesday&#160;4pm))<br></tt>


a. On Friday morning, Ben wants to query the data base for all the meetings that occur that day. What query should he use?


b. Alyssa P. Hacker is unimpressed. She thinks it would be much more useful to be able to ask for her meetings by specifying her name. So she designs a rule that says that a person's meetings include all <tt>whole-company</tt> meetings plus all meetings of that person's division. Fill in the body of Alyssa's rule.


<tt>(rule&#160;(meeting-time&#160;?person&#160;?day-and-time)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&lt;*rule-body*&gt;)<br></tt>


c. Alyssa arrives at work on Wednesday morning and wonders what meetings she has to attend that day. Having defined the above rule, what query should she make to find this out?


<a name="%_thm_4.60"></a> **Exercise 4.60.**&#160;&#160;<a name="index_term_5154"></a>By giving the query


<tt>(lives-near&#160;?person&#160;(Hacker&#160;Alyssa&#160;P))<br></tt>


Alyssa P. Hacker is able to find people who live near her, with whom she can ride to work. On the other hand, when she tries to find all pairs of people who live near each other by querying


<tt>(lives-near&#160;?person-1&#160;?person-2)<br></tt>


she notices that each pair of people who live near each other is listed twice; for example,


<tt>(lives-near&#160;(Hacker&#160;Alyssa&#160;P)&#160;(Fect&#160;Cy&#160;D))<br>
(lives-near&#160;(Fect&#160;Cy&#160;D)&#160;(Hacker&#160;Alyssa&#160;P))<br></tt>


Why does this happen? Is there a way to find a list of people who live near each other, in which each pair appears only once? Explain.


<a name="%_sec_Temp_664"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_664">Logic as programs</a></h4>

<a name="index_term_5156"></a> We can regard a rule as a kind of logical implication: *If* an assignment of values to pattern variables satisfies the body, *then* it satisfies the conclusion. Consequently, we can regard the query language as having the ability to perform *logical deductions* based upon the rules. As an example, consider the <tt>append</tt> operation described at the beginning of section&#160;<a href="#%_sec_4.4">4.4</a>. As we said, <tt>append</tt> can be characterized by the following two rules:

<ul>
  <li>For any list <tt>y</tt>, the empty list and <tt>y</tt> <tt>append</tt> to form <tt>y</tt>.</li>

  <li>For any <tt>u</tt>, <tt>v</tt>, <tt>y</tt>, and <tt>z</tt>, <tt>(cons u v)</tt> and <tt>y</tt> <tt>append</tt> to form <tt>(cons&#160;u&#160;z)</tt> if <tt>v</tt> and <tt>y</tt> <tt>append</tt> to form <tt>z</tt>.</li>
</ul>

To express this in our query language, we define two rules for a relation


<tt>(append-to-form&#160;x&#160;y&#160;z)<br></tt>


which we can interpret to mean "<tt>x</tt> and <tt>y</tt> <tt>append</tt> to form <tt>z</tt>":


<tt><a name="index_term_5158"></a>(rule&#160;(append-to-form&#160;()&#160;?y&#160;?y))<br>
(rule&#160;(append-to-form&#160;(?u&#160;.&#160;?v)&#160;?y&#160;(?u&#160;.&#160;?z))<br>
&#160;&#160;&#160;&#160;&#160;&#160;(append-to-form&#160;?v&#160;?y&#160;?z))<br></tt>


<a name="index_term_5160"></a>The first rule has no body, which means that the conclusion holds for any value of <tt>?y</tt>. Note how the second rule makes use of <a name="index_term_5162"></a>dotted-tail notation to name the <tt>car</tt> and <tt>cdr</tt> of a list.


Given these two rules, we can formulate queries that compute the <tt>append</tt> of two lists:


<tt><i>;;;&#160;Query&#160;input:</i><br>
(append-to-form&#160;(a&#160;b)&#160;(c&#160;d)&#160;?z)<br>
<i>;;;&#160;Query&#160;results:</i><br>
(append-to-form&#160;(a&#160;b)&#160;(c&#160;d)&#160;(a&#160;b&#160;c&#160;d))<br></tt>


What is more striking, we can use the same rules to ask the question "Which list, when <tt>append</tt>ed to <tt>(a b)</tt>, yields <tt>(a b c d)</tt>?" This is done as follows:


<tt><i>;;;&#160;Query&#160;input:</i><br>
(append-to-form&#160;(a&#160;b)&#160;?y&#160;(a&#160;b&#160;c&#160;d))<br>
<i>;;;&#160;Query&#160;results:</i><br>
(append-to-form&#160;(a&#160;b)&#160;(c&#160;d)&#160;(a&#160;b&#160;c&#160;d))<br></tt>


We can also ask for all pairs of lists that <tt>append</tt> to form <tt>(a b c d)</tt>:


<tt><i>;;;&#160;Query&#160;input:</i><br>
(append-to-form&#160;?x&#160;?y&#160;(a&#160;b&#160;c&#160;d))<br>
<i>;;;&#160;Query&#160;results:</i><br>
(append-to-form&#160;()&#160;(a&#160;b&#160;c&#160;d)&#160;(a&#160;b&#160;c&#160;d))<br>
(append-to-form&#160;(a)&#160;(b&#160;c&#160;d)&#160;(a&#160;b&#160;c&#160;d))<br>
(append-to-form&#160;(a&#160;b)&#160;(c&#160;d)&#160;(a&#160;b&#160;c&#160;d))<br>
(append-to-form&#160;(a&#160;b&#160;c)&#160;(d)&#160;(a&#160;b&#160;c&#160;d))<br>
(append-to-form&#160;(a&#160;b&#160;c&#160;d)&#160;()&#160;(a&#160;b&#160;c&#160;d))<br></tt>


The query system may seem to exhibit quite a bit of intelligence in using the rules to deduce the answers to the queries above. Actually, as we will see in the next section, the system is following a well-determined algorithm in unraveling the rules. Unfortunately, although the system works impressively in the <tt>append</tt> case, the general methods may break down in more complex cases, as we will see in section&#160;<a href="#%_sec_4.4.3">4.4.3</a>.


<a name="%_thm_4.61"></a> **Exercise 4.61.**&#160;&#160;The following rules implement a <tt>next-to</tt> relation that finds adjacent elements of a list:


<tt><a name="index_term_5164"></a>(rule&#160;(?x&#160;next-to&#160;?y&#160;in&#160;(?x&#160;?y&#160;.&#160;?u)))<br>
<br>
(rule&#160;(?x&#160;next-to&#160;?y&#160;in&#160;(?v&#160;.&#160;?z))<br>
&#160;&#160;&#160;&#160;&#160;&#160;(?x&#160;next-to&#160;?y&#160;in&#160;?z))<br></tt>


What will the response be to the following queries?


<tt>(?x&#160;next-to&#160;?y&#160;in&#160;(1&#160;(2&#160;3)&#160;4))<br>
<br>
(?x&#160;next-to&#160;1&#160;in&#160;(2&#160;1&#160;3&#160;1))<br></tt>


<a name="%_thm_4.62"></a> **Exercise 4.62.**&#160;&#160;<a name="index_term_5166"></a>Define rules to implement the <tt>last-pair</tt> operation of exercise&#160;<a href="chapter_2_section_2.html#exercise_2_17">2.17</a>, which returns a list containing the last element of a nonempty list. Check your rules on queries such as <tt>(last-pair (3) ?x)</tt>, <tt>(last-pair (1 2 3) ?x)</tt>, and <tt>(last-pair (2 ?x) (3))</tt>. Do your rules work correctly on queries such as <tt>(last-pair ?x (3))</tt> ?


<a name="%_thm_4.63"></a> **Exercise 4.63.**&#160;&#160;<a name="index_term_5168"></a><a name="index_term_5170"></a>The following data base (see Genesis 4) traces the genealogy of the descendants of Ada back to Adam, by way of Cain:


<tt>(son&#160;Adam&#160;Cain)<br>
(son&#160;Cain&#160;Enoch)<br>
(son&#160;Enoch&#160;Irad)<br>
(son&#160;Irad&#160;Mehujael)<br>
(son&#160;Mehujael&#160;Methushael)<br>
(son&#160;Methushael&#160;Lamech)<br>
(wife&#160;Lamech&#160;Ada)<br>
(son&#160;Ada&#160;Jabal)<br>
(son&#160;Ada&#160;Jubal)<br></tt>


Formulate rules such as "If *S* is the son of *F*, and *F* is the son of *G*, then *S* is the grandson of *G*" and "If *W* is the wife of *M*, and *S* is the son of *W*, then *S* is the son of *M*" (which was supposedly more true in biblical times than today) that will enable the query system to find the grandson of Cain; the sons of Lamech; the grandsons of Methushael. (See exercise&#160;<a href="#%_thm_4.69">4.69</a> for some rules to deduce more complicated relationships.)


<a name="%_sec_4.4.2"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_4.4.2">4.4.2&#160;&#160;How the Query System Works</a></h3>

<a name="index_term_5172"></a> In section&#160;<a href="#%_sec_4.4.4">4.4.4</a> we will present an implementation of the query interpreter as a collection of procedures. In this section we give an overview that explains the general structure of the system independent of low-level implementation details. After describing the implementation of the interpreter, we will be in a position to understand some of its limitations and some of the subtle ways in which the query language's logical operations differ from the operations of mathematical logic.


It should be apparent that the query evaluator must perform some kind of search in order to match queries against facts and rules in the data base. One way to do this would be to implement the query system as a nondeterministic program, using the <tt>amb</tt> evaluator of section&#160;<a href="chapter_4_section_3.html#%_sec_4.3">4.3</a> (see exercise&#160;<a href="#%_thm_4.78">4.78</a>). Another possibility is to manage the search with the aid of streams. Our implementation follows this second approach.


The query system is organized around two central operations called *pattern matching* and *unification*. We first describe pattern matching and explain how this operation, together with the organization of information in terms of streams of frames, enables us to implement both simple and compound queries. We next discuss unification, a generalization of pattern matching needed to implement rules. Finally, we show how the entire query interpreter fits together through a procedure that classifies expressions in a manner analogous to the way <tt>eval</tt> classifies expressions for the interpreter described in section&#160;<a href="chapter_4_section_1.html#%_sec_4.1">4.1</a>.


<a name="%_sec_Temp_668"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_668">Pattern matching</a></h4>

<a name="index_term_5174"></a><a name="index_term_5176"></a> A *pattern matcher* is a program that tests whether some datum fits a specified pattern. For example, the data list <tt>((a b) c (a b))</tt> matches the pattern <tt>(?x c ?x)</tt> with the pattern variable <tt>?x</tt> bound to <tt>(a b)</tt>. The same data list matches the pattern <tt>(?x ?y ?z)</tt> with <tt>?x</tt> and <tt>?z</tt> both bound to <tt>(a b)</tt> and <tt>?y</tt> bound to <tt>c</tt>. It also matches the pattern <tt>((?x&#160;?y)&#160;c&#160;(?x&#160;?y))</tt> with <tt>?x</tt> bound to <tt>a</tt> and <tt>?y</tt> bound to <tt>b</tt>. However, it does not match the pattern <tt>(?x a ?y)</tt>, since that pattern specifies a list whose second element is the symbol <tt>a</tt>.


<a name="index_term_5178"></a><a name="index_term_5180"></a>The pattern matcher used by the query system takes as inputs a pattern, a datum, and a *frame* that specifies bindings for various pattern variables. It checks whether the datum matches the pattern in a way that is consistent with the bindings already in the frame. If so, it returns the given frame augmented by any bindings that may have been determined by the match. Otherwise, it indicates that the match has failed.


For example, using the pattern <tt>(?x ?y ?x)</tt> to match <tt>(a b a)</tt> given an empty frame will return a frame specifying that <tt>?x</tt> is bound to <tt>a</tt> and <tt>?y</tt> is bound to <tt>b</tt>. Trying the match with the same pattern, the same datum, and a frame specifying that <tt>?y</tt> is bound to <tt>a</tt> will fail. Trying the match with the same pattern, the same datum, and a frame in which <tt>?y</tt> is bound to <tt>b</tt> and <tt>?x</tt> is unbound will return the given frame augmented by a binding of <tt>?x</tt> to <tt>a</tt>.


<a name="index_term_5182"></a>The pattern matcher is all the mechanism that is needed to process simple queries that don't involve rules. For instance, to process the query


<tt>(job&#160;?x&#160;(computer&#160;programmer))<br></tt>


we scan through all assertions in the data base and select those that match the pattern with respect to an initially empty frame. For each match we find, we use the frame returned by the match to instantiate the pattern with a value for <tt>?x</tt>. <a name="%_sec_Temp_669"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_669">Streams of frames</a></h4>

<a name="index_term_5184"></a><a name="index_term_5186"></a> The testing of patterns against frames is organized through the use of streams. Given a single frame, the matching process runs through the data-base entries one by one. For each data-base entry, the matcher generates either a special symbol indicating that the match has failed or an extension to the frame. The results for all the data-base entries are collected into a stream, which is passed through a filter to weed out the failures. The result is a stream of all the frames that extend the given frame via a match to some assertion in the data base.<a name="call_footnote_Temp_670" href="#footnote_Temp_670" id="call_footnote_Temp_670"><sup><small>67</small></sup></a>


In our system, a query takes an input stream of frames and performs the above matching operation for every frame in the stream, as indicated in figure&#160;<a href="#%_fig_4.4">4.4</a>. That is, for each frame in the input stream, the query generates a new stream consisting of all extensions to that frame by matches to assertions in the data base. All these streams are then combined to form one huge stream, which contains all possible extensions of every frame in the input stream. This stream is the output of the query.


<a name="%_fig_4.4"></a>

<div align="left">
  <div align="left">
    **Figure 4.4:**&#160;&#160;A query processes a stream of frames.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_4_image_04.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="index_term_5194"></a>To answer a simple query, we use the query with an input stream consisting of a single empty frame. The resulting output stream contains all extensions to the empty frame (that is, all answers to our query). This stream of frames is then used to generate a stream of copies of the original query pattern with the variables instantiated by the values in each frame, and this is the stream that is finally printed.


<a name="%_sec_Temp_671"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_671">Compound queries</a></h4>

<a name="index_term_5196"></a> The real elegance of the stream-of-frames implementation is evident when we deal with compound queries. The processing of compound queries makes use of the ability of our matcher to demand that a match <a name="index_term_5198"></a>be consistent with a specified frame. For example, to handle the <tt>and</tt> of two queries, such as


<tt>(and&#160;(can-do-job&#160;?x&#160;(computer&#160;programmer&#160;trainee))<br>
&#160;&#160;&#160;&#160;&#160;(job&#160;?person&#160;?x))<br></tt>


(informally, "Find all people who can do the job of a computer programmer trainee"), we first find all entries that match the pattern


<tt>(can-do-job&#160;?x&#160;(computer&#160;programmer&#160;trainee))<br></tt>


This produces a stream of frames, each of which contains a binding for <tt>?x</tt>. Then for each frame in the stream we find all entries that match


<tt>(job&#160;?person&#160;?x)<br></tt>


in a way that is consistent with the given binding for <tt>?x</tt>. Each such match will produce a frame containing bindings for <tt>?x</tt> and <tt>?person</tt>. The <tt>and</tt> of two queries can be viewed as a series combination of the two component queries, as shown in figure&#160;<a href="#%_fig_4.5">4.5</a>. The frames that pass through the first query filter are filtered and further extended by the second query.


<a name="%_fig_4.5"></a>

<div align="left">
  <div align="left">
    **Figure 4.5:**&#160;&#160;The <tt>and</tt> combination of two queries is produced by operating on the stream of frames in series.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_4_image_05.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="index_term_5200"></a>Figure&#160;<a href="#%_fig_4.6">4.6</a> shows the analogous method for computing the <tt>or</tt> of two queries as a parallel combination of the two component queries. The input stream of frames is extended separately by each query. The two resulting streams are then merged to produce the final output stream.


<a name="%_fig_4.6"></a>

<div align="left">
  <div align="left">
    **Figure 4.6:**&#160;&#160;The <tt>or</tt> combination of two queries is produced by operating on the stream of frames in parallel and merging the results.
  </div>

  <table width="100%">
    <tr>
      <td><img src="img/chapter_4_image_06.gif" border="0"></td>
    </tr>

    <tr>
      <td></td>
    </tr>
  </table>
</div>

<a name="index_term_5202"></a>Even from this high-level description, it is apparent that the processing of compound queries can be slow. For example, since a query may produce more than one output frame for each input frame, and each query in an <tt>and</tt> gets its input frames from the previous query, an <tt>and</tt> query could, in the worst case, have to perform a number of matches that is exponential in the number of queries (see exercise&#160;<a href="#%_thm_4.76">4.76</a>).<a name="call_footnote_Temp_672" href="#footnote_Temp_672" id="call_footnote_Temp_672"><sup><small>68</small></sup></a> Though systems for handling only simple queries are quite practical, dealing with complex queries is extremely difficult.<a name="call_footnote_Temp_673" href="#footnote_Temp_673" id="call_footnote_Temp_673"><sup><small>69</small></sup></a>


<a name="index_term_5204"></a>From the stream-of-frames viewpoint, the <tt>not</tt> of some query acts as a filter that removes all frames for which the query can be satisfied. For instance, given the pattern


<tt>(not&#160;(job&#160;?x&#160;(computer&#160;programmer)))<br></tt>


we attempt, for each frame in the input stream, to produce extension frames that satisfy <tt>(job ?x (computer programmer))</tt>. We remove from the input stream all frames for which such extensions exist. The result is a stream consisting of only those frames in which the binding for <tt>?x</tt> does not satisfy <tt>(job ?x (computer programmer))</tt>. For example, in processing the query


<tt>(and&#160;(supervisor&#160;?x&#160;?y)<br>
&#160;&#160;&#160;&#160;&#160;(not&#160;(job&#160;?x&#160;(computer&#160;programmer))))<br></tt>


the first clause will generate frames with bindings for <tt>?x</tt> and <tt>?y</tt>. The <tt>not</tt> clause will then filter these by removing all frames in which the binding for <tt>?x</tt> satisfies the restriction that <tt>?x</tt> is a computer programmer.<a name="call_footnote_Temp_674" href="#footnote_Temp_674" id="call_footnote_Temp_674"><sup><small>70</small></sup></a>


<a name="index_term_5206"></a>The <tt>lisp-value</tt> special form is implemented as a similar filter on frame streams. We use each frame in the stream to instantiate any variables in the pattern, then apply the Lisp predicate. We remove from the input stream all frames for which the predicate fails. <a name="%_sec_Temp_675"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_675">Unification</a></h4>

<a name="index_term_5208"></a><a name="index_term_5210"></a> In order to handle rules in the query language, we must be able to find the rules whose conclusions match a given query pattern. Rule conclusions are like assertions except that they can contain variables, so we will need a generalization of pattern matching -- called *unification* -- in which both the "pattern" and the "datum" may contain variables.


A unifier takes two patterns, each containing constants and variables, and determines whether it is possible to assign values to the variables that will make the two patterns equal. If so, it returns a frame containing these bindings. For example, unifying <tt>(?x a ?y)</tt> and <tt>(?y ?z a)</tt> will specify a frame in which <tt>?x</tt>, <tt>?y</tt>, and <tt>?z</tt> must all be bound to <tt>a</tt>. On the other hand, unifying <tt>(?x ?y a)</tt> and <tt>(?x b ?y)</tt> will fail, because there is no value for <tt>?y</tt> that can make the two patterns equal. (For the second elements of the patterns to be equal, <tt>?y</tt> would have to be <tt>b</tt>; however, for the third elements to be equal, <tt>?y</tt> would have to be <tt>a</tt>.) The unifier used in the query system, like the pattern matcher, takes a frame as input and performs unifications that are consistent with this frame.


The unification algorithm is the most technically difficult part of the query system. With complex patterns, performing unification may seem to require deduction. To unify <tt>(?x ?x)</tt> and <tt>((a ?y c) (a b ?z))</tt>, for example, the algorithm must infer that <tt>?x</tt> should be <tt>(a b c)</tt>, <tt>?y</tt> should be <tt>b</tt>, and <tt>?z</tt> should be <tt>c</tt>. We may think of this process as solving a set of equations among the pattern components. In general, these are simultaneous equations, which may require substantial manipulation to solve.<a name="call_footnote_Temp_676" href="#footnote_Temp_676" id="call_footnote_Temp_676"><sup><small>71</small></sup></a> For example, unifying <tt>(?x ?x)</tt> and <tt>((a ?y c) (a b ?z))</tt> may be thought of as specifying the simultaneous equations


<tt>?x&#160; = &#160;(a&#160;?y&#160;c)<br>
?x&#160; = &#160;(a&#160;b&#160;?z)<br></tt>


These equations imply that


<tt>(a&#160;?y&#160;c)&#160; = &#160;(a&#160;b&#160;?z)<br></tt>


which in turn implies that


<tt>a&#160; = &#160;a,&#160;?y&#160; = &#160;b,&#160;c&#160; = &#160;?z,<br></tt>


and hence that


<tt>?x&#160; = &#160;(a&#160;b&#160;c)<br></tt>


<a name="index_term_5212"></a><a name="index_term_5214"></a>In a successful pattern match, all pattern variables become bound, and the values to which they are bound contain only constants. This is also true of all the examples of unification we have seen so far. In general, however, a successful unification may not completely determine the variable values; some variables may remain unbound and others may be bound to values that contain variables.


Consider the unification of <tt>(?x a)</tt> and <tt>((b ?y) ?z)</tt>. We can deduce that <tt>?x = (b ?y)</tt> and <tt>a = ?z</tt>, but we cannot further solve for <tt>?x</tt> or&#160;<tt>?y</tt>. The unification doesn't fail, since it is certainly possible to make the two patterns equal by assigning values to <tt>?x</tt> and <tt>?y</tt>. Since this match in no way restricts the values <tt>?y</tt> can take on, no binding for <tt>?y</tt> is put into the result frame. The match does, however, restrict the value of&#160;<tt>?x</tt>. Whatever value <tt>?y</tt> has, <tt>?x</tt> must be <tt>(b ?y)</tt>. A binding of <tt>?x</tt> to the pattern <tt>(b ?y)</tt> is thus put into the frame. If a value for <tt>?y</tt> is later determined and added to the frame (by a pattern match or unification that is required to be consistent with this frame), the previously bound <tt>?x</tt> will refer to this value.<a name="call_footnote_Temp_677" href="#footnote_Temp_677" id="call_footnote_Temp_677"><sup><small>72</small></sup></a> <a name="%_sec_Temp_678"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_678">Applying rules</a></h4>

<a name="index_term_5216"></a> Unification is the key to the component of the query system that makes inferences from rules. To see how this is accomplished, consider processing a query that involves applying a rule, such as


<tt>(lives-near&#160;?x&#160;(Hacker&#160;Alyssa&#160;P))<br></tt>


To process this query, we first use the ordinary pattern-match procedure described above to see if there are any assertions in the data base that match this pattern. (There will not be any in this case, since our data base includes no direct assertions about who lives near whom.) The next step is to attempt to unify the query pattern with the conclusion of each rule. We find that the pattern unifies with the conclusion of the rule


<tt>(rule&#160;(lives-near&#160;?person-1&#160;?person-2)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(and&#160;(address&#160;?person-1&#160;(?town&#160;.&#160;?rest-1))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(address&#160;?person-2&#160;(?town&#160;.&#160;?rest-2))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(not&#160;(same&#160;?person-1&#160;?person-2))))<br></tt>


resulting in a frame specifying that <tt>?person-2</tt> is bound to <tt>(Hacker Alyssa P)</tt> and that <tt>?x</tt> should be bound to (have the same value as) <tt>?person-1</tt>. Now, relative to this frame, we evaluate the compound query given by the body of the rule. Successful matches will extend this frame by providing a binding for <tt>?person-1</tt>, and consequently a value for <tt>?x</tt>, which we can use to instantiate the original query pattern.


In general, the query evaluator uses the following method to apply a rule when trying to establish a query pattern in a frame that specifies bindings for some of the pattern variables:

<ul>
  <li>Unify the query with the conclusion of the rule to form, if successful, an extension of the original frame.</li>

  <li>Relative to the extended frame, evaluate the query formed by the body of the rule.</li>
</ul>

<a name="index_term_5218"></a>Notice how similar this is to the method for applying a procedure in the <tt>eval</tt>/<tt>apply</tt> evaluator for Lisp:

<ul>
  <li>Bind the procedure's parameters to its arguments to form a frame that extends the original procedure environment.</li>

  <li>Relative to the extended environment, evaluate the expression formed by the body of the procedure.</li>
</ul>

The similarity between the two evaluators should come as no surprise. Just as procedure definitions are the means of abstraction in Lisp, rule definitions are the means of abstraction in the query language. In each case, we unwind the abstraction by creating appropriate bindings and evaluating the rule or procedure body relative to these.


<a name="%_sec_Temp_679"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_679">Simple queries</a></h4>

<a name="index_term_5220"></a> We saw earlier in this section how to evaluate simple queries in the absence of rules. Now that we have seen how to apply rules, we can describe how to evaluate simple queries by using both rules and assertions.


Given the query pattern and a stream of frames, we produce, for each frame in the input stream, two streams:

<ul>
  <li>a stream of extended frames obtained by matching the pattern against all assertions in the data base (using the pattern matcher), and</li>

  <li>a stream of extended frames obtained by applying all possible rules (using the unifier).<a name="call_footnote_Temp_680" href="#footnote_Temp_680" id="call_footnote_Temp_680"><sup><small>73</small></sup></a></li>
</ul>

Appending these two streams produces a stream that consists of all the ways that the given pattern can be satisfied consistent with the original frame. These streams (one for each frame in the input stream) are now all combined to form one large stream, which therefore consists of all the ways that any of the frames in the original input stream can be extended to produce a match with the given pattern. <a name="%_sec_Temp_681"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_681">The query evaluator and the driver loop</a></h4>

<a name="index_term_5226"></a> Despite the complexity of the underlying matching operations, the system is organized much like an evaluator for any language. The procedure that coordinates the matching operations is called <a name="index_term_5228"></a><a name="index_term_5230"></a><tt>qeval</tt>, and it plays a role analogous to that of the <tt>eval</tt> procedure for Lisp. <tt>Qeval</tt> takes as inputs a query and a stream of frames. Its output is a stream of frames, corresponding to successful matches to the query pattern, that extend some frame in the input stream, as indicated in figure&#160;<a href="#%_fig_4.4">4.4</a>. Like <tt>eval</tt>, <tt>qeval</tt> classifies the different types of expressions (queries) and dispatches to an appropriate procedure for each. There is a procedure for each special form (<tt>and</tt>, <tt>or</tt>, <tt>not</tt>, and <tt>lisp-value</tt>) and one for simple queries.


<a name="index_term_5232"></a><a name="index_term_5234"></a>The driver loop, which is analogous to the <tt>driver-loop</tt> procedure for the other evaluators in this chapter, reads queries from the terminal. For each query, it calls <tt>qeval</tt> with the query and a stream that consists of a single empty frame. This will produce the stream of all possible matches (all possible extensions to the empty frame). For each frame in the resulting stream, it instantiates the original query using the values of the variables found in the frame. This stream of instantiated queries is then printed.<a name="call_footnote_Temp_682" href="#footnote_Temp_682" id="call_footnote_Temp_682"><sup><small>74</small></sup></a>


<a name="index_term_5240"></a><a name="index_term_5242"></a>The driver also checks for the special command <tt>assert!</tt>, which signals that the input is not a query but rather an assertion or rule to be added to the data base. For instance,


<tt>(assert!&#160;(job&#160;(Bitdiddle&#160;Ben)&#160;(computer&#160;wizard)))<br>
(assert!&#160;(rule&#160;(wheel&#160;?person)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(and&#160;(supervisor&#160;?middle-manager&#160;?person)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(supervisor&#160;?x&#160;?middle-manager))))<br></tt>


<a name="%_sec_4.4.3"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_4.4.3">4.4.3&#160;&#160;Is Logic Programming Mathematical Logic?</a></h3>

<a name="index_term_5244"></a><a name="index_term_5246"></a> The means of combination used in the query language may at first seem identical to the operations <tt>and</tt>, <tt>or</tt>, and <tt>not</tt> of mathematical logic, and the application of query-language rules is in fact accomplished through a legitimate method of <a name="index_term_5248"></a>inference.<a name="call_footnote_Temp_683" href="#footnote_Temp_683" id="call_footnote_Temp_683"><sup><small>75</small></sup></a> This identification of the query language with mathematical logic is not really valid, though, because the query language provides a <a name="index_term_5252"></a>*control structure* that interprets the logical statements procedurally. We can often take advantage of this control structure. For example, to find all of the supervisors of programmers we could formulate a query in either of two logically equivalent forms:


<tt>(and&#160;(job&#160;?x&#160;(computer&#160;programmer))<br>
&#160;&#160;&#160;&#160;&#160;(supervisor&#160;?x&#160;?y))<br></tt>


or


<tt>(and&#160;(supervisor&#160;?x&#160;?y)<br>
&#160;&#160;&#160;&#160;&#160;(job&#160;?x&#160;(computer&#160;programmer)))<br></tt>


<a name="index_term_5254"></a>If a company has many more supervisors than programmers (the usual case), it is better to use the first form rather than the second because the data base must be scanned for each intermediate result (frame) produced by the first clause of the <tt>and</tt>.


<a name="index_term_5256"></a><a name="index_term_5258"></a>The aim of logic programming is to provide the programmer with techniques for decomposing a computational problem into two separate problems: "what" is to be computed, and "how" this should be computed. This is accomplished by selecting a subset of the statements of mathematical logic that is powerful enough to be able to describe anything one might want to compute, yet weak enough to have a controllable procedural interpretation. The intention here is that, on the one hand, a program specified in a logic programming language should be an effective program that can be carried out by a computer. Control ("how" to compute) is effected by using the order of evaluation of the language. We should be able to arrange the order of clauses and the order of subgoals within each clause so that the computation is done in an order deemed to be effective and efficient. At the same time, we should be able to view the result of the computation ("what" to compute) as a simple consequence of the laws of logic.


Our query language can be regarded as just such a procedurally interpretable subset of mathematical logic. An assertion represents a simple fact (an atomic proposition). A rule represents the implication that the rule conclusion holds for those cases where the rule body holds. A rule has a natural procedural interpretation: To establish the conclusion of the rule, establish the body of the rule. Rules, therefore, specify computations. However, because rules can also be regarded as statements of mathematical logic, we can justify any "inference" accomplished by a logic program by asserting that the same result could be obtained by working entirely within mathematical logic.<a name="call_footnote_Temp_684" href="#footnote_Temp_684" id="call_footnote_Temp_684"><sup><small>76</small></sup></a>


<a name="%_sec_Temp_685"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_685">Infinite loops</a></h4>

<a name="index_term_5260"></a> A consequence of the procedural interpretation of logic programs is that it is possible to construct hopelessly inefficient programs for solving certain problems. An extreme case of inefficiency occurs when the system falls into infinite loops in making deductions. As a simple example, suppose we are setting up a data base of famous marriages, including


<a name="index_term_5262"></a>


<tt>(assert!&#160;(married&#160;Minnie&#160;Mickey))<br></tt>


If we now ask


<tt>(married&#160;Mickey&#160;?who)<br></tt>


we will get no response, because the system doesn't know that if *A* is married to *B*, then *B* is married to *A*. So we assert the rule


<tt>(assert!&#160;(rule&#160;(married&#160;?x&#160;?y)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(married&#160;?y&#160;?x)))<br></tt>


and again query


<tt>(married&#160;Mickey&#160;?who)<br></tt>


Unfortunately, this will drive the system into an infinite loop, as follows:

<ul>
  <li>The system finds that the <tt>married</tt> rule is applicable; that is, the rule conclusion <tt>(married ?x ?y)</tt> successfully unifies with the query pattern <tt>(married Mickey ?who)</tt> to produce a frame in which <tt>?x</tt> is bound to <tt>Mickey</tt> and <tt>?y</tt> is bound to <tt>?who</tt>. So the interpreter proceeds to evaluate the rule body <tt>(married ?y ?x)</tt> in this frame -- in effect, to process the query <tt>(married ?who Mickey)</tt>.</li>

  <li>One answer appears directly as an assertion in the data base: <tt>(married Minnie Mickey)</tt>.</li>

  <li>The <tt>married</tt> rule is also applicable, so the interpreter again evaluates the rule body, which this time is equivalent to <tt>(married Mickey ?who)</tt>.</li>
</ul>

The system is now in an infinite loop. Indeed, whether the system will find the simple answer <tt>(married Minnie Mickey)</tt> before it goes into the loop depends on implementation details concerning the order in which the system checks the items in the data base. This is a very simple example of the kinds of loops that can occur. Collections of interrelated rules can lead to loops that are much harder to anticipate, and the appearance of a loop can depend on the order of clauses in an <tt>and</tt> (see exercise&#160;<a href="#%_thm_4.64">4.64</a>) or on low-level details concerning the order in which the system processes queries.<a name="call_footnote_Temp_686" href="#footnote_Temp_686" id="call_footnote_Temp_686"><sup><small>77</small></sup></a> <a name="%_sec_Temp_687"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_687">Problems with <tt>not</tt></a></h4>

<a name="index_term_5266"></a> <a name="index_term_5268"></a>Another quirk in the query system concerns <tt>not</tt>. Given the data base of section&#160;<a href="#%_sec_4.4.1">4.4.1</a>, consider the following two queries:


<tt>(and&#160;(supervisor&#160;?x&#160;?y)<br>
&#160;&#160;&#160;&#160;&#160;(not&#160;(job&#160;?x&#160;(computer&#160;programmer))))<br>
(and&#160;(not&#160;(job&#160;?x&#160;(computer&#160;programmer)))<br>
&#160;&#160;&#160;&#160;&#160;(supervisor&#160;?x&#160;?y))<br></tt>


These two queries do not produce the same result. The first query begins by finding all entries in the data base that match <tt>(supervisor ?x ?y)</tt>, and then filters the resulting frames by removing the ones in which the value of <tt>?x</tt> satisfies <tt>(job ?x (computer programmer))</tt>. The second query begins by filtering the incoming frames to remove those that can satisfy <tt>(job ?x (computer programmer))</tt>. Since the only incoming frame is empty, it checks the data base to see if there are any patterns that satisfy <tt>(job ?x (computer programmer))</tt>. Since there generally are entries of this form, the <tt>not</tt> clause filters out the empty frame and returns an empty stream of frames. Consequently, the entire compound query returns an empty stream.


The trouble is that our implementation of <tt>not</tt> really is meant to serve as a filter on values for the variables. If a <tt>not</tt> clause is processed with a frame in which some of the variables remain unbound (as does <tt>?x</tt> in the example above), the system will produce unexpected results. Similar problems occur with the use of <a name="index_term_5270"></a><tt>lisp-value</tt> -- the Lisp predicate can't work if some of its arguments are unbound. See exercise&#160;<a href="#%_thm_4.77">4.77</a>.


There is also a much more serious way in which the <tt>not</tt> of the query language differs from the <tt>not</tt> of mathematical logic. In logic, we interpret the statement "not *P*" to mean that *P* is not true. In the query system, however, "not *P*" means that *P* is not deducible from the knowledge in the data base. For example, given the personnel data base of section&#160;<a href="#%_sec_4.4.1">4.4.1</a>, the system would happily deduce all sorts of <tt>not</tt> statements, such as that Ben Bitdiddle is not a baseball fan, that it is not raining outside, and that 2 + 2 is not 4.<a name="call_footnote_Temp_688" href="#footnote_Temp_688" id="call_footnote_Temp_688"><sup><small>78</small></sup></a> In other words, the <tt>not</tt> of logic programming languages reflects the so-called <a name="index_term_5272"></a>*closed world assumption* that all relevant information has been included in the data base.<a name="call_footnote_Temp_689" href="#footnote_Temp_689" id="call_footnote_Temp_689"><sup><small>79</small></sup></a>


<a name="%_thm_4.64"></a> **Exercise 4.64.**&#160;&#160;<a name="index_term_5276"></a>Louis Reasoner mistakenly deletes the <tt>outranked-by</tt> rule (section&#160;<a href="#%_sec_4.4.1">4.4.1</a>) from the data base. When he realizes this, he quickly reinstalls it. Unfortunately, he makes a slight change in the rule, and types it in as


<tt>(rule&#160;(outranked-by&#160;?staff-person&#160;?boss)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(or&#160;(supervisor&#160;?staff-person&#160;?boss)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(and&#160;(outranked-by&#160;?middle-manager&#160;?boss)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(supervisor&#160;?staff-person&#160;?middle-manager))))<br></tt>


Just after Louis types this information into the system, DeWitt Aull comes by to find out who outranks Ben Bitdiddle. He issues the query


<tt>(outranked-by&#160;(Bitdiddle&#160;Ben)&#160;?who)<br></tt>


After answering, the system goes into an infinite loop. Explain why.


<a name="%_thm_4.65"></a> **Exercise 4.65.**&#160;&#160;<a name="index_term_5278"></a>Cy D. Fect, looking forward to the day when he will rise in the organization, gives a query to find all the wheels (using the <tt>wheel</tt> rule of section&#160;<a href="#%_sec_4.4.1">4.4.1</a>):


<tt>(wheel&#160;?who)<br></tt>


To his surprise, the system responds


<tt><i>;;;&#160;Query&#160;results:</i><br>
(wheel&#160;(Warbucks&#160;Oliver))<br>
(wheel&#160;(Bitdiddle&#160;Ben))<br>
(wheel&#160;(Warbucks&#160;Oliver))<br>
(wheel&#160;(Warbucks&#160;Oliver))<br>
(wheel&#160;(Warbucks&#160;Oliver))<br></tt>


Why is Oliver Warbucks listed four times?


<a name="%_thm_4.66"></a> **Exercise 4.66.**&#160;&#160;<a name="index_term_5280"></a>Ben has been generalizing the query system to provide statistics about the company. For example, to find the total salaries of all the computer programmers one will be able to say


<tt>(sum&#160;?amount<br>
&#160;&#160;&#160;&#160;&#160;(and&#160;(job&#160;?x&#160;(computer&#160;programmer))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(salary&#160;?x&#160;?amount)))<br></tt>


In general, Ben's new system allows expressions of the form


<tt>(accumulation-function&#160;&lt;*variable*&gt;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&lt;*query&#160;pattern*&gt;)<br></tt>


where <tt>accumulation-function</tt> can be things like <tt>sum</tt>, <tt>average</tt>, or <tt>maximum</tt>. Ben reasons that it should be a cinch to implement this. He will simply feed the query pattern to <tt>qeval</tt>. This will produce a stream of frames. He will then pass this stream through a mapping function that extracts the value of the designated variable from each frame in the stream and feed the resulting stream of values to the accumulation function. Just as Ben completes the implementation and is about to try it out, Cy walks by, still puzzling over the <tt>wheel</tt> query result in exercise&#160;<a href="#%_thm_4.65">4.65</a>. When Cy shows Ben the system's response, Ben groans, ``Oh, no, my simple accumulation scheme won't work!"


What has Ben just realized? Outline a method he can use to salvage the situation.


<a name="%_thm_4.67"></a> **Exercise 4.67.**&#160;&#160;<a name="index_term_5282"></a><a name="index_term_5284"></a>Devise a way to install a loop detector in the query system so as to avoid the kinds of simple loops illustrated in the text and in exercise&#160;<a href="#%_thm_4.64">4.64</a>. The general idea is that the system should maintain some sort of history of its current chain of deductions and should not begin processing a query that it is already working on. Describe what kind of information (patterns and frames) is included in this history, and how the check should be made. (After you study the details of the query-system implementation in section&#160;<a href="#%_sec_4.4.4">4.4.4</a>, you may want to modify the system to include your loop detector.)


<a name="%_thm_4.68"></a> **Exercise 4.68.**&#160;&#160;<a name="index_term_5286"></a>Define rules to implement the <tt>reverse</tt> operation of exercise&#160;<a href="chapter_2_section_2.html#exercise_2_18">2.18</a>, which returns a list containing the same elements as a given list in reverse order. (Hint: Use <tt>append-to-form</tt>.) Can your rules answer both <tt>(reverse (1 2 3) ?x)</tt> and <tt>(reverse ?x (1 2 3))</tt> ?


<a name="%_thm_4.69"></a> **Exercise 4.69.**&#160;&#160;Beginning with the data base and the rules you formulated in exercise&#160;<a href="#%_thm_4.63">4.63</a>, devise a rule for adding "greats" to a grandson relationship. This should enable the system to deduce that Irad is the great-grandson of Adam, or that Jabal and Jubal are the great-great-great-great-great-grandsons of Adam. (Hint: Represent the fact about Irad, for example, as <tt>((great grandson) Adam Irad)</tt>. Write rules that determine if a list ends in the word <tt>grandson</tt>. Use this to express a rule that allows one to derive the relationship <tt>((great . ?rel) ?x ?y)</tt>, where <tt>?rel</tt> is a list ending in <tt>grandson</tt>.) Check your rules on queries such as <tt>((great grandson) ?g ?ggs)</tt> and <tt>(?relationship Adam Irad)</tt>.


<a name="%_sec_4.4.4"></a>

<h3><a href="sicp_part_04.html#%_toc_%_sec_4.4.4">4.4.4&#160;&#160;Implementing the Query System</a></h3>

Section&#160;<a href="#%_sec_4.4.2">4.4.2</a> described how the query system works. Now we fill in the details by presenting a complete implementation of the system.


<a name="%_sec_4.4.4.1"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_4.4.4.1">4.4.4.1&#160;&#160;The Driver Loop and Instantiation</a></h4>

<a name="index_term_5288"></a><a name="index_term_5290"></a>The driver loop for the query system repeatedly reads input expressions. If the expression is a rule or assertion to be added to the data base, then the information is added. Otherwise the expression is assumed to be a query. The driver passes this query to the evaluator <tt>qeval</tt> together with an initial frame stream consisting of a single empty frame. The result of the evaluation is a stream of frames generated by satisfying the query with variable values found in the data base. These frames are used to form a new stream consisting of copies of the original query in which the variables are instantiated with values supplied by the stream of frames, and this final stream is printed at the terminal:


<tt><a name="index_term_5292"></a>(define&#160;input-prompt&#160;&quot;;;;&#160;Query&#160;input:&quot;)<br>
(define&#160;output-prompt&#160;&quot;;;;&#160;Query&#160;results:&quot;)<br>
<a name="index_term_5294"></a>(define&#160;(query-driver-loop)<br>
&#160;&#160;(prompt-for-input&#160;input-prompt)<br>
&#160;&#160;(let&#160;((q&#160;(query-syntax-process&#160;(read))))<br>
&#160;&#160;&#160;&#160;(cond&#160;((assertion-to-be-added?&#160;q)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(add-rule-or-assertion!&#160;(add-assertion-body&#160;q))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(newline)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(display&#160;&quot;Assertion&#160;added&#160;to&#160;data&#160;base.&quot;)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(query-driver-loop))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(newline)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(display&#160;output-prompt)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(display-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-map<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda&#160;(frame)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(instantiate&#160;q<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;frame<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda&#160;(v&#160;f)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(contract-question-mark&#160;v))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(qeval&#160;q&#160;(singleton-stream&#160;'()))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(query-driver-loop)))))<br></tt>


<a name="index_term_5296"></a>Here, as in the other evaluators in this chapter, we use an abstract syntax for the expressions of the query language. The implementation of the expression syntax, including the predicate <tt>assertion-to-be-added?</tt> and the selector <tt>add-assertion-body</tt>, is given in section&#160;<a href="#%_sec_4.4.4.7">4.4.4.7</a>. <tt>Add-rule-or-assertion!</tt> is defined in section&#160;<a href="#%_sec_4.4.4.5">4.4.4.5</a>.


Before doing any processing on an input expression, the driver loop transforms it syntactically into a form that makes the processing more efficient. This involves changing the <a name="index_term_5298"></a><a name="index_term_5300"></a>representation of pattern variables. When the query is instantiated, any variables that remain unbound are transformed back to the input representation before being printed. These transformations are performed by the two procedures <tt>query-syntax-process</tt> and <tt>contract-question-mark</tt> (section &#160;<a href="#%_sec_4.4.4.7">4.4.4.7</a>).


<a name="index_term_5302"></a>To instantiate an expression, we copy it, replacing any variables in the expression by their values in a given frame. The values are themselves instantiated, since they could contain variables (for example, if <tt>?x</tt> in <tt>exp</tt> is bound to <tt>?y</tt> as the result of unification and <tt>?y</tt> is in turn bound to&#160;5). The action to take if a variable cannot be instantiated is given by a procedural argument to <tt>instantiate</tt>.


<tt><a name="index_term_5304"></a>(define&#160;(instantiate&#160;exp&#160;frame&#160;unbound-var-handler)<br>
&#160;&#160;(define&#160;(copy&#160;exp)<br>
&#160;&#160;&#160;&#160;(cond&#160;((var?&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(let&#160;((binding&#160;(binding-in-frame&#160;exp&#160;frame)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;binding<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(copy&#160;(binding-value&#160;binding))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(unbound-var-handler&#160;exp&#160;frame))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((pair?&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons&#160;(copy&#160;(car&#160;exp))&#160;(copy&#160;(cdr&#160;exp))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;exp)))<br>
&#160;&#160;(copy&#160;exp))<br></tt>


The procedures that manipulate bindings are defined in section&#160;<a href="#%_sec_4.4.4.8">4.4.4.8</a>.


<a name="%_sec_4.4.4.2"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_4.4.4.2">4.4.4.2&#160;&#160;The Evaluator</a></h4>

<a name="index_term_5306"></a>The <tt>qeval</tt> procedure, called by the <tt>query-driver-loop</tt>, is the basic evaluator of the query system. It takes as inputs a query and a stream of frames, and it returns a stream of extended frames. It identifies special forms by a <a name="index_term_5308"></a>data-directed dispatch using <tt>get</tt> and <tt>put</tt>, just as we did in implementing generic operations in chapter&#160;2. Any query that is not identified as a special form is assumed to be a simple query, to be processed by <tt>simple-query</tt>.


<tt><a name="index_term_5310"></a>(define&#160;(qeval&#160;query&#160;frame-stream)<br>
&#160;&#160;(let&#160;((qproc&#160;(get&#160;(type&#160;query)&#160;'qeval)))<br>
&#160;&#160;&#160;&#160;(if&#160;qproc<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(qproc&#160;(contents&#160;query)&#160;frame-stream)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(simple-query&#160;query&#160;frame-stream))))<br></tt>


<tt>Type</tt> and <tt>contents</tt>, defined in section&#160;<a href="#%_sec_4.4.4.7">4.4.4.7</a>, implement the abstract syntax of the special forms.


<a name="%_sec_Temp_696"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_696">Simple queries</a></h4>

<a name="index_term_5312"></a> The <tt>simple-query</tt> procedure handles simple queries. It takes as arguments a simple query (a pattern) together with a stream of frames, and it returns the stream formed by extending each frame by all data-base matches of the query.


<tt><a name="index_term_5314"></a>(define&#160;(simple-query&#160;query-pattern&#160;frame-stream)<br>
&#160;&#160;(stream-flatmap<br>
&#160;&#160;&#160;(lambda&#160;(frame)<br>
&#160;&#160;&#160;&#160;&#160;(stream-append-delayed<br>
&#160;&#160;&#160;&#160;&#160;&#160;(find-assertions&#160;query-pattern&#160;frame)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(delay&#160;(apply-rules&#160;query-pattern&#160;frame))))<br>
&#160;&#160;&#160;frame-stream))<br></tt>


For each frame in the input stream, we use <tt>find-assertions</tt> (section&#160;<a href="#%_sec_4.4.4.3">4.4.4.3</a>) to match the pattern against all assertions in the data base, producing a stream of extended frames, and we use <tt>apply-rules</tt> (section&#160;<a href="#%_sec_4.4.4.4">4.4.4.4</a>) to apply all possible rules, producing another stream of extended frames. These two streams are combined (using <tt>stream-append-delayed</tt>, section&#160;<a href="#%_sec_4.4.4.6">4.4.4.6</a>) to make a stream of all the ways that the given pattern can be satisfied consistent with the original frame (see exercise&#160;<a href="#%_thm_4.71">4.71</a>). The streams for the individual input frames are combined using <tt>stream-flatmap</tt> (section&#160;<a href="#%_sec_4.4.4.6">4.4.4.6</a>) to form one large stream of all the ways that any of the frames in the original input stream can be extended to produce a match with the given pattern. <a name="%_sec_Temp_697"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_697">Compound queries</a></h4>

<a name="index_term_5316"></a> <a name="index_term_5318"></a><tt>And</tt> queries are handled as illustrated in figure&#160;<a href="#%_fig_4.5">4.5</a> by the <tt>conjoin</tt> procedure. <tt>Conjoin</tt> takes as inputs the conjuncts and the frame stream and returns the stream of extended frames. First, <tt>conjoin</tt> processes the stream of frames to find the stream of all possible frame extensions that satisfy the first query in the conjunction. Then, using this as the new frame stream, it recursively applies <tt>conjoin</tt> to the rest of the queries.


<tt><a name="index_term_5320"></a>(define&#160;(conjoin&#160;conjuncts&#160;frame-stream)<br>
&#160;&#160;(if&#160;(empty-conjunction?&#160;conjuncts)<br>
&#160;&#160;&#160;&#160;&#160;&#160;frame-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;(conjoin&#160;(rest-conjuncts&#160;conjuncts)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(qeval&#160;(first-conjunct&#160;conjuncts)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;frame-stream))))<br></tt>


The expression


<tt>(put&#160;'and&#160;'qeval&#160;conjoin)<br></tt>


sets up <tt>qeval</tt> to dispatch to <tt>conjoin</tt> when an <tt>and</tt> form is encountered.


<a name="index_term_5322"></a><tt>Or</tt> queries are handled similarly, as shown in figure&#160;<a href="#%_fig_4.6">4.6</a>. The output streams for the various disjuncts of the <tt>or</tt> are computed separately and merged using the <tt>interleave-delayed</tt> procedure from section&#160;<a href="#%_sec_4.4.4.6">4.4.4.6</a>. (See exercises&#160;<a href="#%_thm_4.71">4.71</a> and&#160;<a href="#%_thm_4.72">4.72</a>.)


<tt><a name="index_term_5324"></a>(define&#160;(disjoin&#160;disjuncts&#160;frame-stream)<br>
&#160;&#160;(if&#160;(empty-disjunction?&#160;disjuncts)<br>
&#160;&#160;&#160;&#160;&#160;&#160;the-empty-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;(interleave-delayed<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(qeval&#160;(first-disjunct&#160;disjuncts)&#160;frame-stream)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(delay&#160;(disjoin&#160;(rest-disjuncts&#160;disjuncts)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;frame-stream)))))<br>
(put&#160;'or&#160;'qeval&#160;disjoin)<br></tt>


The predicates and selectors for the syntax of conjuncts and disjuncts are given in section&#160;<a href="#%_sec_4.4.4.7">4.4.4.7</a>.


<a name="%_sec_Temp_698"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_698">Filters</a></h4>

<a name="index_term_5326"></a><tt>Not</tt> is handled by the method outlined in section&#160;<a href="#%_sec_4.4.2">4.4.2</a>. We attempt to extend each frame in the input stream to satisfy the query being negated, and we include a given frame in the output stream only if it cannot be extended.


<tt><a name="index_term_5328"></a>(define&#160;(negate&#160;operands&#160;frame-stream)<br>
&#160;&#160;(stream-flatmap<br>
&#160;&#160;&#160;(lambda&#160;(frame)<br>
&#160;&#160;&#160;&#160;&#160;(if&#160;(stream-null?&#160;(qeval&#160;(negated-query&#160;operands)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(singleton-stream&#160;frame)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(singleton-stream&#160;frame)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;the-empty-stream))<br>
&#160;&#160;&#160;frame-stream))<br>
(put&#160;'not&#160;'qeval&#160;negate)<br></tt>


<a name="index_term_5330"></a><tt>Lisp-value</tt> is a filter similar to <tt>not</tt>. Each frame in the stream is used to instantiate the variables in the pattern, the indicated predicate is applied, and the frames for which the predicate returns false are filtered out of the input stream. An error results if there are unbound pattern variables.


<tt><a name="index_term_5332"></a>(define&#160;(lisp-value&#160;call&#160;frame-stream)<br>
&#160;&#160;(stream-flatmap<br>
&#160;&#160;&#160;(lambda&#160;(frame)<br>
&#160;&#160;&#160;&#160;&#160;(if&#160;(execute<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(instantiate<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;call<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;frame<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(lambda&#160;(v&#160;f)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Unknown&#160;pat&#160;var&#160;--&#160;LISP-VALUE&quot;&#160;v))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(singleton-stream&#160;frame)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;the-empty-stream))<br>
&#160;&#160;&#160;frame-stream))<br>
(put&#160;'lisp-value&#160;'qeval&#160;lisp-value)<br></tt>


<tt>Execute</tt>, which applies the predicate to the arguments, must <tt>eval</tt> the predicate expression to get the procedure to apply. However, it must not evaluate the arguments, since they are already the actual arguments, not expressions whose evaluation (in Lisp) will produce the arguments. Note that <tt>execute</tt> is implemented using <a name="index_term_5334"></a><tt>eval</tt> and <tt>apply</tt> from the underlying Lisp system.


<tt>(define&#160;(execute&#160;exp)<br>
&#160;&#160;(apply&#160;(eval&#160;(predicate&#160;exp)&#160;user-initial-environment)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(args&#160;exp)))<br></tt>


The <tt>always-true</tt> special form provides for a query that is always satisfied. It ignores its contents (normally empty) and simply passes through all the frames in the input stream. <tt>Always-true</tt> is used by the <tt>rule-body</tt> selector (section&#160;<a href="#%_sec_4.4.4.7">4.4.4.7</a>) <a name="index_term_5336"></a>to provide bodies for rules that were defined without bodies (that is, rules whose conclusions are always satisfied).


<tt><a name="index_term_5338"></a>(define&#160;(always-true&#160;ignore&#160;frame-stream)&#160;frame-stream)<br>
(put&#160;'always-true&#160;'qeval&#160;always-true)<br></tt>


The selectors that define the syntax of <tt>not</tt> and <tt>lisp-value</tt> are given in section&#160;<a href="#%_sec_4.4.4.7">4.4.4.7</a>.


<a name="%_sec_4.4.4.3"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_4.4.4.3">4.4.4.3&#160;&#160;Finding Assertions by Pattern Matching</a></h4>

<a name="index_term_5340"></a><a name="index_term_5342"></a><tt>Find-assertions</tt>, called by <tt>simple-query</tt> (section&#160;<a href="#%_sec_4.4.4.2">4.4.4.2</a>), takes as input a pattern and a frame. It returns a stream of frames, each extending the given one by a data-base match of the given pattern. It uses <tt>fetch-assertions</tt> (section&#160;<a href="#%_sec_4.4.4.5">4.4.4.5</a>) to get a stream of all the assertions in the data base that should be checked for a match against the pattern and the frame. The reason for <tt>fetch-assertions</tt> here is that we can often apply simple tests that will eliminate many of the entries in the data base from the pool of candidates for a successful match. The system would still work if we eliminated <tt>fetch-assertions</tt> and simply checked a stream of all assertions in the data base, but the computation would be less efficient because we would need to make many more calls to the matcher.


<tt><a name="index_term_5344"></a>(define&#160;(find-assertions&#160;pattern&#160;frame)<br>
&#160;&#160;(stream-flatmap&#160;(lambda&#160;(datum)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(check-an-assertion&#160;datum&#160;pattern&#160;frame))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(fetch-assertions&#160;pattern&#160;frame)))<br></tt>


<tt>Check-an-assertion</tt> takes as arguments a pattern, a data object (assertion), and a frame and returns either a one-element stream containing the extended frame or <tt>the-empty-stream</tt> if the match fails.


<tt>(define&#160;(check-an-assertion&#160;assertion&#160;query-pat&#160;query-frame)<br>
&#160;&#160;(let&#160;((match-result<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(pattern-match&#160;query-pat&#160;assertion&#160;query-frame)))<br>
&#160;&#160;&#160;&#160;(if&#160;(eq?&#160;match-result&#160;'failed)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;the-empty-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(singleton-stream&#160;match-result))))<br></tt>


The basic pattern matcher returns either the symbol <tt>failed</tt> or an extension of the given frame. The basic idea of the matcher is to check the pattern against the data, element by element, accumulating bindings for the pattern variables. If the pattern and the data object are the same, the match succeeds and we return the frame of bindings accumulated so far. Otherwise, if the pattern is a variable we extend the current frame by binding the variable to the data, so long as this is consistent with the bindings already in the frame. If the pattern and the data are both pairs, we (recursively) match the <tt>car</tt> of the pattern against the <tt>car</tt> of the data to produce a frame; in this frame we then match the <tt>cdr</tt> of the pattern against the <tt>cdr</tt> of the data. If none of these cases are applicable, the match fails and we return the symbol <tt>failed</tt>.


<tt><a name="index_term_5346"></a>(define&#160;(pattern-match&#160;pat&#160;dat&#160;frame)<br>
&#160;&#160;(cond&#160;((eq?&#160;frame&#160;'failed)&#160;'failed)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((equal?&#160;pat&#160;dat)&#160;frame)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((var?&#160;pat)&#160;(extend-if-consistent&#160;pat&#160;dat&#160;frame))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((and&#160;(pair?&#160;pat)&#160;(pair?&#160;dat))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(pattern-match&#160;(cdr&#160;pat)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cdr&#160;dat)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(pattern-match&#160;(car&#160;pat)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(car&#160;dat)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;frame)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;'failed)))<br></tt>


Here is the procedure that extends a frame by adding a new binding, if this is consistent with the bindings already in the frame:


<tt><a name="index_term_5348"></a>(define&#160;(extend-if-consistent&#160;var&#160;dat&#160;frame)<br>
&#160;&#160;(let&#160;((binding&#160;(binding-in-frame&#160;var&#160;frame)))<br>
&#160;&#160;&#160;&#160;(if&#160;binding<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(pattern-match&#160;(binding-value&#160;binding)&#160;dat&#160;frame)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(extend&#160;var&#160;dat&#160;frame))))<br></tt>


If there is no binding for the variable in the frame, we simply add the binding of the variable to the data. Otherwise we match, in the frame, the data against the value of the variable in the frame. If the stored value contains only constants, as it must if it was stored during pattern matching by <tt>extend-if-consistent</tt>, then the match simply tests whether the stored and new values are the same. If so, it returns the unmodified frame; if not, it returns a failure indication. The stored value may, however, contain pattern variables if it was stored during unification (see section&#160;<a href="#%_sec_4.4.4.4">4.4.4.4</a>). The recursive match of the stored pattern against the new data will add or check bindings for the variables in this pattern. For example, suppose we have a frame in which <tt>?x</tt> is bound to <tt>(f ?y)</tt> and <tt>?y</tt> is unbound, and we wish to augment this frame by a binding of <tt>?x</tt> to <tt>(f b)</tt>. We look up <tt>?x</tt> and find that it is bound to <tt>(f ?y)</tt>. This leads us to match <tt>(f ?y)</tt> against the proposed new value <tt>(f b)</tt> in the same frame. Eventually this match extends the frame by adding a binding of <tt>?y</tt> to <tt>b</tt>. <tt>?X</tt> remains bound to <tt>(f ?y)</tt>. We never modify a stored binding and we never store more than one binding for a given variable.


The procedures used by <tt>extend-if-consistent</tt> to manipulate bindings are defined in section&#160;<a href="#%_sec_4.4.4.8">4.4.4.8</a>.


<a name="%_sec_Temp_699"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_Temp_699">Patterns with dotted tails</a></h4>

<a name="index_term_5350"></a> If a pattern contains a dot followed by a pattern variable, the pattern variable matches the rest of the data list (rather than the next element of the data list), just as one would expect with the dotted-tail notation described in exercise&#160;<a href="chapter_2_section_2.html#exercise_2_20">2.20</a>. Although the pattern matcher we have just implemented doesn't look for dots, it does behave as we want. This is because the Lisp <tt>read</tt> primitive, which is used by <tt>query-driver-loop</tt> to read the query and represent it as a list structure, treats dots in a special way.


<a name="index_term_5352"></a><a name="index_term_5354"></a>When <tt>read</tt> sees a dot, instead of making the next item be the next element of a list (the <tt>car</tt> of a <tt>cons</tt> whose <tt>cdr</tt> will be the rest of the list) it makes the next item be the <tt>cdr</tt> of the list structure. For example, the list structure produced by <tt>read</tt> for the pattern <tt>(computer ?type)</tt> could be constructed by evaluating the expression <tt>(cons 'computer (cons&#160;'?type '()))</tt>, and that for <tt>(computer . ?type)</tt> could be constructed by evaluating the expression <tt>(cons 'computer '?type)</tt>.


Thus, as <tt>pattern-match</tt> recursively compares <tt>car</tt>s and <tt>cdr</tt>s of a data list and a pattern that had a dot, it eventually matches the variable after the dot (which is a <tt>cdr</tt> of the pattern) against a sublist of the data list, binding the variable to that list. For example, matching the pattern <tt>(computer . ?type)</tt> against <tt>(computer&#160;programmer&#160;trainee)</tt> will match <tt>?type</tt> against the list <tt>(programmer trainee)</tt>. <a name="%_sec_4.4.4.4"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_4.4.4.4">4.4.4.4&#160;&#160;Rules and Unification</a></h4>

<a name="index_term_5356"></a><tt>Apply-rules</tt> is the rule analog of <tt>find-assertions</tt> (section&#160;<a href="#%_sec_4.4.4.3">4.4.4.3</a>). It takes as input a pattern and a frame, and it forms a stream of extension frames by applying rules from the data base. <tt>Stream-flatmap</tt> maps <tt>apply-a-rule</tt> down the stream of possibly applicable rules (selected by <tt>fetch-rules</tt>, section&#160;<a href="#%_sec_4.4.4.5">4.4.4.5</a>) and combines the resulting streams of frames.


<tt><a name="index_term_5358"></a>(define&#160;(apply-rules&#160;pattern&#160;frame)<br>
&#160;&#160;(stream-flatmap&#160;(lambda&#160;(rule)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(apply-a-rule&#160;rule&#160;pattern&#160;frame))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(fetch-rules&#160;pattern&#160;frame)))<br></tt>


<tt>Apply-a-rule</tt> applies rules using the method outlined in section <a href="#%_sec_4.4.2">4.4.2</a>. It first augments its argument frame by unifying the rule conclusion with the pattern in the given frame. If this succeeds, it evaluates the rule body in this new frame.


Before any of this happens, however, the program renames all the variables in the rule with unique new names. The reason for this is to prevent the variables for different rule applications from becoming confused with each other. For instance, if two rules both use a variable named <tt>?x</tt>, then each one may add a binding for <tt>?x</tt> to the frame when it is applied. These two <tt>?x</tt>'s have nothing to do with each other, and we should not be fooled into thinking that the two bindings must be consistent. Rather than rename variables, we could devise a more clever environment structure; however, the renaming approach we have chosen here is the most straightforward, even if not the most efficient. (See exercise&#160;<a href="#%_thm_4.79">4.79</a>.) Here is the <tt>apply-a-rule</tt> procedure:


<tt>(define&#160;(apply-a-rule&#160;rule&#160;query-pattern&#160;query-frame)<br>
&#160;&#160;(let&#160;((clean-rule&#160;(rename-variables-in&#160;rule)))<br>
&#160;&#160;&#160;&#160;(let&#160;((unify-result<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(unify-match&#160;query-pattern<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(conclusion&#160;clean-rule)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;query-frame)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;(eq?&#160;unify-result&#160;'failed)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;the-empty-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(qeval&#160;(rule-body&#160;clean-rule)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(singleton-stream&#160;unify-result))))))<br></tt>


The selectors <tt>rule-body</tt> and <tt>conclusion</tt> that extract parts of a rule are defined in section&#160;<a href="#%_sec_4.4.4.7">4.4.4.7</a>.


We generate unique variable names by associating a unique identifier (such as a number) with each rule application and combining this identifier with the original variable names. For example, if the rule-application identifier is 7, we might change each <tt>?x</tt> in the rule to <tt>?x-7</tt> and each <tt>?y</tt> in the rule to <tt>?y-7</tt>. (<tt>Make-new-variable</tt> and <tt>new-rule-application-id</tt> are included with the syntax procedures in section&#160;<a href="#%_sec_4.4.4.7">4.4.4.7</a>.)


<tt>(define&#160;(rename-variables-in&#160;rule)<br>
&#160;&#160;(let&#160;((rule-application-id&#160;(new-rule-application-id)))<br>
&#160;&#160;&#160;&#160;(define&#160;(tree-walk&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cond&#160;((var?&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(make-new-variable&#160;exp&#160;rule-application-id))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((pair?&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons&#160;(tree-walk&#160;(car&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(tree-walk&#160;(cdr&#160;exp))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;exp)))<br>
&#160;&#160;&#160;&#160;(tree-walk&#160;rule)))<br></tt>


<a name="index_term_5360"></a><a name="index_term_5362"></a>The unification algorithm is implemented as a procedure that takes as inputs two patterns and a frame and returns either the extended frame or the symbol <tt>failed</tt>. The unifier is like the pattern matcher except that it is symmetrical -- variables are allowed on both sides of the match. <tt>Unify-match</tt> is basically the same as <tt>pattern-match</tt>, except that there is extra code (marked "<tt>***</tt>" below) to handle the case where the object on the right side of the match is a variable.


<tt><a name="index_term_5364"></a>(define&#160;(unify-match&#160;p1&#160;p2&#160;frame)<br>
&#160;&#160;(cond&#160;((eq?&#160;frame&#160;'failed)&#160;'failed)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((equal?&#160;p1&#160;p2)&#160;frame)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((var?&#160;p1)&#160;(extend-if-possible&#160;p1&#160;p2&#160;frame))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((var?&#160;p2)&#160;(extend-if-possible&#160;p2&#160;p1&#160;frame))&#160;&#160;*;&#160;****<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((and&#160;(pair?&#160;p1)&#160;(pair?&#160;p2))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(unify-match&#160;(cdr&#160;p1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cdr&#160;p2)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(unify-match&#160;(car&#160;p1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(car&#160;p2)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;frame)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;'failed)))<br></tt>


In unification, as in one-sided pattern matching, we want to accept a proposed extension of the frame only if it is consistent with existing bindings. The procedure <tt>extend-if-possible</tt> used in unification is the same as the <tt>extend-if-consistent</tt> used in pattern matching except for two special checks, marked "<tt>***</tt>" in the program below. In the first case, if the variable we are trying to match is not bound, but the value we are trying to match it with is itself a (different) variable, it is necessary to check to see if the value is bound, and if so, to match its value. If both parties to the match are unbound, we may bind either to the other.


The second check deals with attempts to bind a variable to a pattern that includes that variable. Such a situation can occur whenever a variable is repeated in both patterns. Consider, for example, unifying the two patterns <tt>(?x ?x)</tt> and <tt>(?y &lt;*expression involving <tt>?y</tt>*&gt;)</tt> in a frame where both <tt>?x</tt> and <tt>?y</tt> are unbound. First <tt>?x</tt> is matched against <tt>?y</tt>, making a binding of <tt>?x</tt> to <tt>?y</tt>. Next, the same <tt>?x</tt> is matched against the given expression involving <tt>?y</tt>. Since <tt>?x</tt> is already bound to <tt>?y</tt>, this results in matching <tt>?y</tt> against the expression. If we think of the unifier as finding a set of values for the pattern variables that make the patterns the same, then these patterns imply instructions to find a <tt>?y</tt> such that <tt>?y</tt> is equal to the expression involving <tt>?y</tt>. There is no general method for solving such equations, so we reject such bindings; these cases are recognized by the predicate <tt>depends-on?</tt>.<a name="call_footnote_Temp_700" href="#footnote_Temp_700" id="call_footnote_Temp_700"><sup><small>80</small></sup></a> On the other hand, we do not want to reject attempts to bind a variable to itself. For example, consider unifying <tt>(?x&#160;?x)</tt> and <tt>(?y&#160;?y)</tt>. The second attempt to bind <tt>?x</tt> to <tt>?y</tt> matches <tt>?y</tt> (the stored value of <tt>?x</tt>) against <tt>?y</tt> (the new value of <tt>?x</tt>). This is taken care of by the <tt>equal?</tt> clause of <tt>unify-match</tt>.


<tt><a name="index_term_5370"></a>(define&#160;(extend-if-possible&#160;var&#160;val&#160;frame)<br>
&#160;&#160;(let&#160;((binding&#160;(binding-in-frame&#160;var&#160;frame)))<br>
&#160;&#160;&#160;&#160;(cond&#160;(binding<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(unify-match<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(binding-value&#160;binding)&#160;val&#160;frame))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((var?&#160;val)&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*;&#160;****<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(let&#160;((binding&#160;(binding-in-frame&#160;val&#160;frame)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;binding<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(unify-match<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;var&#160;(binding-value&#160;binding)&#160;frame)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(extend&#160;var&#160;val&#160;frame))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((depends-on?&#160;val&#160;var&#160;frame)&#160;&#160;&#160;&#160;&#160;*;&#160;****<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;'failed)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;(extend&#160;var&#160;val&#160;frame)))))<br></tt>


<tt>Depends-on?</tt> is a predicate that tests whether an expression proposed to be the value of a pattern variable depends on the variable. This must be done relative to the current frame because the expression may contain occurrences of a variable that already has a value that depends on our test variable. The structure of <tt>depends-on?</tt> is a simple recursive tree walk in which we substitute for the values of variables whenever necessary.


<tt>(define&#160;(depends-on?&#160;exp&#160;var&#160;frame)<br>
&#160;&#160;(define&#160;(tree-walk&#160;e)<br>
&#160;&#160;&#160;&#160;(cond&#160;((var?&#160;e)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;(equal?&#160;var&#160;e)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;true<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(let&#160;((b&#160;(binding-in-frame&#160;e&#160;frame)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(if&#160;b<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(tree-walk&#160;(binding-value&#160;b))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;false))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((pair?&#160;e)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(or&#160;(tree-walk&#160;(car&#160;e))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(tree-walk&#160;(cdr&#160;e))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;false)))<br>
&#160;&#160;(tree-walk&#160;exp))<br></tt>


<a name="%_sec_4.4.4.5"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_4.4.4.5">4.4.4.5&#160;&#160;Maintaining the Data Base</a></h4>

<a name="index_term_5372"></a> <a name="index_term_5374"></a><a name="index_term_5376"></a>One important problem in designing logic programming languages is that of arranging things so that as few irrelevant data-base entries as possible will be examined in checking a given pattern. In our system, in addition to storing all assertions in one big stream, we store all assertions whose <tt>car</tt>s are constant symbols in separate streams, in a table indexed by the symbol. To fetch an assertion that may match a pattern, we first check to see if the <tt>car</tt> of the pattern is a constant symbol. If so, we return (to be tested using the matcher) all the stored assertions that have the same <tt>car</tt>. If the pattern's <tt>car</tt> is not a constant symbol, we return all the stored assertions. Cleverer methods could also take advantage of information in the frame, or try also to optimize the case where the <tt>car</tt> of the pattern is not a constant symbol. We avoid building our criteria for indexing (using the <tt>car</tt>, handling only the case of constant symbols) into the program; instead we call on predicates and selectors that embody our criteria.


<tt>(define&#160;THE-ASSERTIONS&#160;the-empty-stream)<br>
<a name="index_term_5378"></a>(define&#160;(fetch-assertions&#160;pattern&#160;frame)<br>
&#160;&#160;(if&#160;(use-index?&#160;pattern)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(get-indexed-assertions&#160;pattern)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(get-all-assertions)))<br>
(define&#160;(get-all-assertions)&#160;THE-ASSERTIONS)<br>
(define&#160;(get-indexed-assertions&#160;pattern)<br>
&#160;&#160;(get-stream&#160;(index-key-of&#160;pattern)&#160;'assertion-stream))<br></tt>


<tt>Get-stream</tt> looks up a stream in the table and returns an empty stream if nothing is stored there.


<tt>(define&#160;(get-stream&#160;key1&#160;key2)<br>
&#160;&#160;(let&#160;((s&#160;(get&#160;key1&#160;key2)))<br>
&#160;&#160;&#160;&#160;(if&#160;s&#160;s&#160;the-empty-stream)))<br></tt>


Rules are stored similarly, using the <tt>car</tt> of the rule conclusion. Rule conclusions are arbitrary patterns, however, so they differ from assertions in that they can contain variables. A pattern whose <tt>car</tt> is a constant symbol can match rules whose conclusions start with a variable as well as rules whose conclusions have the same <tt>car</tt>. Thus, when fetching rules that might match a pattern whose <tt>car</tt> is a constant symbol we fetch all rules whose conclusions start with a variable as well as those whose conclusions have the same <tt>car</tt> as the pattern. For this purpose we store all rules whose conclusions start with a variable in a separate stream in our table, indexed by the symbol <tt>?</tt>.


<tt>(define&#160;THE-RULES&#160;the-empty-stream)<br>
<a name="index_term_5380"></a>(define&#160;(fetch-rules&#160;pattern&#160;frame)<br>
&#160;&#160;(if&#160;(use-index?&#160;pattern)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(get-indexed-rules&#160;pattern)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(get-all-rules)))<br>
(define&#160;(get-all-rules)&#160;THE-RULES)<br>
(define&#160;(get-indexed-rules&#160;pattern)<br>
&#160;&#160;(stream-append<br>
&#160;&#160;&#160;(get-stream&#160;(index-key-of&#160;pattern)&#160;'rule-stream)<br>
&#160;&#160;&#160;(get-stream&#160;'?&#160;'rule-stream)))<br></tt>


<tt>Add-rule-or-assertion!</tt> is used by <tt>query-driver-loop</tt> to add assertions and rules to the data base. Each item is stored in the index, if appropriate, and in a stream of all assertions or rules in the data base.


<tt><a name="index_term_5382"></a>(define&#160;(add-rule-or-assertion!&#160;assertion)<br>
&#160;&#160;(if&#160;(rule?&#160;assertion)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(add-rule!&#160;assertion)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(add-assertion!&#160;assertion)))<br>
(define&#160;(add-assertion!&#160;assertion)<br>
&#160;&#160;(store-assertion-in-index&#160;assertion)<br>
&#160;&#160;(let&#160;((old-assertions&#160;THE-ASSERTIONS))<br>
&#160;&#160;&#160;&#160;(set!&#160;THE-ASSERTIONS<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream&#160;assertion&#160;old-assertions))<br>
&#160;&#160;&#160;&#160;'ok))<br>
(define&#160;(add-rule!&#160;rule)<br>
&#160;&#160;(store-rule-in-index&#160;rule)<br>
&#160;&#160;(let&#160;((old-rules&#160;THE-RULES))<br>
&#160;&#160;&#160;&#160;(set!&#160;THE-RULES&#160;(cons-stream&#160;rule&#160;old-rules))<br>
&#160;&#160;&#160;&#160;'ok))<br></tt>


To actually store an assertion or a rule, we check to see if it can be indexed. If so, we store it in the appropriate stream.


<tt>(define&#160;(store-assertion-in-index&#160;assertion)<br>
&#160;&#160;(if&#160;(indexable?&#160;assertion)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(let&#160;((key&#160;(index-key-of&#160;assertion)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(let&#160;((current-assertion-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(get-stream&#160;key&#160;'assertion-stream)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(put&#160;key<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;'assertion-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream&#160;assertion<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;current-assertion-stream))))))<br>
(define&#160;(store-rule-in-index&#160;rule)<br>
&#160;&#160;(let&#160;((pattern&#160;(conclusion&#160;rule)))<br>
&#160;&#160;&#160;&#160;(if&#160;(indexable?&#160;pattern)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(let&#160;((key&#160;(index-key-of&#160;pattern)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(let&#160;((current-rule-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(get-stream&#160;key&#160;'rule-stream)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(put&#160;key<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;'rule-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream&#160;rule<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;current-rule-stream)))))))<br></tt>


The following procedures define how the data-base index is used. A pattern (an assertion or a rule conclusion) will be stored in the table if it starts with a variable or a constant symbol.


<tt>(define&#160;(indexable?&#160;pat)<br>
&#160;&#160;(or&#160;(constant-symbol?&#160;(car&#160;pat))<br>
&#160;&#160;&#160;&#160;&#160;&#160;(var?&#160;(car&#160;pat))))<br></tt>


The key under which a pattern is stored in the table is either <tt>?</tt> (if it starts with a variable) or the constant symbol with which it starts.


<tt>(define&#160;(index-key-of&#160;pat)<br>
&#160;&#160;(let&#160;((key&#160;(car&#160;pat)))<br>
&#160;&#160;&#160;&#160;(if&#160;(var?&#160;key)&#160;'?&#160;key)))<br></tt>


The index will be used to retrieve items that might match a pattern if the pattern starts with a constant symbol.


<tt>(define&#160;(use-index?&#160;pat)<br>
&#160;&#160;(constant-symbol?&#160;(car&#160;pat)))<br></tt>


<a name="%_thm_4.70"></a> **Exercise 4.70.**&#160;&#160;What is the purpose of the <tt>let</tt> bindings in the procedures <tt>add-assertion!</tt> and <tt>add-rule!</tt> ? What would be wrong with the following implementation of <tt>add-assertion!</tt> ? Hint: Recall the definition of the infinite stream of ones in section&#160;<a href="chapter_3_section_5.html#%_sec_3.5.2">3.5.2</a>: <tt>(define ones (cons-stream 1 ones))</tt>.


<tt>(define&#160;(add-assertion!&#160;assertion)<br>
&#160;&#160;(store-assertion-in-index&#160;assertion)<br>
&#160;&#160;(set!&#160;THE-ASSERTIONS<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream&#160;assertion&#160;THE-ASSERTIONS))<br>
&#160;&#160;'ok)<br></tt>


<a name="%_sec_4.4.4.6"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_4.4.4.6">4.4.4.6&#160;&#160;Stream Operations</a></h4>

<a name="index_term_5384"></a> The query system uses a few stream operations that were not presented in chapter&#160;3.


<tt>Stream-append-delayed</tt> and <tt>interleave-delayed</tt> are just like <tt>stream-append</tt> and <tt>interleave</tt> (section&#160;<a href="chapter_3_section_5.html#%_sec_3.5.3">3.5.3</a>), except that they take a delayed argument (like the <tt>integral</tt> procedure in section&#160;<a href="chapter_3_section_5.html#%_sec_3.5.4">3.5.4</a>). This postpones looping in some cases (see exercise&#160;<a href="#%_thm_4.71">4.71</a>).


<tt><a name="index_term_5386"></a>(define&#160;(stream-append-delayed&#160;s1&#160;delayed-s2)<br>
&#160;&#160;(if&#160;(stream-null?&#160;s1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(force&#160;delayed-s2)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-car&#160;s1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-append-delayed&#160;(stream-cdr&#160;s1)&#160;delayed-s2))))<br>
<a name="index_term_5388"></a>(define&#160;(interleave-delayed&#160;s1&#160;delayed-s2)<br>
&#160;&#160;(if&#160;(stream-null?&#160;s1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(force&#160;delayed-s2)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cons-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-car&#160;s1)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(interleave-delayed&#160;(force&#160;delayed-s2)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(delay&#160;(stream-cdr&#160;s1))))))<br></tt>


<tt>Stream-flatmap</tt>, which is used throughout the query evaluator to map a procedure over a stream of frames and combine the resulting streams of frames, is the stream analog of the <tt>flatmap</tt> procedure introduced for ordinary lists in section&#160;<a href="chapter_2_section_2.html#%_sec_2.2.3">2.2.3</a>. Unlike ordinary <tt>flatmap</tt>, however, we accumulate the streams with an interleaving process, rather than simply appending them (see exercises&#160;<a href="#%_thm_4.72">4.72</a> and &#160;<a href="#%_thm_4.73">4.73</a>).


<tt><a name="index_term_5390"></a>(define&#160;(stream-flatmap&#160;proc&#160;s)<br>
&#160;&#160;(flatten-stream&#160;(stream-map&#160;proc&#160;s)))<br>
<a name="index_term_5392"></a>(define&#160;(flatten-stream&#160;stream)<br>
&#160;&#160;(if&#160;(stream-null?&#160;stream)<br>
&#160;&#160;&#160;&#160;&#160;&#160;the-empty-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;(interleave-delayed<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-car&#160;stream)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(delay&#160;(flatten-stream&#160;(stream-cdr&#160;stream))))))<br></tt>


The evaluator also uses the following simple procedure to generate a stream consisting of a single element:


<tt><a name="index_term_5394"></a>(define&#160;(singleton-stream&#160;x)<br>
&#160;&#160;(cons-stream&#160;x&#160;the-empty-stream))<br></tt>


<a name="%_sec_4.4.4.7"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_4.4.4.7">4.4.4.7&#160;&#160;Query Syntax Procedures</a></h4>

<a name="index_term_5396"></a> <tt>Type</tt> and <tt>contents</tt>, used by <tt>qeval</tt> (section&#160;<a href="#%_sec_4.4.4.2">4.4.4.2</a>), specify that a special form is identified by the symbol in its <tt>car</tt>. They are the same as the <tt>type-tag</tt> and <tt>contents</tt> procedures in section&#160;<a href="chapter_2_section_4.html#%_sec_2.4.2">2.4.2</a>, except for the error message.


<tt>(define&#160;(type&#160;exp)<br>
&#160;&#160;(if&#160;(pair?&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(car&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Unknown&#160;expression&#160;TYPE&quot;&#160;exp)))<br>
(define&#160;(contents&#160;exp)<br>
&#160;&#160;(if&#160;(pair?&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(cdr&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(error&#160;&quot;Unknown&#160;expression&#160;CONTENTS&quot;&#160;exp)))<br></tt>


The following procedures, used by <tt>query-driver-loop</tt> (in section <a href="#%_sec_4.4.4.1">4.4.4.1</a>), specify that rules and assertions are added to the data base by expressions of the form <tt>(assert! &lt;*rule-or-assertion*&gt;):</tt>


<tt>(define&#160;(assertion-to-be-added?&#160;exp)<br>
&#160;&#160;(eq?&#160;(type&#160;exp)&#160;'assert!))<br>
(define&#160;(add-assertion-body&#160;exp)<br>
&#160;&#160;(car&#160;(contents&#160;exp)))<br></tt>


Here are the syntax definitions for the <tt>and</tt>, <tt>or</tt>, <tt>not</tt>, and <tt>lisp-value</tt> special forms (section&#160;<a href="#%_sec_4.4.4.2">4.4.4.2</a>):


<tt>(define&#160;(empty-conjunction?&#160;exps)&#160;(null?&#160;exps))<br>
(define&#160;(first-conjunct&#160;exps)&#160;(car&#160;exps))<br>
(define&#160;(rest-conjuncts&#160;exps)&#160;(cdr&#160;exps))<br>
(define&#160;(empty-disjunction?&#160;exps)&#160;(null?&#160;exps))<br>
(define&#160;(first-disjunct&#160;exps)&#160;(car&#160;exps))<br>
(define&#160;(rest-disjuncts&#160;exps)&#160;(cdr&#160;exps))<br>
(define&#160;(negated-query&#160;exps)&#160;(car&#160;exps))<br>
(define&#160;(predicate&#160;exps)&#160;(car&#160;exps))<br>
(define&#160;(args&#160;exps)&#160;(cdr&#160;exps))<br></tt>


The following three procedures define the syntax of rules:


<tt>(define&#160;(rule?&#160;statement)<br>
&#160;&#160;(tagged-list?&#160;statement&#160;'rule))<br>
(define&#160;(conclusion&#160;rule)&#160;(cadr&#160;rule))<br>
(define&#160;(rule-body&#160;rule)<br>
&#160;&#160;(if&#160;(null?&#160;(cddr&#160;rule))<br>
&#160;&#160;&#160;&#160;&#160;&#160;'(always-true)<br>
&#160;&#160;&#160;&#160;&#160;&#160;(caddr&#160;rule)))<br></tt>


<a name="index_term_5398"></a><a name="index_term_5400"></a><tt>Query-driver-loop</tt> (section&#160;<a href="#%_sec_4.4.4.1">4.4.4.1</a>) calls <tt>query-syntax-process</tt> to transform pattern variables in the expression, which have the form <tt>?symbol</tt>, into the internal format <tt>(? symbol)</tt>. That is to say, a pattern such as <tt>(job ?x ?y)</tt> is actually represented internally by the system as <tt>(job (? x) (? y))</tt>. This increases the efficiency of query processing, since it means that the system can check to see if an expression is a pattern variable by checking whether the <tt>car</tt> of the expression is the symbol <tt>?</tt>, rather than having to extract characters from the symbol. The syntax transformation is accomplished by the following procedure:<a name="call_footnote_Temp_702" href="#footnote_Temp_702" id="call_footnote_Temp_702"><sup><small>81</small></sup></a>


<tt>(define&#160;(query-syntax-process&#160;exp)<br>
&#160;&#160;(map-over-symbols&#160;expand-question-mark&#160;exp))<br>
<a name="index_term_5412"></a>(define&#160;(map-over-symbols&#160;proc&#160;exp)<br>
&#160;&#160;(cond&#160;((pair?&#160;exp)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(cons&#160;(map-over-symbols&#160;proc&#160;(car&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(map-over-symbols&#160;proc&#160;(cdr&#160;exp))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;((symbol?&#160;exp)&#160;(proc&#160;exp))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(else&#160;exp)))<br>
(define&#160;(expand-question-mark&#160;symbol)<br>
&#160;&#160;(let&#160;((chars&#160;(symbol-&gt;string&#160;symbol)))<br>
&#160;&#160;&#160;&#160;(if&#160;(string=?&#160;(substring&#160;chars&#160;0&#160;1)&#160;&quot;?&quot;)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(list&#160;'?<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(string-&gt;symbol<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(substring&#160;chars&#160;1&#160;(string-length&#160;chars))))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;symbol)))<br></tt>


Once the variables are transformed in this way, the variables in a pattern are lists starting with <tt>?</tt>, and the constant symbols (which need to be recognized for data-base indexing, section&#160;<a href="#%_sec_4.4.4.5">4.4.4.5</a>) are just the symbols.


<tt>(define&#160;(var?&#160;exp)<br>
&#160;&#160;(tagged-list?&#160;exp&#160;'?))<br>
(define&#160;(constant-symbol?&#160;exp)&#160;(symbol?&#160;exp))<br></tt>


Unique variables are constructed during rule application (in section <a href="#%_sec_4.4.4.4">4.4.4.4</a>) by means of the following procedures. The unique identifier for a rule application is a number, which is incremented each time a rule is applied.


<tt>(define&#160;rule-counter&#160;0)<br>
(define&#160;(new-rule-application-id)<br>
&#160;&#160;(set!&#160;rule-counter&#160;(+&#160;1&#160;rule-counter))<br>
&#160;&#160;rule-counter)<br>
(define&#160;(make-new-variable&#160;var&#160;rule-application-id)<br>
&#160;&#160;(cons&#160;'?&#160;(cons&#160;rule-application-id&#160;(cdr&#160;var))))<br></tt>


When <tt>query-driver-loop</tt> instantiates the query to print the answer, it converts any unbound pattern variables back to the right form for printing, using


<tt>(define&#160;(contract-question-mark&#160;variable)<br>
&#160;&#160;(string-&gt;symbol<br>
&#160;&#160;&#160;(string-append&#160;&quot;?&quot;&#160;<br>
&#160;&#160;&#160;&#160;&#160;(if&#160;(number?&#160;(cadr&#160;variable))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(string-append&#160;(symbol-&gt;string&#160;(caddr&#160;variable))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&quot;-&quot;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(number-&gt;string&#160;(cadr&#160;variable)))<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(symbol-&gt;string&#160;(cadr&#160;variable))))))<br></tt>


<a name="%_sec_4.4.4.8"></a>

<h4><a href="sicp_part_04.html#%_toc_%_sec_4.4.4.8">4.4.4.8&#160;&#160;Frames and Bindings</a></h4>

<a name="index_term_5414"></a><a name="index_term_5416"></a>Frames are represented as lists of bindings, which are variable-value pairs:


<tt>(define&#160;(make-binding&#160;variable&#160;value)<br>
&#160;&#160;(cons&#160;variable&#160;value))<br>
(define&#160;(binding-variable&#160;binding)<br>
&#160;&#160;(car&#160;binding))<br>
(define&#160;(binding-value&#160;binding)<br>
&#160;&#160;(cdr&#160;binding))<br>
(define&#160;(binding-in-frame&#160;variable&#160;frame)<br>
&#160;&#160;(assoc&#160;variable&#160;frame))<br>
(define&#160;(extend&#160;variable&#160;value&#160;frame)<br>
&#160;&#160;(cons&#160;(make-binding&#160;variable&#160;value)&#160;frame))<br></tt>


<a name="%_thm_4.71"></a> **Exercise 4.71.**&#160;&#160;Louis Reasoner wonders why the <tt>simple-query</tt> and <tt>disjoin</tt> procedures (section&#160;<a href="#%_sec_4.4.4.2">4.4.4.2</a>) are implemented using explicit <tt>delay</tt> operations, rather than being defined as follows:


<tt>(define&#160;(simple-query&#160;query-pattern&#160;frame-stream)<br>
&#160;&#160;(stream-flatmap<br>
&#160;&#160;&#160;(lambda&#160;(frame)<br>
&#160;&#160;&#160;&#160;&#160;(stream-append&#160;(find-assertions&#160;query-pattern&#160;frame)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(apply-rules&#160;query-pattern&#160;frame)))<br>
&#160;&#160;&#160;frame-stream))<br>
(define&#160;(disjoin&#160;disjuncts&#160;frame-stream)<br>
&#160;&#160;(if&#160;(empty-disjunction?&#160;disjuncts)<br>
&#160;&#160;&#160;&#160;&#160;&#160;the-empty-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;(interleave<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(qeval&#160;(first-disjunct&#160;disjuncts)&#160;frame-stream)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(disjoin&#160;(rest-disjuncts&#160;disjuncts)&#160;frame-stream))))<br></tt>


Can you give examples of queries where these simpler definitions would lead to undesirable behavior?


<a name="%_thm_4.72"></a> **Exercise 4.72.**&#160;&#160;Why do <tt>disjoin</tt> and <tt>stream-flatmap</tt> interleave the streams rather than simply append them? Give examples that illustrate why interleaving works better. (Hint: Why did we use <tt>interleave</tt> in section&#160;<a href="chapter_3_section_5.html#%_sec_3.5.3">3.5.3</a>?)


<a name="%_thm_4.73"></a> **Exercise 4.73.**&#160;&#160;Why does <tt>flatten-stream</tt> use <tt>delay</tt> explicitly? What would be wrong with defining it as follows:


<tt>(define&#160;(flatten-stream&#160;stream)<br>
&#160;&#160;(if&#160;(stream-null?&#160;stream)<br>
&#160;&#160;&#160;&#160;&#160;&#160;the-empty-stream<br>
&#160;&#160;&#160;&#160;&#160;&#160;(interleave<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-car&#160;stream)<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;(flatten-stream&#160;(stream-cdr&#160;stream)))))<br></tt>


<a name="%_thm_4.74"></a> **Exercise 4.74.**&#160;&#160;<a name="index_term_5418"></a>Alyssa P. Hacker proposes to use a simpler version of <tt>stream-flatmap</tt> in <tt>negate</tt>, <tt>lisp-value</tt>, and <tt>find-assertions</tt>. She observes that the procedure that is mapped over the frame stream in these cases always produces either the empty stream or a singleton stream, so no interleaving is needed when combining these streams.


a. Fill in the missing expressions in Alyssa's program.


<tt>(define&#160;(simple-stream-flatmap&#160;proc&#160;s)<br>
&#160;&#160;(simple-flatten&#160;(stream-map&#160;proc&#160;s)))<br>
<br>
(define&#160;(simple-flatten&#160;stream)<br>
&#160;&#160;(stream-map&#160;&lt;*??*&gt;<br>
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;(stream-filter&#160;&lt;*??*&gt;&#160;stream)))<br></tt>


b. Does the query system's behavior change if we change it in this way?


<a name="%_thm_4.75"></a> **Exercise 4.75.**&#160;&#160;<a name="index_term_5420"></a><a name="index_term_5422"></a><a name="index_term_5424"></a>Implement for the query language a new special form called <tt>unique</tt>. <tt>Unique</tt> should succeed if there is precisely one item in the data base satisfying a specified query. For example,


<tt>(unique&#160;(job&#160;?x&#160;(computer&#160;wizard)))<br></tt>


should print the one-item stream


<tt>(unique&#160;(job&#160;(Bitdiddle&#160;Ben)&#160;(computer&#160;wizard)))<br></tt>


since Ben is the only computer wizard, and


<tt>(unique&#160;(job&#160;?x&#160;(computer&#160;programmer)))<br></tt>


should print the empty stream, since there is more than one computer programmer. Moreover,


<tt>(and&#160;(job&#160;?x&#160;?j)&#160;(unique&#160;(job&#160;?anyone&#160;?j)))<br></tt>


should list all the jobs that are filled by only one person, and the people who fill them.


There are two parts to implementing <tt>unique</tt>. The first is to write a procedure that handles this special form, and the second is to make <tt>qeval</tt> dispatch to that procedure. The second part is trivial, since <tt>qeval</tt> does its dispatching in a data-directed way. If your procedure is called <tt>uniquely-asserted</tt>, all you need to do is


<tt>(put&#160;'unique&#160;'qeval&#160;uniquely-asserted)<br></tt>


and <tt>qeval</tt> will dispatch to this procedure for every query whose <tt>type</tt> (<tt>car</tt>) is the symbol <tt>unique</tt>.


The real problem is to write the procedure <tt>uniquely-asserted</tt>. This should take as input the <tt>contents</tt> (<tt>cdr</tt>) of the <tt>unique</tt> query, together with a stream of frames. For each frame in the stream, it should use <tt>qeval</tt> to find the stream of all extensions to the frame that satisfy the given query. Any stream that does not have exactly one item in it should be eliminated. The remaining streams should be passed back to be accumulated into one big stream that is the result of the <tt>unique</tt> query. This is similar to the implementation of the <tt>not</tt> special form.


Test your implementation by forming a query that lists all people who supervise precisely one person.


<a name="%_thm_4.76"></a> **Exercise 4.76.**&#160;&#160;<a name="index_term_5426"></a><a name="index_term_5428"></a><a name="index_term_5430"></a>Our implementation of <tt>and</tt> as a series combination of queries (figure&#160;<a href="#%_fig_4.5">4.5</a>) is elegant, but it is inefficient because in processing the second query of the <tt>and</tt> we must scan the data base for each frame produced by the first query. If the data base has *N* elements, and a typical query produces a number of output frames proportional to *N* (say *N*/*k*), then scanning the data base for each frame produced by the first query will require *N*<sup>2</sup>/*k* calls to the pattern matcher. Another approach would be to process the two clauses of the <tt>and</tt> separately, then look for all pairs of output frames that are compatible. If each query produces *N*/*k* output frames, then this means that we must perform *N*<sup>2</sup>/*k*<sup>2</sup> compatibility checks -- a factor of *k* fewer than the number of matches required in our current method.


Devise an implementation of <tt>and</tt> that uses this strategy. You must implement a procedure that takes two frames as inputs, checks whether the bindings in the frames are compatible, and, if so, produces a frame that merges the two sets of bindings. This operation is similar to unification.


<a name="%_thm_4.77"></a> **Exercise 4.77.**&#160;&#160;<a name="index_term_5432"></a><a name="index_term_5434"></a><a name="index_term_5436"></a><a name="index_term_5438"></a><a name="index_term_5440"></a>In section&#160;<a href="#%_sec_4.4.3">4.4.3</a> we saw that <tt>not</tt> and <tt>lisp-value</tt> can cause the query language to give "wrong" answers if these filtering operations are applied to frames in which variables are unbound. Devise a way to fix this shortcoming. One idea is to perform the filtering in a "delayed" manner by appending to the frame a "promise" to filter that is fulfilled only when enough variables have been bound to make the operation possible. We could wait to perform filtering until all other operations have been performed. However, for efficiency's sake, we would like to perform filtering as soon as possible so as to cut down on the number of intermediate frames generated.


<a name="%_thm_4.78"></a> **Exercise 4.78.**&#160;&#160;<a name="index_term_5442"></a>Redesign the query language as a nondeterministic program to be implemented using the evaluator of section&#160;<a href="chapter_4_section_3.html#%_sec_4.3">4.3</a>, rather than as a stream process. In this approach, each query will produce a single answer (rather than the stream of all answers) and the user can type <tt>try-again</tt> to see more answers. You should find that much of the mechanism we built in this section is subsumed by nondeterministic search and backtracking. You will probably also find, however, that your new query language has subtle differences in behavior from the one implemented here. Can you find examples that illustrate this difference?


<a name="%_thm_4.79"></a> **Exercise 4.79.**&#160;&#160;<a name="index_term_5444"></a><a name="index_term_5446"></a>When we implemented the Lisp evaluator in section&#160;<a href="chapter_4_section_1.html#%_sec_4.1">4.1</a>, we saw how to use local environments to avoid name conflicts between the parameters of procedures. For example, in evaluating


<tt>(define&#160;(square&#160;x)<br>
&#160;&#160;(*&#160;x&#160;x))<br>
(define&#160;(sum-of-squares&#160;x&#160;y)<br>
&#160;&#160;(+&#160;(square&#160;x)&#160;(square&#160;y)))<br>
(sum-of-squares&#160;3&#160;4)<br></tt>


there is no confusion between the <tt>x</tt> in <tt>square</tt> and the <tt>x</tt> in <tt>sum-of-squares</tt>, because we evaluate the body of each procedure in an environment that is specially constructed to contain bindings for the local variables. In the query system, we used a different strategy to avoid name conflicts in applying rules. Each time we apply a rule we rename the variables with new names that are guaranteed to be unique. The analogous strategy for the Lisp evaluator would be to do away with local environments and simply rename the variables in the body of a procedure each time we apply the procedure.


<a name="index_term_5448"></a><a name="index_term_5450"></a><a name="index_term_5452"></a><a name="index_term_5454"></a>Implement for the query language a rule-application method that uses environments rather than renaming. See if you can build on your environment structure to create constructs in the query language for dealing with large systems, such as the rule analog of block-structured procedures. Can you relate any of this to the problem of making deductions in a context (e.g., "If I supposed that *P* were true, then I would be able to deduce *A* and *B*.") as a method of problem solving? (This problem is open-ended. A good answer is probably worth a Ph.D.)

<div class="smallprint">
  <hr>
</div>

<div class="footnote">
  
<a name="footnote_Temp_645" href="#call_footnote_Temp_645" id="footnote_Temp_645"><sup><small>58</small></sup></a> Logic programming has grown out of a long <a name="index_term_5040"></a><a name="index_term_5042"></a>history of research in automatic theorem proving. Early theorem-proving programs could accomplish very little, because they exhaustively searched the space of possible proofs. The major breakthrough that made such a search plausible was the discovery in the early 1960s of the <a name="index_term_5044"></a>*unification algorithm* and the <a name="index_term_5046"></a>*resolution principle* (Robinson 1965). Resolution was used, for example, by <a name="index_term_5048"></a><a name="index_term_5050"></a>Green and Raphael (1968) (see also Green 1969) as the basis for a deductive question-answering system. During most of this period, researchers concentrated on algorithms that are guaranteed to find a proof if one exists. Such algorithms were difficult to control and to direct toward a proof. <a name="index_term_5052"></a>Hewitt (1969) recognized the possibility of merging the control structure of a programming language with the operations of a logic-manipulation system, leading to the work in automatic search mentioned in section&#160;<a href="chapter_4_section_3.html#%_sec_4.3.1">4.3.1</a> (footnote&#160;<a href="chapter_4_section_3.html#footnote_Temp_603">47</a>). At the same time that this was being done, <a name="index_term_5054"></a>Colmerauer, in Marseille, was developing rule-based systems for manipulating natural language (see Colmerauer et al. 1973). He invented a programming language called <a name="index_term_5056"></a>Prolog for representing those rules. <a name="index_term_5058"></a>Kowalski (1973; 1979), in Edinburgh, recognized that execution of a Prolog program could be interpreted as proving theorems (using a proof technique called linear <a name="index_term_5060"></a>Horn-clause resolution). The merging of the last two strands led to the logic-programming movement. Thus, in assigning credit for the development of logic programming, the French can point to Prolog's genesis at the <a name="index_term_5062"></a>University of Marseille, while the British can highlight the work at the <a name="index_term_5064"></a>University of Edinburgh. According to people at <a name="index_term_5066"></a>MIT, logic programming was developed by these groups in an attempt to figure out what Hewitt was talking about in his brilliant but impenetrable Ph.D. thesis. For a history of logic <a name="index_term_5068"></a>programming, see Robinson 1983.

  
<a name="footnote_Temp_646" href="#call_footnote_Temp_646" id="footnote_Temp_646"><sup><small>59</small></sup></a> To see the correspondence between the rules and the procedure, let <tt>x</tt> in the procedure (where <tt>x</tt> is nonempty) correspond to <tt>(cons u v)</tt> in the rule. Then <tt>z</tt> in the rule corresponds to the <tt>append</tt> of <tt>(cdr x)</tt> and <tt>y</tt>.

  
<a name="footnote_Temp_647" href="#call_footnote_Temp_647" id="footnote_Temp_647"><sup><small>60</small></sup></a> This certainly does not relieve the user of the entire problem of how to compute the answer. There are many different mathematically equivalent sets of rules for formulating the <tt>append</tt> relation, only some of which can be turned into effective devices for computing in any direction. In addition, sometimes "what is" information gives no clue "how to" compute an answer. For example, consider the problem of computing the *y* such that *y*<sup>2</sup> = *x*.

  
<a name="footnote_Temp_648" href="#call_footnote_Temp_648" id="footnote_Temp_648"><sup><small>61</small></sup></a> Interest in logic programming peaked <a name="index_term_5080"></a><a name="index_term_5082"></a><a name="index_term_5084"></a>during the early 80s when the Japanese government began an ambitious project aimed at building superfast computers optimized to run logic programming languages. The speed of such computers was to be measured in LIPS (Logical Inferences Per Second) rather than the usual FLOPS (FLoating-point Operations Per Second). Although the project succeeded in developing hardware and software as originally planned, the international computer industry moved in a different direction. See <a name="index_term_5086"></a><a name="index_term_5088"></a>Feigenbaum and Shrobe 1993 for an overview evaluation of the Japanese project. The logic programming community has also moved on to consider relational programming based on techniques other than simple pattern matching, such as the ability to deal with numerical constraints such as the ones illustrated in the constraint-propagation system of section&#160;<a href="chapter_3_section_3.html#%_sec_3.3.5">3.3.5</a>.

  
<a name="footnote_Temp_651" href="#call_footnote_Temp_651" id="footnote_Temp_651"><sup><small>62</small></sup></a> This uses the dotted-tail notation introduced in exercise&#160;<a href="chapter_2_section_2.html#exercise_2_20">2.20</a>.

  
<a name="footnote_Temp_654" href="#call_footnote_Temp_654" id="footnote_Temp_654"><sup><small>63</small></sup></a> Actually, this description of <tt>not</tt> is valid only for simple cases. The real behavior of <tt>not</tt> is more complex. We will examine <tt>not</tt>'s peculiarities in sections&#160;<a href="#%_sec_4.4.2">4.4.2</a> and&#160;<a href="#%_sec_4.4.3">4.4.3</a>.

  
<a name="footnote_Temp_655" href="#call_footnote_Temp_655" id="footnote_Temp_655"><sup><small>64</small></sup></a> <tt>Lisp-value</tt> should be used only to perform an operation not <a name="index_term_5134"></a>provided in the query language. In particular, it should not be used to test equality (since that is what the matching in the query language is designed to do) or inequality (since that can be done with the <tt>same</tt> rule shown below).

  
<a name="footnote_Temp_658" href="#call_footnote_Temp_658" id="footnote_Temp_658"><sup><small>65</small></sup></a> Notice that we do not need <tt>same</tt> in order to make two things be the same: We just use the same pattern variable for each -- in effect, we have one thing instead of two things in the first place. For example, see <tt>?town</tt> in the <tt>lives-near</tt> rule and <tt>?middle-manager</tt> in the <tt>wheel</tt> rule below. <tt>Same</tt> is useful when we want to force two things to be different, such as <tt>?person-1</tt> and <tt>?person-2</tt> in the <tt>lives-near</tt> rule. Although using the same pattern variable in two parts of a query forces the same value to appear in both places, using different pattern variables does not force different values to appear. (The values assigned to different pattern variables may be the same or different.)

  
<a name="footnote_Temp_659" href="#call_footnote_Temp_659" id="footnote_Temp_659"><sup><small>66</small></sup></a> We will also allow rules without bodies, as in <tt><a name="index_term_5146"></a>same</tt>, and we will interpret such a rule to mean that the rule conclusion is satisfied by any values of the variables.

  
<a name="footnote_Temp_670" href="#call_footnote_Temp_670" id="footnote_Temp_670"><sup><small>67</small></sup></a> Because matching is generally very expensive, we would <a name="index_term_5188"></a>like to avoid applying the full matcher to every element of the data base. This is usually arranged by breaking up the process into a fast, coarse match and the final match. The coarse match filters the data base to produce a small set of candidates for the final match. With care, we can arrange our data base so that some of the work of coarse matching can be done when the data base is constructed rather <a name="index_term_5190"></a><a name="index_term_5192"></a>then when we want to select the candidates. This is called *indexing* the data base. There is a vast technology built around data-base-indexing schemes. Our implementation, described in section&#160;<a href="#%_sec_4.4.4">4.4.4</a>, contains a simple-minded form of such an optimization.

  
<a name="footnote_Temp_672" href="#call_footnote_Temp_672" id="footnote_Temp_672"><sup><small>68</small></sup></a> But this kind of exponential explosion is not common in <tt>and</tt> queries because the added conditions tend to reduce rather than expand the number of frames produced.

  
<a name="footnote_Temp_673" href="#call_footnote_Temp_673" id="footnote_Temp_673"><sup><small>69</small></sup></a> There is a large literature on data-base-management systems that is concerned with how to handle complex queries efficiently.

  
<a name="footnote_Temp_674" href="#call_footnote_Temp_674" id="footnote_Temp_674"><sup><small>70</small></sup></a> There is a subtle difference between this filter implementation of <tt>not</tt> and the usual meaning of <tt>not</tt> in mathematical logic. See section&#160;<a href="#%_sec_4.4.3">4.4.3</a>.

  
<a name="footnote_Temp_676" href="#call_footnote_Temp_676" id="footnote_Temp_676"><sup><small>71</small></sup></a> In one-sided pattern matching, all the equations that contain pattern variables are explicit and already solved for the unknown (the pattern variable).

  
<a name="footnote_Temp_677" href="#call_footnote_Temp_677" id="footnote_Temp_677"><sup><small>72</small></sup></a> Another way to think of unification is that it generates the most general pattern that is a specialization of the two input patterns. That is, the unification of <tt>(?x a)</tt> and <tt>((b&#160;?y)&#160;?z)</tt> is <tt>((b ?y) a)</tt>, and the unification of <tt>(?x a ?y)</tt> and <tt>(?y ?z a)</tt>, discussed above, is <tt>(a a a)</tt>. For our implementation, it is more convenient to think of the result of unification as a frame rather than a pattern.

  
<a name="footnote_Temp_680" href="#call_footnote_Temp_680" id="footnote_Temp_680"><sup><small>73</small></sup></a> Since unification is a <a name="index_term_5222"></a><a name="index_term_5224"></a>generalization of matching, we could simplify the system by using the unifier to produce both streams. Treating the easy case with the simple matcher, however, illustrates how matching (as opposed to full-blown unification) can be useful in its own right.

  
<a name="footnote_Temp_682" href="#call_footnote_Temp_682" id="footnote_Temp_682"><sup><small>74</small></sup></a> The reason we use streams (rather than lists) of frames is that the <a name="index_term_5236"></a><a name="index_term_5238"></a>recursive application of rules can generate infinite numbers of values that satisfy a query. The delayed evaluation embodied in streams is crucial here: The system will print responses one by one as they are generated, regardless of whether there are a finite or infinite number of responses.

  
<a name="footnote_Temp_683" href="#call_footnote_Temp_683" id="footnote_Temp_683"><sup><small>75</small></sup></a> That a particular method of inference is legitimate is not a trivial assertion. One must prove that if one starts with true premises, only true conclusions can be derived. The method of inference represented by rule applications is <a name="index_term_5250"></a>*modus ponens*, the familiar method of inference that says that if *A* is true and *A implies B* is true, then we may conclude that *B* is true.

  
<a name="footnote_Temp_684" href="#call_footnote_Temp_684" id="footnote_Temp_684"><sup><small>76</small></sup></a> We must qualify this statement by agreeing that, in speaking of the "inference" accomplished by a logic program, we assume that the computation terminates. Unfortunately, even this qualified statement is false for our implementation of the query language (and also false for programs in Prolog and most other current logic programming languages) because of our use of <tt>not</tt> and <tt>lisp-value</tt>. As we will describe below, the <tt>not</tt> implemented in the query language is not always consistent with the <tt>not</tt> of mathematical logic, and <tt>lisp-value</tt> introduces additional complications. We could implement a language consistent with mathematical logic by simply removing <tt>not</tt> and <tt>lisp-value</tt> from the language and agreeing to write programs using only simple queries, <tt>and</tt>, and <tt>or</tt>. However, this would greatly restrict the expressive power of the language. One of the major concerns of research in logic programming is to find ways to achieve more consistency with mathematical logic without unduly sacrificing expressive power.

  
<a name="footnote_Temp_686" href="#call_footnote_Temp_686" id="footnote_Temp_686"><sup><small>77</small></sup></a> This is not a problem of the logic but one of the procedural interpretation of the logic provided by our interpreter. We could write an interpreter that would not fall into a loop here. For example, we could enumerate all the proofs derivable from our assertions and our rules in a breadth-first rather than a depth-first order. However, such a system makes it more difficult to take advantage of the order of deductions in our programs. One attempt to build sophisticated control into such a program is described in <a name="index_term_5264"></a>deKleer et al. 1977. Another technique, which does not lead to such serious control problems, is to put in special knowledge, such as detectors for particular kinds of loops (exercise&#160;<a href="#%_thm_4.67">4.67</a>). However, there can be no general scheme for reliably preventing a system from going down infinite paths in performing deductions. Imagine a diabolical rule of the form "To show *P*(*x*) is true, show that *P*(*f*(*x*)) is true," for some suitably chosen function *f*.

  
<a name="footnote_Temp_688" href="#call_footnote_Temp_688" id="footnote_Temp_688"><sup><small>78</small></sup></a> Consider the query <tt>(not (baseball-fan (Bitdiddle Ben)))</tt>. The system finds that <tt>(baseball-fan (Bitdiddle Ben))</tt> is not in the data base, so the empty frame does not satisfy the pattern and is not filtered out of the initial stream of frames. The result of the query is thus the empty frame, which is used to instantiate the input query to produce <tt>(not (baseball-fan (Bitdiddle Ben)))</tt>.

  
<a name="footnote_Temp_689" href="#call_footnote_Temp_689" id="footnote_Temp_689"><sup><small>79</small></sup></a> A discussion and justification of this <a name="index_term_5274"></a>treatment of <tt>not</tt> can be found in the article by Clark (1978).

  
<a name="footnote_Temp_700" href="#call_footnote_Temp_700" id="footnote_Temp_700"><sup><small>80</small></sup></a> In general, unifying <tt>?y</tt> with an expression involving <a name="index_term_5366"></a><tt>?y</tt> would require our being able to find a fixed point of the equation <tt>?y</tt> = &lt;*expression involving <tt>?y</tt>*&gt;. It is sometimes possible to syntactically form an expression that appears to be the solution. For example, <tt>?y</tt>&#160; = &#160;<tt>(f ?y)</tt> seems to have the fixed point <tt>(f (f (f</tt> ... ))), which we can produce by beginning with the expression <tt>(f ?y)</tt> and repeatedly substituting <tt>(f ?y)</tt> for <tt>?y</tt>. Unfortunately, not every such equation has a meaningful fixed point. The issues that arise here are similar to the issues of manipulating <a name="index_term_5368"></a>infinite series in mathematics. For example, we know that 2 is the solution to the equation *y* = 1 + *y*/2. Beginning with the expression 1 + *y*/2 and repeatedly substituting 1 + *y*/2 for *y* gives

  <div align="left">
    <img src="img/chapter_4_image_07.gif" border="0">
  </div>

  
which leads to

  <div align="left">
    <img src="img/chapter_4_image_08.gif" border="0">
  </div>

  
However, if we try the same manipulation beginning with the observation that - 1 is the solution to the equation *y* = 1 + 2*y*, we obtain

  <div align="left">
    <img src="img/chapter_4_image_09.gif" border="0">
  </div>

  
which leads to

  <div align="left">
    <img src="img/chapter_4_image_10.gif" border="0">
  </div>

  
Although the formal manipulations used in deriving these two equations are identical, the first result is a valid assertion about infinite series but the second is not. Similarly, for our unification results, reasoning with an arbitrary syntactically constructed expression may lead to errors.

  
<a name="footnote_Temp_702" href="#call_footnote_Temp_702" id="footnote_Temp_702"><sup><small>81</small></sup></a> Most Lisp systems give the user the ability to modify the ordinary <tt>read</tt> procedure to perform such transformations by defining <a name="index_term_5402"></a><a name="index_term_5404"></a><a name="index_term_5406"></a><a name="index_term_5408"></a>*reader macro characters*. Quoted expressions are already handled in this way: The reader automatically translates <tt>'expression</tt> into <tt>(quote expression)</tt> before the evaluator sees it. We could arrange for <tt>?expression</tt> to be transformed into <tt>(? expression)</tt> in the same way; however, for the sake of clarity we have included the transformation procedure here explicitly.

  
<a name="index_term_5410"></a><tt>Expand-question-mark</tt> and <tt>contract-question-mark</tt> use several procedures with <tt>string</tt> in their names. These are Scheme primitives.

</div>