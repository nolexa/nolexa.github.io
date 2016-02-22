---
layout: post
title: "Visualizing a cloud of skills with Python"
modified:
categories: blog
excerpt: ""
tags: [python, tag cloud]
image:
  feature:
date: "2016-02-22"
comments: true
share: true
---

As a consultant and a freelance programmer, I need to be aware of which are the most demanded skills in my area of interest.
What I did until now was scanning contract openings trying to remember how often certain skills appear in the specifications.
But people are bad intuitive statisticians, a scientific fact proven by [Daniel Kahneman][1]. At least I am. 
It is hard for me to objectively aggregate statistic information in my head, and it's kind of boring to keep notes and crunch the numbers manually. 
It's much more fun to have a picture, 
which, as they say, is worth a thousand words. So I wanted to visualize the skills demands as a [tag cloud][3]. 

## Input

First I dump all skills specifications into a file `skills.txt`. Skills are comma or slash separated. 
Leading and trailing dashes and spaces are insignificant. 

Here is a sample: 

{% highlight text %}
- C++
- Forex/FX, J2EE, Spring, Hibernate, REST
- Clojure
- Scala

Java Enterprise
JEE/J2EE
Hibernate
Spring
Struts

Java 7, Java 8, Agile, TDD, NoSQL
Kafka, Scala Test, Scala, Python, Hazelcast, Cassandra, Docker
{% endhighlight %}

## Script

A cloud image is generated with a small `Python` script using a package [wordcloud][2], 
which has to be installed either with `pip` 

{% highlight bash %}
pip install wordcloud
{% endhighlight %}

or from the [source][2].
  
Getting a cloud image with [wordcloud][2] is pretty straightforward.
The `generate_from_frequencies` method expects a list of tuples, where the first member of a tuple is a term, 
and second is a term's frequency.
For example: `[('java', 15), ('scala', 12), ('spring', 12)]`.
  
There are two ways to assign weights to the terms: by rank and by frequency. 
Parameter `relative_scaling` defines a ratio by which these two strategies are mixed.
With `relative_scaling=0`, only ranks are considered.
With `relative_scaling=1`, only frequencies are considered.
In the example above, *java* has rank of *1*, *scala* - of *2*, *spring* - of *3*, 
whereas frequencies are distributed as *15, 12, 12*.
 
When frequency distribution is smooth, with many terms occupying same frequency, 
the cloud looks better if the frequency-based strategy gets more influence, so that differences in font size would not be huge.
In this example `relative_scaling` is set to *0.8*.


{% highlight python %}
import re
from collections import Counter

from wordcloud import WordCloud


lines = file("skills.txt", "r").read().splitlines()
words = []

# split terms separated by commas and slashes
for line in lines:
    words.extend(re.split("[,/]+", line))

# strip any leading and trailing whitespaces and dashes
words = map(lambda s: s.strip(' -').lower(), words)

# remove stopwords and empty strings
stopWords = {"agile", "tdd", "bdd"}
words = filter(lambda s: s and s not in stopWords, words)

# count frequencies
cnt = Counter()
for word in words:
    cnt[word] += 1

# generate a cloud image prioritizing frequency over rank with weight 0.8
wordcloud = WordCloud(width=800, height=600, relative_scaling=.8)\
    .generate_from_frequencies(cnt.items())
wordcloud.to_file("skills-cloud.png")

# Display the image. Remove the lines below if you don't have matplotlib installed 
# and don't want the interactive display. 
import matplotlib.pyplot as plt

plt.imshow(wordcloud)
plt.axis("off")
plt.show()
{% endhighlight %}

## The image

The generated image is saved to a file and displayed on screen. 
The lines displaying the image can be removed if interactivity is not needed, and the `matplotlib` package is not installed.   

<figure>
	<img src="/images/2016-02-22/skills-cloud.png" alt="image">
	<figcaption>Generated cloud image</figcaption>
</figure>

## More things to do

- Ignore comment lines starting with `#`. Keep dates, job details in comments.
- Read multiple files.

## Links

1. [Daniel Kahneman][1]
2. [wordcloud on GitHub][2]
3. [Tag cloud][3]

[1]: https://en.wikipedia.org/wiki/Daniel_Kahneman
[2]: https://github.com/amueller/word_cloud 
[3]: https://en.wikipedia.org/wiki/Tag_cloud
