---
layout: post
title: Rails - routes
permalink: path-vs-url.html
categories: ruby rails url routes
---

## Small notes

In computer science a path is relative whereas a url is absolute
In rails you've got helper to have your user navigating into differents link.

One of the key differences between the path and url helper are as the [rails guide](https://edgeguides.rubyonrails.org/routing.html#path-and-url-helpers) said :
> photos_path returns /photos <br>
photos_path returns /photos <br>
new_photo_path returns /photos/new <br>
<br>
Each of these helpers has a corresponding _url helper (such as photos_url) which returns the same path prefixed with the current host, port, and path prefix.

To recap the situation you will probably have relative url (_path) in view while you'll be using _url helper in controller with `redirect_to`. Another use of _url helper is for email template, it is a bit like a redirect a the end : you need to make your user naviguating from one place (the email inbox) to another (your website, or another one)
