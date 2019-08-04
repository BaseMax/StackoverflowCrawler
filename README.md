# Stackoverflow Crawler

A web crawler which crawls the **Stack Overflow** website. (Questions and Answers / Q&A)

https://stackoverflow.com/

### Try API

- https://stackoverflow.com/questions/tagged/c
- https://api.stackexchange.com/2.2/questions?order=desc&sort=activity&site=stackoverflow&pagesize=100
- https://api.stackexchange.com/2.2/questions/1163244?order=desc&sort=activity&site=askubuntu
- https://api.stackexchange.com/2.2/questions/1160229/answers?order=desc&sort=activity&site=askubuntu
- https://api.stackexchange.com/2.2/questions?order=desc&sort=activity&site=askubuntu&pagesize=100
- https://api.stackexchange.com/docs/questions

### Stack Exchange API

- https://api.stackexchange.com/
- https://api.stackexchange.com/docs
- https://stackapps.com/questions/7/how-to-list-your-application-library-wrapper-script-here
- http://stackapps.com/apps/oauth/register
- https://api.stackexchange.com/docs/throttle
- https://api.stackexchange.com/docs/authentication
- https://api.stackexchange.com/docs/write

-------

It's same as a fork and depends on some other projects on the GitHub:

- https://www.crummy.com/software/BeautifulSoup/bs4/doc/
- https://pypi.org/project/bs4/
- https://pypi.org/project/beautifulsoup4/
- https://www.crummy.com/software/BeautifulSoup/bs4/doc/#get-text
- https://github.com/topics/stackoverflow-crawler
- https://github.com/rjstyles/StackOverflow-Crawler
- https://github.com/topics/stackoverfolw-website
- ...

I did not remember that at now.

---------

## Testing

```
$ python3 craw.py
crawling page 1: https://stackoverflow.com/questions/tagged/c%2b%2b?sort=votes&page=1&pageSize=15
------------------------------
Why is processing a sorted array faster than processing an unsorted array?
<div class="post-text" itemprop="text">
<p>Here is a piece of C++ code that shows some very peculiar behavior. For some strange reason, sorting the data miraculously makes the code almost six times faster:</p>
<pre class="lang-cpp prettyprint-override"><code>#include &lt;algorithm&gt;
#include &lt;ctime&gt;
#include &lt;iostream&gt;

int main()
{
	// Generate data
	const unsigned arraySize = 32768;
	int data[arraySize];

	for (unsigned c = 0; c &lt; arraySize; ++c)
		data[c] = std::rand() % 256;


	// !!! With this, the next loop runs faster.
	std::sort(data, data + arraySize);


	// Test
	clock_t start = clock();
	long long sum = 0;

	for (unsigned i = 0; i &lt; 100000; ++i)
	{
		// Primary loop
		for (unsigned c = 0; c &lt; arraySize; ++c)
		{
			if (data[c] &gt;= 128)
				sum += data[c];
		}
	}

	double elapsedTime = static_cast&lt;double&gt;(clock() - start) / CLOCKS_PER_SEC;

	std::cout &lt;&lt; elapsedTime &lt;&lt; std::endl;
	std::cout &lt;&lt; "sum = " &lt;&lt; sum &lt;&lt; std::endl;
}
</code></pre>
<ul>
<li>Without <code>std::sort(data, data + arraySize);</code>, the code runs in 11.54 seconds.</li>
<li>With the sorted data, the code runs in 1.93 seconds.</li>
</ul>
<hr/>
<p>Initially I thought this might be just a language or compiler anomaly, so I tried Java:</p>
<pre class="lang-java prettyprint-override"><code>import java.util.Arrays;
import java.util.Random;

public class Main
{
	public static void main(String[] args)
	{
		// Generate data
		int arraySize = 32768;
		int data[] = new int[arraySize];

		Random rnd = new Random(0);
		for (int c = 0; c &lt; arraySize; ++c)
			data[c] = rnd.nextInt() % 256;


		// !!! With this, the next loop runs faster
		Arrays.sort(data);


		// Test
		long start = System.nanoTime();
		long sum = 0;

		for (int i = 0; i &lt; 100000; ++i)
		{
			// Primary loop
			for (int c = 0; c &lt; arraySize; ++c)
			{
				if (data[c] &gt;= 128)
					sum += data[c];
			}
		}

		System.out.println((System.nanoTime() - start) / 1000000000.0);
		System.out.println("sum = " + sum);
	}
}
</code></pre>
<p>with a similar but less extreme result.</p>
<hr/>
<p>My first thought was that sorting brings the data into the cache, but then I thought how silly that was because the array was just generated.</p>
<ul>
<li>What is going on?</li>
<li>Why is processing a sorted array faster than processing an unsorted array? The code is summing up some independent terms, so the order should not matter.</li>
</ul>
</div>
List of answers:


===>
Why is processing a sorted array faster than processing an unsorted array?

You are a victim of branch prediction fail.

What is Branch Prediction?
Consider a railroad junction:

Image by Mecanismo, via Wikimedia Commons. Used under the CC-By-SA 3.0 license.
Now for the sake of argument, suppose this is back in the 1800s - before long distance or radio communication.
You are the operator of a junction and you hear a train coming. You have no idea which way it is supposed to go. You stop the train to ask the driver which direction they want. And then you set the switch appropriately.
Trains are heavy and have a lot of inertia. So they take forever to start up and slow down.
Is there a better way? You guess which direction the train will go!

If you guessed right, it continues on.
If you guessed wrong, the captain will stop, back up, and yell at you to flip the switch. Then it can restart down the other path.

If you guess right every time, the train will never have to stop.
If you guess wrong too often, the train will spend a lot of time stopping, backing up, and restarting.

Consider an if-statement: At the processor level, it is a branch instruction:

You are a processor and you see a branch. You have no idea which way it will go. What do you do? You halt execution and wait until the previous instructions are complete. Then you continue down the correct path.
Modern processors are complicated and have long pipelines. So they take forever to "warm up" and "slow down".
Is there a better way? You guess which direction the branch will go!

If you guessed right, you continue executing.
If you guessed wrong, you need to flush the pipeline and roll back to the branch. Then you can restart down the other path.

If you guess right every time, the execution will never have to stop.
If you guess wrong too often, you spend a lot of time stalling, rolling back, and restarting.

This is branch prediction. I admit it's not the best analogy since the train could just signal the direction with a flag. But in computers, the processor doesn't know which direction a branch will go until the last moment.
So how would you strategically guess to minimize the number of times that the train must back up and go down the other path? You look at the past history! If the train goes left 99% of the time, then you guess left. If it alternates, then you alternate your guesses. If it goes one way every three times, you guess the same...
In other words, you try to identify a pattern and follow it. This is more or less how branch predictors work.
Most applications have well-behaved branches. So modern branch predictors will typically achieve >90% hit rates. But when faced with unpredictable branches with no recognizable patterns, branch predictors are virtually useless.
Further reading: "Branch predictor" article on Wikipedia.

As hinted from above, the culprit is this if-statement:
if (data[c] >= 128)
	sum += data[c];

Notice that the data is evenly distributed between 0 and 255. When the data is sorted, roughly the first half of the iterations will not enter the if-statement. After that, they will all enter the if-statement.
This is very friendly to the branch predictor since the branch consecutively goes the same direction many times. Even a simple saturating counter will correctly predict the branch except for the few iterations after it switches direction.
Quick visualization:
T = branch taken
N = branch not taken

data[] = 0, 1, 2, 3, 4, ... 126, 127, 128, 129, 130, ... 250, 251, 252, ...
branch = N  N  N  N  N  ...   N    N    T    T    T  ...   T    T    T  ...

	   = NNNNNNNNNNNN ... NNNNNNNTTTTTTTTT ... TTTTTTTTTT  (easy to predict)

However, when the data is completely random, the branch predictor is rendered useless, because it can't predict random data. Thus there will probably be around 50% misprediction (no better than random guessing).
data[] = 226, 185, 125, 158, 198, 144, 217, 79, 202, 118,  14, 150, 177, 182, 133, ...
branch =   T,   T,   N,   T,   T,   T,   T,  N,   T,   N,   N,   T,   T,   T,   N  ...

	   = TTNTTTTNTNNTTTN ...   (completely random - hard to predict)


So what can be done?
If the compiler isn't able to optimize the branch into a conditional move, you can try some hacks if you are willing to sacrifice readability for performance.
Replace:
if (data[c] >= 128)
	sum += data[c];

with:
int t = (data[c] - 128) >> 31;
sum += ~t & data[c];

This eliminates the branch and replaces it with some bitwise operations.
(Note that this hack is not strictly equivalent to the original if-statement. But in this case, it's valid for all the input values of data[].)
Benchmarks: Core i7 920 @ 3.5 GHz
C++ - Visual Studio 2010 - x64 Release
//  Branch - Random
seconds = 11.777

//  Branch - Sorted
seconds = 2.352

//  Branchless - Random
seconds = 2.564

//  Branchless - Sorted
seconds = 2.587

Java - NetBeans 7.1.1 JDK 7 - x64
//  Branch - Random
seconds = 10.93293813

//  Branch - Sorted
seconds = 5.643797077

//  Branchless - Random
seconds = 3.113581453

//  Branchless - Sorted
seconds = 3.186068823

Observations:

With the Branch: There is a huge difference between the sorted and unsorted data.
With the Hack: There is no difference between sorted and unsorted data.
In the C++ case, the hack is actually a tad slower than with the branch when the data is sorted.

A general rule of thumb is to avoid data-dependent branching in critical loops (such as in this example).

Update:

GCC 4.6.1 with -O3 or -ftree-vectorize on x64 is able to generate a conditional move. So there is no difference between the sorted and unsorted data - both are fast.
VC++ 2010 is unable to generate conditional moves for this branch even under /Ox.
Intel C++ Compiler (ICC) 11 does something miraculous. It interchanges the two loops, thereby hoisting the unpredictable branch to the outer loop. So not only is it immune the mispredictions, it is also twice as fast as whatever VC++ and GCC can generate! In other words, ICC took advantage of the test-loop to defeat the benchmark...
If you give the Intel compiler the branchless code, it just out-right vectorizes it... and is just as fast as with the branch (with the loop interchange).

This goes to show that even mature modern compilers can vary wildly in their ability to optimize code...

===>
Why is processing a sorted array faster than processing an unsorted array?

Branch prediction.
With a sorted array, the condition data[c] >= 128 is first false for a streak of values, then becomes true for all later values. That's easy to predict. With an unsorted array, you pay for the branching cost.

===>
Why is processing a sorted array faster than processing an unsorted array?

The reason why performance improves drastically when the data is sorted is that the branch prediction penalty is removed, as explained beautifully in Mysticial's answer.
Now, if we look at the code
if (data[c] >= 128)
	sum += data[c];

we can find that the meaning of this particular if... else... branch is to add something when a condition is satisfied. This type of branch can be easily transformed into a conditional move statement, which would be compiled into a conditional move instruction: cmovl, in an x86 system. The branch and thus the potential branch prediction penalty is removed.
In C, thus C++, the statement, which would compile directly (without any optimization) into the conditional move instruction in x86, is the ternary operator ... ? ... : .... So we rewrite the above statement into an equivalent one:
sum += data[c] >=128 ? data[c] : 0;

While maintaining readability, we can check the speedup factor.
On an Intel Core i7-2600K @ 3.4 GHz and Visual Studio 2010 Release Mode, the benchmark is (format copied from Mysticial):
x86
//  Branch - Random
seconds = 8.885

//  Branch - Sorted
seconds = 1.528

//  Branchless - Random
seconds = 3.716

//  Branchless - Sorted
seconds = 3.71

x64
//  Branch - Random
seconds = 11.302

//  Branch - Sorted
 seconds = 1.830

//  Branchless - Random
seconds = 2.736

//  Branchless - Sorted
seconds = 2.737

The result is robust in multiple tests. We get a great speedup when the branch result is unpredictable, but we suffer a little bit when it is predictable. In fact, when using a conditional move, the performance is the same regardless of the data pattern.
Now let's look more closely by investigating the x86 assembly they generate. For simplicity, we use two functions max1 and max2.
max1 uses the conditional branch if... else ...:
int max1(int a, int b) {
	if (a > b)
		return a;
	else
		return b;
}

max2 uses the ternary operator ... ? ... : ...:
int max2(int a, int b) {
	return a > b ? a : b;
}

On a x86-64 machine, GCC -S generates the assembly below.
:max1
	movl    %edi, -4(%rbp)
	movl    %esi, -8(%rbp)
	movl    -4(%rbp), %eax
	cmpl    -8(%rbp), %eax
	jle     .L2
	movl    -4(%rbp), %eax
	movl    %eax, -12(%rbp)
	jmp     .L4
.L2:
	movl    -8(%rbp), %eax
	movl    %eax, -12(%rbp)
.L4:
	movl    -12(%rbp), %eax
	leave
	ret

:max2
	movl    %edi, -4(%rbp)
	movl    %esi, -8(%rbp)
	movl    -4(%rbp), %eax
	cmpl    %eax, -8(%rbp)
	cmovge  -8(%rbp), %eax
	leave
	ret

max2 uses much less code due to the usage of instruction cmovge. But the real gain is that max2 does not involve branch jumps, jmp, which would have a significant performance penalty if the predicted result is not right.
So why does a conditional move perform better?
In a typical x86 processor, the execution of an instruction is divided into several stages. Roughly, we have different hardware to deal with different stages. So we do not have to wait for one instruction to finish to start a new one. This is called pipelining.
In a branch case, the following instruction is determined by the preceding one, so we cannot do pipelining. We have to either wait or predict.
In a conditional move case, the execution conditional move instruction is divided into several stages, but the earlier stages like Fetch and Decode does not depend on the result of the previous instruction; only latter stages need the result. Thus, we wait a fraction of one instruction's execution time. This is why the conditional move version is slower than the branch when prediction is easy.
The book Computer Systems: A Programmer's Perspective, second edition explains this in detail. You can check Section 3.6.6 for Conditional Move Instructions, entire Chapter 4 for Processor Architecture, and Section 5.11.2 for a special treatment for Branch Prediction and Misprediction Penalties.
Sometimes, some modern compilers can optimize our code to assembly with better performance, sometimes some compilers can't (the code in question is using Visual Studio's native compiler). Knowing the performance difference between branch and conditional move when unpredictable can help us write code with better performance when the scenario gets so complex that the compiler can not optimize them automatically.

------------------------------
What is the “-->” operator in C++?
<div class="post-text" itemprop="text">
<p>After reading <a href="http://groups.google.com/group/comp.lang.c++.moderated/msg/33f173780d58dd20" rel="noreferrer">Hidden Features and Dark Corners of C++/STL</a> on <code>comp.lang.c++.moderated</code>, I was completely surprised that the following snippet compiled and worked in both Visual Studio 2008 and G++ 4.4.</p>
<p>Here's the code:</p>
<pre><code>#include &lt;stdio.h&gt;
int main()
{
	int x = 10;
	while (x --&gt; 0) // x goes to 0
	{
		printf("%d ", x);
	}
}
</code></pre>
<p>I'd assume this is C, since it works in GCC as well. Where is this defined in the standard, and where has it come from?</p>
</div>
List of answers:


===>
What is the “-->” operator in C++?

--> is not an operator. It is in fact two separate operators, -- and >.
The conditional's code decrements x, while returning x's original (not decremented) value, and then compares the original value with 0 using the > operator.
To better understand, the statement could be written as follows:
while( (x--) > 0 )


===>
What is the “-->” operator in C++?

Or for something completely different... x slides to 0
while (x --\
			\
			 \
			  \
			   > 0)
	 printf("%d ", x);

Not so mathematical, but... every picture paints a thousand words...

===>
What is the “-->” operator in C++?

That's a very complicated operator, so even ISO/IEC JTC1 (Joint Technical Committee 1) placed its description in two different parts of the C++ Standard.
Joking aside, they are two different operators: -- and > described respectively in §5.2.6/2 and §5.9 of the C++03 Standard.

------------------------------
The Definitive C++ Book Guide and List
<div class="post-text" itemprop="text">
<p>This question attempts to collect the few pearls among the dozens of bad C++ books that are published every year.</p>
<p>Unlike many other programming languages, which are often picked up on the go from tutorials found on the Internet, few are able to quickly pick up C++ without studying a well-written C++ book. It is way too big and complex for doing this. In fact, it is so big and complex, that <strong><em>there are very many very bad C++ books</em></strong> out there. And we are not talking about bad style, but things like sporting <em>glaringly obvious factual errors</em> and <em>promoting abysmally bad programming styles</em>.</p>
<p>Please edit the accepted answer to provide <strong>quality books</strong> and an approximate skill level — <em>preferably</em> <strong>after</strong> <em>discussing your addition in <a href="http://chat.stackoverflow.com/rooms/10/loungec">the C++ chat room</a></em>. (The regulars might mercilessly undo your work if they disagree with a recommendation.) Add a short blurb/description about each book that you have personally read/benefited from. Feel free to debate quality, headings, etc. Books that meet the criteria will be added to the list.  Books that have reviews by the Association of C and C++ Users (ACCU) have links to the review. </p>
<p><sub>*Note: FAQs and other resources can be found in the <a href="https://stackoverflow.com/tags/c%2b%2b/info">C++ tag info</a> and under <a class="post-tag" href="/questions/tagged/c%2b%2b-faq" rel="tag" title="show questions tagged 'c++-faq'">c++-faq</a>. </sub></p>
</div>
List of answers:


===>
The Definitive C++ Book Guide and List

Beginner
Introductory, no previous programming experience

C++ Primer * (Stanley Lippman, Josée Lajoie, and Barbara E. Moo)  (updated for C++11) Coming at 1k pages, this is a very thorough introduction into C++ that covers just about everything in the language in a very accessible format and in great detail. The fifth edition (released August 16, 2012) covers C++11. [Review]
Programming: Principles and Practice Using C++ (Bjarne Stroustrup, 2nd Edition - May 25, 2014) (updated for C++11/C++14) An introduction to programming using C++ by the creator of the language. A good read, that assumes no previous programming experience, but is not only for beginners.


* Not to be confused with C++ Primer Plus (Stephen Prata), with a significantly less favorable review.

Introductory, with previous programming experience

A Tour of C++ (Bjarne Stroustrup) (2nd edition  for C++17) The “tour” is a quick (about 180 pages and 14 chapters) tutorial overview of all of standard C++ (language and standard library, and using C++11) at a moderately high level for people who already know C++ or at least are experienced programmers. This book is an extended version of the material that constitutes Chapters 2-5 of The C++ Programming Language, 4th edition.
Accelerated C++ (Andrew Koenig and Barbara Moo, 1st Edition - August 24, 2000)  This basically covers the same ground as the C++ Primer, but does so on a fourth of its space. This is largely because it does not attempt to be an introduction to programming, but an introduction to C++ for people who've previously programmed in some other language. It has a steeper learning curve, but, for those who can cope with this, it is a very compact introduction to the language. (Historically, it broke new ground by being the first beginner's book to use a modern approach to teaching the language.) Despite this, the C++
it teaches is purely C++98. [Review]

Best practices

Effective C++ (Scott Meyers, 3rd Edition - May 22, 2005)  This was written with the aim of being the best second book C++ programmers should read, and it succeeded. Earlier editions were aimed at programmers coming from C, the third edition changes this and targets programmers coming from languages like Java. It presents ~50 easy-to-remember rules of thumb along with their rationale in a very accessible (and enjoyable) style. For C++11 and C++14 the examples and a few issues are outdated and Effective Modern C++ should be preferred. [Review]
Effective Modern C++ (Scott Meyers) This is basically the new version of Effective C++, aimed at C++ programmers making the transition from C++03 to C++11 and C++14.
Effective STL (Scott Meyers)  This aims to do the same to the part of the standard library coming from the STL what Effective C++ did to the language as a whole: It presents rules of thumb along with their rationale. [Review]


Intermediate

More Effective C++ (Scott Meyers) Even more rules of thumb than Effective C++. Not as important as the ones in the first book, but still good to know.
Exceptional C++ (Herb Sutter)  Presented as a set of puzzles, this has one of the best and thorough discussions of the proper resource management and exception safety in C++ through Resource Acquisition is Initialization (RAII) in addition to in-depth coverage of a variety of other topics including the pimpl idiom, name lookup, good class design, and the C++ memory model. [Review]
More Exceptional C++ (Herb Sutter)  Covers additional exception safety topics not covered in Exceptional C++, in addition to discussion of effective object-oriented programming in C++ and correct use of the STL. [Review]
Exceptional C++ Style (Herb Sutter)  Discusses generic programming, optimization, and resource management; this book also has an excellent exposition of how to write modular code in C++ by using non-member functions and the single responsibility principle. [Review]
C++ Coding Standards (Herb Sutter and Andrei Alexandrescu) “Coding standards” here doesn't mean “how many spaces should I indent my code?”  This book contains 101 best practices, idioms, and common pitfalls that can help you to write correct, understandable, and efficient C++ code. [Review]
C++ Templates: The Complete Guide (David Vandevoorde and Nicolai M. Josuttis) This is the book about templates as they existed before C++11.  It covers everything from the very basics to some of the most advanced template metaprogramming and explains every detail of how templates work (both conceptually and at how they are implemented) and discusses many common pitfalls.  Has excellent summaries of the One Definition Rule (ODR) and overload resolution in the appendices. A second edition covering C++11, C++14 and C++17 has been already published . [Review]
C++ 17 - The Complete Guide (Nicolai M. Josuttis) This book describes all the new features introduced in the C++17 Standard covering everything from the simple ones like 'Inline Variables', 'constexpr if' all the way up to 'Polymorphic Memory Resources' and 'New and Delete with overaligned Data'.


Advanced

Modern C++ Design (Andrei Alexandrescu)  A groundbreaking book on advanced generic programming techniques.  Introduces policy-based design, type lists, and fundamental generic programming idioms then explains how many useful design patterns (including small object allocators, functors, factories, visitors, and multi-methods) can be implemented efficiently, modularly, and cleanly using generic programming. [Review]
C++ Template Metaprogramming (David Abrahams and Aleksey Gurtovoy)
C++ Concurrency In Action (Anthony Williams) A book covering C++11 concurrency support including the thread library, the atomics library, the C++ memory model, locks and mutexes, as well as issues of designing and debugging multithreaded applications.
Advanced C++ Metaprogramming (Davide Di Gennaro) A pre-C++11 manual of TMP techniques, focused more on practice than theory.  There are a ton of snippets in this book, some of which are made obsolete by type traits, but the techniques, are nonetheless useful to know.  If you can put up with the quirky formatting/editing, it is easier to read than Alexandrescu, and arguably, more rewarding.  For more experienced developers, there is a good chance that you may pick up something about a dark corner of C++ (a quirk) that usually only comes about through extensive experience.


Reference Style - All Levels

The C++ Programming Language (Bjarne Stroustrup) (updated for C++11) The classic introduction to C++ by its creator. Written to parallel the classic K&R, this indeed reads very much like it and covers just about everything from the core language to the standard library, to programming paradigms to the language's philosophy. [Review] Note: All releases of the C++ standard are tracked in this question: Where do I find the current C++ standard.
C++ Standard Library Tutorial and Reference (Nicolai Josuttis) (updated for C++11) The introduction and reference for the C++ Standard Library. The second edition (released on April 9, 2012) covers C++11. [Review]
The C++ IO Streams and Locales (Angelika Langer and Klaus Kreft)  There's very little to say about this book except that, if you want to know anything about streams and locales, then this is the one place to find definitive answers. [Review]

C++11/14/17/… References:

The C++11/14/17 Standard (INCITS/ISO/IEC 14882:2011/2014/2017) This, of course, is the final arbiter of all that is or isn't C++. Be aware, however, that it is intended purely as a reference for experienced users willing to devote considerable time and effort to its understanding. The C++17 standard is released in electronic form for 198 Swiss Francs.
The C++17 standard is available, but seemingly not in an economical form – directly from the ISO it costs 198 Swiss Francs (about $200 US). For most people, the final draft before standardization is more than adequate (and free). Many will prefer an even newer draft, documenting new features that are likely to be included in C++20.
Overview of the New C++ (C++11/14) (PDF only) (Scott Meyers) (updated for C++14) These are the presentation materials (slides and some lecture notes) of a three-day training course offered by Scott Meyers, who's a highly respected author on C++. Even though the list of items is short, the quality is high.
The C++ Core Guidelines (C++11/14/17/…) (edited by Bjarne Stroustrup and Herb Sutter) is an evolving online document consisting of a set of guidelines for using modern C++ well. The guidelines are focused on relatively higher-level issues, such as interfaces, resource management, memory management and concurrency affecting application architecture and library design. The project was announced at CppCon'15 by Bjarne Stroustrup and others and welcomes contributions from the community. Most guidelines are supplemented with a rationale and examples as well as discussions of possible tool support. Many rules are designed specifically to be automatically checkable by static analysis tools.
The C++ Super-FAQ (Marshall Cline, Bjarne Stroustrup and others) is an effort by the Standard C++ Foundation to unify the C++ FAQs previously maintained individually by Marshall Cline and Bjarne Stroustrup and also incorporating new contributions. The items mostly address issues at an intermediate level and are often written with a humorous tone. Not all items might be fully up to date with the latest edition of the C++ standard yet.
cppreference.com (C++03/11/14/17/…) (initiated by Nate Kohl) is a wiki that summarizes the basic core-language features and has extensive documentation of the C++ standard library. The documentation is very precise but is easier to read than the official standard document and provides better navigation due to its wiki nature. The project documents all versions of the C++ standard and the site allows filtering the display for a specific version. The project was presented by Nate Kohl at CppCon'14.


Classics / Older
Note: Some information contained within these books may not be up-to-date or no longer considered best practice.

The Design and Evolution of C++ (Bjarne Stroustrup)  If you want to know why the language is the way it is, this book is where you find answers. This covers everything before the standardization of C++.
Ruminations on C++ - (Andrew Koenig and Barbara Moo) [Review]
Advanced C++ Programming Styles and Idioms (James Coplien)  A predecessor of the pattern movement, it describes many C++-specific “idioms”. It's certainly a very good book and might still be worth a read if you can spare the time, but quite old and not up-to-date with current C++.
Large Scale C++ Software Design (John Lakos)  Lakos explains techniques to manage very big C++ software projects. Certainly, a good read, if it only was up to date. It was written long before C++ 98 and misses on many features (e.g. namespaces) important for large-scale projects. If you need to work in a big C++ software project, you might want to read it, although you need to take more than a grain of salt with it. The first volume of a new edition is expected in 2018.
Inside the C++ Object Model (Stanley Lippman)  If you want to know how virtual member functions are commonly implemented and how base objects are commonly laid out in memory in a multi-inheritance scenario, and how all this affects performance, this is where you will find thorough discussions of such topics.
The Annotated C++ Reference Manual (Bjarne Stroustrup, Margaret A. Ellis) This book is quite outdated in the fact that it explores the 1989 C++ 2.0 version - Templates, exceptions, namespaces and new casts were not yet introduced. Saying that however, this book goes through the entire C++ standard of the time explaining the rationale, the possible implementations, and features of the language. This is not a book to learn programming principles and patterns on C++, but to understand every aspect of the C++ language.
Thinking in C++ (Bruce Eckel, 2nd Edition, 2000).  Two volumes; is a tutorial style free set of intro level books. Downloads: vol 1, vol 2. Unfortunately they’re marred by a number of trivial errors (e.g. maintaining that temporaries are automatically const), with no official errata list. A partial 3rd party errata list is available at (http://www.computersciencelab.com/Eckel.htm), but it’s apparently not maintained.
Scientific and Engineering C++: An Introduction to Advanced Techniques and Examples (John Barton and Lee Nackman)
It is a comprehensive and very detailed book that tried to explain and make use of all the features available in C++, in the context of numerical methods. It introduced at the time several new techniques, such as the Curiously Recurring Template Pattern (CRTP, also called Barton-Nackman trick).
It pioneered several techniques such as dimensional analysis and automatic differentiation.
It came with a lot of compilable and useful code, ranging from an expression parser to a Lapack wrapper.
The code is still available here: http://www.informit.com/store/scientific-and-engineering-c-plus-plus-an-introduction-9780201533934.
Unfortunately, the books have become somewhat outdated in the style and C++ features, however, it was an incredible tour-de-force at the time (1994, pre-STL).
The chapters on dynamics inheritance are a bit complicated to understand and not very useful.
An updated version of this classic book that includes move semantics and the lessons learned from the STL would be very nice.


------------------------------
What are the differences between a pointer variable and a reference variable in C++?
<div class="post-text" itemprop="text">
<p>I know references are syntactic sugar, so code is easier to read and write.</p>
<p>But what are the differences?</p>
<hr/>
<p>Summary from answers and links below:</p>
<ol>
<li>A pointer can be re-assigned any number of times while a reference cannot be re-assigned after binding.</li>
<li>Pointers can point nowhere (<code>NULL</code>), whereas a reference always refers to an object.</li>
<li>You can't take the address of a reference like you can with pointers.</li>
<li>There's no "reference arithmetic" (but you can take the address of an object pointed by a reference and do pointer arithmetic on it as in <code>&amp;obj + 5</code>).</li>
</ol>
<p>To clarify a misconception:</p>
<blockquote>
<p><em>The C++ standard is very careful to avoid dictating how a compiler may
  implement references, but every C++ compiler implements
  references as pointers. That is, a declaration such as:</em></p>
<pre><code>int &amp;ri = i;
</code></pre>
<p><strong><em>if it's not optimized away entirely</em></strong>, <em>allocates the same amount of storage
  as a pointer, and places the address
  of <code>i</code> into that storage.</em></p>
</blockquote>
<p><strong><em>So, a pointer and a reference both use the same amount of memory.</em></strong></p>
<p>As a general rule,</p>
<ul>
<li>Use references in function parameters and return types to provide useful and self-documenting interfaces.</li>
<li>Use pointers for implementing algorithms and data structures.</li>
</ul>
<p>Interesting read:</p>
<ul>
<li>My all-time favorite <a href="http://yosefk.com/c++fqa/ref.html" rel="noreferrer">C++ FAQ lite</a>.</li>
<li><a href="http://www.embedded.com/electronics-blogs/programming-pointers/4023307/References-vs-Pointers" rel="noreferrer">References vs. Pointers</a>.</li>
<li><a href="http://www.embedded.com/electronics-blogs/programming-pointers/4024641/An-Introduction-to-References" rel="noreferrer">An Introduction to References</a>.</li>
<li><a href="http://www.embedded.com/electronics-blogs/programming-pointers/4023290/References-and-const" rel="noreferrer">References and const</a>.</li>
</ul>
</div>
List of answers:


===>
What are the differences between a pointer variable and a reference variable in C++?


A pointer can be re-assigned:
int x = 5;
int y = 6;
int *p;
p =  &x;
p = &y;
*p = 10;
assert(x == 5);
assert(y == 10);

A reference cannot, and must be assigned at initialization:
int x = 5;
int y = 6;
int &r = x;

A pointer has its own memory address and size on the stack (4 bytes on x86), whereas a reference shares the same memory address (with the original variable) but also takes up some space on the stack.  Since a reference has the same address as the original variable itself, it is safe to think of a reference as another name for the same variable.  Note: What a pointer points to can be on the stack or heap.  Ditto a reference. My claim in this statement is not that a pointer must point to the stack.  A pointer is just a variable that holds a memory address.  This variable is on the stack.  Since a reference has its own space on the stack, and since the address is the same as the variable it references.  More on stack vs heap.  This implies that there is a real address of a reference that the compiler will not tell you.
int x = 0;
int &r = x;
int *p = &x;
int *p2 = &r;
assert(p == p2);

You can have pointers to pointers to pointers offering extra levels of indirection.  Whereas references only offer one level of indirection.
int x = 0;
int y = 0;
int *p = &x;
int *q = &y;
int **pp = &p;
pp = &q;//*pp = q
**pp = 4;
assert(y == 4);
assert(x == 0);

Pointer can be assigned nullptr directly, whereas reference cannot. If you try hard enough, and you know how, you can make the address of a reference nullptr.  Likewise, if you try hard enough you can have a reference to a pointer, and then that reference can contain nullptr.
int *p = nullptr;
int &r = nullptr; <--- compiling error
int &r = *p;  <--- likely no compiling error, especially if the nullptr is hidden behind a function call, yet it refers to a non-existent int at address 0

Pointers can iterate over an array, you can use ++ to go to the next item that a pointer is pointing to, and + 4 to go to the 5th element.  This is no matter what size the object is that the pointer points to.
A pointer needs to be dereferenced with * to access the memory location it points to, whereas a reference can be used directly.  A pointer to a class/struct uses -> to access it's members whereas a reference uses a ..
A pointer is a variable that holds a memory address.  Regardless of how a reference is implemented, a reference has the same memory address as the item it references.
References cannot be stuffed into an array, whereas pointers can be (Mentioned by user @litb)
Const references can be bound to temporaries. Pointers cannot (not without some indirection):
const int &x = int(12); //legal C++
int *y = &int(12); //illegal to dereference a temporary.

This makes const& safer for use in argument lists and so forth.


===>
What are the differences between a pointer variable and a reference variable in C++?

What's a C++ reference (for C programmers)
A reference can be thought of as a constant pointer (not to be confused with a pointer to a constant value!) with automatic indirection, ie the compiler will apply the * operator for you.
All references must be initialized with a non-null value or compilation will fail. It's neither possible to get the address of a reference - the address operator will return the address of the referenced value instead - nor is it possible to do arithmetics on references.
C programmers might dislike C++ references as it will no longer be obvious when indirection happens or if an argument gets passed by value or by pointer without looking at function signatures.
C++ programmers might dislike using pointers as they are considered unsafe - although references aren't really any safer than constant pointers except in the most trivial cases - lack the convenience of automatic indirection and carry a different semantic connotation.
Consider the following statement from the C++ FAQ:

Even though a reference is often implemented using an address in the
  underlying assembly language, please do not think of a reference as a
  funny looking pointer to an object. A reference is the object. It is
  not a pointer to the object, nor a copy of the object. It is the
  object.

But if a reference really were the object, how could there be dangling references? In unmanaged languages, it's impossible for references to be any 'safer' than pointers - there generally just isn't a way to reliably alias values across scope boundaries!
Why I consider C++ references useful
Coming from a C background, C++ references may look like a somewhat silly concept, but one should still use them instead of pointers where possible: Automatic indirection is convenient, and references become especially useful when dealing with RAII - but not because of any perceived safety advantage, but rather because they make writing idiomatic code less awkward.
RAII is one of the central concepts of C++, but it interacts non-trivially with copying semantics. Passing objects by reference avoids these issues as no copying is involved. If references were not present in the language, you'd have to use pointers instead, which are more cumbersome to use, thus violating the language design principle that the best-practice solution should be easier than the alternatives.

===>
What are the differences between a pointer variable and a reference variable in C++?

If you want to be really pedantic, there is one thing you can do with a reference that you can't do with a pointer: extend the lifetime of a temporary object. In C++ if you bind a const reference to a temporary object, the lifetime of that object becomes the lifetime of the reference.
std::string s1 = "123";
std::string s2 = "456";

std::string s3_copy = s1 + s2;
const std::string& s3_reference = s1 + s2;

In this example s3_copy copies the temporary object that is a result of the concatenation. Whereas s3_reference in essence becomes the temporary object. It's really a reference to a temporary object that now has the same lifetime as the reference.
If you try this without the const it should fail to compile. You cannot bind a non-const reference to a temporary object, nor can you take its address for that matter.

------------------------------
How do I iterate over the words of a string?
<div class="post-text" itemprop="text">
<p>I'm trying to iterate over the words of a string.</p>
<p>The string can be assumed to be composed of words separated by whitespace.</p>
<p>Note that I'm not interested in C string functions or that kind of character manipulation/access. Also, please give precedence to elegance over efficiency in your answer.</p>
<p>The best solution I have right now is:</p>
<pre><code>#include &lt;iostream&gt;
#include &lt;sstream&gt;
#include &lt;string&gt;

using namespace std;

int main()
{
	string s = "Somewhere down the road";
	istringstream iss(s);

	do
	{
		string subs;
		iss &gt;&gt; subs;
		cout &lt;&lt; "Substring: " &lt;&lt; subs &lt;&lt; endl;
	} while (iss);
}
</code></pre>
<p>Is there a more elegant way to do this?</p>
</div>
List of answers:


===>
How do I iterate over the words of a string?

For what it's worth, here's another way to extract tokens from an input string, relying only on standard library facilities. It's an example of the power and elegance behind the design of the STL.
#include <iostream>
#include <string>
#include <sstream>
#include <algorithm>
#include <iterator>

int main() {
	using namespace std;
	string sentence = "And I feel fine...";
	istringstream iss(sentence);
	copy(istream_iterator<string>(iss),
		 istream_iterator<string>(),
		 ostream_iterator<string>(cout, "\n"));
}

Instead of copying the extracted tokens to an output stream, one could insert them into a container, using the same generic copy algorithm.
vector<string> tokens;
copy(istream_iterator<string>(iss),
	 istream_iterator<string>(),
	 back_inserter(tokens));

... or create the vector directly:
vector<string> tokens{istream_iterator<string>{iss},
					  istream_iterator<string>{}};


===>
How do I iterate over the words of a string?

I use this to split string by a delimiter. The first puts the results in a pre-constructed vector, the second returns a new vector.
#include <string>
#include <sstream>
#include <vector>
#include <iterator>

template<typename Out>
void split(const std::string &s, char delim, Out result) {
	std::stringstream ss(s);
	std::string item;
	while (std::getline(ss, item, delim)) {
		*(result++) = item;
	}
}

std::vector<std::string> split(const std::string &s, char delim) {
	std::vector<std::string> elems;
	split(s, delim, std::back_inserter(elems));
	return elems;
}


Note that this solution does not skip empty tokens, so the following will find 4 items, one of which is empty:
std::vector<std::string> x = split("one:two::three", ':');


===>
How do I iterate over the words of a string?

A possible solution using Boost might be:
#include <boost/algorithm/string.hpp>
std::vector<std::string> strs;
boost::split(strs, "string to split", boost::is_any_of("\t "));

This approach might be even faster than the stringstream approach. And since this is a generic template function it can be used to split other types of strings (wchar, etc. or UTF-8) using all kinds of delimiters.
See the documentation for details.

------------------------------
What does the explicit keyword mean?
<div class="post-text" itemprop="text">
<p>What does the <code>explicit</code> keyword mean in C++?</p>
</div>
List of answers:


===>
What does the explicit keyword mean?

The compiler is allowed to make one implicit conversion to resolve the parameters to a function. What this means is that the compiler can use constructors callable with a single parameter to convert from one type to another in order to get the right type for a parameter.
Here's an example class with a constructor that can be used for implicit conversions:
class Foo
{
public:
  // single parameter constructor, can be used as an implicit conversion
  Foo (int foo) : m_foo (foo)
  {
  }

  int GetFoo () { return m_foo; }

private:
  int m_foo;
};

Here's a simple function that takes a Foo object:
void DoBar (Foo foo)
{
  int i = foo.GetFoo ();
}

and here's where the DoBar function is called.
int main ()
{
  DoBar (42);
}

The argument is not a Foo object, but an int. However, there exists a constructor for Foo that takes an int so this constructor can be used to convert the parameter to the correct type.
The compiler is allowed to do this once for each parameter.
Prefixing the explicit keyword to the constructor prevents the compiler from using that constructor for implicit conversions. Adding it to the above class will create a compiler error at the function call DoBar (42).  It is now necessary to call for conversion explicitly with  DoBar (Foo (42))
The reason you might want to do this is to avoid accidental construction that can hide bugs.  Contrived example:

You have a MyString(int size) class with a constructor that constructs a string of the given size.  You have a function print(const MyString&), and you call print(3) (when you actually intended to call print("3")).  You expect it to print "3", but it prints an empty string of length 3 instead.


===>
What does the explicit keyword mean?

Suppose, you have a class String:
class String {
public:
	String(int n); // allocate n bytes to the String object
	String(const char *p); // initializes object with char *p
};

Now, if you try:
String mystring = 'x';

The character 'x' will be implicitly converted to int and then the String(int) constructor will be called. But, this is not what the user might have intended. So, to prevent such conditions, we shall define the constructor as explicit:
class String {
public:
	explicit String (int n); //allocate n bytes
	String(const char *p); // initialize sobject with string p
};


===>
What does the explicit keyword mean?

In C++, a constructor with only one required parameter is considered an implicit conversion function.  It converts the parameter type to the class type.  Whether this is a good thing or not depends on the semantics of the constructor.
For example, if you have a string class with constructor String(const char* s), that's probably exactly what you want.  You can pass a const char* to a function expecting a String, and the compiler will automatically construct a temporary String object for you.
On the other hand, if you have a buffer class whose constructor Buffer(int size) takes the size of the buffer in bytes, you probably don't want the compiler to quietly turn ints into Buffers.  To prevent that, you declare the constructor with the explicit keyword:
class Buffer { explicit Buffer(int size); ... }

That way,
void useBuffer(Buffer& buf);
useBuffer(4);

becomes a compile-time error.  If you want to pass a temporary Buffer object, you have to do so explicitly:
useBuffer(Buffer(4));

In summary, if your single-parameter constructor converts the parameter into an object of your class, you probably don't want to use the explicit keyword.  But if you have a constructor that simply happens to take a single parameter, you should declare it as explicit to prevent the compiler from surprising you with unexpected conversions.

------------------------------
Why is “using namespace std;” considered bad practice?
<div class="post-text" itemprop="text">
<p>I've been told by others that writing <code>using namespace std;</code> in code is wrong, and that I should use <code>std::cout</code> and <code>std::cin</code> directly instead.</p>
<p>Why is <code>using namespace std;</code> considered a bad practice? Is it inefficient or does it risk declaring ambiguous variables (variables that share the same name as a function in <code>std</code> namespace)? Does it impact performance?</p>
</div>
List of answers:


===>
Why is “using namespace std;” considered bad practice?

This is not related to performance at all. But consider this: you are using two libraries called Foo and Bar:
using namespace foo;
using namespace bar;

Everything works fine, you can call Blah() from Foo and Quux() from Bar without problems. But one day you upgrade to a new version of Foo 2.0, which now offers a function called Quux(). Now you've got a conflict: Both Foo 2.0 and Bar import Quux() into your global namespace. This is going to take some effort to fix, especially if the function parameters happen to match.
If you had used foo::Blah() and bar::Quux(), then the introduction of foo::Quux() would have been a non-event.

===>
Why is “using namespace std;” considered bad practice?

I agree with everything Greg wrote, but I'd like to add: It can even get worse than Greg said!
Library Foo 2.0 could introduce a function, Quux(), that is an unambiguously better match for some of your calls to Quux() than the bar::Quux() your code called for years. Then your code still compiles, but it silently calls the wrong function and does god-knows-what. That's about as bad as things can get.
Keep in mind that the std namespace has tons of identifiers, many of which are very common ones (think list, sort, string, iterator, etc.) which are very likely to appear in other code, too.
If you consider this unlikely: There was a question asked here on Stack Overflow where pretty much exactly this happened (wrong function called due to omitted std:: prefix) about half a year after I gave this answer. Here is another, more recent example of such a question.
So this is a real problem.

Here's one more data point: Many, many years ago, I also used to find it annoying having to prefix everything from the standard library with std::. Then I worked in a project where it was decided at the start that both using directives and declarations are banned except for function scopes. Guess what? It took most of us very few weeks to get used to writing the prefix, and after a few more weeks most of us even agreed that it actually made the code more readable. There's a reason for that: Whether you like shorter or longer prose is subjective, but the prefixes objectively add clarity to the code. Not only the compiler, but you, too, find it easier to see which identifier is referred to.
In a decade, that project grew to have several million lines of code. Since these discussions come up again and again, I once was curious how often the (allowed) function-scope using actually was used in the project. I grep'd the sources for it and only found one or two dozen places where it was used. To me this indicates that, once tried, developers don't find std:: painful enough to employ using directives even once every 100 kLoC even where it was allowed to be used.

Bottom line: Explicitly prefixing everything doesn't do any harm, takes very little getting used to, and has objective advantages. In particular, it makes the code easier to interpret by the compiler and by human readers — and that should probably be the main goal when writing code.

===>
Why is “using namespace std;” considered bad practice?

The problem with putting using namespace in the header files of your classes is that it forces anyone who wants to use your classes (by including your header files) to also be 'using' (i.e. seeing everything in) those other namespaces.
However, you may feel free to put a using statement in your (private) *.cpp files.

Beware that some people disagree with my saying "feel free" like this -- because although a using statement in a cpp file is better than in a header (because it doesn't affect people who include your header file), they think it's still not good (because depending on the code it could make the implementation of the class more difficult to maintain). This FAQ topic says,

The using-directive exists for legacy C++ code and to ease the transition to namespaces, but you probably shouldn’t use it on a regular basis, at least not in your new C++ code.

The FAQ suggests two alternatives:

A using-declaration:
using std::cout; // a using-declaration lets you use cout without qualification
cout << "Values:";

Just typing std::
std::cout << "Values:";



------------------------------
How do you set, clear, and toggle a single bit?
<div class="post-text" itemprop="text">
<p>How do you set, clear, and toggle a bit in C/C++?</p>
</div>
List of answers:


===>
How do you set, clear, and toggle a single bit?

Setting a bit
Use the bitwise OR operator (|) to set a bit.
number |= 1UL << n;

That will set the nth bit of number. n should be zero, if you want to set the 1st bit and so on upto n-1, if you want to set the nth bit.
Use 1ULL if number is wider than unsigned long; promotion of 1UL << n doesn't happen until after evaluating 1UL << n where it's undefined behaviour to shift by more than the width of a long.  The same applies to all the rest of the examples.
Clearing a bit
Use the bitwise AND operator (&) to clear a bit.
number &= ~(1UL << n);

That will clear the nth bit of number. You must invert the bit string with the bitwise NOT operator (~), then AND it.
Toggling a bit
The XOR operator (^) can be used to toggle a bit.
number ^= 1UL << n;

That will toggle the nth bit of number.
Checking a bit
You didn't ask for this, but I might as well add it.
To check a bit, shift the number n to the right, then bitwise AND it:
bit = (number >> n) & 1U;

That will put the value of the nth bit of number into the variable bit.
Changing the nth bit to x
Setting the nth bit to either 1 or 0 can be achieved with the following on a 2's complement C++ implementation:
number ^= (-x ^ number) & (1UL << n);

Bit n will be set if x is 1, and cleared if x is 0.  If x has some other value, you get garbage.  x = !!x will booleanize it to 0 or 1.
To make this independent of 2's complement negation behaviour (where -1 has all bits set, unlike on a 1's complement or sign/magnitude C++ implementation), use unsigned negation.
number ^= (-(unsigned long)x ^ number) & (1UL << n);

or
unsigned long newbit = !!x;    // Also booleanize to force 0 or 1
number ^= (-newbit ^ number) & (1UL << n);

It's generally a good idea to use unsigned types for portable bit manipulation.
or
number = (number & ~(1UL << n)) | (x << n);

(number & ~(1UL << n)) will clear the nth bit and (x << n) will set the nth bit to x.
It's also generally a good idea to not to copy/paste code in general and so many people use preprocessor macros (like the community wiki answer further down) or some sort of encapsulation.

===>
How do you set, clear, and toggle a single bit?

Using the Standard C++ Library: std::bitset<N>.
Or the Boost version: boost::dynamic_bitset.
There is no need to roll your own:
#include <bitset>
#include <iostream>

int main()
{
	std::bitset<5> x;

	x[1] = 1;
	x[2] = 0;
	// Note x[0-4]  valid

	std::cout << x << std::endl;
}


[Alpha:] > ./a.out
00010

The Boost version allows a runtime sized bitset compared with a standard library compile-time sized bitset.

===>
How do you set, clear, and toggle a single bit?

The other option is to use bit fields:
struct bits {
	unsigned int a:1;
	unsigned int b:1;
	unsigned int c:1;
};

struct bits mybits;

defines a 3-bit field (actually, it's three 1-bit felds). Bit operations now become a bit (haha) simpler:
To set or clear a bit:
mybits.b = 1;
mybits.c = 0;

To toggle a bit:
mybits.a = !mybits.a;
mybits.b = ~mybits.b;
mybits.c ^= 1;  /* all work */

Checking a bit:
if (mybits.c)  //if mybits.c is non zero the next line below will execute

This only works with fixed-size bit fields. Otherwise you have to resort to the bit-twiddling techniques described in previous posts.

------------------------------
When should static_cast, dynamic_cast, const_cast and reinterpret_cast be used?
<div class="post-text" itemprop="text">
<p>What are the proper uses of:</p>
<ul>
<li><code>static_cast</code></li>
<li><code>dynamic_cast</code></li>
<li><code>const_cast</code></li>
<li><code>reinterpret_cast</code></li>
<li>C-style cast <code>(type)value</code></li>
<li>Function-style cast <code>type(value)</code></li>
</ul>
<p>How does one decide which to use in which specific cases?</p>
</div>
List of answers:


===>
When should static_cast, dynamic_cast, const_cast and reinterpret_cast be used?

static_cast is the first cast you should attempt to use. It does things like implicit conversions between types (such as int to float, or pointer to void*), and it can also call explicit conversion functions (or implicit ones). In many cases, explicitly stating static_cast isn't necessary, but it's important to note that the T(something) syntax is equivalent to (T)something and should be avoided (more on that later). A T(something, something_else) is safe, however, and guaranteed to call the constructor.
static_cast can also cast through inheritance hierarchies. It is unnecessary when casting upwards (towards a base class), but when casting downwards it can be used as long as it doesn't cast through virtual inheritance. It does not do checking, however, and it is undefined behavior to static_cast down a hierarchy to a type that isn't actually the type of the object.

const_cast can be used to remove or add const to a variable; no other C++ cast is capable of removing it (not even reinterpret_cast). It is important to note that modifying a formerly const value is only undefined if the original variable is const; if you use it to take the const off a reference to something that wasn't declared with const, it is safe. This can be useful when overloading member functions based on const, for instance. It can also be used to add const to an object, such as to call a member function overload.
const_cast also works similarly on volatile, though that's less common.

dynamic_cast is exclusively used for handling polymorphism. You can cast a pointer or reference to any polymorphic type to any other class type (a polymorphic type has at least one virtual function, declared or inherited). You can use it for more than just casting downwards – you can cast sideways or even up another chain. The dynamic_cast will seek out the desired object and return it if possible. If it can't, it will return nullptr in the case of a pointer, or throw std::bad_cast in the case of a reference.
dynamic_cast has some limitations, though. It doesn't work if there are multiple objects of the same type in the inheritance hierarchy (the so-called 'dreaded diamond') and you aren't using virtual inheritance. It also can only go through public inheritance - it will always fail to travel through protected or private inheritance. This is rarely an issue, however, as such forms of inheritance are rare.

reinterpret_cast is the most dangerous cast, and should be used very sparingly. It turns one type directly into another — such as casting the value from one pointer to another, or storing a pointer in an int, or all sorts of other nasty things. Largely, the only guarantee you get with reinterpret_cast is that normally if you cast the result back to the original type, you will get the exact same value (but not if the intermediate type is smaller than the original type). There are a number of conversions that reinterpret_cast cannot do, too. It's used primarily for particularly weird conversions and bit manipulations, like turning a raw data stream into actual data, or storing data in the low bits of a pointer to aligned data.

C-style cast and function-style cast are casts using (type)object or type(object), respectively, and are functionally equivalent. They are defined as the first of the following which succeeds:

const_cast
static_cast (though ignoring access restrictions)
static_cast (see above), then const_cast
reinterpret_cast
reinterpret_cast, then const_cast

It can therefore be used as a replacement for other casts in some instances, but can be extremely dangerous because of the ability to devolve into a reinterpret_cast, and the latter should be preferred when explicit casting is needed, unless you are sure static_cast will succeed or reinterpret_cast will fail. Even then, consider the longer, more explicit option.
C-style casts also ignore access control when performing a static_cast, which means that they have the ability to perform an operation that no other cast can. This is mostly a kludge, though, and in my mind is just another reason to avoid C-style casts.

===>
When should static_cast, dynamic_cast, const_cast and reinterpret_cast be used?

Use dynamic_cast for converting pointers/references within an inheritance hierarchy.
Use static_cast for ordinary type conversions.
Use reinterpret_cast for low-level reinterpreting of bit patterns.  Use with extreme caution.
Use const_cast for casting away const/volatile.  Avoid this unless you are stuck using a const-incorrect API.

===>
When should static_cast, dynamic_cast, const_cast and reinterpret_cast be used?

(A lot of theoretical and conceptual explanation has been given above)
Below are some of the practical examples when I used static_cast, dynamic_cast, const_cast, reinterpret_cast.
(Also referes this to understand the explaination : http://www.cplusplus.com/doc/tutorial/typecasting/)
static_cast :
OnEventData(void* pData)

{
  ......

  //  pData is a void* pData,

  //  EventData is a structure e.g.
  //  typedef struct _EventData {
  //  std::string id;
  //  std:: string remote_id;
  //  } EventData;

  // On Some Situation a void pointer *pData
  // has been static_casted as
  // EventData* pointer

  EventData *evtdata = static_cast<EventData*>(pData);
  .....
}

dynamic_cast :
void DebugLog::OnMessage(Message *msg)
{
	static DebugMsgData *debug;
	static XYZMsgData *xyz;

	if(debug = dynamic_cast<DebugMsgData*>(msg->pdata)){
		// debug message
	}
	else if(xyz = dynamic_cast<XYZMsgData*>(msg->pdata)){
		// xyz message
	}
	else/* if( ... )*/{
		// ...
	}
}

const_cast :
// *Passwd declared as a const

const unsigned char *Passwd


// on some situation it require to remove its constness

const_cast<unsigned char*>(Passwd)

reinterpret_cast :
typedef unsigned short uint16;

// Read Bytes returns that 2 bytes got read.

bool ByteBuffer::ReadUInt16(uint16& val) {
  return ReadBytes(reinterpret_cast<char*>(&val), 2);
}


------------------------------
Why are elementwise additions much faster in separate loops than in a combined loop?
<div class="post-text" itemprop="text">
<p>Suppose <code>a1</code>, <code>b1</code>, <code>c1</code>, and <code>d1</code> point to heap memory and my numerical code has the following core loop.</p>
<pre><code>const int n = 100000;

for (int j = 0; j &lt; n; j++) {
	a1[j] += b1[j];
	c1[j] += d1[j];
}
</code></pre>
<p>This loop is executed 10,000 times via another outer <code>for</code> loop. To speed it up, I changed the code to:</p>
<pre><code>for (int j = 0; j &lt; n; j++) {
	a1[j] += b1[j];
}

for (int j = 0; j &lt; n; j++) {
	c1[j] += d1[j];
}
</code></pre>
<p>Compiled on MS <a href="http://en.wikipedia.org/wiki/Visual_C++#32-bit_versions" rel="noreferrer">Visual C++ 10.0</a> with full optimization and <a href="http://en.wikipedia.org/wiki/SSE2" rel="noreferrer">SSE2</a> enabled for 32-bit on a <a href="http://en.wikipedia.org/wiki/Intel_Core_2" rel="noreferrer">Intel Core 2</a> Duo (x64), the first example takes 5.5 seconds and the double-loop example takes only 1.9 seconds. My question is: (Please refer to the my rephrased question at the bottom)</p>
<p>PS: I am not sure, if this helps:</p>
<p>Disassembly for the first loop basically looks like this (this block is repeated about five times in the full program):</p>
<pre><code>movsd       xmm0,mmword ptr [edx+18h]
addsd       xmm0,mmword ptr [ecx+20h]
movsd       mmword ptr [ecx+20h],xmm0
movsd       xmm0,mmword ptr [esi+10h]
addsd       xmm0,mmword ptr [eax+30h]
movsd       mmword ptr [eax+30h],xmm0
movsd       xmm0,mmword ptr [edx+20h]
addsd       xmm0,mmword ptr [ecx+28h]
movsd       mmword ptr [ecx+28h],xmm0
movsd       xmm0,mmword ptr [esi+18h]
addsd       xmm0,mmword ptr [eax+38h]
</code></pre>
<p>Each loop of the double loop example produces this code (the following block is repeated about three times):</p>
<pre><code>addsd       xmm0,mmword ptr [eax+28h]
movsd       mmword ptr [eax+28h],xmm0
movsd       xmm0,mmword ptr [ecx+20h]
addsd       xmm0,mmword ptr [eax+30h]
movsd       mmword ptr [eax+30h],xmm0
movsd       xmm0,mmword ptr [ecx+28h]
addsd       xmm0,mmword ptr [eax+38h]
movsd       mmword ptr [eax+38h],xmm0
movsd       xmm0,mmword ptr [ecx+30h]
addsd       xmm0,mmword ptr [eax+40h]
movsd       mmword ptr [eax+40h],xmm0
</code></pre>
<p>The question turned out to be of no relevance, as the behavior severely depends on the sizes of the arrays (n) and the CPU cache. So if there is further interest, I rephrase the question:</p>
<p><strong>Could you provide some solid insight into the details that lead to the different cache behaviors as illustrated by the five regions on the following graph?</strong></p>
<p><strong>It might also be interesting to point out the differences between CPU/cache architectures, by providing a similar graph for these CPUs.</strong></p>
<p>PPS: Here is the full code. It uses <a href="https://www.threadingbuildingblocks.org/" rel="noreferrer">TBB</a> <code>Tick_Count</code> for higher resolution timing, which can be disabled by not defining the <code>TBB_TIMING</code> Macro:</p>
<pre><code>#include &lt;iostream&gt;
#include &lt;iomanip&gt;
#include &lt;cmath&gt;
#include &lt;string&gt;

//#define TBB_TIMING

#ifdef TBB_TIMING
#include &lt;tbb/tick_count.h&gt;
using tbb::tick_count;
#else
#include &lt;time.h&gt;
#endif

using namespace std;

//#define preallocate_memory new_cont

enum { new_cont, new_sep };

double *a1, *b1, *c1, *d1;


void allo(int cont, int n)
{
	switch(cont) {
	  case new_cont:
		a1 = new double[n*4];
		b1 = a1 + n;
		c1 = b1 + n;
		d1 = c1 + n;
		break;
	  case new_sep:
		a1 = new double[n];
		b1 = new double[n];
		c1 = new double[n];
		d1 = new double[n];
		break;
	}

	for (int i = 0; i &lt; n; i++) {
		a1[i] = 1.0;
		d1[i] = 1.0;
		c1[i] = 1.0;
		b1[i] = 1.0;
	}
}

void ff(int cont)
{
	switch(cont){
	  case new_sep:
		delete[] b1;
		delete[] c1;
		delete[] d1;
	  case new_cont:
		delete[] a1;
	}
}

double plain(int n, int m, int cont, int loops)
{
#ifndef preallocate_memory
	allo(cont,n);
#endif

#ifdef TBB_TIMING
	tick_count t0 = tick_count::now();
#else
	clock_t start = clock();
#endif

	if (loops == 1) {
		for (int i = 0; i &lt; m; i++) {
			for (int j = 0; j &lt; n; j++){
				a1[j] += b1[j];
				c1[j] += d1[j];
			}
		}
	} else {
		for (int i = 0; i &lt; m; i++) {
			for (int j = 0; j &lt; n; j++) {
				a1[j] += b1[j];
			}
			for (int j = 0; j &lt; n; j++) {
				c1[j] += d1[j];
			}
		}
	}
	double ret;

#ifdef TBB_TIMING
	tick_count t1 = tick_count::now();
	ret = 2.0*double(n)*double(m)/(t1-t0).seconds();
#else
	clock_t end = clock();
	ret = 2.0*double(n)*double(m)/(double)(end - start) *double(CLOCKS_PER_SEC);
#endif

#ifndef preallocate_memory
	ff(cont);
#endif

	return ret;
}


void main()
{
	freopen("C:\\test.csv", "w", stdout);

	char *s = " ";

	string na[2] ={"new_cont", "new_sep"};

	cout &lt;&lt; "n";

	for (int j = 0; j &lt; 2; j++)
		for (int i = 1; i &lt;= 2; i++)
#ifdef preallocate_memory
			cout &lt;&lt; s &lt;&lt; i &lt;&lt; "_loops_" &lt;&lt; na[preallocate_memory];
#else
			cout &lt;&lt; s &lt;&lt; i &lt;&lt; "_loops_" &lt;&lt; na[j];
#endif

	cout &lt;&lt; endl;

	long long nmax = 1000000;

#ifdef preallocate_memory
	allo(preallocate_memory, nmax);
#endif

	for (long long n = 1L; n &lt; nmax; n = max(n+1, long long(n*1.2)))
	{
		const long long m = 10000000/n;
		cout &lt;&lt; n;

		for (int j = 0; j &lt; 2; j++)
			for (int i = 1; i &lt;= 2; i++)
				cout &lt;&lt; s &lt;&lt; plain(n, m, j, i);
		cout &lt;&lt; endl;
	}
}
</code></pre>
<p>(It shows FLOP/s for different values of <code>n</code>.)</p>
<p><img alt="enter image description here" src="https://i.stack.imgur.com/keuWU.gif"/></p>
</div>
List of answers:


===>
Why are elementwise additions much faster in separate loops than in a combined loop?

Upon further analysis of this, I believe this is (at least partially) caused by data alignment of the four pointers. This will cause some level of cache bank/way conflicts.
If I've guessed correctly on how you are allocating your arrays, they are likely to be aligned to the page line.
This means that all your accesses in each loop will fall on the same cache way. However, Intel processors have had 8-way L1 cache associativity for a while. But in reality, the performance isn't completely uniform. Accessing 4-ways is still slower than say 2-ways.
EDIT : It does in fact look like you are allocating all the arrays separately.
Usually when such large allocations are requested, the allocator will request fresh pages from the OS. Therefore, there is a high chance that large allocations will appear at the same offset from a page-boundary.
Here's the test code:
int main(){
	const int n = 100000;

#ifdef ALLOCATE_SEPERATE
	double *a1 = (double*)malloc(n * sizeof(double));
	double *b1 = (double*)malloc(n * sizeof(double));
	double *c1 = (double*)malloc(n * sizeof(double));
	double *d1 = (double*)malloc(n * sizeof(double));
#else
	double *a1 = (double*)malloc(n * sizeof(double) * 4);
	double *b1 = a1 + n;
	double *c1 = b1 + n;
	double *d1 = c1 + n;
#endif

	//  Zero the data to prevent any chance of denormals.
	memset(a1,0,n * sizeof(double));
	memset(b1,0,n * sizeof(double));
	memset(c1,0,n * sizeof(double));
	memset(d1,0,n * sizeof(double));

	//  Print the addresses
	cout << a1 << endl;
	cout << b1 << endl;
	cout << c1 << endl;
	cout << d1 << endl;

	clock_t start = clock();

	int c = 0;
	while (c++ < 10000){

#if ONE_LOOP
		for(int j=0;j<n;j++){
			a1[j] += b1[j];
			c1[j] += d1[j];
		}
#else
		for(int j=0;j<n;j++){
			a1[j] += b1[j];
		}
		for(int j=0;j<n;j++){
			c1[j] += d1[j];
		}
#endif

	}

	clock_t end = clock();
	cout << "seconds = " << (double)(end - start) / CLOCKS_PER_SEC << endl;

	system("pause");
	return 0;
}


Benchmark Results:
EDIT: Results on an actual Core 2 architecture machine:
2 x Intel Xeon X5482 Harpertown @ 3.2 GHz:
#define ALLOCATE_SEPERATE
#define ONE_LOOP
00600020
006D0020
007A0020
00870020
seconds = 6.206

#define ALLOCATE_SEPERATE
//#define ONE_LOOP
005E0020
006B0020
00780020
00850020
seconds = 2.116

//#define ALLOCATE_SEPERATE
#define ONE_LOOP
00570020
00633520
006F6A20
007B9F20
seconds = 1.894

//#define ALLOCATE_SEPERATE
//#define ONE_LOOP
008C0020
00983520
00A46A20
00B09F20
seconds = 1.993

Observations:

6.206 seconds with one loop and 2.116 seconds with two loops. This reproduces the OP's results exactly.
In the first two tests, the arrays are allocated separately. You'll notice that they all have the same alignment relative to the page.
In the second two tests, the arrays are packed together to break that alignment. Here you'll notice both loops are faster. Furthermore, the second (double) loop is now the slower one as you would normally expect.

As @Stephen Cannon points out in the comments, there is very likely possibility that this alignment causes false aliasing in the load/store units or the cache. I Googled around for this and found that Intel actually has a hardware counter for partial address aliasing stalls:
http://software.intel.com/sites/products/documentation/doclib/stdxe/2013/~amplifierxe/pmw_dp/events/partial_address_alias.html

5 Regions - Explanations
Region 1:
This one is easy. The dataset is so small that the performance is dominated by overhead like looping and branching.
Region 2:
Here, as the data sizes increases, the amount of relative overhead goes down and the performance "saturates". Here two loops is slower because it has twice as much loop and branching overhead.
I'm not sure exactly what's going on here... Alignment could still play an effect as Agner Fog mentions cache bank conflicts. (That link is about Sandy Bridge, but the idea should still be applicable to Core 2.)
Region 3:
At this point, the data no longer fits in L1 cache. So performance is capped by the L1 <-> L2 cache bandwidth.
Region 4:
The performance drop in the single-loop is what we are observing. And as mentioned, this is due to the alignment which (most likely) causes false aliasing stalls in the processor load/store units.
However, in order for false aliasing to occur, there must be a large enough stride between the datasets. This is why you don't see this in region 3.
Region 5:
At this point, nothing fits in cache. So you're bound by memory bandwidth.





===>
Why are elementwise additions much faster in separate loops than in a combined loop?

OK, the right answer definitely has to do something with the CPU cache. But to use the cache argument can be quite difficult, especially without data.
There are many answers, that led to a lot of discussion, but let's face it: Cache issues can be very complex and are not one dimensional. They depend heavily on the size of the data, so my question was unfair: It turned out to be at a very interesting point in the cache graph.
@Mysticial's answer convinced a lot of people (including me), probably because it was the only one that seemed to rely on facts, but it was only one "data point" of the truth.
That's why I combined his test (using a continuous vs. separate allocation) and @James' Answer's advice.
The graphs below shows, that most of the answers and especially the majority of comments to the question and answers can be considered completely wrong or true depending on the exact scenario and parameters used.
Note that my initial question was at n = 100.000. This point (by accident) exhibits special behavior:

It possesses the greatest discrepancy between the one and two loop'ed version (almost a factor of three)
It is the only point, where one-loop (namely with continuous allocation) beats the two-loop version. (This made Mysticial's answer possible, at all.)

The result using initialized data:

The result using uninitialized data (this is what Mysticial tested):

And this is a hard-to-explain one: Initialized data, that is allocated once and reused for every following test case of different vector size:

Proposal
Every low-level performance related question on Stack Overflow should be required to provide MFLOPS information for the whole range of cache relevant data sizes! It's a waste of everybody's time to think of answers and especially discuss them with others without this information.

===>
Why are elementwise additions much faster in separate loops than in a combined loop?

The second loop involves a lot less cache activity, so it's easier for the processor to keep up with the memory demands.

------------------------------
What is the difference between #include <filename> and #include “filename”?
<div class="post-text" itemprop="text">
<p>In the C and C++ programming languages, what is the difference between using angle brackets and using quotes in an <code>include</code> statement, as follows?</p>
<ol>
<li><code>#include &lt;filename&gt;</code> </li>
<li><code>#include "filename"</code></li>
</ol>
</div>
List of answers:


===>
What is the difference between #include <filename> and #include “filename”?

In practice, the difference is in the location where the preprocessor searches for the included file.
For #include <filename> the preprocessor searches in an implementation dependent manner, normally in search directories pre-designated by the compiler/IDE. This method is normally used to include standard library header files.
For #include "filename" the preprocessor searches first in the same directory as the file containing the directive, and then follows the search path used for the #include <filename> form. This method is normally used to include programmer-defined header files.
A more complete description is available in the GCC documentation on search paths.

===>
What is the difference between #include <filename> and #include “filename”?

The only way to know is to read your implementation's documentation.
In the C standard, section 6.10.2, paragraphs 2 to 4 state:


A preprocessing directive of the form
#include <h-char-sequence> new-line

searches a sequence of implementation-defined places for a header identified uniquely by the specified sequence between the < and > delimiters, and causes the replacement of that directive by the entire contents of the header. How the places are specified or the header identified is implementation-defined.
A preprocessing directive of the form
#include "q-char-sequence" new-line

causes the replacement of that directive by the entire contents of the source file identified by the specified sequence between the " delimiters. The named source file is searched for in an implementation-defined manner. If this search is not supported, or if the search fails, the directive is reprocessed as if it read
#include <h-char-sequence> new-line

with the identical contained sequence (including > characters, if any) from the original
  directive.
A preprocessing directive of the form
#include pp-tokens new-line

(that does not match one of the two previous forms) is permitted. The preprocessing tokens after include in the directive are processed just as in normal text. (Each identifier currently defined as a macro name is replaced by its replacement list of preprocessing tokens.) The directive resulting after all replacements shall match one of the two previous forms. The method by which a sequence of preprocessing tokens between a < and a > preprocessing token pair or a pair of " characters is combined into a single header name preprocessing token is implementation-defined.

Definitions:

h-char: any member of the source character set except the new-line character and >
q-char: any member of the source character set except the new-line character and "



===>
What is the difference between #include <filename> and #include “filename”?

The sequence of characters between < and > uniquely refer to a header, which isn't necessarily a file. Implementations are pretty much free to use the character sequence as they wish. (Mostly, however, just treat it as a file name and do a search in the include path, as the other posts state.)
If the #include "file" form is used, the implementation first looks for a file of the given name, if supported. If not (supported), or if the search fails, the implementation behaves as though the other (#include <file>) form was used.
Also, a third form exists and is used when the #include directive doesn't match either of the forms above. In this form, some basic preprocessing (such as macro expansion) is done on the "operands" of the #include directive, and the result is expected to match one of the two other forms.

------------------------------
What is The Rule of Three?
<div class="post-text" itemprop="text">
<ul>
<li>What does <em>copying an object</em> mean?</li>
<li>What are the <em>copy constructor</em> and the <em>copy assignment operator</em>?</li>
<li>When do I need to declare them myself?</li>
<li>How can I prevent my objects from being copied?</li>
</ul>
</div>
List of answers:


===>
What is The Rule of Three?

Introduction
C++ treats variables of user-defined types with value semantics.
This means that objects are implicitly copied in various contexts,
and we should understand what "copying an object" actually means.
Let us consider a simple example:
class person
{
	std::string name;
	int age;

public:

	person(const std::string& name, int age) : name(name), age(age)
	{
	}
};

int main()
{
	person a("Bjarne Stroustrup", 60);
	person b(a);   // What happens here?
	b = a;         // And here?
}

(If you are puzzled by the name(name), age(age) part,
this is called a member initializer list.)
Special member functions
What does it mean to copy a person object?
The main function shows two distinct copying scenarios.
The initialization person b(a); is performed by the copy constructor.
Its job is to construct a fresh object based on the state of an existing object.
The assignment b = a is performed by the copy assignment operator.
Its job is generally a little more complicated,
because the target object is already in some valid state that needs to be dealt with.
Since we declared neither the copy constructor nor the assignment operator (nor the destructor) ourselves,
these are implicitly defined for us. Quote from the standard:

The [...] copy constructor and copy assignment operator, [...] and destructor are special member functions.
  [ Note: The implementation will implicitly declare these member functions
  for some class types when the program does not explicitly declare them.
  The implementation will implicitly define them if they are used. [...] end note ]
  [n3126.pdf section 12 §1]

By default, copying an object means copying its members:

The implicitly-defined copy constructor for a non-union class X performs a memberwise copy of its subobjects.
  [n3126.pdf section 12.8 §16]
The implicitly-defined copy assignment operator for a non-union class X performs memberwise copy assignment
  of its subobjects.
  [n3126.pdf section 12.8 §30]

Implicit definitions
The implicitly-defined special member functions for person look like this:
// 1. copy constructor
person(const person& that) : name(that.name), age(that.age)
{
}

// 2. copy assignment operator
person& operator=(const person& that)
{
	name = that.name;
	age = that.age;
	return *this;
}

// 3. destructor
~person()
{
}

Memberwise copying is exactly what we want in this case:
name and age are copied, so we get a self-contained, independent person object.
The implicitly-defined destructor is always empty.
This is also fine in this case since we did not acquire any resources in the constructor.
The members' destructors are implicitly called after the person destructor is finished:

After executing the body of the destructor and destroying any automatic objects allocated within the body,
  a destructor for class X calls the destructors for X's direct [...] members
  [n3126.pdf 12.4 §6]

Managing resources
So when should we declare those special member functions explicitly?
When our class manages a resource, that is,
when an object of the class is responsible for that resource.
That usually means the resource is acquired in the constructor
(or passed into the constructor) and released in the destructor.
Let us go back in time to pre-standard C++.
There was no such thing as std::string, and programmers were in love with pointers.
The person class might have looked like this:
class person
{
	char* name;
	int age;

public:

	// the constructor acquires a resource:
	// in this case, dynamic memory obtained via new[]
	person(const char* the_name, int the_age)
	{
		name = new char[strlen(the_name) + 1];
		strcpy(name, the_name);
		age = the_age;
	}

	// the destructor must release this resource via delete[]
	~person()
	{
		delete[] name;
	}
};

Even today, people still write classes in this style and get into trouble:
"I pushed a person into a vector and now I get crazy memory errors!"
Remember that by default, copying an object means copying its members,
but copying the name member merely copies a pointer, not the character array it points to!
This has several unpleasant effects:

Changes via a can be observed via b.
Once b is destroyed, a.name is a dangling pointer.
If a is destroyed, deleting the dangling pointer yields undefined behavior.
Since the assignment does not take into account what name pointed to before the assignment,
sooner or later you will get memory leaks all over the place.

Explicit definitions
Since memberwise copying does not have the desired effect, we must define the copy constructor and the copy assignment operator explicitly to make deep copies of the character array:
// 1. copy constructor
person(const person& that)
{
	name = new char[strlen(that.name) + 1];
	strcpy(name, that.name);
	age = that.age;
}

// 2. copy assignment operator
person& operator=(const person& that)
{
	if (this != &that)
	{
		delete[] name;
		// This is a dangerous point in the flow of execution!
		// We have temporarily invalidated the class invariants,
		// and the next statement might throw an exception,
		// leaving the object in an invalid state :(
		name = new char[strlen(that.name) + 1];
		strcpy(name, that.name);
		age = that.age;
	}
	return *this;
}

Note the difference between initialization and assignment:
we must tear down the old state before assigning to name to prevent memory leaks.
Also, we have to protect against self-assignment of the form x = x.
Without that check, delete[] name would delete the array containing the source string,
because when you write x = x, both this->name and that.name contain the same pointer.
Exception safety
Unfortunately, this solution will fail if new char[...] throws an exception due to memory exhaustion.
One possible solution is to introduce a local variable and reorder the statements:
// 2. copy assignment operator
person& operator=(const person& that)
{
	char* local_name = new char[strlen(that.name) + 1];
	// If the above statement throws,
	// the object is still in the same state as before.
	// None of the following statements will throw an exception :)
	strcpy(local_name, that.name);
	delete[] name;
	name = local_name;
	age = that.age;
	return *this;
}

This also takes care of self-assignment without an explicit check.
An even more robust solution to this problem is the copy-and-swap idiom,
but I will not go into the details of exception safety here.
I only mentioned exceptions to make the following point: Writing classes that manage resources is hard.
Noncopyable resources
Some resources cannot or should not be copied, such as file handles or mutexes.
In that case, simply declare the copy constructor and copy assignment operator as private without giving a definition:
private:

	person(const person& that);
	person& operator=(const person& that);

Alternatively, you can inherit from boost::noncopyable or declare them as deleted (in C++11 and above):
person(const person& that) = delete;
person& operator=(const person& that) = delete;

The rule of three
Sometimes you need to implement a class that manages a resource.
(Never manage multiple resources in a single class,
this will only lead to pain.)
In that case, remember the rule of three:

If you need to explicitly declare either the destructor,
  copy constructor or copy assignment operator yourself,
  you probably need to explicitly declare all three of them.

(Unfortunately, this "rule" is not enforced by the C++ standard or any compiler I am aware of.)
The rule of five
From C++11 on, an object has 2 extra special member functions: the move constructor and move assignment. The rule of five states to implement these functions as well.
An example with the signatures:
class person
{
	std::string name;
	int age;

public:
	person(const std::string& name, int age);        // Ctor
	person(const person &) = default;                // Copy Ctor
	person(person &&) noexcept = default;            // Move Ctor
	person& operator=(const person &) = default;     // Copy Assignment
	person& operator=(person &&) noexcept = default; // Move Assignment
	~person() noexcept = default;                    // Dtor
};

The rule of zero
The rule of 3/5 is also referred to as the rule of 0/3/5. The zero part of the rule states that you are allowed to not write any of the special member functions when creating your class.
Advice
Most of the time, you do not need to manage a resource yourself,
because an existing class such as std::string already does it for you.
Just compare the simple code using a std::string member
to the convoluted and error-prone alternative using a char* and you should be convinced.
As long as you stay away from raw pointer members, the rule of three is unlikely to concern your own code.

===>
What is The Rule of Three?

The Rule of Three is a rule of thumb for C++, basically saying

If your class needs any of

a copy constructor,
an assignment operator,
or a destructor,

defined explictly, then it is likely to need all three of them.

The reasons for this is that all three of them are usually used to manage a resource, and if your class manages a resource, it usually needs to manage copying as well as freeing.
If there is no good semantic for copying the resource your class manages, then consider to forbid copying by declaring (not defining) the copy constructor and assignment operator as private.
(Note that the forthcoming new version of the C++ standard (which is C++11) adds move semantics to C++, which will likely change the Rule of Three. However, I know too little about this to write a C++11 section about the Rule of Three.)

===>
What is The Rule of Three?

The law of the big three is as specified above.
An easy example, in plain English, of the kind of problem it solves:
Non default destructor
You allocated memory in your constructor and so you need to write a destructor to delete it.  Otherwise you will cause a memory leak.
You might think that this is job done.
The problem will be, if a copy is made of your object, then the copy will point to the same memory as the original object.
Once, one of these deletes the memory in its destructor, the other will have a pointer to invalid memory (this is called a dangling pointer) when it tries to use it things are going to get hairy.
Therefore, you write a copy constructor so that it allocates new objects their own pieces of memory to destroy.
Assignment operator and copy constructor
You allocated memory in your constructor to a member pointer of your class.  When you copy an object of this class the default assignment operator and copy constructor will copy the value of this member pointer to the new object.
This means that the new object and the old object will be pointing at the same piece of memory so when you change it in one object it will be changed for the other objerct too.  If one object deletes this memory the other will carry on trying to use it - eek.
To resolve this you write your own version of the copy constructor and assignment operator.  Your versions allocate separate memory to the new objects and copy across the values that the first pointer is pointing to rather than its address.

------------------------------
What are the basic rules and idioms for operator overloading?
<div class="post-text" itemprop="text">
<p>Note: The answers were given in <em>a specific order</em>, but since many users sort answers according to votes, rather than the time they were given, here's an <strong><em>index of the answers</em></strong> in the order in which they make most sense:</p>
<ul>
<li><a href="https://stackoverflow.com/questions/4421706/operator-overloading-in-c/4421715#4421715">The General Syntax of operator overloading in C++</a></li>
<li><a href="https://stackoverflow.com/questions/4421706/operator-overloading-in-c/4421708#4421708">The Three Basic Rules of Operator Overloading in C++</a></li>
<li><a href="https://stackoverflow.com/questions/4421706/operator-overloading-in-c/4421729#4421729">The Decision between Member and Non-member</a></li>
<li><a href="https://stackoverflow.com/questions/4421706/operator-overloading-in-c/4421719#4421719">Common operators to overload</a>
<ul>
<li>Assignment Operator</li>
<li>Input and Output Operators</li>
<li>Function call operator</li>
<li>Comparison operators</li>
<li>Arithmetic Operators</li>
<li>Array Subscripting</li>
<li>Operators for Pointer-like Types</li>
<li><a href="https://stackoverflow.com/questions/4421706/operator-overloading/16615725#16615725">Conversion Operators</a></li>
</ul></li>
<li><a href="https://stackoverflow.com/questions/4421706/operator-overloading-in-c/4421791#4421791">Overloading new and delete</a> </li>
</ul>
<p><sub>
<em>(Note: This is meant to be an entry to <a href="https://stackoverflow.com/questions/tagged/c++-faq">Stack Overflow's C++ FAQ</a>. If you want to critique the idea of providing an FAQ in this form, then <a href="https://meta.stackexchange.com/questions/68647/setting-up-a-faq-for-the-c-tag">the posting on meta that started all this</a> would be the place to do that. Answers to that question are monitored in the <a href="http://chat.stackoverflow.com/rooms/10/c-lounge">C++ chatroom</a>, where the FAQ idea started out in the first place, so your answer is very likely to get read by those who came up with the idea.)</em>
</sub> </p>
</div>
List of answers:


===>
What are the basic rules and idioms for operator overloading?

Common operators to overload
Most of the work in overloading operators is boiler-plate code. That is little wonder, since operators are merely syntactic sugar, their actual work could be done by (and often is forwarded to) plain functions. But it is important that you get this boiler-plate code right. If you fail, either your operator’s code won’t compile or your users’ code won’t compile or your users’ code will behave surprisingly.
Assignment Operator
There's a lot to be said about assignment. However, most of it has already been said in GMan's famous Copy-And-Swap FAQ, so I'll skip most of it here, only listing the perfect assignment operator for reference:
X& X::operator=(X rhs)
{
  swap(rhs);
  return *this;
}

Bitshift Operators (used for Stream I/O)
The bitshift operators << and >>, although still used in hardware interfacing for the bit-manipulation functions they inherit from C, have become more prevalent as overloaded stream input and output operators in most applications.  For guidance overloading as bit-manipulation operators, see the section below on Binary Arithmetic Operators.  For implementing your own custom format and parsing logic when your object is used with iostreams, continue.
The stream operators, among the most commonly overloaded operators, are binary infix operators for which the syntax specifies no restriction on whether they should be members or non-members.
Since they change their left argument (they alter the stream’s state), they should, according to the rules of thumb, be implemented as members of their left operand’s type. However, their left operands are streams from the standard library, and while most of the stream output and input operators defined by the standard library are indeed defined as members of the stream classes, when you implement output and input operations for your own types, you cannot change the standard library’s stream types. That’s why you need to implement these operators for your own types as non-member functions.
The canonical forms of the two are these:
std::ostream& operator<<(std::ostream& os, const T& obj)
{
  // write obj to stream

  return os;
}

std::istream& operator>>(std::istream& is, T& obj)
{
  // read obj from stream

  if( /* no valid object of T found in stream */ )
	is.setstate(std::ios::failbit);

  return is;
}

When implementing operator>>, manually setting the stream’s state is only necessary when the reading itself succeeded, but the result is not what would be expected.
Function call operator
The function call operator, used to create function objects, also known as functors, must be defined as a member function, so it always has the implicit this argument of member functions. Other than this, it can be overloaded to take any number of additional arguments, including zero.
Here's an example of the syntax:
class foo {
public:
	// Overloaded call operator
	int operator()(const std::string& y) {
		// ...
	}
};

Usage:
foo f;
int a = f("hello");

Throughout the C++ standard library, function objects are always copied. Your own function objects should therefore be cheap to copy. If a function object absolutely needs to use data which is expensive to copy, it is better to store that data elsewhere and have the function object refer to it.
Comparison operators
The binary infix comparison operators should, according to the rules of thumb, be implemented as non-member functions1. The unary prefix negation ! should (according to the same rules) be implemented as a member function. (but it is usually not a good idea to overload it.)
The standard library’s algorithms (e.g. std::sort()) and types (e.g. std::map) will always only expect operator< to be present. However, the users of your type will expect all the other operators to be present, too, so if you define operator<, be sure to follow the third fundamental rule of operator overloading and also define all the other boolean comparison operators. The canonical way to implement them is this:
inline bool operator==(const X& lhs, const X& rhs){ /* do actual comparison */ }
inline bool operator!=(const X& lhs, const X& rhs){return !operator==(lhs,rhs);}
inline bool operator< (const X& lhs, const X& rhs){ /* do actual comparison */ }
inline bool operator> (const X& lhs, const X& rhs){return  operator< (rhs,lhs);}
inline bool operator<=(const X& lhs, const X& rhs){return !operator> (lhs,rhs);}
inline bool operator>=(const X& lhs, const X& rhs){return !operator< (lhs,rhs);}

The important thing to note here is that only two of these operators actually do anything, the others are just forwarding their arguments to either of these two to do the actual work.
The syntax for overloading the remaining binary boolean operators (||, &&) follows the rules of the comparison operators. However, it is very unlikely that you would find a reasonable use case for these2.
1 As with all rules of thumb, sometimes there might be reasons to break this one, too. If so, do not forget that the left-hand operand of the binary comparison operators, which for member functions will be *this, needs to be const, too. So a comparison operator implemented as a member function would have to have this signature:
bool operator<(const X& rhs) const { /* do actual comparison with *this */ }

(Note the const at the end.)
2 It should be noted that the built-in version of || and && use shortcut semantics. While the user defined ones (because they are syntactic sugar for method calls) do not use shortcut semantics. User will expect these operators to have shortcut semantics, and their code may depend on it, Therefore it is highly advised NEVER to define them.
Arithmetic Operators
Unary arithmetic operators
The unary increment and decrement operators come in both prefix and postfix flavor. To tell one from the other, the postfix variants take an additional dummy int argument. If you overload increment or decrement, be sure to always implement both prefix and postfix versions.
Here is the canonical implementation of increment, decrement follows the same rules:
class X {
  X& operator++()
  {
	// do actual increment
	return *this;
  }
  X operator++(int)
  {
	X tmp(*this);
	operator++();
	return tmp;
  }
};

Note that the postfix variant is implemented in terms of prefix. Also note that postfix does an extra copy.2
Overloading unary minus and plus is not very common and probably best avoided. If needed, they should probably be overloaded as member functions.
2 Also note that the postfix variant does more work and is therefore less efficient to use than the prefix variant. This is a good reason to generally prefer prefix increment over postfix increment. While compilers can usually optimize away the additional work of postfix increment for built-in types, they might not be able to do the same for user-defined types (which could be something as innocently looking as a list iterator). Once you got used to do i++, it becomes very hard to remember to do ++i instead when i is not of a built-in type (plus you'd have to change code when changing a type), so it is better to make a habit of always using prefix increment, unless postfix is explicitly needed.
Binary arithmetic operators
For the binary arithmetic operators, do not forget to obey the third basic rule operator overloading: If you provide +, also provide +=, if you provide -, do not omit -=, etc. Andrew Koenig is said to have been the first to observe that the compound assignment operators can be used as a base for their non-compound counterparts. That is, operator + is implemented in terms of +=, - is implemented in terms of -= etc.
According to our rules of thumb, + and its companions should be non-members, while their compound assignment counterparts (+= etc.), changing their left argument, should be a member. Here is the exemplary code for += and +; the other binary arithmetic operators should be implemented in the same way:
class X {
  X& operator+=(const X& rhs)
  {
	// actual addition of rhs to *this
	return *this;
  }
};
inline X operator+(X lhs, const X& rhs)
{
  lhs += rhs;
  return lhs;
}

operator+= returns its result per reference, while operator+ returns a copy of its result. Of course, returning a reference is usually more efficient than returning a copy, but in the case of operator+, there is no way around the copying. When you write a + b, you expect the result to be a new value, which is why operator+ has to return a new value.3
Also note that operator+ takes its left operand by copy rather than by const reference. The reason for this is the same as the reason giving for operator= taking its argument per copy.
The bit manipulation operators ~ & | ^ << >> should be implemented in the same way as the arithmetic operators. However, (except for overloading << and >> for output and input) there are very few reasonable use cases for overloading these.
3 Again, the lesson to be taken from this is that a += b is, in general, more efficient than a + b and should be preferred if possible.
Array Subscripting
The array subscript operator is a binary operator which must be implemented as a class member. It is used for container-like types that allow access to their data elements by a key.
The canonical form of providing these is this:
class X {
		value_type& operator[](index_type idx);
  const value_type& operator[](index_type idx) const;
  // ...
};

Unless you do not want users of your class to be able to change data elements returned by operator[] (in which case you can omit the non-const variant), you should always provide both variants of the operator.
If value_type is known to refer to a built-in type, the const variant of the operator should better return a copy instead of a const reference:
class X {
  value_type& operator[](index_type idx);
  value_type  operator[](index_type idx) const;
  // ...
};

Operators for Pointer-like Types
For defining your own iterators or smart pointers, you have to overload the unary prefix dereference operator * and the binary infix pointer member access operator ->:
class my_ptr {
		value_type& operator*();
  const value_type& operator*() const;
		value_type* operator->();
  const value_type* operator->() const;
};

Note that these, too, will almost always need both a const and a non-const version.
For the -> operator, if value_type is of class (or struct or union) type, another operator->() is called recursively, until an operator->() returns a value of non-class type.
The unary address-of operator should never be overloaded.
For operator->*() see this question. It's rarely used and thus rarely ever overloaded. In fact, even iterators do not overload it.

Continue to Conversion Operators

===>
What are the basic rules and idioms for operator overloading?

The Three Basic Rules of Operator Overloading in C++
When it comes to operator overloading in C++, there are three basic rules you should follow. As with all such rules, there are indeed exceptions. Sometimes people have deviated from them and the outcome was not bad code, but such positive deviations are few and far between. At the very least, 99 out of 100 such deviations I have seen were unjustified. However, it might just as well have been 999 out of 1000. So you’d better stick to the following rules.

Whenever the meaning of an operator is not obviously clear and undisputed, it should not be overloaded. Instead, provide a function with a well-chosen name.
Basically, the first and foremost rule for overloading operators, at its very heart, says: Don’t do it. That might seem strange, because there is a lot to be known about operator overloading and so a lot of articles, book chapters, and other texts deal with all this. But despite this seemingly obvious evidence, there are only a surprisingly few cases where operator overloading is appropriate. The reason is that actually it is hard to understand the semantics behind the application of an operator unless the use of the operator in the application domain is well known and undisputed. Contrary to popular belief, this is hardly ever the case.
Always stick to the operator’s well-known semantics.
C++ poses no limitations on the semantics of overloaded operators. Your compiler will happily accept code that implements the binary + operator to subtract from its right operand. However, the users of such an operator would never suspect the expression a + b to subtract a from b. Of course, this supposes that the semantics of the operator in the application domain is undisputed.
Always provide all out of a set of related operations.
Operators are related to each other and to other operations. If your type supports a + b, users will expect to be able to call a += b, too. If it supports prefix increment ++a, they will expect a++ to work as well. If they can check whether a < b, they will most certainly expect to also to be able to check whether a > b. If they can copy-construct your type, they expect assignment to work as well.


Continue to The Decision between Member and Non-member.

===>
What are the basic rules and idioms for operator overloading?

The General Syntax of operator overloading in C++
You cannot change the meaning of operators for built-in types in C++, operators can only be overloaded for user-defined types1. That is, at least one of the operands has to be of a user-defined type. As with other overloaded functions, operators can be overloaded for a certain set of parameters only once.
Not all operators can be overloaded in C++. Among the operators that cannot be overloaded are: . :: sizeof typeid .* and the only ternary operator in C++, ?:
Among the operators that can be overloaded in C++ are these:

arithmetic operators: + - * / % and += -= *= /= %= (all binary infix); + - (unary prefix); ++ -- (unary prefix and postfix)
bit manipulation: & | ^ << >> and &= |= ^= <<= >>= (all binary infix); ~ (unary prefix)
boolean algebra: == != < > <= >= || && (all binary infix); ! (unary prefix)
memory management: new new[] delete delete[]
implicit conversion operators
miscellany: = [] -> ->* ,  (all binary infix); * & (all unary prefix) () (function call, n-ary infix)

However, the fact that you can overload all of these does not mean you should do so. See the basic rules of operator overloading.
In C++, operators are overloaded in the form of functions with special names. As with other functions, overloaded operators can generally be implemented either as a member function of their left operand's type or as non-member functions. Whether you are free to choose or bound to use either one depends on several criteria.2 A unary operator @3, applied to an object x, is invoked either as operator@(x) or as x.operator@(). A binary infix operator @, applied to the objects x and y, is called either as operator@(x,y) or as x.operator@(y).4
Operators that are implemented as non-member functions are sometimes friend of their operand’s type.
1 The term “user-defined” might be slightly misleading. C++ makes the distinction between built-in types and user-defined types. To the former belong for example int, char, and double; to the latter belong all struct, class, union, and enum types, including those from the standard library, even though they are not, as such, defined by users.
2 This is covered in a later part of this FAQ.
3 The @ is not a valid operator in C++ which is why I use it as a placeholder.
4 The only ternary operator in C++ cannot be overloaded and the only n-ary operator must always be implemented as a member function.

Continue to The Three Basic Rules of Operator Overloading in C++.

------------------------------
What is the copy-and-swap idiom?

Stopped by user!

Done!
```
