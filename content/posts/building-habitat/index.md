---
title: "I'm Building Habitat"
date: 2024-06-24
description: "I've decided to stop putting it off and jump right in."
tags: ["fediverse", "projects", "habitat"]
showComments: true
---

A few months ago I created this blog for the very purpose of sharing an idea of a [decentralised social 
platform based on your location](/posts/location-based-social-network). In posting and sharing it, I was hoping to find
out if anyone else would find such a social network valuable, and if anyone with any experience building for the
fediverse had any reason to believe that it wouldn't be technically possible.

At the time of writing the post, I wasn't convinced that I'd be building the social platform myself. If I'm honest, I 
was hoping that someone with experience building for the fediverse would feel inspired to take up the project 
themselves! After all, what do I know about building a social network?

It was only after reading and hearing such positive feedback, and finding that other people were as excited as I am 
about a local platform, that I decided I was going to give it my best.

## Your Feedback

Thankfully, I didn't get any feedback that indicated that there would be any technical issues I've overlooked which
would prevent a network of Habitat instances from communicating with one another effectively, but most of the feedback 
I received online appears to have been from users of fediverse services, not developers. So I'm still fearfully 
awaiting that bombshell!

I did have one federated forum developer let me know that he'll be stealing the idea of [using Gossip Protocol for 
instance discovery](/posts/location-based-social-network/#gossip-protocol), and I'm all for it -- steal away!

I've had a great deal of feedback from fediverse users and friends and family who are generally excited about the idea
of a social platform that is focused on local information.

It's not all sunshine and rainbows though, I have received some doubt, criticism and questions, so I thought I'd go 
into some of that here.

### "It won't catch on"

I think this is probably the most vocal criticism I've received about the idea, and at first, I saw it as a valid 
reason not to attempt it. Initially, I saw this issue as the most likely cause for failure. If you build a 
social platform around the concept of segmenting groups of people by location, there is a very real possibility that 
people would fall into the catch-22 of not using it because nobody uses it.

In fact, in my original post, [I explored a potential solution for this very 
problem](/posts/location-based-social-network/#how-big-is-too-big). After some thought, I've decided to revoke that
idea. [One Lemmy user](https://feddit.uk/comment/8646093) put it very well:

> One advantage to a system like this, is that you don’t need it to catch on in a bunch of different communities for it
> to work. It can catch on even in one place, and it will already be useful for the people there.

Don't get me wrong, I would love the [Nearby feed](/posts/location-based-social-network/#the-nearby-feed) to work 
anywhere in the world, and for that to be a reality, it would need to catch on. But I now feel that it's crucial to 
take the approach of fostering one's own community. If I can make it a success in my local town, perhaps you could make
it a success in yours.

### "Isn't this just Nextdoor?"

I had variations of this question for a bunch of social platforms, but Nextdoor was the most common comparison. Putting 
aside that Habitat will be [federated](https://en.wikipedia.org/wiki/Fediverse), [free and open-source 
software](https://en.wikipedia.org/wiki/Free_and_open-source_software), a _Neighbourhood Watch_ kind of social network
is not what I'm going for. In theory, if an instance administrator wanted to [make a 
category](/posts/location-based-social-network/#categories) for such discussions, they could. They could also make a 
_Spotted_ category to mirror those kinds of Facebook groups if they wanted. But the purpose is to provide an experience 
around _location_ as the subject with no direction on the kinds of discussions that happen around those locations, only
to categorise them accordingly.

Personally, I'm most excited about posting and asking questions about local landmarks and areas of natural beauty. But
it's up to the communities themselves to decide the conversation.

### "Why reinvent the wheel?"

I had a lot of people telling me that I should just launch an instance of Mastodon or Lemmy and give it the name of my
local town. I have two answers to this:

1. There is nothing that I can find that provides the functionality I think would prove useful for local communities
2. Where's the fun in that?!

Why _not_ reinvent the wheel? I've already learned a great deal in just starting this project, and I'm excited to learn
a great deal more.

## The Plan

Habitat will be built in three phases, and while I may be posting here to explore some ideas for the future phases, I
won't begin work on a phase until I'm satisfied that the prior phases have been completed to a good standard.

### Phase 1: Core

Before worrying about how Habitat instances will connect, it's first important to ensure that the functionality will 
provide a good and usable experience in any isolated instance. This is why phase 1 will focus solely on the user 
experience in their local area. Getting this right before moving on to the next step is crucial to build something 
worthwhile.

### Phase 2: Mesh

The Mesh phase will focus on [communication between Habitat 
instances](/posts/location-based-social-network/#connecting-instances) to allow the user to see posts from any
locationally relevant instance that is connected to the network. It will need to be designed carefully to prevent 
bad actors from flooding the network with spam while ensuring that newly created instances can be adopted into the
network with ease.

### Phase 3: Flow

The third and final phase will introduce the ability for users to interact with posts between instances. The user 
should not need to sign up to a different instance to comment on a post from another instance. This phase will
most likely introduce the use of [ActivityPub](https://www.w3.org/TR/activitypub/) and may require a great deal of 
rewriting.

## A Sneak Peek

I thought I'd quickly demo the end-user experience as it is at the moment. Forgive the black bars and the placeholder 
content, I'm still developing locally in browser mode for the time being.

### Browsing

{{< video src="browse.mp4" width="346" >}}

### Posting

{{< video src="post.mp4" width="346" >}}

## Get Involved

If you're a developer with experience or desire to work with [Docker](https://www.docker.com/), 
[Symfony](https://symfony.com/), [HTMX](https://htmx.org/), [Bootstrap](https://getbootstrap.com/) or 
[Leaflet](https://leafletjs.com/), I would greatly appreciate any help or input.

If you have no desire to work on this project but can give advice on managing Open Source projects, that would also be
tremendously valuable.

If you have no experience in software development in any capacity but are interested in the idea of Habitat, I'd also 
love to hear any feedback you have.

You can [get a look at the repository](https://github.com/carlnewton/habitat) in its very much pre-alpha state on 
GitHub. Give it a watch or a star or something so that I know you're interested.

I have a [project board](https://github.com/users/carlnewton/projects/2) which indicates the current priorities. At the
moment, I'm just committing to main like a madman, but if I see someone take up an issue, I'll start branching.

You can get in touch with any of the social links below. Keep an eye on this blog if you're interested, I have some
ideas on the Mesh phase that I'll be sharing in the near future.
