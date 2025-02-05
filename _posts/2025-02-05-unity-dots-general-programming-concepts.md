---
layout: post
author: Gustavo Alberola
title: Unity's DOTS general programming concepts
description: An introduction to the general programming concepts used in Unity's Data Oriented Technology Stack (DOTS).

# caption.svg - 600 x 400
# main.png - Main caption: 616 x 353
# vertical.png - 374 x 448


hero_caption: true  # assets/post/{name}/main.png - 616 x 353
card_captions: true # assets/post/{name}/vertical.png - 374 x 448
caption: main.png   # assets/post/{name}/{caption} - 600 x 400
---

## Who is this for?

If you are **getting started** with Unity's **Data Oriented Technology Stack** (DOTS), or trying to **wrap your head around all the terminology**, and what is what, this is for you.

<div class="alert alert-success">
  🤚 This post focuses on the <b>general programming concepts</b> from which Unity's DOTS is based, <b>not</b> in Unity's implementation of them.
</div>


## Programming Concepts vs Unity's Concepts

![](/assets{{ page.url }}/intro.png)

You can think of these as the **foundation of Unity's DOTS**. So, to understand DOTS, we need to understand its foundations first.

## Data-Oriented Design

<div class="alert alert-info">
  💡 Data-Oriented Design (DOD) is an <b>optimization approach</b> to create programs that make <b>efficient use of the <code>CPU-cache</code></b>.
  <br/>
  <small>Source <a href="https://en.wikipedia.org/wiki/Data-oriented_design" target="_blank">Wikipedia</a></small>
</div>


You can think of the `CPU-Cache` as a small, but very fast memory (`RAM`) on the `CPU`.

![](/assets{{ page.url }}/cpu-cache.png)

When the `CPU` needs to fetch some data, it will **#1 Check the `CPU-cache`*** first. If the **data is present**, we get a **1.A `Cache-hit`**, and the `CPU` can resume processing.

If the **data was not found**, we get instead a **#1.B `Cache-miss`** and need to fetch the value from `RAM`.

But the **catch** is that we can't copy just an _individual value_ from `RAM`. We need to copy **#2 Copy an entire chunk of memory**.


<small>* `CPU`s have several levels of `Cache` (`L1-Cache`, `L2-Cache`, `L3-Cache`). For simplification I just draw one. But the process is similar. Check `L1`, if not check `L2`, if not check `L3`, if not check `RAM`</small>

### Why is might be a problem?

When we program using an `Object-Oriented` approach, using `MonoBehaviors`, we commonly find something like this:

```cs
public class MyButton : MonoBehavior {

  public MyDoor DoorGO;
  // ... more attributes

  private void Update() {
    // ... some code logic

    if (ifButtonWasPressed) {
      DoorGO?.OpenDoor();
    }
  }
}
```

We normally don't tend to think about **data locality** <small>--where data lives in memory--</small> much in this form.


The result is something like this:

![](/assets{{ page.url }}/cpu-cache-example-1.png)

Without having control of where data is stored, we end up **jumping around memory** quite often, resulting in a bunch of `cache-misses`.

### A solution

Now that we know how the **underlying hardware** operates, imagine we re-order the data from the previous scenario to look like this:

![](/assets{{ page.url }}/cpu-cache-example-2.png)

By arranging the elements sequentially, we incur only a **`cache-miss` on the first element**, the rest are already in the `CPU-cache` and we get a `cache-hit`, hence reducing the time it takes the `CPU` to process the data.

<div class="alert alert-info">
  🧠 <b>Concept</b> 
  <br/>
  Data-Oriented Design is about designing programs that <b>efficiently use the <code>CPU-cache</code></b>. Said otherwise, tend to <b>reduce the amount of <code>CPU-cache</code> misses</b>.
</div>

## Memory Management

If you are making games in Unity, you use C#, which has its own Memory Management System.

![](/assets{{ page.url }}/cs-memory-management.png)

The 👍 good thing is that we **don't have to think** about the underlying memory when programming.

The 👎 bad thing is that we **don't have control** about how the underlying memory is access / managed.

<div class="alert alert-danger">
  🔥 If you remember the chapter about Data-Oriented Design and <code>CPU-cache</code>, you can start seeying why this might be a problem.
  <br />
  <small>This also has implications when talking about multi-threading.</small>
</div>

### How does it work in lower-level languages (C, C++)?

When we look at languages that are **not** Garbage Collected, we usually start talking about `Pointers` to memory and manual memory allocation.

![](/assets{{ page.url }}/cpp-memory-management.png)

The 👍 good thing is that we **gain control** about how the underlying memory is access / managed.

The 👎 bad thing is that we **have un-restricted access** to memory, and we are responsible of managing this with caution, and to **manually remove** the data that is no longer in use.

<div class="alert alert-info">
  🧠 <b>Concept</b> 
  <br/>
  Data-Oriented Design requires the ability to <b>self manage memory allocation</b>, in order to place data in memory to make efficient use of the <code>CPU-cache</code>.
</div>

## Multi-threading

You probably know that `CPU`s have multiple cores. And each core is capable of doing operations at the same time.

![](/assets{{ page.url }}/multi-threading.png)

One Core is the one running the Engine (Unity), and it's known as the `main thread`. The others are known as `worker threads`.

When multi-threading there are two aspects of it:

1. Running a part of our application in a `worker thread`
2. Splitting a part of our application in several `task(s)` that can run in **parallel**.

<div class="alert alert-info">
  🧠 <b>Concept</b> 
  <br/>
  Multi-threading is about <b>running parts of your program in other <code>cores</code></b>, outside of the <code>main thread</code>, possibly even splitting them into several <code>tasks</code> that run in parallel.
</div>

<small>This is <b>easier said than done</b>. Multi-thread programming is a deep topic, and one that is quite complex at its core. But for the purpose of this post, its as far as we go here.</small>

## Compiler

![](/assets{{ page.url }}/compiler.png)

An oversimplification of the process, but the idea remains. Normally C# _code_ is _compiled_ into an **Intermediate Language (IL)***. In Unity's land, <a href="https://docs.unity3d.com/6000.0/Documentation/Manual/scripting-backends-mono.html" target="_blank">Mono</a> is also an IL.

<small>* Common Intermediate Language (CIL), formerly called Microsoft Intermediate Language (MSIL). <a href="https://en.wikipedia.org/wiki/Common_Intermediate_Language" target="_blank">Wikipedia</a>.</small>

This _code_ is later read by a _virtual machine_ and translated into platform specific _code_*.

<small>* This is also known as just-in-time (JIT) compilation. <a href="https://en.wikipedia.org/wiki/Just-in-time_compilation" target="_blank">Wikipedia</a>.</small>

### A better way

An alternative to this, is to _compile_ into _platform specific code_ once for each **target platform** we want to target our game.

And there is more 😎. By doing `Data-Oriented Design` we can leverage other compiler optimizations, like <a href="https://en.wikipedia.org/wiki/Single_instruction,_multiple_data" target="_blank">Single Instruction Multiple Data (SIMD)</a> instructions.

<div class="alert alert-secondary">
  👻 <b>Foreshadow</b> 
  <br/>
  It's outside the scope of this post, but Unity is providing us exactly this with his <a href="https://docs.unity3d.com/Manual/com.unity.burst.html" target="_blank">Burst Compiler</a>. It's amazing 🎉💃
</div>

## Entity Component System (ECS)

![](/assets{{ page.url }}/ecs.png)

An `Entity` is an `ID` that identifies a unique entity in the world.

<small>E.g. You can think of them as the equivalent of a <code>GameObject's Instance</code></small>

Each `Entity` is formed by one <small>--or more--</small> `Components`, each with it's own data.

`Systems` operate over the `Components` of `Entities` that match a given _criteria_.

<small>E.g. In the image we see a <code>System</code> called <code>MoveSystem</code> that operates over <code>Entities</code> with a <code>Location</code> and <code>Acceleration</code>.<small>

### Key differences between ECS and Object-Oriented

![](/assets{{ page.url }}/ecs-vs-oo-overview.png)

So while `Objects` tend to hold both `data` and `code` together, in `ECS`, `Components` hold the `data`, and `Systems` hold the `code`.

And in `Objects`, we tend to think in _single_ units <small>(batch operations are the exception, not the rule)</small>, like `Player.Move()`.

On the other side, `Systems` tend to operate in _batches_ of units, like "for all `Entities` with a `Location` and `Acceleration`, I will _move_ them".

### ECS and Data-Oriented Design

<div class="alert alert-info">
  🧠 <b>Concept</b> 
  <br/>
  While <code>ECS</code> does <b>not</b> require a <code>Data-Orieted Design</code> per se, <b>it pairs really well</b> with it.
</div>


By separating `data` from `code`, we can store `data` taking into consideration _data locality_.

By having `Systems` operating over batch of `Entities` at a time based on a _criteria_ <small>--<code>Components</code> present in an <code>Entity</code>--</small>, we can have several `Systems` instead of dealing with _inheritance_.

This <small>--not dealing with <i>inheritance</i>--</small> has an positive impact on the `CPU-cache`, but not just on the _data_, but also on the _code_.

<small>Yes, <code>CPU-cache</code> also affect your <i>code</i>. When you have <a href="https://en.wikipedia.org/wiki/Polymorphism_(computer_science)" target="_blank">Polymorphism</a>, you are likely incurring <code>CPU-cache</code> misses.</small>

# Next steps

**I wrote this because** when I started with Unity's DOTS I was a bit lost. The concepts are not that hard to grasp <small>(mastery is another thing entirely 😅)</small>, but when thrown at you all at once, it was a bit overwhelming.

As next steps you can:

1. Dive deeper into the concepts described in here. Specially around `Data-Oriented Design`.

2. Start delving into Unity's DOTS implementation.


## Here are some links

To start learning about **Unity's DOTS implementations**:

Start here 👉 <a href="https://learn.unity.com/tutorial/65bbbee8edbc2a1bb56409d4?uv=6&projectId=65b3d3cfedbc2a5399ce3740#">Basics of DOTS: Jobs and Entities, Unity Learn project</a>

<small>This is an intro to the concepts</small>

<a href="https://unity.com/campaign/data-oriented-design-bootcamp" target="_blank">Unity's DOTS bootcamp</a>

<small><b>I really recommend this</b> bootcamp. But be wary, it's not holding your hand. Expect to revisit parts of it more than once, and have the code open while going through the video. Pause, experiment a bit, repeat.</small>

<a href="https://github.com/Unity-Technologies/EntityComponentSystemSamples" target="_blank">Unity's Github repo with DOTS samples</a>

To learn more about **Data-Oriented Design**: 

<a href="https://www.youtube.com/watch?v=4B00hV3wmMY" target="_blank">🎬 Solving the Right Problems for Engine Programmers, Mike Acton, TGC 2017</a><br/>
<small>This is a talk aimed towards <b>Game Engine Programmers</b>, so be aware. You are going down the rabbit hole fast.</small>

A topic outside of this blog is **composition over inheritance**:

<a href="https://www.youtube.com/watch?v=JxI3Eu5DPwE" target="_blank">🎬 Is There More to Game Architecture than ECS?, Bob Nystrom, Roguelike Celebration</a>

Nystrom is also the writter of the <a href="https://gameprogrammingpatterns.com/" target="_blank">📚 Game Programming Patterns</a> book. A **really good read** 😉 and the main inspiration for this blog.
