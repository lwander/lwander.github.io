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

First we start with a standard Turing-complete register machine, where we have 
a set of registers \\(R\\), a program \\(P\\) that manipulates these registers,
and an instruction pointer \\(i\\) referring to the next instruction 
\\(P_i\\) to execute. Let's make two assumptions about this register machine:

  1. Once \\(i \geq \mathrm{len}(P)\\) we are finished executing. Keep in mind
     that this does not imply the program takes at most \\(\mathrm{len}(P)\\)
     steps, since our program's control flow can contain loops and conditional
     branches.
  2. Each instruction operates on a constant number of registers at a time.
  
Now we can define \\(S_c = (R, P, i)\\), which represents the state of a 
register machine program after \\(c\\) instructions have been executed. Assume 
we have a machine \\(M\\) that take a state, executes a single instruction, 
and returns the following state: \\(M(S_c) = S_{c+1}\\).

Now typically, in some meta-language, we would execute programs like this:

```
func execute(P):
    R = 0..                       # instantiate an infinite tape of 0's
    i = 0
    while i < len(P):             # by assumption 1
        (R, _, i) = M(R, P, i)

    return R
```

Let's start extending this language with the concept of a channel \\((s, r)\\),
which is a tuple of \\(s\\), a sender, and \\(r\\), a reciever. Each message
transferred through the channel is tagged with a numeric id, and the message 
received is the one sent with the greatest id. However, this channel is special 
in the sense that anything sent through \\(s\\) arrives at \\(r\\) when 
\\(r\\) was first created. Assume we can create such a channel with 
\\(T\\), our time machine.

We can give sample usage of our magical channel here:


```
func demo(msg):
    (s, r) = T()
    print r()

    id = 1
    s(id, msg)
```

This prints whatever we supply to `demo`.

Before we continue, let's assume that machine \\(M\\) has a counterpart
\\(M'\\) which given a state \\(S_c\\) returns \\(\Delta S_c\\) that contains
only the (constant, by assumption 2) difference between \\(S_{c+1}\\) and
\\(S_c\\). Similarly, we can apply in constant time \\(\Delta S_c\\) to
\\(S_c\\) to get \\(S_{c+1}\\)


Now let's adjust how `execute` behaves:

```
func execute(P):
    R = 0..                       # instantiate an infinite tape of 0's
    i = 0
    c = 0 
    (s, r) = T()

    msg = r()
    if msg != nil:
        S = (R, P, i)
    else:
        (S, c) = msg

    c += 1
    dS = M'(S)
    s(c, dS)
```

This still needs a little work, but you can see the beginnings of an execution
model that can execute any program written in a register machine in constant
time.
