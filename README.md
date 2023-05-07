Download Link: https://assignmentchef.com/product/solved-operatingsystems-problem-3-linux-completely-fair-scheduler
<br>



<strong>Problem 3.1: </strong><em>Linux completely fair scheduler</em>

The Completely Fair Scheduler (CFS) was introduced with the Linux kernel 2.6.23 and it was revised in kernel version 2.6.24. Read about the design of the (revised) CFS scheduler and answer the following questions:

<ol>

 <li>What does fairness mean in the context of the CFS?</li>

 <li>How does the CFS scheduler select tasks to run? What is the data structure used to maintain the necessary information and why was it chosen?</li>

 <li>Does the CFS scheduler use time-slices? Are there parameters affecting CFS time calculations?</li>

 <li>How do priorities (nice values) affect the selection of tasks?</li>

</ol>

<strong>Problem 3.2: </strong><em>student running group                                                                                                           </em>

After an intense weekend, a group of students decide to do something for their fitness and their health. They form a running group that meets regularly to go for a run together. Unfortunately, the students are very busy and hence not always on time for the running sessions. So they decide that whoever arrives first for a running session becomes the leader of the running session. The running session leader waits for a fixed time for other runners to arrive. Once the time has passed, all runners that have arrived start running. Since runners may run at different pace, not all runners complete the run together. But the runners show real team spirit and they always wait until every runner participating in the running session has arrived before they leave to study again.

In order to increase the pressure to attend running sessions, the students decide that whoever has missed five running sessions has to leave the running group. In addition, it occasionally happens that the leader of a running session has to run alone if nobody else shows up. This is, of course, a bit annoying and hence runners who did run ten times alone leave the running group voluntarily.

Write a C program to simulate the running group. Every runner is implemented as a separate thread. The time between running sessions is modeled by sleeping for a random number of microseconds. The time the session leader waits for additional runners to arrive is also randomized. Runners arriving too late simply try to make the next running session.

Your program should use pthread mutexes and condition variables and timed wait functions. Do not use any other synchronization primitives. Your program should implement the option -n to set the number of runners initially participating in the running group (default value is one runner).

While you generally have to handle all possible runtime errors, it is acceptable for this assignment to assume that calls to lock or unlock mutexes or calls to wait or signal condition variables generally succeed. (This rule hopefully keeps your code more readable and makes it easier to review your solutions.) In general, try to write clean and well organized code. Only submit your source code files, do not upload any object code files or executables or any other files that are of no value for us.

Below is the non-debugging output of a solution for this problem:

<table width="446">

 <tbody>

  <tr>

   <td width="216">./runner -n 10 2&gt;/dev/null</td>

   <td width="230"> </td>

  </tr>

  <tr>

   <td width="216">r0 stopped after 17 run(s):</td>

   <td width="230">2 run(s) missed, 10 run(s) alone</td>

  </tr>

  <tr>

   <td width="216">r1 stopped after                  8 run(s):</td>

   <td width="230">5 run(s) missed,                       0 run(s) alone</td>

  </tr>

  <tr>

   <td width="216">r2 stopped after                  4 run(s):</td>

   <td width="230">5 run(s) missed,                       1 run(s) alone</td>

  </tr>

  <tr>

   <td width="216">r3 stopped after                  8 run(s):</td>

   <td width="230">5 run(s) missed,                       0 run(s) alone</td>

  </tr>

  <tr>

   <td width="216">r4 stopped after 12 run(s):</td>

   <td width="230">5 run(s) missed,                       4 run(s) alone</td>

  </tr>

  <tr>

   <td width="216">r5 stopped after 17 run(s):</td>

   <td width="230">1 run(s) missed, 10 run(s) alone</td>

  </tr>

  <tr>

   <td width="216">r6 stopped after 21 run(s):</td>

   <td width="230">4 run(s) missed, 10 run(s) alone</td>

  </tr>

  <tr>

   <td width="216">r7 stopped after 12 run(s):</td>

   <td width="230">5 run(s) missed,                       2 run(s) alone</td>

  </tr>

  <tr>

   <td width="216">r8 stopped after                  6 run(s):</td>

   <td width="230">5 run(s) missed,                       3 run(s) alone</td>

  </tr>

  <tr>

   <td width="216">r9 stopped after 13 run(s):</td>

   <td width="230">5 run(s) missed,                       3 run(s) alone</td>

  </tr>

 </tbody>

</table>

A template of the program is provided below and on the course web page so that you can concentrate on filling in the missing parts that implement the logic of the student running group.

<ul>

 <li><em>/*</em></li>

 <li><em>* p3-runner/runner-template.c —</em></li>

 <li><em>*</em></li>

 <li><em>* Runners meeting regularly (if they manage). See Operating </em>5      <em>*              Systems ‘2018 assignment #3 for more details.</em></li>

</ul>

6                 <em>*/</em>

7

<ul>

 <li><em>#define _REENTRANT</em></li>

 <li><em>#define _DEFAULT_SOURCE</em></li>

</ul>

10

<ul>

 <li><em>#include </em><em>&lt;stdio.h&gt;</em></li>

 <li><em>#include </em><em>&lt;stdlib.h&gt;</em></li>

 <li><em>#include </em><em>&lt;string.h&gt;</em></li>

 <li><em>#include </em><em>&lt;unistd.h&gt;</em></li>

 <li><em>#include </em><em>&lt;getopt.h&gt;</em></li>

 <li><em>#include </em><em>&lt;assert.h&gt;</em></li>

 <li><em>#include </em><em>&lt;pthread.h&gt;</em></li>

 <li><em>#include </em><em>&lt;errno.h&gt;</em></li>

 <li><em>#include </em><em>&lt;sys/time.h&gt;</em></li>

</ul>

20

<ul>

 <li><em>#define LATE_THRESHOLD 5</em></li>

 <li><em>#define LONELY_THRESHOLD 10</em></li>

</ul>

23

<ul>

 <li><em>#define GROUP_SLEEPING 0x01</em></li>

 <li><em>#define GROUP_ASSEMBLING 0x02</em></li>

 <li><em>#define GROUP_RUNNING 0x03</em></li>

 <li><em>#define GROUP_FINISHING 0x04</em></li>

</ul>

28

<ul>

 <li><em>#define RUNNER_SLEEPING 0x01</em></li>

 <li><em>#define RUNNER_LEADING 0x02</em></li>

 <li><em>#define RUNNER_ASSEMBLING 0x03</em></li>

 <li><em>#define RUNNER_RUNNING 0x04</em></li>

 <li><em>#define RUNNER_WAITING 0x05</em></li>

</ul>

34

<ul>

 <li>typedef struct group {</li>

 <li>unsigned state;      <em>/* the state of the running group */</em></li>

 <li>unsigned arriving; <em>/* number of runnners arriving */</em></li>

 <li>unsigned running; <em>/* number of runnners running */</em></li>

 <li>} group_t;</li>

</ul>

40

<ul>

 <li>typedef struct runner {</li>

 <li>unsigned id;           <em>/* identity of the runner */</em></li>

 <li>pthread_t tid;          <em>/* thread identifier */</em></li>

 <li>unsigned state;      <em>/* state of the runner */</em></li>

 <li>unsigned late;        <em>/* number of runs missed (late arrival) */</em></li>

 <li>unsigned lonely;    <em>/* number of runs without any other runners */</em></li>

 <li>unsigned runs;       <em>/* number of runs completed */</em></li>

 <li>group_t *group;  <em>/* the group this runner belongs to */</em></li>

 <li>} runner_t;</li>

</ul>

50

51 static const char *progname = “runner”;

52

<ul>

 <li>static void*</li>

 <li>runners_life(void *data)</li>

 <li>{</li>

 <li>runner_t *runner = (runner_t *) data;</li>

 <li>group_t *group = runner-&gt;group;</li>

</ul>

58

59                                   assert(runner &amp;&amp; runner-&gt;group);

60

<ul>

 <li>while (runner-&gt;late &lt; LATE_THRESHOLD</li>

 <li>&amp;&amp; runner-&gt;lonely &lt; LONELY_THRESHOLD) {</li>

</ul>

63

<ul>

 <li>runner-&gt;state = RUNNER_SLEEPING;</li>

 <li>(void) fprintf(stderr, “r%d sleeping
”, runner-&gt;id);</li>

 <li><em>/* not very random but good enough here */ </em>67 usleep(172000+random()%10000);</li>

</ul>

68

69                                                            <em>/* add additional logic to model the runners here */</em>

70

<ul>

 <li><em>/* the session leader is expected to wait for some time for</em></li>

 <li><em>additional runners to arrive and join the session */</em></li>

 <li>usleep(3600+random()%100);</li>

 <li>}</li>

</ul>

75

<ul>

 <li>return NULL;</li>

 <li>}</li>

</ul>

78

<ul>

 <li>int</li>

 <li>main(int argc, char *argv[])</li>

 <li>{</li>

 <li>int c, n = 1;</li>

 <li>runner_t *runner = NULL;</li>

 <li>int rc = EXIT_SUCCESS;</li>

</ul>

85

<ul>

 <li>group_t group = {</li>

 <li>.state = GROUP_SLEEPING,</li>

 <li>.arriving = 0,</li>

 <li>.running = 0,</li>

 <li>};</li>

</ul>

91

<ul>

 <li>while ((c = getopt(argc, argv, “n:h”)) &gt;= 0) {</li>

 <li>switch (c) { 94 case ‘n’:</li>

 <li>if ((n = atoi(optarg)) &lt;= 0) {</li>

 <li>(void) fprintf(stderr, “number of runners must be &gt; 0
”); 97 exit(EXIT_FAILURE);</li>

 <li>}</li>

 <li>break;</li>

 <li>case ‘h’:</li>

 <li>(void) printf(“Usage: %s [-n runners] [-h]
”, progname); 102 exit(EXIT_SUCCESS);</li>

 <li>}</li>

 <li>}</li>

</ul>

105

<ul>

 <li>runner = calloc(n, sizeof(runner_t));</li>

 <li>if (! runner) {</li>

 <li>(void) fprintf(stderr, “%s: calloc() failed
”, progname);</li>

 <li>rc = EXIT_FAILURE;</li>

 <li>goto cleanup;</li>

 <li>}</li>

</ul>

112

<ul>

 <li>for (int i = 0; i &lt; n; i++) {</li>

 <li>runner[i].id = i;</li>

 <li>runner[i].state = RUNNER_SLEEPING;</li>

 <li>runner[i].late = 0;</li>

 <li>runner[i].runs = 0;</li>

 <li>runner[i].group = &amp;group;</li>

</ul>

119

120                                                      <em>/* XXX create thread for runner[i] XXX */</em>

121

122                        }

123

124                                           for (int i = 0; i &lt; n; i++) {

125

126                                                      <em>/* XXX join thread for runner[i] XXX */</em>

127

128                        }

129

<ul>

 <li>for (int i = 0; i &lt; n; i++) {</li>

 <li>(void) printf(“r%d stopped after %3d run(s): “</li>

 <li>“%3d run(s) missed, %3d run(s) alone
”,</li>

 <li>runner[i].id, runner[i].runs,</li>

 <li>runner[i].late, runner[i].lonely);</li>

 <li>}</li>

</ul>

136

137 cleanup:

138

139                                          if (runner) { (void) free(runner); }

140

<ul>

 <li>return rc;</li>

 <li>}</li>

</ul>