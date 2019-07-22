---
layout: post
title: Web - Tabnabbing
permalink: tabnabbing.html
categories: web security
---

At my job one day, I discover some weird html attribute that I never saw before : `rel="noopener noreferrer"`. After making some research on the topic, I realize this was a protection against phishing and the `target=_blank` attribute. Let me explain that to you.

### Nabtabbing
The previous protection is against a security issue call tabnabbing. To describe you more how this works, let's take the following scenario :

- You are on twitter and someone posted an exciting tweet about the new trendy tech that GraphQL is.
- You click the link and it opens it in a new tab. (Thanks to the `target="_blank"` attribute)
- You read the article, you are happy to learn why this is an amazing tech, you get back on twitter, but ... 
weird they ask you your password. "Ok I might have been disconnected because of cache or session issue, whatever here is my password"

You have been just phished. This was a fake website and you give it your twitter's password.

### How nabtapping work technically ?

The link on twitter that you open in a new tab was containing the following piece of code :

```javascript
window.opener.location="http://phishingtwitter.com"
```

According to [W3C](https://www.w3schools.com/jsref/prop_win_opener.asp) :
> The opener property returns a reference to the window that created the window

What this mean is that through a link that you click and open in a new tab, the newly opened tab (children) has control over the parent tab (twitter) and can thus redirect it transparently to a phishing website using the location method.

This is currently working on up-to-date major browsers like Chrome.

### How to prevent the security issue ?
To prevent the control of the parent tab through the children one, just the add this to your link :
```javascript
rel="noopener noreffer"
```

This HTML code prevent from accessing the `window.opener` object, and also actually ask the browser to set his referrer to "no-reffer" ([see here](https://html.spec.whatwg.org/multipage/links.html#link-type-noreferrer))

If you want to experiment the attack, I recommend going on [this website](https://mathiasbynens.github.io/rel-noopener/)

### A performance impact
When I first encounter this and understand to security issue, I was against this idea because the link containing the `target="_blank"` are not user's link but link that are pointing to our official website, so I was pretty confident no script would run to change the `location` property.

That having said, adding this attribute are always a good idea because they are also an optimisation for certain browsers.

From Jake Archibald's website :
> A Javascript running on one domain name runs on a different thread to a window/tab running another

Using `target="_blank"` the new created window will run in the same process && thread; Using `rel="noopener"` prevent from using `window.opener` so the browser won't use the same threads to open the new tab.

After having experiment this performance impact [here](https://jakearchibald.com/2016/performance-benefits-of-rel-noopener/) I see it only working for Safari and Firefox. Chrome sounds to have solved this issue as well as Chromium. 

Sources
- [https://mathiasbynens.github.io/rel-noopener](https://mathiasbynens.github.io/rel-noopener/){:target="_blank"}
- [https://www.jitbit.com/alexblog/256-targetblank---the-most-underestimated-vulnerability-ever](https://www.jitbit.com/alexblog/256-targetblank---the-most-underestimated-vulnerability-ever/){:target="_blank"}
- [https://developers.google.com/web/tools/lighthouse/audits/noopener](https://developers.google.com/web/tools/lighthouse/audits/noopener){:target="_blank"}
- [https://html.spec.whatwg.org/multipage/links.html#link-type-noreferrer](https://html.spec.whatwg.org/multipage/links.html#link-type-noreferrer){:target="_blank"}

