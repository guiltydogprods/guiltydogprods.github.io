---
layout: post
title: "Reality Engine - The tech behind Air Hockey Arcade."
date: 2020-10-10
---

A few of months ago I introduced my new game Air Hockey Arcade via the video linked in my previous [blog post](https://guiltydogprods.github.io/blog/2020/08/05/introducing-air-hockey-arcade). More recently I posted a video showing it running on Oculus Quest after finally figuring out how to get gamma correction working correctly on Quest (see https://youtu.be/ZB-xahYmvZQ).

In this post, I'm going to write a bit about the tech behind Air Hockey Arcade, my new C99 based game engine focused purely at VR, the not so imaginatively named 'Reality Engine'.



**Reality Engine**

Reality Engine was born out of a small experiment exploring C99 / C11.  I've always had a soft spot for C since first learning it at college in the early 90, and was dragged kicking and screaming into the C++ camp when working at Sony Studio Liverpool.  I find modern C++ pretty hideous and have wanted to switch away for some time.  The original iOS game that Air Hockey Arcade was built from was written in C too. Technically it was C99 as I used the 'restrict' keyword a bit that I'd first learnt about from this article by Mike Acton [Demystifying The Restrict Keyword](https://cellperformance.beyond3d.com/articles/2006/05/demystifying-the-restrict-keyword.html).

However, it wasn't until I read a couple of great blog posts by Andre Weissflog (@FlohOfWoe on Twitter) [One year of C](https://floooh.github.io/2018/06/02/one-year-of-c.html) and [Modern C for C++ Peeps](https://floooh.github.io/2019/09/27/modern-c-for-cpp-peeps.html) and several other folks posting about their experiences on Twitter that I decided it was time to take another look at it properly.

You can find my early experiment with C99 & Clang here [SublimeClang](https://github.com/guiltydogprods/SublimeClang).



**Extended Reality**

Prior to my SublimeClang project I'd pretty much been using Visual Studio exclusively since I switched away from OS X.  As Visual Studio's C compiler doesn't support C99, I had to look elsewhere.

Clang does support C99 and I'd used it a fair bit already on PS4.  I knew it was pretty much supported everywhere I wanted to be (PC, PS4 & Android, basically all the places VR is pretty strong) so it was pretty much a no brainer to switch to it across the board.  Not that I had much choice if I wanted to switch to C on Windows. At the time I didn't even know I could use clang-cl directly from Visual Studio so I was using Sublime Text 3 as my editor and building from the command line, hence the project name SublimeClang.

SublimeClang also used some of the extensions to C/C++ that Clang supports.  

Vector Extensions which allows me to treat 2, 3 or 4 element vectors as native types.  C++ obviously has support for this built in and it's one of the things I felt I might miss going back to C.

So for example, you can create a 'vec4' type, as in GLSL by adding the following somewhere in your code:

``` c
typedef float vec4 __attribute__((ext_vector_type(4)));
```

then use as follows
``` c
	vec4 res = v2 + v1 * scale;
```

which is so much nicer than
``` c
	vec4 v1, v2;
	float scale;
	vec res = vec4_add(v2, vec4_scale(v1, scale));
```

You still need to use SIMD intrinsics for more complex operations but thats pretty much the base with GLSL / HLSL too.

There is a similar extension for Matrix types on the way in Clang 12 but for now I build matrices from a struct of vec4s i.e.

``` c
typedef struct mat4x4
{
	union
	{
		struct
		{
			vec4 xAxis;
			vec4 yAxis;
			vec4 zAxis;
			vec4 wAxis;
		};
		vec4 axis[4];
	};
} mat4x4 __attribute__ ((aligned(16)));
```

I'm also potentially going to use Apple's 'Blocks' extension, which gives me lambda expressions in C and will allow me to do libDispatch style multi-threading: 

``` c
dispatch_async( concurrentQueue, ^{
	// Do some stuff that can be done on any thread.
  dispatch_async( renderQueue, ^{
  	// Do some stuff that has to be done on the render thread.
  });
});
``` 

