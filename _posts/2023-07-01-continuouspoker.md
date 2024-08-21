---
title: continuous poker
tags: math
comments: true
math: true
---

i mean, we did [continuous blackjack](https://www.example.com) so this is just continuing a trend. what's next? continuous roulette? 

<!--more-->

### problem

let's play *continuous poker*. at the table, the game starts with the $n$ players each putting in \$1 to the pot. each round of poker goes as follows: 

* each player gets *dealt* a real number chosen uniformly at random between 0 and 1 inclusive.
* simultaneously, each of the players decides if they want to play this round, or *fold*. 
* if everyone folds, the "folder" who got dealt the highest value has to match the pot. 
* if only one person plays, they get all the money in the pot and the game ends. 
* otherwise, multiple people play; the player who got dealt the highest takes the pot and everyone else (who's playing) must match the pot. 
* keep playing rounds until the pot is empty.

*matching the pot* means a player putting an amount equal to the current value of the pot into the pot. 

given that everyone plays optimally, what's the strategy? 

### solution

first off, there's no point in bluffing. you bluff in poker to force people to fold, but in continuous poker each player makes the decision to play or fold without knowing anyone else's decisions so you don't even get the opportunity to influence anyone else by bluffing. then, the strategy is pretty much the same as the one in continuous blackjack: if we get dealt anything higher than a certain cutoff $p$, play; fold otherwise. (the argument for this strategy is exactly the same.) then, we find $p$ by noticing that if a player is dealt $p$, they should be indifferent as to whether they should play or fold. let $E_p$ be the player's expected winnings given they play when dealt $p$, and $E_f$ be the expected winnings given they fold. 

if they play, they win if everyone else got dealt stuff less than $p$, which happens with probability $p^{n - 1}$. otherwise, they lose the current round and the pot, if it were initially at value $P$, increases to $(k - 1)P$ if $k$ players decided to play this round. then, we claim that the expected winnings from this increased pot is $\frac{(k - 1)P}{n}$ (and subtract $P$ for having to match the pot). think about it this way: since this game is symmetric, it's equally likely that each player wins, and note that $\text{winnings} = \text{value of pot} - \text{bets paid into pot} = (k - 1)P$ always, by law of conservation. if $k$ players play, then it means that $n - k$ players got dealt less than $p$, which happens with probability $\binom{n - 1}{k - 1} (1 - p)^kp^{n - k - 1}$ and letting $p = 1$ without loss of generality we have 

\begin{equation}
E_p = p^{n - 1} - \left(1 - p^{n - 1}\right) + \sum_{k = 2}^n \dfrac{(k - 1)\binom{n - 1}{k - 1} (1 - p)^{k - 1} p^{n - k}}{n}.
\end{equation}

if they fold, then either everyone else folds or someone else decides to play. if everyone else folds, the expected winnings is $-1 + \frac{2}{n}$; and casing on if $k$ players decide to play ($1 < k < n$) we have 

\begin{equation}
E_f = p^{n - 1}\left(\dfrac{2}{n} - 1\right) + \sum_{k = 2}^{n - 1} \dfrac{(k - 1)\binom{n - 1}{k} (1 - p)^k p^{n - k - 1}}{n}.
\end{equation}

setting $E_p = E_f$ and rearranging a bit gives 

\begin{equation}
3p^{n - 1} = 1 + \dfrac{2p^{n - 1}}{n} + \sum_{k = 2}^{n - 1} \dfrac{(k - 1)\binom{n - 1}{k} (1 - p)^k p^{n - k - 1}}{n} - \sum_{k = 2}^n \dfrac{(k - 1)\binom{n - 1}{k - 1} (1 - p)^{k - 1} p^{n - k}}{n}
\end{equation}

and the binomial sums look awful. fortunately, unlike continuous blackjack, this does yield a simplification: that last sum can be written as 

\begin{equation}
\dfrac{(n - 1)(1 - p)p^{n - 2}}{n} + \sum_{k = 2}^{n - 1} \dfrac{k\binom{n - 1}{k} (1 - p)^k p^{n - k - 1}}{n}
\end{equation}

and so our right-hand side simplifies down to 

\begin{equation}
1 + \dfrac{2p^{n - 1}}{n} - \dfrac{(n - 1)(1 - p)p^{n - 2}}{n} - \sum_{k = 2}^{n - 1} \dfrac{\binom{n - 1}{k} (1 - p)^k p^{n - k - 1}}{n}
\end{equation}

and now note that our binomial sum is now just part of the expansion of $\left[p + (1 - p)\right]^{n - 1}$ so simplifying further we get 

\begin{equation}
1 + \dfrac{2p^{n - 1}}{n} - \dfrac{(n - 1)(1 - p)p^{n - 2}}{n} - \dfrac{1}{n} + \dfrac{p^{n - 1} + (n - 1)(1 - p)p^{n - 2}}{n} = \dfrac{n - 1}{n} + \dfrac{3p^{n - 1}}{n}
\end{equation}

and revisiting our left-hand side we get

\begin{equation}
3p^{n - 1} = \dfrac{n - 1}{n} + \dfrac{3p^{n - 1}}{n} \iff p = \mathbf{\sqrt[n - 1]{\dfrac{1}{3}}}
\end{equation}

and we are done.

also this is a relevant [paper](https://arxiv.org/abs/2108.06556) on it