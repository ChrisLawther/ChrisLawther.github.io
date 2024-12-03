---
layout: post
title:  "LetsEncrypt SSL certificates for nginx in Docker"
date:   2024-10-01 12:00:00 +0000
categories: docker
excerpt_separator: <!--more-->
---

nginx is a commonly used reverse-proxy which serves as a front-end for web services, allowing requests to easily be routed to different back-end services based on their URL (e.g. nginx could handle requests for static content, while directing API calls to another service). As the public-facing interface to your service, it also serves as a convenient "SSL terminator", relieving your services of that responsibility.

Historically, acquiring SSL certificates was expensive, but for the last 10 years LetsEncrypt has offered a free solution.

The need to periodically refresh certificates complicates things, especially if you are using Docker to containerise your (micro-)services. Googling for a solution offer turns up a particular solution, but discussion on letsencrypt forums don't recommend it.