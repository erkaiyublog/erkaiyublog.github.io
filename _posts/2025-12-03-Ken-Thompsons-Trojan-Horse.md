---
published: true
title: Ken Thompson's Trojan Horse 
tags: compiler security C
---

In this post, I will briefly introduce the idea of Ken Thompson's Trojan Horse, which I recently learned about while reading a book on compiler design.

As the name suggests, Ken Thompson's Trojan Horse is a backdoor that can be inserted into the C compiler that would allow you to log into affected machines as any user. The motivation of this backdoor came from a challenge he had with his colleagues in UCB to write the shortest self-reproducing program (this type of program is known as [quine](https://en.wikipedia.org/wiki/Quine_(computing))). Later, Ken presented this work during his Turing award lecture (it was originally intended to be a paper, but he gave a talk with slides instead) in 1983.

An example of quine in Python looks like this:

```python
s='s=%r;print(s%%s)';print(s%s)
```

The idea behind such a backdoor is straightforward: you want to make a compiler such that:
* Property 1: It inserts a backdoor into the binary whenever compiling a login application.
* Property 2: It inserts both property 1 and 2 into the binary whenever compiling the compiler itself.

To achieve property 1 and 2 without revealing anything in the source code, Ken took advantage of the fact that when the compiler is first compiled, there must inevitably be some hard-coded bytes due to limitations of the simpler C translator used before the real compiler exists. For example, when trying to parse escape sequences like `\n`, the relevant compiler code may look like:

```c
c = next();
if(c == '\\') {
    c = next();
    if(c == 'n')
        c = '\n';
}
```

When the compiler is compiling itself, the bootstrap compiler used at the beginning does not support parsing these escape sequences. Thus, to produce a correct binary, the compiler has to embed fixed byte values directly rather than using the literal escape sequences from the source.

By exploiting the fact that certain byte sequences must be baked into the binary and cannot be inferred from the source code alone, Ken was able to embed properties 1 and 2 into the compiler’s binary. In particular, implementing property 2 requires the use of a quine-like technique, and once you understand quines, the mechanism becomes fairly straightforward in spirit.

# References

1. [That Time Ken Thompson Wrote a Backdoor into the C Compiler](https://micahkepe.com/blog/thompson-trojan-horse/)
2. [Running the "Reflections on Trusting Trust" Compiler](https://research.swtch.com/nih)
