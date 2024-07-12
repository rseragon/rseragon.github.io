---
title: "SleepSort: The zenith of sorting algorithms"
date: 2024-07-01 10:00:03 +0530
categories: [Algorithms, beginner]
tags: [Sleepsort, Algorithms]
image: https://cdn.jsdelivr.net/gh/rseragon/blog-assets@main/sleep-sort/anim/tanya_beyond_science.png
---

<details>
<summary>
What do programmers and polar bears have in common?
  </summary>
<b class="margin-left: 4px;">We sleep a lot!</b>
</details>

<br />

A cliche, nonetheless if you are a programmer you definitely had a **Eureka!** moment
during sleep or whilst taking a bath or while doing an mundane task. These moments define us, they give 
us the reason to pursue programming. Ironically, sleepsort is one such kind of invention.<br/>


A 4Chan invented[^4chan] sorting algorithm that, theoretically, runs on **O(1)** space & **O(k)** time complexity.<br/>

*What?*\
Yeah, you heard me right! Can't believe me? take a look at the implementation below.


### Bash
> Not comfortable with Bash? Have a look at other implementations [here](https://rosettacode.org/wiki/Sorting_algorithms/Sleep_sort)
{: .prompt-info }
```bash
#!/bin/bash
function f() {
    sleep "$1"
    echo "$1"
}
while [ -n "$1" ]
do
    f "$1" &
    shift
done
wait
```

Running it output's the following
```console
$ bash sleepsort.sh 3 2 1 
1
2
3
```
Amazing, right?

Even though, theoretically, it is sorting the given list of `n` numbers in `k` time, the `k` isn't a constant. Here, `k` represents the **biggest number** in the given list.
For example, if we were to sort an array which has the number 99999 in it. It's gonna take minimum 99999 seconds to sort the array. Absurd, isn't it? <br />

*So, how the hell is it at the zenith of sorting algorithms?* \
If you remember about the "Eureka!" I've mentioned at the start, that's how I felt when I first looked at its implementation. Confused, baffled, perplexed at this uncanny atrocity.
Most of all, **it works!** All it does is sleep. No complex merging with recursion or finding a pesky pivot. **IT'S LITERALLY SLEEPING**. I cannot emphasize the word sleeping anymore.

With this troubling my mind, I've dug deeper into this rabbit hole. I've uprooted an interesting approach to solve the problem of sleep in sleepsort. How about we map
the numbers through a function and reduce them into a smaller and more acceptable range for sleeping. Wouldn't that solve the 99999 problem?

For sake of my sanity, let's just entertain this approach for a bit.

# Improving SleepSort

Imagine we have an array of numbers [1, 100, 231, 444, 555, 766\]

And now plug them into this formula



<!-- For some reason, the normal mathjax isn't working -->
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>


<p>
$$
  f(x) = {\sqrt x \over max (N)}
$$
</p>

Here,\
$$ x $$ is an element of the list whose domain is [$$ x \in [0, \infty) $$](#2-root-of-negative-number)\
$$ max(N) $$ represents the largest number in the list $$ N $$. 


A rudimentary question arises here, why divide it with $$ max(N) $$? \
That is because I had to scale down the numbers to incur the least sleep time. This was essential as just the root of the number ($$\sqrt x$$) isn't necessarily enough to bring the number to a palatable range.


Plotting those numbers on the co-ordinate line and transforming them through $$ f(x) $$, gives us the following graph. 


<center>
    <img src="https://cdn.jsdelivr.net/gh/rseragon/blog-assets@main/sleep-sort/anim/scaled_numbers.gif" width="720"/>
</center>

> NOTE: The scaled numbers were adjusted to be visible in the animation 
{: .prompt-info}

Here, the $$ Y $$ axis is only considered to show the transformation scale of the function.

# Execution!
Time to put theory aside and be true realists. Let's write the code and execute it!
Given below is python implementation of the following idea


```python
#!/usr/bin/env python3
from asyncio import run, sleep, wait, Task
import math


async def f(n, t):
    await sleep(t)
    print(n)


def my_func(n, max):
    return math.sqrt(n) / max



async def runner():
    numbers = [1, 100, 231, 444, 555, 766]

    print(numbers)
    maximum = max(numbers)
    reduced_numbers = [my_func(i, maximum) for i in numbers]
    print(reduced_numbers)
    tasks = [Task(f(n, t)) for n, t in zip(numbers, reduced_numbers)]
    await wait(tasks)


if __name__ == "__main__":
    run(runner())

```

```console
Original Numbers: [1, 100, 231, 444, 555, 766]
Scaled Numbers: [0.0013054830287206266, 0.013054830287206266, 0.019841624221371625, 0.027508234341652057, 0.030755140964464092, 0.036131468676496206]
1
100
231
444
555
766

```

It works! Great, isn't it? In fact this works on $$ O({n \over \sqrt m})  $$ time complexity. Where $$ m $$ is the maximum number in the list.

The derivation is quite simple as shown below.

Consider $$ n $$ to be the total number of elements in the list $$ L $$ and $$ m $$ to be the maximum element in the list i.e. $$ max(L) = m $$.

For the worst case with the sleep time $$ m $$, the function outputs as follows

$$ 
  f(m) = {\sqrt m \over max (L)} = {\sqrt m \over m} = {1 \over \sqrt m}
$$

From this we can conclude, that the function, at it's worst case will sleep for at least $$ {1 \over \sqrt m} $$ seconds.

And for the worst case scenario calculation, if all the elements in the list are maximum i.e. the list is made up of $$ n $$ elements with the value of $$ m $$ ($$ [m, m, m, ...(n)times] $$), then Big O notation for each element would be $$ O({ 1 \over \sqrt m}) $$.

So, for the entire array in the worst case, the Big O expression would be.

$$
O({n \over \sqrt m})
$$


## Caveats
To be honest, $$ O({n \over \sqrt m}) $$ isn't the correct time complexity of the algorithm. In fact, it's $$ O(n) $$. Though, it's only for a few cases.

This is the beauty of mathematics. That is because, if the denominator in the fraction is under 1, then the total result would be greater than the numerator, conversely, if the denominator is greater than 1, then the result would be lesser than the numerator (Strictly speaking for positive numbers). 

Now, if we consider the entire algorithm, there are two parts to it
1. Find out the maximum number is $$ O(n) $$.
2. Sleep for every number is $$ O({n \over \sqrt m}) $$.

So, the worst case scenario would be 

$$ 
 O(n) + O({n \over \sqrt m}) = O(n) 
$$

Since, $$ O(n) >= O({n \over \sqrt m}) $$

To summarize,

$$
  Time Complexity =
\begin{cases}
O(n),  & \text{if $n$ > 1} \\
O({n \over \sqrt m}), & \text{if $0 < n < 1$ }
\end{cases}
$$

But, this is quite speculative on algorithmic standpoint.

# Why does it fail?

### 1. Non-deterministic execution anomalies
The most pressing reason for it's failure is because of its *non-deterministic execution anomalies*. BIG WORDS, aren't they?
To be direct, we just can't determine its execution sequence. As threads are maintained by the kernel and we don't control
which threads executes first and which doesn't. That means it's impossible to predict the sequence of execution if the difference of sleep between threads is infinitesimally small.

To be honest, it's like an imitation of BogoSort[^bogosort] but more systematic.
In short, we pray for the entropy of randomness to be on our side during execution and expect it to yield favorable results. 

### 2. Root of negative number
Since the formula contains square root of the given number, negative numbers would cause problem to digress into the complex plane. Additionally, the denominator
representing the maximum number would skew the result towards the positive side disregarding the negative numbers. Therefore, this formula loses its essence if negative numbers are involved.


### 3. Sparse or non-continuous array input
If the data is spread sparsely, then it might cause the formula to bias towards the bigger numbers leaving the smaller numbers to the dust (as the division would put the smaller numbers very close to 0). 
This would increase the chances of non-determinism in the execution of the threads leading to inadvertent flaws in the output.


Well, that's the end of it. Was this article useful or a faux pas, I don't know. I'll let you decide.

# References
[^4chan]: [TinyChan](https://archive.tinychan.net/read/prog/1295544154)
[^bogosort]: [Wikipedia](https://en.wikipedia.org/wiki/Bogosort)
