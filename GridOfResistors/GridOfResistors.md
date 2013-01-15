<HTML>

<HEAD>
<TITLE>Homework 1: Grid of Resistors</TITLE>
<link rel="stylesheet" href="../styles/style.css"> 
</head>

<body class="class1" alink="#800080" vlink ="#800080" link="#800080"> 

<H1>Homework 1: Grid of Resistors</H1>
<p><em>Due Thursday, 3/11/2004 </em></p>

<h2>Hints</h2>

General:
<ul>
  <li>It might help to draw a good picture of the grid. Number all
  the rows and columns, mark out the two source positions, and
  identify the red and the black nodes.</li>
  <li>When you split the domain, think about which nodes you have
  to update on each of the two processes, both for the red and the
  black updates. Note that the calculation should be done exactly as
  in the serial version.</li>
</ul>

Question 2:
<ul>
  <li>It is a good idea to begin experimenting with mm-mode
  using a very simple function. Try to pass data to and from
  the function, and make sure you understand how it works.</li>
  <li>Depending on where you put your functions, you might have
  to use the <code>mmpath</code> command.</li>
  <li>You have to write (at least) two functions: One main routine,
  which you will run on the front-end, and one subroutine which you
  will run using the mm-mode.</li>
  <li>In the main routine, you allocate the "buffer"
  <code>A=zeros(2*n+1,2*p)</code>. The for-loop for the iterations
  also goes here, and you call mm-mode to update the red and the
  black nodes.</li>
  <li>The "buffer" has to be sent both to and from your subroutine,
  for example:
  <pre>
   [buffer,&lt;other outputs&gt;]=mm('subroutine',buffer,&lt;other inputs&gt;); 
  </pre>
  </li>
  <li>After updating the red or the black nodes, you have to switch
  the columns of <code>A</code> according to the description below.</li>
  <li>Inside the subroutine, you store a matrix with all the data for
  each subregion. Use <code>persistent</code> or <code>global</code>
  variables (see help text in MATLAB) to keep this data between the
  mm-calls.</li>
  <li>In the subroutine you move data between the internal matrix and
  the column of the buffer matrix.</li>
</ul>

<h2>Downloads</h2>

<pre>
<a href="sor2d.m">sor2d.m</a>
<a href="sor2d.c">sor2d.c</a>
</pre>

<h2>Description</h2>

<p> The problem is to compute the voltages and the effective
resistance of a 2<i>n</i>+1 by 2<i>n</i>+2 grid of 1 ohm resistors if
a battery is connected to the two center points. This is a discrete
version of finding the lines of force using iron filings for a magnet.
The picture below describes the two dimensional problem.</p>

<center>
<img src="battery.gif">
</center>

<p> The method of solution that we will use here is <i>successive
overrelaxation</i> (SOR) with red-black ordering. This is certainly
not the fastest way to solve the problem, but it does illustrate many
important programming ideas. </p>

<p> It is not so important that you know the details of SOR. Some of
the basic ideas may be found on pages 407-409 of Gil Strang's <a
href="http://www-math.mit.edu/%7Egs/books/itam_toc.html">Introduction
to Applied Mathematics</a>. A somewhat more in-depth discussion may
be found in any serious numerical analysis text such as Stoer and
Bulirsch's <em>Introduction to Numerical Analysis</em>. What is
important is that you see that the nodes are divided in half into red
nodes and black nodes. During the first pass, the red nodes obtain the
voltages as a weighted average of their original voltage, the input
(if any) <i>and the four surrounding black nodes</i>. During the
second pass, the black nodes obtain voltages from the four surrounding
red nodes. The process converges in the limit to the correct answer
for the finite grid.</p>

<h2>Question 1 - Introduction</h2>

   <p>The MATLAB code <code>sor2d.m</code> solves the problem in 2-D. The problem
size <i>n</i> and the number of iterations can be changed by editing the file.
Read the source code carefully and make sure you understand exactly how it
works.</p>

   <p>A more general version of the code can be found in <code>sornd.m</code>.
Here, the problem is solved for <em>arbitrary dimensions d</em>. You don't need
to understand every detail of this code (unless you are interested in some advanced
MATLAB programming).</p>

<ul>
  <li>Run <code>sornd.m</code> for <i>d</i>=2, 3, 4, and 5. Use as large <i>n</i>
  as possible, and try to guess the value of the resistance for an infinite
  grid for each dimension.</li>
  <li>Based on your experiments, make a conjecture about the value of the
  resistance as a function of <i>d</i>. <em>Optional</em>: Prove your answer by physical reasoning.</li>
</ul>

<h2>Question 2 - STAR-P</h2>

   <p>You should now try to parallelize the 2-D code <code>sor2d.m</code> with
STAR-P and two processors. With the current version of STAR-P, you will not
get good performance by simple making the arrays distributed (that is,
by multiplying by <code>*p</code>). Instead, you will
have to do the parallelization explicitly yourself.</p>

   <p>Split the domain into two regions, of size 2<i>n</i>+1 by
<i>n</i>+1 each. To be able to update the values along the boundary
between the regions, you also need one additional vector of
overlapping data. Between the iterations, this data has to be
communicated between the processors. Use the STAR-P mm-mode to perform
the updates in each region separately. To send data between the
processors, copy the data into a distributed matrix with two columns,
and switch the columns:
<pre>
  A=randn(100,2*p);   % Just an example
  A=A(:,[2,1]);       % Switch data between processors
</pre>
You will have to do this between every red or black iteration.

<ul>
  <li>Write the STAR-P version for two processors. Verify that your
  calculations are correct by comparing the results (for example the
  resistance) with the ones from the serial program.</li>
  <li>Measure the time / iteration for a series of values of <i>n</i>,
  both for the serial MATLAB code and your parallel MATLAB code. Comment on the results.</li>
</ul>

   <p>This example shows how to combine the mm-mode and the *p-mode
   for a non-embarrassingly parallel problem. The independent node
   updates are done in parallel using the mm-mode, and the communication
   is done by the high-level *p-syntax instead of explicit sends and
   receives.</p>   

<h2>Question 3 - OpenMP</h2>

The C code <code>sor2d.c</code> solves the problem in 2-D. Read it, understand it, compile it, and run it. Remember to turn optimization on (we want high performance!).

<ul>
  <li>In the loops over all nodes, the second index j is in the outer loop and the first index i is in the inner loop. Why is that? Change the code to get the indices in the opposite order (both red and black) and compare the running times of the two versions (easy but not trivial, you need to handle the special case i=2n-1). Explain the results.</li>
  <li>Use OpenMP to parallelize <code>sor2d.c</code> for a two processor machine with shared memory (note: this is relatively easy).</li>
  <li>Measure the time / iteration for a series of values of <i>n</i>,
  both for the serial C code and your parallel C code. Comment on the results. Did you get perfect speedup (twice as fast)? If not, explain why.</li>
</ul>

<h2>Question 4 - MPI</h2>

   <p>You should now parallelize the serial C code <code>sor2d.c</code> for a
distributed memory machine with two processors using MPI. Use the same
technique as for the STAR-P program, that is, split the region in two, update
the regions separately, and send the boundary data between every red or black iteration.<p>

<ul>
  <li>Implement the MPI program using <code>MPI_Send</code> and <code>MPI_Recv</code>
  (<em>blocking</em> send). Again, verify the results by comparing with the serial
  version.</li>
  <li>Modify your code to use <code>MPI_Isend</code> and <code>MPI_Irecv</code>
  (<em>non-blocking</em> send). Try to overlap the communications and the computations
  by updating the values that are independent of the communication while the send is
  in progress.</li>
  <li>Measure the time / iteration for a series of values of <i>n</i>,
  both for the serial C code and your two versions of
  parallel C code. Comment on the results.</li>
</ul>

<h2>How to submit</h2>
<UL>
  <LI>mkdir hw1; mkdir hw1/q1 hw1/q2 hw1/q3 hw1/q4
  <LI>copy question 1 files - source code, Makefile (if any), write-ups, plots 
  (if any) into hw1/q1 
  <LI>same thing for q2, q3, and q4 
  <LI>tar cf `whoami`-hw1.tar hw1/ 
  <LI>gzip `whoami`-hw1.tar 
  <LI>cp `whoami`-hw1.tar.gz ~ 
  <LI>It will be collected automatically by a cron job.</LI>
</UL>

<ADDRESS><A href="mailto:persson%20AT%20math.mit.edu">Per-Olof Persson</A></ADDRESS>
  
<!-- hhmts start --> Last modified: Tue Mar  2 12:52:37 GMT 2004 <!-- hhmts end -->
  </BODY>
</HTML>