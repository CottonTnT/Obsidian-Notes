> [原文]([The String Formatters of Manor Farm](http://www.gotw.ca/publications/mill19.htm)) , 


**Keep in mind the larger picture: We're interested in looking at how we would normally choose to format nonstring values as strings in the general case**

	

Consider the following C code that uses sprintf() to *convert an integer value to a human-readable string representation*, perhaps for output on a report or in a GUI window:

```c
void PrettyFormat( int i, char* buf )  
{  
  // Here's the code, neat and simple:  
  sprintf( buf, "%4d", i );
  }
```
The  question is: How would you *do this kind of thing in C++*?

Well, all right, that's not quite the question because, after all, Example 1 is valid C++. The true question is: Throwing off the shackles of the C90 standard [2] on which the C++98 standard [3] is based, if indeed they are shackles, isn't there a superior way to do this in C++ with its classes and templates and so forth?


That's where the question gets interesting, because Example 1 is the first of no fewer than four direct, distinct, and standard ways to accomplish this task. Each of the four ways offers a different tradeoff among *clarity, type safety, runtime safety, and efficiency*. Moreover, all four choices are standard, but some are more standard than others - and, to add insult to injury, not all of them are from the same standard. They are, *in the order I'll discuss them*:
``` C++
sprintf() [2], [3], [4]  
snprintf() [4]  
std::stringstream [3]  
std::strstream [3]
```

Finally, as though that's not enough, there's a fifth not-yet-standard-but-liable-to-become-standard alternative for simple conversions that don't require special formatting:
```c++
boost:lexical_cast[5]
```

Enough chat; let's dig in.

# 1 Option #1: The Joys and Sorrows of sprintf()


`sprintf()` has *two major advantages* and *three distinct disadvantages*. The two advantages are as follows:

_Issue #1: Ease of use and clarity._ Once you've learned the commonly used formatting flags and their combinations, using sprintf() is succinct and obvious, not convoluted. It says directly and concisely what needs to be said. For this, the printf() family is hard to beat in most text formatting work. (True, most of us still have to look up the more rarely-used formatting flags, but they are after all used rarely.)

_Issue #2: Maximum efficiency (ability to directly use existing buffers)._ By using `sprintf()` to put the result directly into an already-provided buffer, `PrettyFormat()` gets the job done without needing to perform any dynamic memory allocations or other extra off-to-the-side work. It's given an already-allocated place to put the output and puts the result directly there.


Caveat lector(读者当留心（拉丁语)): Of course, don't put too much weight on efficiency just yet; your application may well not notice the difference. *Never optimize prematurely, but optimize only when timings show that you really need to do so*. *Write for clarity first, and for speed later if necessary*. In this case, never forget that the efficiency comes at the price of memory management encapsulation -Issue #2 is phrased here as "you get to do your own memory management," but the flip side is "you have to do your own memory management"!


Alas, as most `sprintf()` users know, the story doesn't end quite there. `sprintf()` also has these significant drawbacks:

_Issue #3: Length safety._ Using `sprintf()` is a common source of **buffer overrun errors** if the destination buffer doesn't happen to be big enough for the whole output. For example, consider this calling code:
```c++
char buf[5];  
int value = 42;  
PrettyFormat( value, buf ); // er, well, sort of okay  
assert( value == 42 );

char buf[5];  
int value = 12108642;  
PrettyFormat( value, buf ); // oops  
assert( value == 12108642 ); // likely to fail
```