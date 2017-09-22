---
date: 2016-03-12 22:10:46+00:00
layout: post
categories: 
title: Site Maintenence and Improvement
---
This is just a quick post to explain some recent work. I've been making some changes to the site the past couple days, mostly in the backend. While I'm still working some of the kinks out, there might be issues, so please bear with me. If you do have any trouble, please email me at <laird.avery@gmail.com>. Let me quickly go over some of the changes I've made:

1. Integrated Thebe
2. Upgraded jQuery version
3. Minor style changes

### Thebe

For quite some time now, I've been wanting to streamline my workflow regarding blog posts. I use IPython almost exclusively when testing and creating my projects, so I wanted a way to smoothly transition from creating a project in notebook to published product. This in itself is not too difficult, however I also wanted users to be able to excecute and edit the cells themselves. After seeing how the folks at O'Reilly Media managed to do this in their eBooks, I did some research and found [Thebe](https://github.com/oreillymedia/thebe). I took some time to try and implement it for my own purposes, and so far I'm very happy with the result (I hope you'll be too). I plan to write a guide to doing this yourself, so look out for that over the next couple weeks. Until then, here are some of the things it can do:

#### Simple plots:

    %matplotlib inline 
    from matplotlib import pyplot as plt
 
    y = [x**2 for x in range(1, 100)]
    x = [x for x in range(1, 100)]

    plt.plot(x,y)

#### 3d plots:

    from mpl_toolkits import mplot3d
    import numpy as np
    
    fig = plt.figure()
    ax = plt.axes(projection='3d')

    ax = plt.axes(projection='3d')

    # Data for a 3D line
    zline = np.linspace(0, 15, 1000)
    xline = np.sin(zline)
    yline = np.cos(zline)
    ax.plot3D(xline, yline, zline, 'gray')

    # Data for 3D scattered points
    zdata = 15 * np.random.random(100)
    xdata = np.sin(zdata) + 0.1 * np.random.randn(100)
    ydata = np.cos(zdata) + 0.1 * np.random.randn(100)
    ax.scatter3D(xdata, ydata, zdata, c=zdata, cmap='Greens');

And much more!

### Upgraded jQuery Version, Minor Style Changes

Fairly self explanatory. The load time of the site might be slow for the next bit while I get around to switching from a CDN. The font size of blog posts has been decreased slightly, and I did a few things to tweak the styles for mobile devices.