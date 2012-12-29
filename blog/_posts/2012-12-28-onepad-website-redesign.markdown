---
layout: post
published: true
title: OnePAD Website Redesign
excerpt: I redesigned the OnePAD website and upgraded it from static files to the Middleman static site generator.
---

The original [OnePAD website](http://www.onepadapp.com) was thrown together in Photoshop and [Coda](http://panic.com/coda/) over a weekend and deployed as a static website on Amazon S3. This setup has worked exceptionally well for quite a while now. I had concerns about scaling beyond a one or two page site though, and when I came across the [Middleman](http://middlemanapp.com/) static site generator, converting it over it seemed like a fun project for a Saturday afternoon.

Middleman is very simliar to (based on?) [Jekyll](http://jekyllrb.com/), the static site generator that powers Github pages and this blog, but it mixes in a very rails-like stack of helpers including support for multiple template languages. Being familiar with both Rails and Jekyll it was pretty easy to hit the ground running with Middleman and within a few hours I had the new site published.

Like everything else on the web, the site is far from "finished". I do think it's an improvement on the original iterations though.

[www.onepadapp.com](http://www.onepadapp.com)
