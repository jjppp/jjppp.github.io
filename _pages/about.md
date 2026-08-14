---
permalink: /
title: "About me"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
  - /cv/
  - /resume
---

{% include base_path %}

I am a second-year MSc student supervised by Prof. [Yue Li](https://cs.nju.edu.cn/yueli/) at [PASCAL Research Group](https://pascal-lab.net/), [Institute of Computer Software](https://ics.nju.edu.cn/), [Department of Computer Science and Technology](https://cs.nju.edu.cn/), [Nanjing University](https://www.nju.edu.cn).
I am broadly interested in the theory and implementation of programming languages and static analysis.

Education
======
- 2024.09 - present: Nanjing University, MSc in Computer Science.
- 2020.09 - 2024.06: Nanjing University, Bsc in Information and Computing Science.

Work Experience
======

{% include work-experience.html %}

Publications
======
  <ul>{% for post in site.publications reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>

Talks
======
  <ul>{% for post in site.talks reversed %}
    {% include archive-single-talk-cv.html  %}
  {% endfor %}</ul>

Teaching
======
  <ul>{% for post in site.teaching reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>
