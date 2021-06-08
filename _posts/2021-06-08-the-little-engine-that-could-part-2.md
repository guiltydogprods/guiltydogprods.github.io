---
layout: post
title: "The little engine that could: Part 2"
date: 2021-06-08
---

# Rendering

**Preamble**

I seemed to pick up a few more twitter followers after my last [post](https://guiltydogprods.github.io/blog/2021/05/12/the-little-engine-that-could-reality-engine), and I got some likes and retweets off some industry folks I have a lot of respect for which was nice :)  That doesn't mean it was any good, perhaps they just felt pity on me ;) regardless it encouraged me to have a go at a part 2 on Rendering.  For the record, I began this post on 14th May 2021, and planned to release it on 19th May 2021. For anyone who may have seen any of my previous posts, my record isn't very good on this and as you can see the 19th came and went so now as I write this, I'm aiming for the 26th, which is tomorrow...

...missed it again didn't I :) 

**RGFX**

So 'rgfx' (Reality Graphics) is the prefix I use in place of a namespace in my codebase.  In the last post I explained Reality Engine is coded in C99 which doesn't support namespaces, so all functions and structs etc. are of the form:

``` c
	rgfx_'function' or rgfx_'struct'
```

I'm not entirely happy with my choice of conventions here because there is often no clear differentiator between, structures and function names at this point, but I'll stick with if for now.  I'm the only person working with this for now, and it's not causing me any major problems, it would just get in the way of getting Air Hockey Arcade finished.  Rule #1. Make it work!  If / when I get to a point I feel I could think about open sourcing Reality Engine, I will look into addressing this.

RGFX at it's core is intended to be a thin wrapper on top of whatever Graphics API is running under the hood.  Currently this means OpenGL ES 3.2 on Oculus Quest and OpenGL 4.6 on PC.  Assuming I haven't missed an announcement, these are still the latest versions of OpenGL(ES) available at this time.

The reasons I'm wrapping the underlying Graphics API and not using it directly are two fold:

1. I'll almost certainly want to switch to Vulkan and some point.  For those that don't know, Vulkan is the latest cross platform Graphics open standard by Khronos who also manage the OpenGL standard.  Vulkan is a modern low level graphics API compared to OpenGL, whose roots go back to 1992.  Vulkan is however, a lot more comlicated than OpenGL and while I have enjoyed working with Vulkan to this point, I have a lot more experience with OpenGL and so I was able to get things up and running a lot quicker than had I decided on Vulkan from the start.

2. Some platforms that I'm interested in targetting, namely PlayStation 4 for PSVR, don't support OpenGL or Vulkan sadly and only provide their own custom graphics API.  Based on past experience, I really don't see that situation changing.  

RANT: I would really like Sony to support Open Standards, in fact when I worked for PlayStation R&D I tried to get them to, but to no avail. Without Open Standards, it becomes a lot more work for an independent developer like me to support a platform.  At least with Apple (who also don't support Vulkan and have deprecated OpenGL) there is a solution in the form of MoltenVK, which is an implementation of Vulkan running on top of Apple's Metal API.  It's not perfect but it works well for me.  When I left PlayStation R&D at the end of 2018, I really considered trying to do something similar to MoltenVK for PS4. However with PS5 on the way, I didn't know if Sony would finally make the decision to properly support Open Standards or not so it would have been a risky decision at the time.  It doesn't look like they have so maybe at some point.

This post isn't intended to be a tutorial on OpenGL, so if you are looking for a resource on learning OpenGL, I can wholeheartedly recommend: [Learn OpenGL](https://learnopengl.com) by Joey de Vries (@JoeyDeVriez).

**GPU Instancing and Batching**

As Oculus Quest was my primary focus for RGFX, and at the time all the talk was about drawcalls in OpenGL being expensive and a limiting factor on Mobile GPUs which includes the GPU on Oculus Quest, I designed RGFX around GPU Instancing.

GPU Instancing is a mechanism within most graphics APIs these days, to draw multiple copies of the same model in a single draw call. So instead of doing something like the following:

``` c
for (int32_t i = 0; i < instanceCount; i++)
{
    BindStuffPerModel();												// e.g. Bind Per Model matrices. 
    glDrawElements(GL_TRIANGLES, count, GL_UNSIGNED_SHORT, indices);	// Draw the model.
}
```

You would put all the models transforms into a GPU accessible buffer and then do something like:

``` c
BindMatricesBuffer();
glDrawElementsInstanced(GL_TRIANGLES, count, GL_UNSIGNED_SHORT, indices, instanceCount);
```

Inside your shader, you can then use the builtin GLSL variable **gl_InstanceID** to index into the matrices buffer to obtain the matrix for the instance currently being drawn.

You can also use **gl_InstanceID** to look up various other per model data, for example, per instance material information could be stored in an another buffer, you can even index different textures via a texture array, or more directly using bindless textures on regular OpenGL (bindless textures are not supported in OpenGLES sadly).  I currently use this feature for the pictures on the wall in Air Hockey Arcade, and also for the baked Ambient Occlusion textures on the walls and floor.  More on that later.

