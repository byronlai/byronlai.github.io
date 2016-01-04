---
layout: post
title:  "C Optimization Techniques"
date:   2015-08-10 20:20:10 +0800
categories: jekyll update
---
When optimizing a piece of software, we are often told that micro-optimization is evil and we should focus on macro-optimization. However, sometimes you come to a point that you no longer can improve the asymptotic complexity of the algorithm and the performance is still suboptimal. In this case, I think micro-optimization has its values. Here are some C optimization techniques that I found useful from my experience in a previous project.

## Branch Prediction Hints

{% highlight c %}
if (__builtin_expect(x, 0))
    foo();
{% endhighlight %}

To achieve high performance, modern CPUs have a pipeline architecture. Multiple instructions are executed at the same time. The CPU uses branch prediction to guess which instruction will be executed next. If the prediction is wrong, it will stall the pipeline. In GCC and other compilers, we can use the `__builtin_expect()` function to provide compiler with branch prediction information. For example, if you know that `x` is will be false 90% of the time, you can write `__builtin_expect(x, 0)`. This means that we expect `x` to be false. In my experience, the performance gain will be around 5% depending on whether the code segment is a hotspot.

## Data Prefetching

{% highlight c %}
__builtin_prefetch(ptr, 0, 1);

/* Do something */

while (*ptr)
    /* Do something on ptr */
{% endhighlight %}

Accessing from memory is slow compared to accessing the CPU cache or registers. If you know that you are going to use a piece of memory in the future, you can ask the CPU to prefetch the memory first. This can minimize cache-miss latency. The second argument (zero) of `__builtin_prefetch()` means that the prefetch is preparing for a read and the third argument (one) means that the data has a low degree of temporal locality.

## Function Inlining

{% highlight c %}
inline int foo(int a, int b) {
    /* Do something */
}
{% endhighlight %}

Function calls are not cheap. Whenever you invoke a function, the CPU has to save all the registers to the stack and push the arguments to the stack. When the function returns, it has to restore all the registers. To reduce the overhead, you can qualify the function with the `inline` keyword. The compiler will then perform an inline expansion at the position of the function call, thus reducing the overhead.

## SIMD Instructions

Most modern CPUs include SIMD instructions to improve the performance. For example, most Intel CPUs have the MMX and SSE instruction sets. These SIMD instructions allow you apply the same operation on multiple data points at the same time. The [Intel Instrinsics Guide](https://software.intel.com/sites/landingpage/IntrinsicsGuide/) has very detailed information about all SIMD instructions available.
