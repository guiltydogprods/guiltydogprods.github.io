---
layout: post
title: "The little engine that could: Reality Engine!"
date: 2021-05-12
---

What would have been a few months ago when I first started this post, and is now getting on for a year ago (I'm not much of a blogger to be fair), I introduced my current game Air Hockey Arcade via the video linked in my previous [blog post](https://guiltydogprods.github.io/blog/2020/08/05/introducing-air-hockey-arcade). 

In this post, I'm going to write a bit about the tech behind Air Hockey Arcade, my new C99 based game engine focused purely on VR, the not so imaginatively named 'Reality Engine'.


**Reality Engine**

Reality Engine was born out of a small experiment exploring C99 / C11.  I've always had a soft spot for C since first learning it at college in the early 90, and was dragged kicking and screaming into the C++ camp when working at Sony Studio Liverpool.  I find modern C++ pretty hideous and have wanted to switch away for some time.  The original iOS game that Air Hockey Arcade was built from was written in C too, and I always felt that for some reason I was a lot more productive when I stuck to pure C.

However, it wasn't until I read a couple of great blog posts by Andre Weissflog (@FlohOfWoe on Twitter) [One year of C](https://floooh.github.io/2018/06/02/one-year-of-c.html) and [Modern C for C++ Peeps](https://floooh.github.io/2019/09/27/modern-c-for-cpp-peeps.html) and several other folks posting about their experiences on Twitter that I decided it was time to take another look at it properly.

You can find my early experiment with C99 & Clang here [SublimeClang](https://github.com/guiltydogprods/SublimeClang).  I can't promise it'll still build :)


**Extended Reality**

Prior to my SublimeClang project I'd pretty much been using Visual Studio exclusively since I switched away from OS X.  As Visual Studio's C compiler doesn't support C99, I had to look elsewhere.

Clang does support C99 and I'd used it a fair bit already on PS4.  I knew it was pretty much supported everywhere I wanted to be (PC, PS4 & Android, basically all the places VR is pretty strong) so it was pretty much a no brainer to switch to it across the board.  Not that I had much choice if I wanted to switch to C on Windows. At the time I didn't even know I could use clang-cl directly from Visual Studio so I was using Sublime Text 3 as my editor and building from the command line, hence the project name SublimeClang.

SublimeClang also used some of the extensions to C/C++ that Clang supports.  

Vector Extensions which allows me to treat 2, 3 or 4 element vectors as native types.  C++ obviously has support for this built in via classes and operator overloading and it's one of the things I felt I might miss going back to C.

So for example, you can create a 'vec4' type, similar to the native GLSL type by adding the following somewhere in your code:

``` c
typedef float vec4 __attribute__((ext_vector_type(4)));
```

then use as follows
``` c
	vec4 res = v2 + v1 * scale;
```

which I feel is so much nicer than
``` c
	vec4 v1, v2;
	float scale;
	vec res = vec4_add(v2, vec4_scale(v1, scale));
```

I'm not really using SIMD for performance in this case but it's more for my own convienience as C doesn't support operator overloading.

Apparently there is a similar extension for Matrix types on the way in Clang 12 but for now I build matrices from a struct of vec4s i.e.

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

I'm also using Apple's 'Blocks' extension, which gives me lambda expressions in C.  When I first started this post, I didn't really have a use for Apple blocks, but I was planning on using them for something, most likely multi-threading.  Since then I've added a lot of functionality and have found they are really quite useful for callbacks, so the engine can callback into game code.  This could of course be done via regular function pointers, and I'll likely end up supporting both options, but I personally find being able to write the callback inline as follows is really elegant.  

``` c
	rvrs.initializePlatformOvr(&(rvrs_platformDescOvr) {
		.appId = OCULUS_APP_ID,
		.entitledCallback = ^(uint64_t userId) {		// ^(...) {...} declares a block.
			rvrs.initializeAvatarOvr(OCULUS_APP_ID);
			rvrs.requestAvatarSpecOvr(userId, 0);
			s_gameData.bEntitled = true;
		}
	});
``` 

I should take a little time to explain a couple of other things that may look unusual to people unfamiliar with modern C syntax in the code snippit above.

Firstly the **rvrs.** before the initializePlatformOvr(...) function.  Here 'rvrs' is just a globally declared instance of a struct containing a bunch of function pointers as follows.

``` c
typedef struct rvrsApi
{
	bool(*isInitialized)(void);
	mat4x4(*getViewMatrixForEye)(int32_t index);
	mat4x4(*getProjectionMatrixForEye)(int32_t index);
	...
}rvrsApi;
```

It kind of gives me something akin to namespaces in C and is also a pretty handy way to pass a bunch of engine functions through to the game code which is loaded by the engine or loader as a DLL (or shared library under linux).

The other bit of syntax that may not be familar that I use all over the place is the way I often pass parameters to functions, in this case 

``` c
initializePlatformOvr(&(rvrs_platformDescOvr) {...});  
```

This, is a combination of C99 compound literals and designated initializers, I'm not 100% on the terminology in both cases. @FlohOfWoe covers this much beter than I ever could in his post [Modern C for C++ Peeps](https://floooh.github.io/2019/09/27/modern-c-for-cpp-peeps.html)

I use this syntax **a lot**, to the point that sometimes I worry that I'm over using it :). I do find it gives a similar self documenting quality to function calls as you get with Objective-C though which I like.  Time will tell if this is sensible but these features that were added in C99 really do make C seem like a new language.  

The eagle eyed among you may also note in the above code that you can use designated initializers and apple blocks to declare your callback functions inline as part of the structure initialization.  I make use of this a lot for GUI Widgets such as slider and button actions see below:

``` c
rgui.button(&(rgui_buttonDesc) {
	.x = buttonPos,
	.y = cursorPosY,
	.width = kButtonWidth,
	.height = 0.256f,
	.panel = mainPanel,
	.texture = s_gameData.tableButtonTexture,
	.layoutFlags = kAlignHCenter,
	.action = ^ (void) {
		raud.playSound(s_gameData.guiClickSound, (float*)&s_gameData.guiTransformMain.wAxis, NULL, kVolumeScale, 128);
		s_gameData.bShowCredits = false;
	}
});
```

**Any Downsides / Regrets?**

The only real downside to any of this non standard syntax I can think of is that intellisense really seems to have no idea what I'm doing and thinks my code is just a broken mess.  I should really figure out how to turn it off and get rid of all the annoying squiggles in Visual Studio.

Do I ever regret choosing to use C over C++? Not at all.  Sometimes using C++ would make my life easier.  My policy for using middleware is, I don't mind using a C++ library, if it has a pure C interface.  Not everything does, which is one of the reasons I'm not using Dear imGui. I know there is cimgui and I possibly should have used that but even so I wasn't sure how easy it would be to get imGui working in a VR app.

Saying that, writing my own immediate mode gui was actually a fun challenge and while it's nowhere near complete and will never be as fully featured as Dear imGui, it suits my needs pretty well.

My build / turn around time more than makes up for any slight annoyances, I feel that I'm always making forward progress and I'm never blocked waiting on a build, and my build times are certainly not as low as they should be but I can do a full rebuild of my game and engine on PC in slightly over 10 seconds on a 5 or 6 year old PC, so I have no desperate need to optimise the build just yet.

**Conclusion**

I've not really covered as much as I'd planned, and it's taken me 7 months to get this far!  I'm definitely a coder and not a writer and maybe that's ok.  But part of the reason than I'm writing this and in many ways why I'm making my game using my own tech, is I want to show people that you still can write your own tech and doing so can be really rewarding, not least because you learn a lot in the process but you can get loads of other benefits. 

Your project can take less time to build which for me is pretty much the most important optimisation metric, optimising for programmer time.  I got that nugget of wisdom from watching a Jon Blow talk years ago and it stuck with me.  Not only can your build take less time, but your project can be leaner and in many cases often run faster than when using an off the shelf engine because you can focus on exactly what you need for your game.  

I'm not saying that it's a quicker route to market, far from it, but an interesting observation I've had recently is I am blocked far less often by issues requiring technical support than I ever was using Unity or UE4.

I'd like to talk more about my tech and my approach but I think I may need to find a different way to do it that fits better with me, as 3 blog posts in 3 years isn't going to set the world on fire.  I'm just not sure what this better way might be.

Until next time, which hopefully won't be in another year...
