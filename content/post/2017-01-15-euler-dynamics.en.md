---
title: Computation of the Optimal Policy for the Optimal Growth (Euler Equation)
date: '2017-01-15'
author: Kenji Sato
tags:
- teaching
---

(Mainly for students attending [my class](/teaching/ma/2016Q4).)

As supplementary matrials on our way to understanding the optimal growth
model, I put an R code and related documents on my github repository:
[kenjisato/intro-macro](https://github.com/kenjisato/intro-macro):

- [/r/optimal_growth_euler.R](https://github.com/kenjisato/intro-macro/blob/master/r/optimal_growth_euler.R)
- [/doc/r/optimal_growth_euler.Rmd](https://github.com/kenjisato/intro-macro/blob/master/doc/r/optimal_growth_euler.Rmd)
    - [Compiled PDF](https://github.com/kenjisato/intro-macro/blob/master/doc/r/optimal_growth_euler.pdf)


I do this mainly for a pedagogical reason. As a matter of fact, the above
mentioned script **isn't practical**; it does work if the initial condition is
close to the steady state and the initial guesses of optimal consumption are
sufficiently accurate but otherwise doesn't (it'd take very loong time).

The only utility of this excercise is to show how it is difficult to compute
the optimal policy!

Human beings (as main agents in our economic system) might have a different
decision making strategy such as adaptive learning (when you realize that you
aren't on the right track you will make some adjustments).
Incorporating such notion in formal analysis might be an interesting
direction but is beyond the scope of the course.

We, in fact, have a more practical method to compute the optimal policy, in which
we exploit the technique of dynamic programming. This will be discussed elsewhere.
