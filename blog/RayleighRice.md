+++
title = "Rayleigh-Rice Mixture Model Estimation"
date = Date(2021, 04, 01)
hasmath = true
tags = ["research", "signal processing"]
mintoclevel = 2
maxtoclevel = 2
descr = """
This model provides the means to separate the data into two classes
"""
+++

# {{ title }}

\toc

## Problem in hand
Consider the data with all positive values denoted as $s_n$ for all $n\in\{1,\cdots,N\}$. It is desired to find an unsupervised threshold $\tau_s$ such that we say for $s_n>\tau_s$ the data point $n$ belongs to a class otherwise the data point belongs to the other class. In order to find such a threshold without introducing any extra parameters we use a Rayleigh-Rice mixture model. Note that the values of $s_n$ are positive and Rayleigh-Rice mixture models these values very well. Once the Rayleigh-Rice mixture model is fitted, the intersection of the two distributions is the threshold value.

## Rayleigh-Rice mixture
Let $\alpha$ indicate the probability that a sample $s$ is from a Rayleigh distribution with parameter $b^2$ and $1-\alpha$ the probability that $s$ is from the Rice distribution with parameters $\nu$ and $\sigma^2$. Then, the distribution of the mixture model can be written as
$$
f(s|\Psi) = \alpha f_{Rayl}(s|b^2) + (1-\alpha) f_{Rice} (s|\nu,\sigma^2)
$$
where $\Psi = \{\alpha, b^2, \nu, \sigma^2\}$ and
$$
f_{Rayl}(s|b^2) = \frac{s}{b^2} \exp\left(-\frac{s^2}{2b^2}\right), \quad s\geq0
$$
and
$$
f_{Rice} (s|\nu,\sigma^2) = \frac{s}{\sigma^2} \exp\left(-\frac{s^2+\nu^2}{2\sigma^2}\right) I_0\left(\frac{s\nu}{\sigma^2}\right), \quad s\geq0
$$
with $I_0(\cdot)$ is the 0-th order modified Bessel function of first kind. Given the samples $\{s_n\}_{n=1}^N$ we want to find the parameters of this model ($\Psi = \{\alpha, b^2, \nu, \sigma^2\}$). Expectation-Maximization (EM) algorithm can be used to estimate the model parameters $\Psi$.
Following algorithm shows the steps of the EM for fitting the Rayleigh-Rice model. 

\figenv{}{/assets/blog/RaylRice/EMAlgo.png}{width:95%}

Given the fitted model parameters the threshold value is found by solving the following for $\tau_s$.
$$
\frac{\alpha}{1-\alpha} = \frac{f_{Rayl}(\tau_s|b^2)}{f_{Rice} (\tau_s|\nu,\sigma^2)}.
$$

## Sample Result
A sample histogram of $s_n$ for all $n\in\{1,\cdots,N\}$ is shown in figure. The plotted curves show the fitted Rayleigh and Rice distributions. In this example, the values of $s_n$ denote the standard deviation of electrode activity and the fitted model separates the active and inactive electrodes.

\figenv{}{/assets/blog/RaylRice/Hist.png}{width:85%}

## Matlab implementation
The matlab implementation is provided [here](https://github.com/amirhkhalilian/RayleighRiceModel).
