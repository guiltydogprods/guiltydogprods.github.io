---
layout: post
title: "Reality Engine - The tech behind Air Hockey Arcade."
date: 2020-10-10
---

A couple of months ago I introduced my new game Air Hockey Arcade via the video linked in my previous [blog post](https://guiltydogprods.github.io/blog/2020/08/05/introducing-air-hockey-arcade). This week I posted a video publicly showing it running on Oculus Quest for the first time after finally figuring out how to get gamma correction working correctly on Quest (see https://youtu.be/ZB-xahYmvZQ).

In this post, I'm going to write a bit about the tech behind Air Hockey Arcade, my new C99 based game engine focused purely at VR, the not so imaginatively named 'Reality Engine'.

**Reality Engine**

Reality Engine was born out of a small experiment exploring C99 / C11.  After years of becoming more and more dissatisfied with C++, I'd wanted a new language for some time. But ideally it would be a language that was at least based on C, as I have a lot of experiencethan to return to C for a while. The original iOS game that Air Hockey Arcade was built from was written in C too. Technically it was C99 as I used the 'restrict' keyword after reading an article about it written by Mike Acton (ex TD at Insomniac Games, now heading up the DOTS team at Unity) when I was working at Sony Studio Liverpool, but I think that was pretty much all I really used from the new C standard at the time.

After reading a couple of great blog posts by Andre Weissflog (@FlohOfWoe on Twitter) [One year of C](https://floooh.github.io/2018/06/02/one-year-of-c.html) and [Modern C for C++ Peeps](https://floooh.github.io/2019/09/27/modern-c-for-cpp-peeps.html) and several other folks posting about their experiences on Twitter convinced me it was time to take another look.

You can find my early experiment with C99 & Clang [here](https://github.com/guiltydogprods/SublimeClang).

**Extended Reality**

Prior to my SublimeClang project I'd pretty much been using Visual Studio exclusively since I switched away from OS X.  As Visual Studio's C compiler doesn't support C99, I had to look elsewhere.

Clang does support C99 and I'd used it a fair bit already on PS4.  I knew it was pretty much supported all the places I wanted to be (PC, PS4 & Android, basically all the places VR is pretty strong) so it was pretty much a no brainer to switch to it across the board, not that I had much choice it I wanted to switch to C on Windows.

Over the years when I'd fantasised about moving away from C++ back to C, I'd often think about the things I'd miss from C++.  Like many coders in the Games Industry, I don't hate all of C++:

• Classes can be nice way to encapsulate data and functions that operate on that data. When it becomes the 'only' way, is when it beomes a problem.
• Operator overloading is nice for math functions. 
• Lamdas, yeah I quite like them.


So as much as I never minded 
``` 
	typedef struct vec4 
	{
		float x, y, z, w;
	} vec4;

	vec4 v1, v2, res;
	void vec4_add(vec4* res, const vec4* v1,  const vec4* v2);


```

or even
``` C 
	vec4 v1, v2;
	vec4 vec4_add(vec4 v1, vec4 v2);
```

There is pretty much no denying that the following C++ can be much nicer to work with.

``` C++
	struct vec4
	{
		vec4 operator+(const vec4& right) const;
	private:
		float m_x, m_y, m_z, m_w;
	};

	vec4 v1, v2;
	vec4 res = v1 + v2;
```

Especitally when doing several operations at once.

This
``` C
	vec4 res = v2 + v1 * scale;
```

is so much nicer than

``` C
	vec4 v1, v2;
	float scale;
	vec res = vec4_add(v2, vec4_scale(v1, scale));
```

So in C, when it comes to vector math and operator overloading are we out of luck?  Well yes and no...  

C itself, even the latest standard, doesn't support Vector types. However, it just so happens that Clang supports a number of extensions to standard C based languages including [Vector Extensions](https://clang.llvm.org/docs/LanguageExtensions.html#vectors-and-extended-vectors)

The Vector Extensions enable you to define and use Vector types as follows. 

``` C
typedef float vec4 __attribute__((ext_vector_type(4)));	

vec4 v1 = { 1.0f, 2.0f, 3.0f, 4.0f };
vec4 v2 = { 4.0f, 3.0f, 2.0f, 1.0f };

vec4 res = v1 + v2;

//res will end up with the values { 5.0, 5.0f, 5.0f 5.0f } as you would expect
```

The beauty of Clang's Vector Extensions are they automatically use SIMD instructions at least on Intel and ARM CPUs.

