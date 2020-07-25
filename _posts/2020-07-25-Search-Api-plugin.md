---
title: "Search-API Plugin for IDA"
layout: post
date: 2020-06-20 02:00
tag: [Reverse, CTF, Python, Malware, Analysis]
image: https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/assets/images/profile.png #
headerImage: true
projects: true
hidden: false # don't count this post in blog pagination
description: "IDA Plugin for quickly searching for an API."
category: project
author: coreflood
externalLink: false
---

# Search-API-plugin

A Simple plugin for IDA Pro that automates the process of googling an API.

It Googles the selected function name in a new tab of your default browser.

# Installation
Copy [SearchAPI.py](https://github.com/AlyaGomaa/Search-API-plugin/blob/master/SearchAPI.py) into your IDA plugins folder.

# Usage
The plugin loads automatically when an IDB is opened.

Right click on any API and choose 'Google Search'.

![](https://raw.githubusercontent.com/AlyaGomaa/Search-API-plugin/master/_screenshot.png)
