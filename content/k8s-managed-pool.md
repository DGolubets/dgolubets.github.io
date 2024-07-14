+++
title = "Managed K8s node pool operator"
date = 2024-07-14
weight = 0
[taxonomies] 
tags = ["rust", "k8s"]
+++

[Managed K8s node pool operator](https://github.com/DGolubets/k8s-managed-node-pool) is what I've been working on lately in my spare time. Though the motivation for that comes from my job use case.

We use Digital Ocean managed kubernetes at work.
It works great, and it's cheaper than alternatives.
But there is one limitation they have on their node pools: you can't auto-scale them to zero.

That's usually not a problem at all. For normal work pools.
But sometimes I need to run a Spark job, on beefy, expensive droplets.
You can see where that's going.

At first I used to create and destoy that node pool manually.
I quickly discovered that I often forget to shut it down.
Even though autoscaled to 1, that one pod would run and eat some money.

Then I came up with Python code that would create and destory the pool automatically when Spark app was starting or terminating. That worked much better. Though not perfect. If I wanted to apply a fix to the app I had to restart it and in it's turn that would recreate the pool, which takes time, unnecessarily.

I finally decided to solve it once and for all.
With a custom K8s operator.
