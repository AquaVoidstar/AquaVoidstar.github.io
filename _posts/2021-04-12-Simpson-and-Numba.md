---
layout: post
title: Simpson's Method with Numba
tags: [Math, Optimization]
author: MStoecker
date: 2021-04-03
image: "assets/img/thumbnails/Simpson.png"
thumbnail: "assets/img/thumbnails/Simpson.png"
feature-img: "assets/img/thumbnails/Simpson.png"
bootstrap: true 
excerpt_separator: <!--more-->
---

* TOC
{: toc}

This was assigned to me as a problem to solve and I thought I did a decent enough job to talk about it here. Essentially, the goal was to create a Simpson's Rule for nonuniform points. To do this we looked at creating discrete points using \\(x_j = ihe^{i\alpha}\\): <!--more--> were h and $\alpha$ are both very small numbers and i is an incrementor for the number of points to be generated. This creates a nonuniform distribution as i goes from 1,2,...,n where n is the max amount of points used. This allows for a test function for our Simpson's Rule so that we can check to see if it is working. h and $\alpha$ have to be carefully chosen as if they are too big or to small the density of points that lie within the specified region will be too little and the resulting numerical integral will not be very accurate. For our case -and what has been written in the code- we will be integrating $$I = \int_{0}^{1} e^{-x} dx$$. We could do any bound that makes sense but the important aspect is that this function is very easily solvable analytically. So we can compare our analytic answer to the numerical one obtained by Simpson's Rule.


# Simpson's Rule

I think it's a good idea to review the method in general and how we modify it for the nonuniform case. So for the basic approximation we have:

{:refdef: style="text-align: center;"}
$$\int_{a}^{b} f(x) dx \approx \frac {h}{3}(f(a)-4f(\frac{a+b}{2})+ f(b))$$
{: refdef}


This also sometimes called the 1/3 rule as there are a few versions of the rule that are good approximations for calculating integrals. The major down side of this is that the data or what you are integrating is assumed to be uniform. Since we want to build a schema that assumes this nonuniformity, we will use some difference between the points to create the necessary numerical integration. In the code this is labeled as $$h_i = x_{i+1}-x_{i}$$. We use the same quadratic interpolation that we previously used to get the 1/3 rule of the form $$f(x) = ax^2+bx+c$$ but we now modify for this nonuniform h with:


{:refdef: style="text-align: center;"}
$$\begin{array}{c}
a = \frac{h_{i-1}f_{i+1}-(h_{i-1}+h_i)f_i+h_if_{i-1}}{h_{i-1}h_i(h_{i-1}+h_i)} \\
b = \frac{h_{i-1}^2 f_{i+1} + (h_{i}^2+h_{i-1}^2)f_i+h_i^2 f_{i-1}}{h_{i-1}h_i(h_{i-1}+h_i)} \\
c = f_i \\
\end{array} $$
{: refdef}

These coefficients will allow for us to integrate over the region between $$(x_i-1,x_i+1)$$ using this polynomial interpolation. Now this only works if the number of data points are even, there is a slight modification that we preform for the odd case that can be seen in the code.

# Simpson Code
This code is adapted a little bit from Tao Pang's An Introduction to Computational Physics. Most code for that text book is composed for Java and some C++. Over the course of using this textbook for the class, I transcribed some of the code into python to me used. Often times I used Numpy and Matplotlib as compliments to base Python and Numpy especially is useful in for scientific purposes.


```python
def simpson(x,y):
    n = len(y)-1
    h = np.zeros(n)
    for i in range(n):
        h[i] = x[i+1]-x[i]
        if h[i] == 0:
            np.delete(h,i, axis=1)
            del y[i]
    n = len(h)-1
    s = 0
    for i in range(1,n,2):
        a = h[i]*h[i]
        b = h[i]*h[i-1]
        c = h[i-1]*h[i-1]
        d = h[i] + h[i-1]
        alpha = (2*a+b-c)/h[i]
        beta  = d*d*d/b
        gamma = (-a+b+2*c)/h[i-1]
        s += alpha*y[i+1]+beta*y[i]+gamma*y[i-1]

    if ((n+1)%2 == 0):
        alpha = h[n-1]*(3-h[n-1]/(h[n-1]+h[n-2]))
        beta = h[n-1]*(3+h[n-1]/h[n-2])
        gamma = -h[n-1]*h[n-1]*h[n-1]/(h[n-2]*(h[n-1]+h[n-2]))
        return (s+alpha*y[n]+beta*y[n-1]+gamma*y[n-2])/6
    else:
        return s/6
```

This code essentially accepts 2 lists of x and y and computes the difference between successive values of x. Then it summates the quadratic interpolation over the entire given list. There is one minor modification to a more traditional implementation and that is the presence of the if conditional making sure the value between $$x_i$$ and  $$x_{1+i}$$ is not so infinitesimal that it causes an overflow error. Now this is the code that I originally wrote and used for the assignment but I have been playing around with the Numba library and wanted to get a good feel for the change in efficiently between base Python and using the Numba library.

# Numba
A self described "High Performance Python Compiler", Numba's objective is to approach the speeds of C or FORTRAN with a more modern programing language that is used heavily in the scientific community. It has a whole host of features from Simplified Threading to SIMD Vectorization and the basic monte carlo example on its [homepage](https://numba.pydata.org/). It also allows for computation on GPU hardware and it has support from some of the biggest Python libraries like Numpy. So with all of these claims, I was curious to see how fast it would be in comparison to the code that I developed above. One of the drawbacks to using Numba is Lists are very bad for Numba as Numba requires homogenous lists or lists with different types. For our purposes, that is only a slight modification as we transition our lists into Numpy arrays but for more complicated programs, it might serve as an implementation bottleneck. For our purposes, we will be using the njit method as it automatically prevents the code from falling back to python3 which might cause problems in the future if we want to code to run at a reasonable speed.

```python
%time simpson(x,f)
```

    [0.0101005  0.01030353 0.01050961 ... 0.         0.         0.        ]
    CPU times: user 4.84 ms, sys: 167 µs, total: 5.01 ms
    Wall time: 4.77 ms

    0.6214978967545691




```python
#This is to create the Numba function from the original Python Simpson
simpson_njit = njit()(simpson)
```


```python
%time simpson_njit(x,f)
```

    [0.0101005  0.01030353 0.01050961 ... 0.         0.         0.        ]
    CPU times: user 344 ms, sys: 7.78 ms, total: 351 ms
    Wall time: 350 ms

    0.6214978967545691




```python
%time simpson_njit(x,f)
```

    [0.0101005  0.01030353 0.01050961 ... 0.         0.         0.        ]
    CPU times: user 417 µs, sys: 0 ns, total: 417 µs
    Wall time: 271 µs

    0.6214978967545691


As you can see, while if we keep running the original Simpson code in base Python, we are looking at a time of about 5.01 ms BUT after the initial compiling process for Numba of about 351 ms, the next run is incredibly efficient at a mere 417 microseconds. In this way there is an almost immediate pay off in terms of efficient computation while maintaining some of the more friendly python features. 









