---
layout: post
title: Idempotent
permalink: idempotent.html
categories: computer-science
---

Hi there, just a quick article about the "idempotent" concept in computer-science. I'll show you what it mean by using the PUT and PATCH concept in HTTP. Let's make a quick summary about those two :

> [...] The existing PUT method only allow a complete replacement of a document.

> [...] This proposal adds a new HTTP methods, PATCH to modify an existing HTTP resource.

Basically what it means is that the PUT needs the entire object in parameters. The PATCH allow to modify partially the object.

Idempotent: the principale of idempotent is that an operation producted one or N times will always product the same output.

For PUT this is very simple : since we update an the object entirely with the newly given data, the change producted one times or N times will always be the same and return the same things from the server.

For PATCH, this is a bit shady - this is a bit more difficult. Everything depends on the API's format input.

If we gave it something like :

`
PATCH '/user/42'
{ name: foo }
`

Then yes patch is idempotent. If we recall one or N time the method then we will have the same result. Why ? Because we assume that PATCH is used like PUT be we simply give as parameters the fields we want to update, in another words, it's simply a customized PUT.
But most of the example on the internet are talking about PUT and PATCH and idempotence because PATCH can be used in a different way. We can imagine the following format for our API :

`
PATCH '/calculator'
{ field: count; operation: inc }
`

In this case the operation is not idempotent because we can imagine the server having an error until a certain number.

To be honest so far in my experience I have never seen the use of PATCH with this format, so I am not sure this is the "best" example we can give to most of the people, but we do understand now the principle of idempotence : PUT will always return the same thing because we update the object entirely and if the data are correct one time, they will be safe the next time, however, using PATCH for incrementing a variable or sending a signal can lead to different scenarii depending on how much times we call the API and the constraint applied to the counter.


