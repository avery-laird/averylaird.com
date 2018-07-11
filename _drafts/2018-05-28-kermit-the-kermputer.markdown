---
layout: post
title: "Kermit the Green Kermputer"
categories: hardware computer kermit
---

I have decided to start homebrew computer project, which I have dubbed "Kermit" (after everyone's favourite muppet). I am inspired in part by great projects, like Ben Eater's [breadboard computer](https://eater.net/8bit/). Projects like his are a great educational experience, but the more I thought about what I might actually want to build, I realized I wanted something a bit more sophisticated. Something that is not particularily feasible on just a breadboard. 

My current plan is to design a 32-bit computer with a multicycle CPU and pipelining, using mostly basic components (or as basic as possible). I will design and test each component separately, first using Verilog, then on a breadboard, and finalized on a PCB. As a final step, I might choose to place everything on a PCB.

The instruction set will be heavily based on MIPS. Probably, most MIPS instructions would run on a finished Kermit, but I don't want to constrain myself to a particular existing ISA. MIPS is a good starting point because it is simple, but is also relatively easy to pipeline.

Once Kermit is complete, I would like to build a bare-bones assembler (for Kermit assembly, or KASM), a very simple C compiler, and perhaps a rudimentary operating system (KermOS).

Below you is a list of articles covering the design of each component so far:



