---
title: buffon's needle in a haystack
tags: puzzle math
comments: true
math: true
---

<!--more-->

# problem

in the 2d plane, parallel lines are drawn 1 unit apart. drop a needle of length 1 onto this plane. what is the probability it crosses a line? 

source: [buffon's needle](https://en.wikipedia.org/wiki/Buffon%27s_needle_problem)

# solution

let the lines be $y = 0, y = 1, y = -1, y = 2, y = -2, \cdots$. then, dropping a random needle is equivalent to picking a random $y$-position $Y \in [0, 1)$ and an angle $\theta \in [0, 2\pi)$ to orient the needle; both these random variables are uniformly distributed. this means that the $y$-position of the other endpoint is $Y + \sin \theta$.

by symmetry, we can narrow the angle constraint down to the first quadrant, $\theta \in \left[0, \frac{\pi}{2}\right)$: if the angle is in the fourth quadrants, reflect vertically about $Y = y$; if the angle is in the second quadrant, reflect horizontally about the endpoint; if the angle is in the third quadrant combine the previous two reflections. then, the probability the needle crosses the line $y = 1$ with given $Y, \theta$ is equal to 

\begin{equation}
\text{Pr}(Y + \sin \theta \geq 1) = \text{Pr}(1 > Y \geq 1 - \sin \theta) = \sin \theta
\end{equation}

and now integrate, keeping in mind that both are uniformly distributed: 

\begin{equation}
p = \dfrac{2}{\pi} \displaystyle\int_0^1 \int_0^{\frac{\pi}{2}} \sin\theta \,d\theta \,dy = \dfrac{2}{\pi}
\end{equation}

...but we are not done.

# problem in 2d

tile the plane with unit squares, and then drop the needle. what is the probability the needle crosses exactly one line? 

source: buffon's needle, extended to $\mathbb{R}^2$

# solution

let the gridlines be the lattice lines $x, y = 0, \pm 1, \pm 2, \cdots$. then, dropping a random needle is eqivalent to choosing starting point $(X, Y) \in [0, 1)^2$ and angle $\theta \in \left[0, \frac{\pi}{2}\right)$, where we've shrank the smaple space for $\theta$ by the same symmetry argument. (all three are uniformly distributed.) this gives the other endpoint as $(X + \cos\theta, Y + \sin\theta)$. letting $p_x(\theta), p_y(\theta)$ be the probability the needle at angle $\theta$ crosses the lines $x = 1$ and $y = 1$ respectively, we have $p_x(\theta) = \cos\theta, p_y(\theta) = \sin\theta$. then, since $X$ and $Y$ are independent, letting $p(\theta)$ be the probability that it crosses exactly one lattice line is 

\begin{equation}
p_x(\theta) + p_y(\theta) - 2p_x(\theta)p_y(\theta)
\end{equation}

where the subtraction is our correction for crossing both $x = 1$ and $y = 1$. it is subtracted twice because $p_x, p_y$ each count it, so it's counted twice (and needs to be counted zero times). now we integrate: 

\begin{equation}
\dfrac{2}{\pi} \displaystyle\int_{[0, 1)^2}\int_0^{\frac{\pi}{2}} \cos\theta + \sin\theta - \cos\theta \sin\theta \,d\theta \,dA = \dfrac{2}{\pi}
\end{equation}

...but we are still not done.

# problem in 2d, part 2

keep the grid from last problem, but now the randomly dropped needle has length $d$. what value of $d$ maximizes the probability of the needle crossing exactly one line? 

source: [jane street puzzles, february 2020](https://www.janestreet.com/puzzles/single-cross-index/)

# solution

the difficulty happens when $d > 1$. in that case, it is possible for the needle to cross $x, y = 2$. we will handle this annoying case separately. 

### short needle

if $d \leq 1$ the answer is simple: we get $p_x(\theta) = d\cos\theta$ and $p_y(\theta) = d\sin\theta$. integrating gives 

\begin{equation}
p(d) = \displaystyle\int_0^{\frac{\pi}{2}} d\cos\theta + d\sin\theta - 2d^2 \cos\theta \sin\theta \,d\theta = \dfrac{2d(2 - d)}{\pi}
\end{equation}

which has a maximum at $d = 1$, as we have calculated before. 

### long needle

if $d > 1$ we must also consider the case that the needle crosses $x, y = 2$. so, our probability becomes 

$$\begin{align*}
p_x(\theta) = \text{Pr}(1 \leq X + d\cos\theta < 2) &= \text{Pr}\left(\max(0, 1 - d\cos\theta) \leq X < \min(2 - d\cos\theta, 1)\right) \\ 
&= \min(2 - d\cos\theta, 1) - \max(0, 1 - d\cos\theta) \\
&= \min(d\cos\theta, 2 - d\cos\theta)
\end{align*}$$

and similarly $p_y(\theta) = \min(d\sin\theta, 2 - d\sin\theta)$. now, probability calculation is the same: after some casework the $\min$ function you get

\begin{equation}
\dfrac{4d^2 - 8d\sqrt{d^2 - 1} - 8d + 8\arccos\frac{1}{d} + 6}{\pi}
\end{equation}

which has a maximum at $d = 1$. so, $d = 1$ is indeed the optimal length...

...but we still are not done.

# problem in 3d

fill 3d space with unit cubes, and randomly drop a needle of length $d$. what value of $d$ maximizes the probability that the needle crosses exactly one plane? 

source: [jane street puzzles, august 2023](https://www.janestreet.com/puzzles/single-cross-2-index/)

# solution

### long needle

since the problem implies that there's exactly one maximum, it follows that for all $d$ leading up to the maximum value, $p(d)$ increases, and then decreases for all $d$ after the maximum value. so, we can run a monte carlo simulation at a bunch of points, maybe starting at $d = 0$ and incrementing up by $0.1$ each time. doing so, we find that $p(d)$ starts to decrease starting at $d = 0.8$, so the maximum value of $d$ is less than one. this case is irrelevant.

how do we generate random points on a sphere for our simulation? let $X, Y, Z$ be independent standard normal random variables; then 

\begin{equation}
\vec{p} = \left(\dfrac{X}{\sqrt{X^2 + Y^2 + Z^2}}, \dfrac{Y}{\sqrt{X^2 + Y^2 + Z^2}}, \dfrac{Z}{\sqrt{X^2 + Y^2 + Z^2}}\right)
\end{equation}

is uniformly random on the sphere. this is because the standard normal distribution is radially symmetric with respect to the origin; in other words, given that $(x, y, z)$ and $\left(x', y', z'\right)$ have equal distance from the origin, they're equally likely to be "chosen" since distance, not direction, from the origin is the only thing that matters.

### short needle

we use the same approach as the 2d case, converting to spherical coordinates. by symmetry of a cube, we can restrict our domain to $\varphi, \theta \in \left[0, \frac{\pi}{2}\right)$. we get 

$$\begin{align*}
p_x(d, \varphi, \theta) &= d\cos\theta\sin\varphi \\
p_y(d, \varphi, \theta) &= d\sin\theta\sin\varphi \\
p_z(d, \varphi, \theta) &= d\cos\varphi
\end{align*}$$

and now we use inclusion-exclusion: $p_x + p_y + p_z$ counts the event that the needle crosses two planes twice, when it should be counted zero times. however, correcting for this gives 

\begin{equation}
p_x + p_y + p_z - 2\left(p_xp_y + p_yp_z + p_zp_x\right)
\end{equation}

which counts the event that the needle crosses all three planes negative three times, so our result is

\begin{equation}
p(d, \varphi, \theta) = p_x + p_y + p_z - 2\left(p_xp_y + p_yp_z + p_zp_x\right) + 3p_xp_yp_z
\end{equation}

and now we integrate over $\varphi$ and $\theta$. one problem: how are $\varphi$ and $\theta$ distributed? it makes sense that $\theta$ is uniform, but $\varphi$ actually is not: if it was, points with $\varphi$ near zero (points near the "north pole," if you will) would be sampled more often than points with $\varphi$ near $\frac{\pi}{2}$ (near the "equator"). if you had two circles, one larger than the other, and sampled $N$ points on each circle for large $N$, the points on the smaller circle would be denser than the points on the larger circle: this is why we cannot weight each value of $\varphi$ equally. instead, think of the circle given by a value of $\varphi$, and try and correct this sampling density to equal that of a unit circle. since sampling density is inversely proportional to circumference (and thus radius), sampling $N\cos\varphi$ points for every $N$ points on the unit circle makes the two circles have equal sampling density. thus we get the probability density function of $\varphi$ to be $\sin\varphi$. 

integrating over $\theta, \varphi$ gives the probability, and we can finding the maximum by then differentiating with respect to $d$ and setting equal to zero: 

\begin{equation}
\left(\displaystyle\int_0^{\frac{\pi}{2}}\int_0^{\frac{\pi}{2}} p(d, \varphi, \theta) \sin\varphi \cdot \dfrac{2}{\pi} \,d\varphi\,d\theta\right)' = 0
\end{equation}

can be solved using wolframalpha or similar computer algebra software to get $d = \frac{16 - \sqrt{256 - 54\pi}}{9}$

...but we still are not done.

# long needles are bad

i really don't like the monte carlo "explanation" for why we can ignore the case of long needles. and while the one i came up with isn't entirely rigorous, at least it has some mathematical sense behind it. 

we showed that in a 2d lattice grid, the best value of $d$ is $1$. a 3d lattice grid feels denser than a 2d grid, so it makes sense that we need a lower value of $d$ for a denser lattice. but how can this be formalized? 

take point $(x, y, z)$ in the cube. what's the expected minimum distance from that point to the planes $x, y, z = 1$? we're finding the expectation of $\min(1 - X, 1 - Y, 1 - Z)$ and since all three are standard uniform random variables, this [expectation](https://math.stackexchange.com/questions/786392/expectation-of-minimum-of-n-i-i-d-uniform-random-variables) is $\frac{1}{4}$, down from the expected $\frac{1}{3}$ of the square; indeed, for $n$ dimensions the expected minimum distance to one of the planes $x_1, x_2, \cdots , x_n  = 1$ is $\frac{1}{n + 1}$. on the other hand, the expected coordinate gain from the needle is $\frac{1}{\sqrt{n}}$, which grows larger than $\frac{1}{n + 1}$, meaning that the difference between the two is more pronounced the larger $n$ becomes. (why is it valid to compare expected *minimum* to expected *average*? because the coordinate gains from the needle are independent of the endpoint of the needle, so it makes sense to use average coordinate gain, not minimum coordinate gain.) thus, it seems more likely to get a (hyper)plane crossing the more dimensions there are, and so $d$ should correspondingly decrease. 

we are done. 

# extension to higher dimensions

write $\left(X_1, X_2, \cdots , X_n\right)$ in $n$-spherical coordinates, then calculate individual $x_i = 1$ plane-crossing probabilities $p_i$ for each one. the angle distributions $f_j\left(\theta_j\right)$ for the $n$-sphere can be derived similarly to the derivation of the distribution of $\varphi$ for the sphere. then integrate: 

\begin{equation}
p(d) = \displaystyle\int_{\left[0, \frac{\pi}{2}\right)^{n - 1}} \left[\sum_{i = 1}^n (-1)^{i - 1} i e_i\left(p_1, p_2, \cdots , p_n\right)\right] \prod_{j = 1}^{n - 1} f_j\left(\theta_j\right) \,dV
\end{equation}

where $e_i$ is the $i$-th [elementary symmetric polynomial](https://en.wikipedia.org/wiki/Elementary_symmetric_polynomial). then, solve $p'(d) = 0$ to get your value of $d$. 

i'm sure there's a formula for this. that would also be a way to show that long needles are bad: you can show that $p'(1) < 0$, implying that the optimal value of $d$ happens before $d = 1$. 

but, i'm sure that formula isn't pretty.