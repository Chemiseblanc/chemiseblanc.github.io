---
title: "Introducing Latitude"
date: 2021-10-22T17:14:51-04:00
draft: false
tags:
 - hugo
 - webdev
#resources:
#- name: cover
#  src: relative/from/here.jpg
---
Hello, World!

I decided to give my long neglected homepage a makeover (ignoring my other long neglected websites) but instead of just picking
something interesting from the [Hugo theme list](https://themes.gohugo.io/) I decided to go all in and make a new theme from scratch.
<!--more-->
After looking at some other themes for inspiration I decided to challenge myself and go with the concept of a theme with a horizontally scrolling homepage.
It took a couple days of learning how to correctly massage flexbox into displaying how I wanted I came up with a general design which grew into the theme showing on the website now.

One of my goals was to have a theme that could adapt to different types of posts. Looking at the main types of social media I was familiar with I came up with three categories of content
- Shortform text posts ie. Twitter
- Longform text like traditional blogs
- Image posts with or without captions

Instead of using seperate hugo archetypes for each category I came up with a single template that adapts based off a combination of the content of a post and the supplied metadata.

I've been able to keep the theme almost entirely javascript free but I did include a small snippet on the homepage for translating vertical mouse scrolling into horizontal for ease of use.

My plan is to fine tune the theme over the next few weeks but by and large it is essentially complete.

Overall, developing a theme for hugo was very pleasant once I understood the template lookup order and the built in image processing and sass compiler are wonderful to use.

If I were to add anything else it would be:
- better support for defining theme settings
- being able to handle SVG files with the image processing, mostly as a pass-through but with a way to integrate minification without treating them as a seperate resource like css
