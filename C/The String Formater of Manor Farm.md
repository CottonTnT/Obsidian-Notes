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


_Issue #4: Type safety._ For `sprintf()`, type errors *are runtime errors, not compile-time errors*, and they may not even manifest right away. The `printf()` family uses C's variable argument lists, and C compilers generally *don't check the parameter types for such lists*. 


Granted, the code in Example 1 is so trivial that it's likely easy enough to maintain now when we know we're just throwing a single int at sprintf(), but even so it's not hard to go wrong if your finger happens to hit something other than "d" by mistake. For example, "c" happens to be right next to "d" on most keyboards; if we'd simply mistyped the sprintf() call as
```c++
sprintf( buf, "%4c", i ); // oops
sprintf( buf, "%4s", i ); // oops again
sprintf( buf, "%4ld", i ); // a subtler error,In this case, the format string is telling sprintf() to expect a long int, not just an int, as the first piece of data to be formatted. This too is bad C code, but the trouble is that not only won't this be a compile-time error, but it might not even be a runtime error right away. On many popular platforms, the result will still be the same as before. Why? Because on many popular platforms ints happen to have the same size and layout as longs. You may not notice this error until you port the above code to a platform where int isn't the same size as long, and even then it might not always produce incorrect output or immediate crashes.
```

_Issue #5: Templatability._ It's very hard to use sprintf() in a template. Consider:
```cpp
template<typename T>  
void PrettyFormat( T value, char* buf )  
{  
  sprintf( buf, "%/*what goes here?*/", value );
```

| Standard?                       | Yes:C90, C++98, C99 |
| :------------------------------ | :------------------ |
| Easy to use, good code clarity? | Yes                 |
| Efficient, no extra allocation? | Yes                 |
| Length safe?                    | No                  |
| Type safe?                      | No                  |
| Usable in template?             | No                  |

# 2 Option #2: snprintf()
Of the other choices, `sprintf()`'s closest relative is of course `snprintf()`. `snprintf()` only adds only one new facility to `sprintf()`, but it's an important one: *The ability to specify the maximum length of the output buffer, thereby eliminating buffer overruns*. Of course, *if the buffer is too small then the output will be truncated*.

With snprintf(), we can correctly write the length-checked version we were trying to create earlier:
```c++
// Example 2:  
// Stringizing some data in C,  
// using snprintf().  
//  
void PrettyFormat( int i, char* buf, int buflen )  
{  
  // Here's the code, neat and simple  
  // and now a lot safer:  
  snprintf( buf, buflen, "%4d", i );}
```
**Guideline: Never use sprintf().** If you decide to use C stdio facilities, always use length-checked calls like `snprintf()` even if they're only available as a nonstandard extension on your current compiler. There's no drawback, and there's real benefit, to using snprintf() instead.




|                                 | snprintf                                     | sprintf             |
| :------------------------------ | :------------------------------------------- | ------------------- |
| Standard?                       | Yes: C99 only, but will likely be in C++0x\| | Yes:C90, C++98, C99 |
| Easy to use, good code clarity? | Yes                                          | Yes                 |
| Efficient, no extra allocation? | Yes                                          | Yes                 |
| Length safe?                    | Yes                                          | No                  |
| Type safe?                      | No                                           | No                  |
| Useable in template?            | No                                           | No                  |

# 3 Option #3: std::stringstream

The most common facility in C++ for stringizing data is the `stringstream` family. Here's what Example 1 would look like using an `ostringstream` instead of `sprintf()`:
``` c++
// Example 3:  
// Stringizing some data in C++,  
// using ostringstream.  
//  
void PrettyFormat( int i, string& s )  
{  
  // Not quite as neat and simple:  
  ostringstream temp;  
  temp << setw(4) << i;  
  s = temp.str();}
```

Using `stringstream` *exchanges the advantages and disadvantages of* `sprintf()`. Where `sprintf()` shines, `stringstream` does less well:

_Issue #1: Ease of use and clarity._ Not only *has one line of code turned into three*, but we've *needed to introduce a temporary variable*. This version of the code is superior in several ways, but code clarity isn't one of them. It's not that the manipulators are hard to learn - they're as easy to learn as the sprintf() formatting flags - but that they're generally more clumsy and verbose. I find that code littered with long names like << setprecision(9) and << setw(14) all over the place *is a bear to read* (compared to, say, %14.9), even when all of the manipulators are arranged reasonably well in columns.

_Issue #2: Efficiency (ability to directly use existing buffers)._ A `stringstream` *does its work in an additional buffer off to the side*, and so will usually have to perform extra allocations for that working buffer and for any other helper objects it uses. I tried the Example 3 code on two popular current compilers and instrumented ::operator new() to count the allocations being performed. One platform performed two dynamic memory allocations, and the other performed three.

Where `sprintf()` breaks down, however, stringstream glitters:

_Issue #3: Length safety._ The stringstream's internal basic_stringbuf buffer automatically grows as needed to fit the value being stored.

_Issue #4: Type safety._ Using operator<<() and overload resolution always gets the types right, even for user-defined types that provide their own stream insertion operators. No more obscure runtime errors because of type mismatches.

_Issue #5: Templatability._ Now that the right operator<<() is automatically called, it's trivial to generalize PrettyFormat() to operate on arbitrary data types:

```c++
template<typename T>  
void PrettyFormat( T value, string& s )  
{  
  ostringstream temp;  
  temp << setw(4) << value;  
  s = temp.str();  
}
```



|                                 | stringstream | sprintf             |
| :------------------------------ | :----------- | ------------------- |
| Standard?                       | Yes: C++98   | Yes:C90, C++98, C99 |
| Easy to use, good code clarity? | No           | Yes                 |
| Efficient, no extra allocation? | No           | Yes                 |
| Length safe?                    | Yes          | No                  |
| Type safe?                      | Yes          | No                  |
| Useable in template?            | Yes          | No                  |
# 4 Option #4: std::strstream

Fairly or not, strstream is something of a *persecuted pariah*. Because it has been deprecated in the C++98 standard, ..., *So just forget it*.




# 5 Option #5: boost::lexical_cast

One of the facilities provided in the Boost libraries is `boost::lexical_cast`, which is a handy wrapper around stringstream. Indeed, Kevlin Henney's code is so concise and elegant that I can present it here in its entirety (minus workarounds for older compilers):

```c++
template<typename Target, typename Source>  
Target lexical_cast(Source arg)  
{  
  std::stringstream interpreter;  
  Target result;

  if(!(interpreter << arg) ||  
     !(interpreter >> result) ||  
     !(interpreter >> std::ws).eof())  
    throw bad_lexical_cast();

  return result;  
}
```


Note that `lexical_cast` *is not intended to be a direct competitor for the more general string formatter* `sprintf()`. Instead, `lexical_cast` is *for converting data from one streamable type to another*, and it competes more directly with C's `atoi()` et al. conversion functions as well as with the nonstandard but commonly available `itoa()` et al. functions. It's close enough, however, that it definitely would be an omission not to mention it here.

Here's what Example 1 would look like using `lexical_cast`, minus the at-least-four-character requirement:
```c++
// Example 5:  
// Stringizing some data in C++,  
// using boost::lexical_cast.  
//  
void PrettyFormat( int i, string& s )  
{  
  // Perhaps the neatest and simplest yet,  
  // if it's all you need:  
  s = lexical_cast<string>( i );  
}
```


_Issue #1: Ease of use and clarity._ This code embodies the most direct expression of intent of any of these examples.

_Issue #2: Efficiency (ability to directly use existing buffers)._ Since `lexical_cast` uses stringstream, it's no surprise that *it needs at least as many allocations as stringstream*. On one of the platforms I tried, Example 5 performed one more allocation than the plain stringstream version presented in Example 3; on the other platform, it performed no additional allocations over the plain stringstream version.


In summary, here's how `lexical_cast` compares to sprintf():


|                                 | lexical_cast | sprintf             |
| :------------------------------ | :----------- | ------------------- |
| Standard?                       | Yes: No      | Yes:C90, C++98, C99 |
| Easy to use, good code clarity? | No           | Yes                 |
| Efficient, no extra allocation? | No           | Yes                 |
| Length safe?                    | Yes          | No                  |