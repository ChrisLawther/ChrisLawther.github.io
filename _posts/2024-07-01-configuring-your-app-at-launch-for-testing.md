---
layout: post
title:  "Configuring your app at launch"
date:   2024-08-06 21:55:52 +0100
categories: ios swift testing testability
---

Two (3?) reasons:

 1. You want your tests to have control over the data the app sees
 1. You want ... (dev-mode?)

 Makes use of two features of `UserDefaults`: default values and command-line overrides.

