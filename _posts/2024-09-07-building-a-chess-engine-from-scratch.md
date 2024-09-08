---
published: true
title: Building a Chess Engine from Scratch
tags: chess-engine chess 
---

In this blog, I'm recording the process of me developing my own chess engine, [silkfish](https://github.com/silkrow/silkfish).

---
***Table of Contents***
* TOC
{:toc}

# Motivation
If you are not interested in chess, you are probably not gonna be interested in building a chess engine. Even if you enjoy playing chess, creating your own machine for this game may still make little sense to you, since we already got all those fancy AI chess engines out there with unreachable ratings not only for humans but also for your homemade chess engines. So, why spending the time building your own chess engine?? &#x1F9D0;

I think for most of the readers, their reasons of reading this blog are due to some silly homeworks from some ML introduction courses which require them to build a chess engine using the classical game tree searching algorithms. Similarly, I started my journey of building a chess engine after finishing a machine problem from a ML course which required me to build a tic-tac-toe engine. During the process of building the tic-tac-toe engine, I read this insightful paper called [Programming a Computer for Playing Chess](https://vision.unipv.it/IA1/ProgrammingaComputerforPlayingChess.pdf) from the course materials, to my surprise, the key ideas introduced in this paper by our dear friend Claude E. Shannon have not much difference from the ones I used to create a tic-tac-toe engine, then I started to realize that building my own chess engine might not be such a difficult task. As a chess lover myself, I couldn't help wondering whether it would be possible to create a chess engine following my own understanding of chess to beat myself. It felt like teaching a student and see it gradually become stronger than myself, how exciting! So, I created a new folder, copied the code from my tic-tac-toe engine, and started the journey of creating my own chess engine. 


