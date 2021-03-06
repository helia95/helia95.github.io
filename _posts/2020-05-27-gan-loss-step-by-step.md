---
#title: "Post: Modified Date"
#last_modified_at: 2016-03-09T16:20:02-05:00
title: GAN loss step by step
#categories:
#  - deep learning
#tags:
#  - GANs
#  - generative models
---
### Intro

Generative adversarial networks, are a generative model approach. They are based on a game theoretic assumption as 
generative adversarial neural networks can be formalized as a two-players min-max game, where the competitors are called *generator*, $G$, and *discriminator*, $D$. 
The generator is trained to learn a distribution $p_g$ over data $\mathbf{x}$, where the prior is represented by a variable $\mathbf{z} \sim p_z(\mathbf{z})$ mapped to the data space (i.e. the space of  $\mathbf{x}$) by a function $G(\mathbf{z}, \theta_g)$. 
On the other side the discriminator $D(\mathbf{x}, \theta_d)$, takes data $\mathbf{x}$ as input and outputs a scalar value that represents the probability of $\mathbf{x}$ coming from the real distribution $p_r$ or from the generated one $p_g$. \\
We train $D$ to maximize the probability of correctly classify the input samples, while G is trained to fool the discriminator.
The solution of this game is a saddle point (or Nash equilibrium). If the two networks converges, than the samples generated by $G$ will become indistinguishable for the discriminator, that starts outputting 0.5 for every samples (regardless the fact that they come from real/fake data).

### Objective

The min-max game can be formalized with the following objective function

$$
\begin{equation}
    \min_G \max_D V(G, D) =   \mathbb{E}_{x \sim p_r(x)} \log D(x) + \mathbb{E}_{z \sim p_z(z)} \log(1- D(G(z)))
    \label{eq:loss_gan}
\end{equation}
$$

We can fix one of the two player and analyze the behaviour of the other. Starting from $D$

$$
\begin{equation}
    \max_D \Big[ \mathbb{E}_{x \sim p_r(x)} \log D(x) + \mathbb{E}_{z \sim p_z(z)} \log(1- D(G(z))) \Big]
    \label{eq:loss_D}
\end{equation}
$$

so, we would like to have 


1. $D(x) \longrightarrow 1$, ($\log(1) = 0$).
2. $D(G(z)) \longrightarrow 0$, $\log(1-0) = 0$


to maximize the equation, i.e. we want to correctly classify the samples assign to real samples the label 1 and to fake samples the label 0. \\
On the other side, if we fix $D$ and optimize $G$

$$
\begin{equation}
    \min_G \Big[ \mathbb{E}_{z \sim p_z(z)} \log(1- D(G(z))) \Big]
\end{equation}
$$

* $D(G(z)) \longrightarrow 1$, $\log (1-1) = - \infty$

The training procedure works roughly as follows, starting form $G$ and $D$


1. Fix $G$ and train $D$ until optimality, i.e. the best it can do to recognize real and fake samples.
2. Update $G$ based on make harder for $D$ (which is fixed) to recognize the distribution of samples.

We can prove that this procedure converges (given the two functions have enough capacity) furthermore it converges to $p_g = p_r$.

### Optimal Discriminator
Fixed $G$, the optimal discriminator is

$$
    \begin{equation}
        D^*(x) = \dfrac{p_r(x)}{p_r(x) + p_g(x)}
    \end{equation}
$$

*Proof*. Starting from $\eqref{eq:loss_D}$ and writing properly the expectations, the problem we want to solve is

$$
\begin{equation}
    \begin{split}
        \max_{D}V(G,D) & = \int_x p_r(x) \log(D(x)) dx + \int_z p_z(z) \log(1 - D( G(z) ) ) dz \\
                        & = \int_x p_r(x) \log(D(x)) dx + \int_x p_g(x) \log(1 - D(x) ) dx \\
    \end{split}
\end{equation}
$$

Then, the maximum of $f(y) = a \log y + b \log(1-y)$ in [0,1]. \\

The second passage of the above proof comes form the Jacobian formula. Given a random variable $X$ and a function $f: x \rightarrow y$, it is possible to compute the pdf of $Y$ as:

$$
    |p_Y(y) dy| = |p_X (x) dx|
$$

or:


$$
\begin{equation}
    \begin{split}
    p_Y(y) & = \Big| \dfrac{dx}{dy} \Big| p_X(x) \\
            & = \Big| \dfrac{df^{-1}(y)}{dy}  \Big| p_X(x)
    \end{split}
\end{equation}
$$

In this specific case we have:

$$
\begin{aligned}

    G: z & \rightarrow x \\
    p_Z(z) & = \Big| \dfrac{dx}{dz} \Big| p_g(x)

\end{aligned}
$$

plugging this into the second term of the objective function

$$
\begin{aligned}

    \int_z p_z(z) \log(D(G(z)) dz & = \int_x p_g(x) \Big| \dfrac{dx}{dz} \Big| \log(1 - D(x)) d z \\
            & = \int_x p_g(x) \log(1 - D(x)) dx
\end{aligned}
$$

### Optimal Generator

Once the discriminator is fixed to be the optimal one, we can prove that minimizing $V(G, D^{\*})$ leads to $p_g = p_r$ and the minimum is equal to $- \log4$. \\
*Proof* Let's rewrite the problem we need to solve:

$$    
\begin{equation}
   \min_{G} V(G, D^*) = \mathbb{E}_{x \sim p_r(x)} \log \dfrac{p_r(x)}{p_r(x) + p_g(x)} + \mathbb{E}_{x \sim p_g(x)} \log \dfrac{p_g(x)}{p_r(x) + p_g(x)}
    \label{eq:generator_opt}
\end{equation}
$$

Now, assume $p_g = p_r$. We can see that $V(G^{\*},D^{\*}) = \log \frac{1}{2} + \log \frac{1}{2} = - \log4$ and we can show that this is the best possible value reach if and only if $p_g = p_r$. \\
To show it, let's add and subtract $- \log4$


$$
\begin{split}
V(G, D^*) & = \mathbb{E}_{x \sim p_r(x)} \log \dfrac{p_r(x)}{p_r(x) + p_g(x)} + \mathbb{E}_{x \sim p_g(x)} \log \dfrac{p_g(x)}{p_r(x) + p_g(x)} + \log4 - \log4 \\
    & = \mathbb{E}_{x \sim p_r(x)} \log \dfrac{p_r(x)}{\frac{1}{2}(p_r(x) + p_g(x))} + \mathbb{E}_{x \sim p_g(x)} \log \dfrac{p_g(x)}{ \frac{1}{2} (p_r(x) + p_g(x)) } - \log4 \\
    & = - \log4 + KL \Big(p_r \Big|\Big| \dfrac{p_r + p_g}{2} \Big) + KL \Big(p_g \Big|\Big| \dfrac{p_g+p_r}{2} \Big) \\
    & = -\log(4) + 2 \cdot JS(p_r || p_g)
\end{split}
$$

where $KL(\cdot)$ is the Kullback-Libeler divergence, by definition 

$$
KL(p(x) || q (x)) = \mathbb{E}_{x \sim p_x} \log \dfrac{p(x)}{q(x)} 
$$

$$
\begin{aligned}
-KL(p(x) || q(x)) & = - \sum_x p(x) \log \frac{p(x)}{q(x)}  \\
                  & = \sum_x p(x) \log  \frac{q(x)}{p(x)}   \\
                  & \leq  \log \sum_x p(x)\frac{q(x)}{p(x)} \\
                  & = \log \sum_x q(x) = 0
\end{aligned}
$$

while $JS(\cdot)$ is the Jansen-Shannon divergence, that can be defined as

$$
JS(p(x) || q(x)) = \frac{1}{2} KL (p(x) || q(x)) + \frac{1}{2} KL (q(x) || p(x))
$$


Since Jansen-Shannon Divergence is always positive and it is equal to zero if and only if the two distributions are the same, we proved that the optimal $G$ will produce $p_g = p_r$ and the global minimum is equal to $-\log 4$.