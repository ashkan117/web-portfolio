---
title: "Testing the Right Thing"
date: 2024-05-18T09:38:56+01:00
draft: true
---

Recently something has been bugging me. Let's say that we're building some API endpoint in Elixir it could look something like this.


What should the controller test? Should it test the variations that the nested functions could receive? Well it's easier to test the nested functions directly. 
If we only test the variations in the nested functions then what should the controller do?

Essentially what is the right level of responsibility for each "layer".

## Finding a Solution
Before getting frustrated enough to find a solution, I was brainstorming one myself.

### Handshakes
A fairly intuitive solution that eventually popped up was the concept of a handshake. Essentially if our controller calls some functions, 
we shouldn't have the controller test the variation of input and output for that function for a few reasons.

It's harder. The higher the layer you are the more noise you need you need to worry about. The controller for example is a good example of this.
Let's say our controller has the side effect of creating a user. If you wanted to test everything in the controller level, you also are required to 
test things related to HTTP since that's what the controller does. It takes a request and returns a response. Yet that's not what the focus of
the thing we want to test.

What do we do instead? We unit test the creation of the user at a lower level. WIth Phoenix you'd do that at the context levvel probably.
At the controller level, you just ensure that the right method is called. Through mocks, etc. You are trusting that the other tests leave us with a trusted create_user method,
so you're just worried about the hand-off.

### Manipulation
It's pretty common that your controller make some manipulation to the original request, before it actually hands things off? In this case the controller is responsible.
Mocks

## Entry Point and Exit Point
