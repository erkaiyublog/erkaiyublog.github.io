---
published: true
title: A Domino Puzzle
tags: puzzle
---

I’ve been busy with research and the challenges of job hunting lately, and my brain has been craving some interesting problems to work on. Luckily, I came across a book called [*Mathematical Puzzles, Revised Edition*](https://math.dartmouth.edu/news-resources/electronic/puzzlebook/index.php), which brought me to an interesting problem from [Chapter 7 The Law of Small Numbers](https://math.dartmouth.edu/news-resources/electronic/puzzlebook/book/ch7.pdf). After solving this problem, I was also inspired by the idea behind and managed to solve another puzzle I had in mind for a few months. So, I'm writing this blog to share these two interesting puzzles.

# A Domino Puzzle

This puzzle is called "Domino Task" in the book. It is described as the following:

An 8x8 chessboard is tiled arbitrarily with 32 2x1 dominoes. A new square is added to the right-hand side of the board, making the top row length 9.

At any time you may move a domino from its current position to a new one, provided that after the domino is lifted, there are two adjacent empty squares to receive it.

Can you retile the augmented board so that all the dominoes are horizontal?

![puzzle](/images/posts/domino/puzzle1.jpeg)

## My Thought Process

The moving domino seemed to be a little bit messy at the first glance. However, notice the fact that each domino always cover one black square and one white square no matter how it's placed, it is easy to draw the conclusion that each black square is exactly covered by one domino. Further, imagine there is a hinge at each black square, and the domino covering it is rotating on it.

![puzzle](/images/posts/domino/puzzle2.jpeg)

![puzzle](/images/posts/domino/puzzle3.jpeg)

# Knights on A Chessboard 

The idea of "divide and conquer" (or "solving the problem with a smaller number") of the domino puzzle together with the chessboard reminded me of another puzzle. I remember watching a video clip in which many chess grand masters were asked to answer several puzzles related to chess, and Anish Giri quickly got this puzzle correct (sadly, I couldn't remember the title of that video).

The puzzle is: What is the maximum number of knights one can place on a chessboard without them attacking each other?

## Answer

In the video, Giri quickly arrived at this answer, which was so persuasive that it seemed self-explanatory.

The trivial answer is **32**. Due to the fact that knights on the same colored squares won't attack each other, one can simply place knights all on the dark squares and there are 32 of them.
