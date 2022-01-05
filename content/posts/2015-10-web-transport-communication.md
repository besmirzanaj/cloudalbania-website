---
title: 'Web transport communication - an analogy'
date: "2015-10-20"
draft: false
tags:
  - security
  - communication
  - ssl
---

Web communications are like blind men, in fact, all computers are blind. They require sound to identify each other.
I
magine George. Now, George is a popular guy. People are always screaming from the crowd to get answers from George, "Hey, George!"

Sometimes they only want a single word answer that is freely available information;

> Hey, George! I'm Ralph!
> 
> Hi, Ralph! What's your question?
> 
> George, what's the capital of California?
> 
> Ralph, it's 'Sacramento"

But what if Ralph wants personal information that only he and George know?

> Hey George! It's Ralph!
> 
> Hi, Ralph. What can I do for you?
> 
> George, I need to know my bank balance.

Now here's a good time to take note of a few things:

* Notice how George and Ralph keep addressing each other by name? If Ralph just screamed out "What's my bank balance?" at the very least, no one is going to know who the hell Ralph is talking to. 
* It's also important to note the George just sort of trusts that Ralph is telling the truth, that he really is Ralph. What if Jacque wanted to tell George that he was Ralph and ask for Ralph's bank balance? Shouldn't George protect this information?

So now we arrive at George checking if Ralph is legit:

> Ralph, I can tell you your balance, but I need your password

Now here's the thing about web sessions or "conversations", since computers are blind, the only way that can continue a multi-sentenced conversation in such a noisy crowd is to address each other every time; Ralph, George, Ralph, George. Each sentence is a different REQUEST/RESPONSE.

And when you have to start establishing identity, George is going to have to ask for the password every time Ralph wants something, but that is both tedious and dangerous; someone listening in on the conversation could learn the password. So George and Ralph agree on a shorter, temporary password that REPRESENTS the real one.

This password actually references the STATE the conversation between the two is currently in; Ralph has already provided the password and George has authenticated it.

The STATE of this conversation is that George now knows he's talking to Ralph, and every time the agreed phrase is provided to George by Ralph, George knows to continue where they left off instead of starting a new conversation (in fact, if Ralph continues to forget to provide the "conversation state" enough times and George is lead to believe that it's a new conversation each time, George will eventually stop talking to Ralph).

So after Ralph provides his password, George can respond with this temporary "ticket" that Ralph can give George every time he wants to ask George for resource.
