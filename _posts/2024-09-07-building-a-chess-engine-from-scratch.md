---
published: false
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

Well, my answer to this question is, for fun! The thought of building an engine like the one created by the tech giant IBM around 30 years ago just appeared to be so cool to me. When I took a ML course as a senior undergrad, one of the assignments we got was to implement a tic-tac-toe engine using what we've learned, it was at that time I encountered this insightful paper called [Programming a Computer for Playing Chess](https://vision.unipv.it/IA1/ProgrammingaComputerforPlayingChess.pdf) and realized that a chess engine has not much difference from the toy tic-tac-toe engine I just built! So, I created a new folder, copied the code from my tic-tac-toe engine, and started the journey of creating my own chess engine. 

# Silkfish 1.0
Silkfish 1.0 is deployed on a server with Intel(R) Xeon(R) CPU E5-2698 v4 @ 2.20GHz CPU, 2 cores (1 thread each). It has a rating of **1700** in both rapid and classical time control on lichess. In terms of game strategy, silkfish 1.0 has the following features implemented.  

1. Minimax search
2. Alpha-Beta Pruning
3. Quiescence search
4. Piece-Square Tables
5. Multithreading

## 


