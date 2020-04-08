---
permalink: /csc_448_2020_spring/corrections/
title: "CSC 448 Spring 2020 Schedule"
excerpt: "Anderson Data Science Research Lab."
---

{% include base_path %}

# Table of contents
1. [Syllabus](/csc_448_2020_spring/)
2. [Schedule](/csc_448_2020_spring/schedule/)
3. [Technology](/csc_448_2020_spring/technology/)
4. [Project](/csc_448_2020_spring/project/)
5. [Deadlines](/csc_448_2020_spring/deadlines/)
6. [Deadlines](/csc_448_2020_spring/corrections/)

# Deadlines
**Disclaimer:** This page is manually updated manually, so please check the last updated date.

**Last updated date:** 4/8/20

## Lab 1
### 4/8/20
The loop stub that was provided was not correct. It has been changed from:
<pre>
def frequency_table(text,k):
    freq_map = {}
    n = len(text)
    for i in range(n-k):
    ...
</pre>
To
<pre>
def frequency_table(text,k):
    freq_map = {}
    n = len(text)
    for i in range(n-k+1):
</pre>

The autograder now expects you to correct this problem. If you have already received credit for this, it is
up to you as to whether you need to resubmit, but I recommend you do so because then you have the correct solution.
