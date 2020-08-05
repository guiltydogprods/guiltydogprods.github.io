---
layout: post
title: "Air Hockey Arcade"
date: 2020-08-05
---

**Introduction:**

Air Hockey Arcade is a VR reworking of my iOS / PlayStation&trade; Mobile mini game Flick Hockey, that I released between 2010 and 2012.  Currently the game which is still in development runs on Oculus Rift, SteamVR and Oculus Quest. Hopefully a PSVR version will happen at some point, I just need to find the time to port my engine to PS4 which is a little trickier due to the proprietory graphics API.

Unlike the vast mojority of VR titles it seems, Air Hockey Arcade and it's underlying cross platform engine are written in C99 using just a small handful of libraries to interface with the underlying hardware.  It's entirely self funded because let's face it noone is funding VR titles these days, and even if they were it's unlikely they'd fund me on my lonesome. Oh well...

Currently I'm working on the UI and then I might be able to put it out on Early Access before I start work on Multiplayer support.


**Video:** Warning, doesn't automatically open in a new tab because Github Pages & Github Flavoured Markdown actually seem to suck :(

[![AirHockeyArcade](https://img.youtube.com/vi/b7PPZL-h5pI/0.jpg)](https://www.youtube.com/watch?v=b7PPZL-h5pI "Air Hockey Arcade")


**History:** It may be interesting to someone.

After I left EA Canada and came back to the UK in 2009 I made and self published a little iPhone game called Virtual Air Hockey.  It didn't really sell very well and I re-released it as Flick Hockey perhaps a year later via a publisher.  It did marginally better but lets face it, I still didn't make any money from it.

Undetered, sometime later when I first heard Sony were creating PlayStation&trade; Mobile and looking for launch titles, I contacted SCEE (now SIEE) Strategic Content.  After visiting with them at SCEE HQ in London, they agreed to advanced me a small amount of money to port the game to PlayStation Mobile for launch.  This meant a complete rewrite from C to C# but I had an absolute blast working with PlayStation&trade; Mobile, it really was nice to work with and a shame it's no longer active. This time Flick Hockey sold much better, it paid back the advance and also made me a little extra before Sony shut down Playstation Mobile.  I spent the next couple of years doing contract work and prototying several other game ideas both on my own and with friends but nothing ever really took off.

When Sony announced Project Morpheus (PSVR) at GDC in 2014, I got in touch with Shahid Ahmad right away asking if I could bring it to the platform for launch.  At the time they were really only interested in hardcore games for PSVR so unfortunately I didn't get the chance to do a VR version. In september that year, I re-joined Sony this time in their R&D depaartment based in London working on the PhyreEngine project. I'd applied for a graphics programmer role, but during my interview I expressed an interest in VR so my first task on PhyreEngine was integrating support for PSVR and creating a sample to demonstrate it's use. 

Sometime later after the VR integration and sample had shipped, during one of our two week Innovation sprints where we got to work on pretty much anything we wanted to as long as it was relevant to our department, I decided to spend the time porting my Air Hockey Game to run on PhyreEngine as a PSVR game.  It took me a couple of days to get the game running on PSVR at the native 120Hz using the Move Controllers for Hand Presence and right away I discovered that this was the way the game was meant to be played, second only to a physical Air Hockey Table perhaps.  I showed it to other members of the team and to various other people in R&D and it generated a bit of a buzz. 

Towards the end of 2015 our team learnt that we were to share a PlayStation VR GameEngines stand with Unity and UE4 at the PlayStation area on GDC 2016 Expo Floor. We decided to show Air Hockey VR instead of CaveRoVR which was the offcial PhyreEngine VR sample, as it was more suitable. Unfortuantely due to GDC 2016 occurring before the PSVR launch, lines to try PSVR content would have been too long, so we didn't have PSVR hardware in the end so the game wasn't playable, but we did have it running sans VR (see the image below).  We did get to show it running with PSVR hardware at an internal conference however :)

<img src="/images/booth1.jpg" width="560" height="315" />

I've tinkered with the project ever since, using it as a test subject for my own custom built engines, but I've always been too busy to get it finished. Now with my previous contract coming to an end mid pandemic, I've had a few months to get it to a point where I might be able to release it in some form very soon.

**Comments:** Errr actually don't exist because as I've already said, Github Pages & Github Flavoured Markdown actually seem to suck :(

Maybe I need to dust off wordpress for a blog and it'll be something else I forget to update :)
