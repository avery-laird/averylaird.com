---
date: 2016-03-14 22:03:54+00:00
layout: post
categories: 
title: Matplotlib 3d
---

    ::python
    %matplotlib notebook
    import matplotlib
    #matplotlib.use('webagg')
    import matplotlib.pyplot as plt
    from mpl_toolkits.mplot3d import Axes3D
    import random
    import sys
    import numpy


    def procedure(sampleSize=20):
        A, B = {}, {}
        for x in range(sampleSize):
            A[x] = x
        for x in range(sampleSize):
            t = random.randint(1, sampleSize)
            if t not in B.values():
                B[x] = t
            else:
                while t in B.values():
                    t = random.randint(1, sampleSize)
                B[x] = t

        for key, value in A.iteritems():
            if B[key] == value:
                return 1
        # Another way to check for uniqueness is simply to
        # check if B[x] = x for all x (relies on A being ordered)
        return 0

    def trial(sampleSize, iterations):
        t = []
        for x in range(iterations):
            i = procedure(sampleSize)
            t.append(i)
        return t


    # Generate as a function of fixed interations(100), variable sample size (0-30):
    def generate(iterations, sampleSize):
        x, y, z, a = [], [], [], []
        for t in range(2, sampleSize+2, 2):
            for m in range(2, iterations+2, 2):
                l = trial(t, m)
                z.append(l.count(0)*1.0/len(l))
                x.append(t)
                y.append(m)
                a.append(sum(z)*1.0/len(z))
        return [x, y, z, a]
    # Construct Array


    #print len(x), len(y), len(z)
    fig = plt.figure()
    ax = plt.axes(projection="3d")

    ax.set_xlabel("Sample Size")
    ax.set_ylabel("Iterations")
    ax.set_zlabel("No Matches vs. total iterations")
    l = generate(100,30)
    ax.scatter(l[0], l[1], l[2])


    plt.show()
