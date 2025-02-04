---
layout: documentation
title: MSI example cache protocol
doc: Learning gem5
parent: part3
permalink: /documentation/learning_gem5/part3/cache-intro/
author: Jason Lowe-Power
---


## MSI example cache protocol

Before we implement a cache coherence protocol, it is important to have
a solid understanding of cache coherence. This section leans heavily on
the great book *A Primer on Memory Consistency and Cache Coherence* by
Daniel J. Sorin, Mark D. Hill, and David A. Wood which was published as
part of the Synthesis Lectures on Computer Architecture in 2011
([DOI:10.2200/S00346ED1V01Y201104CAC016](https://doi.org/10.2200/S00346ED1V01Y201104CAC016)).
If you are unfamiliar with cache coherence, I strongly advise reading that book before continuing.

In this chapter, we will be implementing an MSI protocol.
(An MSI protocol has three stable states, modified with read-write permission, shared with read-only permission, and invalid with no permissions.)
We will implement this as a three-hop directory protocol (i.e., caches can send data directly to other caches without going through the directory).
Details for the protocol can be found in Section 8.2 of *A Primer on Memory Consistency and Cache Coherence* (pages 141-149).
It will be helpful to print out Section 8.2 to reference as you are implementing the protocol.

You can download the Second Edition [via this link](https://link.springer.com/content/pdf/10.1007/978-3-031-01764-3.pdf).

## First steps to writing a protocol

Let's start by creating a new directory for our protocol at src/learning\_gem5/MSI\_protocol.
In this directory, like in all gem5 source directories, we need to create a file for SCons to know what to compile.
However, this time, instead of creating a `SConscript` file, we are
going to create a `SConsopts` file. (The `SConsopts` files are processed
before the `SConscript` files and we need to run the SLICC compiler
before SCons executes.)

We need to create a `SConsopts` file with the following:

```python
Import('*')

main.Append(ALL_PROTOCOLS=['MSI'])

main.Append(PROTOCOL_DIRS=[Dir('.')])
```

We do two things in this file. First, we register the name of our
protocol (`'MSI'`). Since we have named our protocol MSI, SCons will
assume that there is a file named `MSI.slicc` which specifies all of the
state machine files and auxiliary files. We will create that file after
writing all of our state machine files. Second, the `SConsopts` files
tells the SCons to look in the current directory for files to pass to
the SLICC compiler.

You can download the `SConsopts` file
[here](https://gem5.googlesource.com/public/gem5/+/refs/heads/stable/src/learning_gem5/part3/SConsopts).

### Writing a state machine file

The next step, and most of the effort in writing a protocol, is to
create the state machine files. State machine files generally follow the
outline:

Parameters
:   These are the parameters for the SimObject that will be generated
    from the SLICC code.

Declaring required structures and functions
:   This section declares the states, events, and many other required
    structures for the state machine.

In port code blocks
:   Contain code that looks at incoming messages from the (`in_port`)
    message buffers and determines what events to trigger.

Actions
:   These are simple one-effect code blocks (e.g., send a message) that
    are executed when going through a transition.

Transitions
:   Specify actions to execute given a starting state and an event and
    the final state. This is the meat of the state machine definition.

Over the next few sections we will go over how to write each of these components of the protocol.
