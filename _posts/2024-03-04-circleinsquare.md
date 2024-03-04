---
title: feeling a bit boxed in
tags: puzzle math
comments: true
---

<!--more-->

# problem

pick two points in a square, uniformly at random. what is the probability that the circle formed by taking the two points as a diameter is completely contained in the square? 

# solution

working with random coordinates $\left(x_1, y_1\right), \left(x_2, y_2\right)$ doesn't really take us anywhere. we switch to looking at two other things that can define a circle: 

* midpoint of the circle $O = (h, k)$
* diameter length $d$ of the circle

in hopes of getting something useful. the distribution of $d$ can be found, after a bit of manipulation from page 3 of [this paper](https://people.kth.se/~johanph/habc.pdf), to 

\begin{equation}
f_D(d) = \begin{cases}
2d\left(d^2 - 4d + \pi\right) & \text{if }d <= 1 \newline
2d\left[4\arcsin\left(\dfrac{1}{d}\right) + 4\sqrt{d^2 - 1} - d^2 - \pi - 2\right] & \text{if }1 < d \leq \sqrt{2} \newline
0 & \text{otherwise}
\end{cases}
\end{equation}

so it remains to deal with $O$. note that, for fixed $d$, $O$ completely determines whether the circle stays contained or not: the circle defined by $O$ and $d$ is contained inside the square if and only if $\frac{d}{2} \leq h, k \leq 1 - \frac{d}{2}$. can we just say the probability the circle is contained is $(1 - d)^2$ then? no -- $O$ is **not uniformly distributed**. if the circle exits the square, the arc outside the square corresponds to diameters that can't actually be formed. for example, if a quarter of the circle is outside the square, it means there are certain point pairs that don't work as diameters. so, for a given $O$ and $d$, draw the circle and think: out of all diameters, which ones have both endpoints inside the circle? shown below is the general case, with the illegal region in red. (the region at the top of the circle is red because any endpoint there would correspond to the other endpoint being outside the square.)

![general case](/assets/img/js%2002.24/1.png)

let the probability of the midpoints with $\frac{d}{2} \leq h, k \leq 1 - \frac{d}{2}$ have arbitrary "scaled density" $\pi$, corresponding to being able to pick all points along one-half of the circle to be an endpoint of a diameter. it remains to think, how much "scaled density" do points outside the central square have? $\pi - \theta$, where $\theta$ is the length of the arc outside the square. so, if $0 \leq k < \frac{d}{2}$ and $\frac{d}{2} \leq h \leq 1 - \frac{d}{2}$, the "scaled density" is $\pi - 2\arccos\left(\frac{2k}{d}\right)$. integrating over $k$, then multiplying by $4(1 - d)$, gives the "scaled density" over all edges (but not the four $\frac{3}{2}$-by-$\frac{d}{2}$ square corners) of the square. 

\begin{equation}
4(1 - d)\displaystyle\int_0^\frac{d}{2} \pi - 2\arccos\left(\dfrac{2h}{d}\right)\,dh = 2d(1 - d)(\pi - 2)
\end{equation}

![edge case](/assets/img/js%2002.24/2.png)

we must be careful with the corner cases. without loss of generality let $0 \leq h, k < \frac{d}{2}$. we claim that there are points in this square that $O$ cannot be. if the circle defined by $O$ and $d$ contains the corner $(0, 0)$, then more than half the circle is outside the square. but then, there's no legal diameter of the circle, contradiction (figure shown below). so, $O$ must be at least $\frac{d}{2}$ distance from the origin, which corresponds to $h^2 + k^2 > \left(\frac{d}{2}\right)^2$. 

![corner case](/assets/img/js%2002.24/3.png)

again by symmetry, multiplying the relevant integral by $4$ gives the "scaled density" over all corners: 

\begin{equation}
4\displaystyle\int_0^\frac{d}{2} \int_\sqrt{\left(\frac{d}{2}\right)^2 - h^2}^\frac{d}{2} \pi - 2\arccos\left(\dfrac{2h}{d}\right) - 2\arccos\left(\dfrac{2k}{d}\right)\,dk\,dh = d^2(\pi - 3)
\end{equation}

so adding in the center cases's "scaled density" of $\pi(1 - d)^2$ we get a probability of 

\begin{equation}
p(d) = \dfrac{\pi(1 - d)^2}{d^2 - 4d + \pi}
\end{equation}

then using law of total probability we have 

$$\begin{align*}
\int_0^\sqrt{2} p(d) f_D(d)\,\mathrm{d}d &= \int_0^1 2d\left(d^2 - 4d + \pi\right)p(d)\,\mathrm{d}d \\
&= \int_0^1 2\pi d(1 - d)^2\,\mathrm{d}d = \mathbf{\dfrac{\pi}{6}}
\end{align*}$$

and we are done.

the [official solution](https://www.janestreet.com/puzzles/some-off-square-solution/) involved less calculation. while the first probability should be clear as an integral, the second probability of $\frac{3}{4}$ might be better understood in the reverse direction. if you know that your diameter (vector extended past the circle's center) must lie within the square, then your circle center must lie within the square scaled down by half, where the center of scaling is your originally chosen diameter endpoint. thus, the ratio of the smaller square, the successful region, to the larger one, is $\frac{1}{4}$. 