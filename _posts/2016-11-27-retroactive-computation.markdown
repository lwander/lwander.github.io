---
layout: post
title: "Retroactive Computation"
date: 2016-11-27 08:49:00 -0400
categories: cs
---

I've been trying to figure out a computation model in a (mythical)
universe that is more powerful than Turing Machines, in the sense that this
model can compute things that Turing Machines cannot. If we allow
time-travel in this universe, it's very possible. Let's loosely describe the
semantics of this model:

First we start with a standard Turing-complete register machine, where we have a set of
registers \\(R\\), a program \\(P\\) that manipulates these registers,
and an instruction pointer \\(i\\) referring to the next instruction 
\\(P_i\\) to execute. Let's say that once \\(i \geq \mathrm{len}(P)\\) we are finished executing.
These three things describe a state \\(S_c = (R, P, i)\\) 
that represent an execution of a register machine program after \\(c\\)
instructions have been executed. Assume we have a machine \\(M\\) that take a 
state, executes a single instruction, and returns the following state:
\\(M(S_c) = S_{c+1}\\).

Now typically, in some meta-language, we would execute programs like this:

```
func execute(P):
    R = 0..       # instatiate an infinite tape of 0's
    i = 0
    while i < len(P):
        (R, _, i) = M(R, P, i)

    return R
```

Let's add some magical operators to this meta-language that makes things
interesting. Let's say that we can create a channel between two points in time
that we can send data between. 
