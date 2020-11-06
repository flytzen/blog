---
layout: post
title: Bitwise shift - what now?
date: '2019-03-06'
author: Frans Lytzen
tags: csharp
modified_time: '2019-03-06'
excerpt: TBD.
---
One of things I have never really got my head around in C# is bit-shifting; I sort of know what they do, but I have never really been able to understand how to use it in practice. If that sounds like you, then this post is for you. Consider it a gentle introduction to understanding bit-shifting so you can go on to other, greater things.

The trigger for this post came when I tried to read [this post about performance improvements with strings](https://www.meziantou.net/2019/03/04/some-performance-tricks-with-net-strings).

I couldn't understand what this code was doing:
```csharp
buffer[0] = s_encode32Chars[(int)(id >> 60) & 31];
buffer[1] = s_encode32Chars[(int)(id >> 55) & 31];
```
It's not really got anything to do with what the article was about, but it really annoyed me that I couldn't follow the code example, so I made it my mission to figure it out (i.e. ask my friend Phil who explained it to me).

In short, that code is part of some code that chops the bits from a `long` into 5-bit long segments, converts those five bits to an integer and then uses that to look up a character in another string to use for encoding it. I'll write a simple version that uses 4 bits later in this post, i.e. to convert a number to a hex string (in a stupid way, just to show the principle :).

*Note: I have used [LinqPad](https://www.linqpad.net/) for the examples, so when you see special syntax like `.Dump()` in the examples then it just means I'm telling LinqPad to write the value to the console.*

# Key operators
Just as a reminder, everything internally is stored as binary, i.e. the number `3` in binary is written as `11`. For various reasons, we almost always work with `bytes`, which is a group of 8 bytes. One byte can have 2^8 different values, for example 0-255 or -127 - +127.
In C# we have a number of "integral" types (meaning they store whole numbers), including `byte` which are one byte long, `short` which is two bytes long, `int` which is four bytes long and `long` which is 8 bytes long.

There are also floating point numbers such as `float` and `double` types which work a bit differently - I will ignore them here.

## Signed vs unsigned
`sbyte`, `short`, `int` and `long` are **signed** meaning the left-most bit is reserved for whether it's a positive or negative number.  
`byte`, `ushort`, `uint` and `ulong` are **unsigned** meaning they can't store negative numbers. Note the inconsistent naming convention for `byte`, which was chosen as in practice it's probably the most intuitive. 

Using `short` as an example, this leaves only 15 bits for the value so a `short` can store a value from -32,768 to +32,767. The reason you can go one "higher" in the negative numbers may be slightly counter intuitive.

| Value | Binary          |
|-------|-----------------|
| 32,767|01111111 11111111|
|      1|00000000 00000001|
|      0|00000000 00000000|
|     -1|11111111 11111111|
|     -2|11111111 11111110|
|-32,767|10000000 00000001|
|-32,768|10000000 00000000|

As you can probably tell, when you get into negative numbers, it all gets a bit more complicated - it's more like "if the left-most bit is set, take the value of the other 15 bits and add them to -32,768". This is markedly different from how we normally think about negative numbers in the everyday world and I shall ignore it for the rest of this post.  
`ushort`, `uint` etc don't have the left-most bit used for the sign so only allows positive numbers; a `ushort` can store values from 0 to 65,535 (i.e. 0 to 11111111_11111111).

## Working with binary values in C#
A lot of the head-hurting arrives when you want to think about bit values but you have to work with numbers.  
For example, say you want to think about a byte like `10011001`. In decimal that happens to be 153 (again, ignoring signing).  
C# allows you to write code like `uint x = 0b_10011001;` which may make more sense than `uint x = 153;`. You can write multi-bytes like this: `uint x = 0b_10011001_11001100;`


## And (&)
A bitwise AND will return a 1 when *both* the input values have a 1. For example:
```
  00110011 &
  10001101
= 00000001
```

## Or (|)
A bitwise OR will return a 1 when *either or both* the input values are 1. For example:
```
  00110011 |
  10001101
= 10111111
```

## Shift (<< and >>)
A *shift* allows you to effectively "shift" the numbers left or right. For example:
```
  00001100 << 1
= 00011000

  00001100 >> 2
= 00000011
```

## Other bit operators
There are some other bit operators, like XOR and NOR that I will ignore here as my focus really is on shifting and the other operators are covered to help.


# Basic usecases
I am going to show a number of use cases here, where bit shifting *could* be used. In many cases it would be a bad idea, because there are better alternatives, and the code examples are not optimised in any way, so do not use them in anything real; they are here to illustrate how it works. *When* you use it is up to your judgement.

## Masking
Okay, so *masking* is not bit-shifting, but you often need to use masking in order to utilise bit-shifting so I will explain it here. I will use "flags" as an example to illustrate.

When you have a number of options in a system that can be either `on` or `off`, bit flags are often used to conserve space. For simplicity's sake, let's assume you have 8 flags that you need to control, for example switch sending of emails on or off. In most cases, you probably want to use some proper configuration so that your code is readable. If you want to use bit flags, you should (at least in C#) [use Enums with the `Flags` attribute](https://docs.microsoft.com/en-us/dotnet/api/system.flagsattribute?view=netframework-4.7.2). But, if you really want to do it yourself, here is how.

A `byte` in C# is just an (unsigned) integral number type that is a single byte (8 bits) long so it can store numbers from 0 to 255. I am going to use it for this example because it's short, so it makes the examples easier.
As it consists of 8 bits you can decide that "if the first bit is set, send emails" and "if the second bit is set, use the production database" (terrible examples, I know). So what you want is to be able to read the individual bits that you want, ignoring all the other bits. That's where bit masking comes in. 

Let's say you wanted to know whether to send emails, you could do something like this:
```
byte flags = 0b_11001101;  // Set all the config options, including switching on emails. Could have been written as flags = 205 if you wanted.
var result = flags & 0b_00000001;  // Returns 00000001
var sendEmails = result == 0b_00000001; // Returns true
```
If you ever see code like this in the wild, it usually uses numbers via defined constants or enums, but this is the underlying principle.
In the same way, you can set different flags one at time like this:
```
int flags = 0;
flags = flags | 0b_00000001; // Set the send email flag
flags = flags | 0b_00000010; // Set the database flag
```

You will see that when you work with Enum Flags in C#, the above is the syntax you use. The Enums basically work as constants that replace the `0b_00000001` etc.


# Packing RGBA into an integer
**Warning: this post is not about RGBA - I do not profess to understand how to process this correctly. I am just using my limited understanding as an example. Do not use this code in anything real without first learning how RGBA should be properly handled!**
A colour code is often encoded as RGBA (Red, Green, Blue, Alpha). The alpha value is a transparency value. Each of the three parts gets assigned a number from 0 to 255 - or 0 to FF in hex. Note: When working with the RGBA, you will often specify the alpha as a number between 0 and 1, such as 0.5 for 50% transparent. As far as I can tell, it's still stored as a number between 0 and 255 with 50.2% = 128.
For example, the banner on this blog is `#485B5A` in hex or R=72, G=91, B=90 in decimal (there is no alpha value so it defaults to 1) You can store the colour value in a string as hex - that will take up 8 bytes. You can store the four different values as individual `byte`s in a Struct. Or you can pack them all into a single integer, which will take up 4 bytes. That last thing seems to be [the canonical way](https://en.wikipedia.org/wiki/RGBA_color_space) and is the example I will use here; Basically taking four individual numbers from 0-255 and packing them into a single 4-byte integer.

```csharp
byte red = 200;   // 11001000
byte green = 100; // 01100100
byte blue = 50;   // 00110010
byte alpha = 127; // 01111111

uint colourAsInt = (uint)(red << 24 | green << 16 | blue << 8 | alpha);
var result = Convert.ToString(colourAsInt, 2); // Just converting the binary value to a string so you can see the result
colourAsInt.Dump(); // 3362009727
result.Dump(); // 11001000011001000011001001111111
```

This is what the shifting is doing to each variable:
```
red   << 24 = 11001000 00000000 00000000 00000000
green << 16 = 00000000 01100100 00000000 00000000
blue  <<  8 = 00000000 00000000 00110010 00000000
alpha       = 00000000 00000000 00000000 01111111
```
You should be able to see that when you OR the four variables together, the result ends up being `11001000 01100100 00110010 01111111`.

Note that the thing about `uint` is not strictly necessary here but it makes it a little bit easier to think about.

In the real world, sometimes the colour values are different on different platforms, i.e. ARGB or RGBA and there are scenarios where little endian and big endian comes into play - it should be obvious how you can use bit-shifting to easily convert between the different formats. See the above article for more information about the various layouts.

## Getting the data back out again
In order to get the data into individual variables again, we need to combine shifting and masking.  
Basically, we need to shift the bits we want to the right-most 8 bits and then use a mask to only get that data. 

Starting with the alpha value - that's already in the right-most 8 bits, so we can just do:
```csharp
uint alpha2 = colourAsInt & 0b_11111111;
```
i.e.:
```
  11001000 01100100 00110010 01111111 & 
  00000000 00000000 00000000 11111111
= 00000000 00000000 00000000 01111111
= 127
```

To get the blue value;
```
uint blue = (colourAsInt >> 8) & 0b_11111111;
```
i.e.
```
  11001000 01100100 00110010 01111111 >> 8
= 00000000 11001000 01100100 00110010


  00000000 11001000 01100100 00110010 & 
  00000000 00000000 00000000 11111111
= 00000000 00000000 00000000 00110010
= 50
```

and so on...

# Math
You can do some basic kinds of math with bit-shifting, such as doubling and (sort-of) halving etc. There are, apparently, situations where that is a good idea - I'll leave it to you to find them.

# Hashcodes

# Converting a number to base-16 (hex)
The article that finally got me to look at this was optimising a function in .Net to convert a number to a base-32 *string*. In order to make this easier to understand and test, I have decided to try to write a function to convert a number to a base-16 (hex) string. Don't do this in the real world; it's silly - but it illustrates how you can use bitwise shifting to do something real. You could easily do the same for converting strings to base-64 [look at the .net framework version? try it myself?].




Look at the hashcode generator functions?

# Base 64


# What was the original code about?

# Here be dragons
There are many subtleties to bitshifting, including (apparently) some unintuitive wrapping behaviour, different behaviour for signed/unsigned and so on. I don't profess to understand those fully and it was not the intention of this post to cover it. If you find yourself using bitshifting for real, it's work checking it out properly and writing tests for your edge cases.