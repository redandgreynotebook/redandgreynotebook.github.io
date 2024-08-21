---
title: just recurse on the reals
tags: math puzzle
comments: true
math: true
---

tw: differential equations

<!--more-->

### problem

alice and bob play *continuous blackjack*. alice goes first, starting at a score of 0. she can choose to either *hit*, adding a number chosen uniformly at random from $[0, 1]$ to her current score, or *stay*, finalizing her current score. she can hit as many times as she wants; however, if her score goes over $1$ before she stays she goes *bust* and her final score will be zero. 

bob then plays according to the same rules, but has no knowledge of alice's results. the player with the higher score is the winner; if they draw they play again until a winner is decided. if alice and bob must tell each other their strategies, and both play to maximize their probability of winning, find the probability that alice will not score zero on her first game. 

source: [article](https://arxiv.org/abs/2011.10315)

### solution

let's first ignore bob, so what alice should do only depends on her current score $s$. it's not clear what her strategy should be, even in singleplayer continuous blackjack, but let's examine edge cases. if $s = 1$ she should obviously stay, and if $s = 0$ she should obviously hit. now, note that for small $\epsilon > 0$ alice should still stay at $s = 1 - \epsilon$ and hit at $s = \epsilon$: after all, since $0$ and $\epsilon$ are so close together, it makes sense that the best move is the same, and similarly for $1$ and $1 - \epsilon$. repeating this argument all the way, and letting the "stay" and "hit" scores meet in the middle, we see that at some point, let's say $s = p$, the best move switches from hitting to staying. and this makes sense: if we stay at $s = p$, why should we hit at $s' = p + \epsilon$, since hitting at $s'$ has an even greater chance to go bust? 

let $X$ be the random variable representing alice's score after she hits a positive number of times; in other words, $X$ represents alice's score up until she decides to stay or she busts.
let $F(x)$ represent this variable's cumulative distribution function and $f(x)$ represent its probability density function. while the law of total probability does have a use case for continuous distributions like these, the expression gets sorta ugly since you have different cases for what you should do, depending on whether $X < p$ or $X > 1$ and it's quite a mess. 

so, let's think of this continuous uniform distribution $[0, 1]$ as the limiting case of a *discrete* uniform distribution, where alice will hit for $\frac{i}{n}$, where $i$ is uniformly distributed on $\{0, 1, \cdots , n\}$. the cutoff $p$ will then change to a cutoff $\frac{p}{n}$ for some natural $p < n$. now, it's pretty clear what our probability recursions for $\text{Pr}\left(X = \frac{x}{n}\right)$ should be: note that $X = \frac{X' + u}{n}$, where $u$ is the value of alice's last hit and $X'$ is alice's score before that hit. we case on $x$:

* if $x \leq p$ then $nX'$ can be anywhere from $0$ to $x$
* if $p < x \leq 1$ then $nX'$ can be anywhere from $0$ to $p$
* if $1 < x < 1 + p$ then $nX'$ can be anywhere from $x - 1$ to $p$
* if $x \geq 1 + p$ then there are no legal values $nX'$ can take

for a probability recursion of 
\begin{equation}
\text{Pr}\left(X = \dfrac{x}{n}\right) = 
\begin{cases}
\displaystyle\sum_{i = 0}^x \dfrac{\text{Pr}\left(X' = \frac{i}{n}\right)}{n + 1} & \text{if }x \leq p \newline
\displaystyle\sum_{i = 0}^p \dfrac{\text{Pr}\left(X' = \frac{i}{n}\right)}{n + 1} & \text{if }p < x \leq 1 \newline
\displaystyle\sum_{i = x - 1}^p \dfrac{\text{Pr}\left(X' = \frac{i}{n}\right)}{n + 1} & \text{if }1 < x < 1 + p \newline
0 & \text{otherwise}
\end{cases}
\end{equation}

where the denominator of $n + 1$ comes from the fact that there are $n + 1$ elements in our discrete uniform distribution, and each of them is equally likely to be chosen. as $n \mapsto \infty, \text{Pr}(X = r) \mapsto f(r)$ where $r = \frac{x}{n}$. applying this abuse of notation, we get the integral equation

\begin{equation}
f(x) = 
\begin{cases}
\displaystyle\int_0^x f(u) \,du & \text{if }x \leq p \newline
\displaystyle\int_0^p f(u) \,du & \text{if }p < x \leq 1 \newline
\displaystyle\int_{x - 1}^p f(u) \,du & \text{if }1 < x < 1 + p \newline
0 & \text{otherwise}
\end{cases}
\end{equation}

let's solve the first case $x \leq p$ first, since every other case depends on it. differentiating and using the second fundamental theorem of calculus we get $f'(x) = f(x) \iff f(x) = Ce^x$; however, we must satisfy initial condition $f(0) = C = \int_0^0 f(u)\,du = 0$


...wait what. that makes no sense. the issue is one of continuity correction: note that $\int_0^x f(u)\,du = F(x) - F(0) = \text{Pr}(X \leq x) - \text{Pr}(X \leq 0) = \text{Pr}(0 < X \leq x)$, with the key being the strict $0 < X$ inequality. in almost any other distribution this wouldn't matter, but remember that $X$ is defined *only* on $[0, 1 + p]$ and so we must make sure to include $0$ as a value $X$ can be. since $f(x) = 0$ for all $x < 0$, simply change the lower bounds from $0$ to $-\infty$. doing this, we get

\begin{equation}
f(x) = 
\begin{cases}
Ce^x & \text{if }x \leq p \newline
Ce^p & \text{if }p < x \leq 1 \newline
C\left(e^p - e^{x - 1}\right) & \text{if }1 < x < 1 + p \newline
0 & \text{otherwise}
\end{cases}
\end{equation}

and doing our integration we get $C = e^{-p}$. 

now let $S$ the random variable representing alice's score, and define $G(s), g(s)$ how you think they'd be defined. again, we case depending on $s$, since the only valid previous scores $X$ are those such that $p \leq X \leq 1$: 

\begin{equation}
g(s) = 
\begin{cases}
\displaystyle\int_p^s f(x)\,dx & \text{if }s < 1 \newline
\displaystyle\int_p^1 f(x)\,dx & \text{if }1 \leq s \leq 1 + p \newline
\displaystyle\int_{s - 1}^1 f(x)\,dx & \text{if }1 + p < s \leq 2 \newline
0 & \text{otherwise}
\end{cases}
\end{equation}

and also we need to calculate $$\text{Pr}(S = 0) = \text{Pr}(X > 1) = \displaystyle\int_1^{1 + p}f(x)\,dx = p - 1 + e^{-p}$$ and after scaling both $g(s)$ and $\text{Pr}(S = 0)$ by a factor of $C = e^p$ (so that the sum of probabilities is $1$) we have 

$$\begin{align*}
g(s) &= 
\begin{cases}
e^p(s - p) & \text{if }s < 1 \newline
e^p(1 - p) & \text{if }1 \leq s \leq 1 + p \newline
e^p(2 - s) f(x)\,dx & \text{if }1 + p < s \leq 2 \newline
0 & \text{otherwise}
\end{cases} \\
\text{Pr}(S = 0) &= 1 - e^p(1 - p)
\end{align*}$$

i think the rest should be rather straightforward. let $w(p, q)$ be the probability that alice wins given that she uses $p$ as her hit/stay cutoff while bob uses cutoff value $q$. since alice wants to maximize this probability and bob wants to minimize it, we must have $\nabla w(p, q) = \vec{0}$. also, both alice and bob will decide on choosing the same cutoff $p$. this is because $p$ is on a spectrum from too safe (so you probably won't bust but your expected score given you don't bust will be low), which happens when $p$ is low, to too risky (so your expected non-busting score is high but you'll probably bust), when $p$ is high. if $p$ is too safe bob can choose $q = p + \epsilon$ and win more often; and if $p$ is too risky bob can choose $q = p - \epsilon$ for some small $\epsilon > 0$. 

a concrete example might help: alice choosing $p = 0.9$ is a very, very risky cutoff; in response, bob choose $q = 0.89$. if neither of them bust, alice's score ever-so-slightly beat bob's; however, this is outweighted by the fact that bob busts less, since busting is basically an auto-lose. for an 100% rigorous proof, you'll find that $\nabla w(p, p) \neq \vec 0$ for such values of $p$ that are either too safe or too risky. 

so, it remains to find $p$ such that $\nabla w(p, p) = \vec 0$. but first, we need to find $w(p, q)$. we case on who, if anyone, busted. here, we let $b(p) = 1 - e^p(1 - p)$ represent the probability a player with cutoff $p$ busts, and let $g(s, p)$ represent the distribution of $S$ given the player plays with cutoff $p$. this is the exact same $g$ function as before, but since we must keep track of both alice's and bob's, we explicity specitfy $p, q$ to avoid confusion:

* if neither player busts then the probability of alice winning is $\text{Pr}(s_a > s_b > 0) = \int_p^2 \int_q^{s_a} g\left(s_a, p\right)g\left(s_b, q\right)\,ds_2\,ds_1$
* if bob busts then the probability of alice winning is $b(q)\left[1 - b(p)\right]$
* if both bust then the game effectively starts over so the probability of alice wining is $\left[1 - b(p)\right]\left[1 - b(q)\right]w(p, q)$

and since the sum of these three probabilities equals $w(p, q),$ solving, we get $$w(p, q) = \dfrac{\left[1 - e^q(1 - q)\right]e^p(1 - p) + \displaystyle\int_p^2 g\left(s_1, p\right)\int_q^{s_1} g\left(s_2, q\right)\,ds_2\,ds_2}{1 - \left[1 - e^p(1 - p)\right]\left[1 - e^q(1 - q)\right]}.$$

i tried for way too long to do that double integral by hand, and i recommend that you don't. you need to split that double integral into five separate cases (and this is even assuming, without loss of generality, that $q \geq p$!): 

* $p \leq s_1 \leq q$
* $q \leq s_1 \leq 1$
* $1 \leq s_1 \leq 1 + p$
* $1 + p \leq s_1 \leq 1 + q$
* $1 + q \leq s_1 \leq 2$

and the integrand in each of these cases is a cubic. yeah, no. my experience in using computer algebra systems up to this point consisted of using a ti-89 in high school calculus. so, i downloaded mathematica and made it do all the work for me. the first step is defining $g(s, p)$:

```mathematica
f[x_, p_] := Exp[p] * Piecewise[{ 
    {x - p, x >= p && x < 1}, 
    {1 - p, x >= 1 && x < 1 + p}, 
    {2 - x, x >= 1 + p && x <= 2} 
    }, 0]
```

then, $w(p, q)$:

```mathematica
w[p_, q_] := (Integrate[Integrate[g[s1, p] * g[s2, q], {s2, q, s1}, 
      Assumptions -> Element[s1, Reals] && p > 0 && p < 1 && Element[q, Reals] && q >= p], 
      {s1, p, 2}, Assumptions -> Element[p, Reals] && p > 0 && p < 1] + 
      (q * Exp[q] - Exp[q] + 1) * (Exp[p] - p * Exp[p])) / 
      (1 - (Exp[q] * (q - 1) + 1) * (Exp[p] * (p - 1) + 1))
```

now we find $\frac{\partial w}{\partial p}$. the criteria was $\nabla w(p, q) = \vec 0$, but since we know in the end $q = p$ that implies $\frac{\partial w}{\partial q} = \frac{\partial w}{\partial p}$:

```mathematica
Simplify[D[w[p, q], p], Element[p, Reals] && Element[q, Reals] && q < 1 && q > p && p > 0 && p < 1]
```

and now substitute in $p$ for $q$:

```mathematica
Simplify[% /. q -> p]
```

we end up with $$\dfrac{\partial w}{\partial p} = \dfrac{3p - e^p(p - 1)^2(2 + p)}{6(p - 1)\left[2 + e^p(p - 1)\right]}$$ when $q = p$; it remains to set this equal to zero and solve. thus, it suffices to solve when the numerator is equal to zero, ignoring the denominator:

```mathematica
Solve[Exp[p] * (2 + p) * (-1 + p)^2 == 3*p && p > 0 && p < 1, p]
```

and it remains to find $b(p) = 1 - e^p(p - 1)$ where $p$ is the solution to $e^p(2 + p)(1 - p)^2 = 3p$:

```mathematica
N[1 - Exp[p] * (1 - p) /. %, 9]
```

for a final answer of around $0.114845886$. we are done. 

it was nice to see "probability recursion" being used in the continuous case. i knew of probability recursion as a technique to solve problems with countably many states, such as who wins a win-by-two-point game given the probability that alice beats bob on any point; or the expected number of coin flips until you get three heads in a row. 

that being said, the final double integral was awful; even mathematica took minutes computing it. if it were up to me, i'd chance the premise of the problem to be a single-player game, with alice wanting to maximize her expected score. that way, the answer is simply the maximum of $$\displaystyle\int_p^2 s g(s)\,ds = \dfrac{e^p\left(p^3 - p + 2\right)}{2}$$ which can be done by hand. however, this does miss out on the whole game theory part of this problem, so you take some you give some i guess.