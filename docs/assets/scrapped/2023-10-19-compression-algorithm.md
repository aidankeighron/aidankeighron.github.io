---
layout: post
title: Compression Algorithm
date: 2023-10-14 12:00:00
categories: [java, FRC]
---

# Project Overview

I wanted to create a compression algorithm, and I couldn't just make a regular algorithm. So created an algorithm that "can" take anything at any size and reduce it to a few kilobytes. 

The code shown here has been modified to make it easier to explain. If you would like to check it out, the full code is available on GitHub: [Compression Algorithm](https://github.com/aidankeighron/Compression-Algorithm){:target="\_blank"}

<details>
<summary><b>Table of Contents</b></summary>
<ul>
<li><a href="#project-overview">Project Overview</a></li>
<li><a href="#why">Why</a></li>
<li><a href="#code-showcase">Code Showcase</a></li>
<li><a href="#conclusion">Conclusion</a></li>
</ul>
</details>

# Why

As you can tell from my [file calculator](https://aidankeighron.github.io/posts/file-calculator/) I like to make things more complicated than they need to be and this is no exception. This was not developed for practical use, I created it because I thought I would be a fun thing to make. I'm not lying when I say it can take anything at any size and reduce it to a few kilobytes, but it does take a bit as you will see.

# Code Showcase

I will be compressing images but this can be used to compress anything. First we covert the image to a

```python
def compress(image):
    image_string = ""
    for row in image:
        for colum in row:
            image_string += str(colum)
            
    iterations = 0
    random.seed(1)
    while True:
        iterations += 1
        print(iterations)
        key = ""
        for _ in range(0, len(image_string)*2):
            key += str(random.choice([0,1]))

        if image_string in str(key):
            index = str(key).find(image_string)
            length = len(image_string)
            return zero(index)+zero(length)+zero(iterations), image_string
```

# Conclusion

Full code: [Compression Algorithm](https://github.com/aidankeighron/Compression-Algorithm){:target="\_blank"}ce it to a few kilobytes. 