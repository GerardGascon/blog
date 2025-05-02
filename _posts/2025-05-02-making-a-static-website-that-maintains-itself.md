---
title: Making a static website that maintains itself
image: /assets/img/covers/game-of-the-day-post-mortem.png
categories: [Personal, Projects]
tags: [post mortem, tools, web]
---

One of the things I don't like to do is get trapped into an eternal development for an already finished "product". With that, I mainly refer to having to maintain that game/thing such that anyone accessing that in the future can still play that.

That's the case of [Game of the Day](https://gameoftheday.org/), a website I made a couple of months ago that, if done wrong, I would have had to have the daily routine of maintaining it (as it is a static website with no backend).

In this post, I will be talking about how this idea came to life and what did I do to avoid this eternal development hell.

## The idea

There's this website called [Find Nice Games (Beta)](https://findnicegames.com/) where [Dragon Wasabi (Yuta Tanaka)](https://bsky.app/profile/dragonwasabipotato.bsky.social) used to post the games they found interesting. I liked that idea of showcasing cute and small games (mostly) from itch.io and sharing it with other people.

With that in mind, I thought that it would be nice if there was a website that every day showcased a small game for anyone to discover.

The website was meant to just give you curiosity. When you enter, the first thing that shows up is an animation revealing the daily game. Also, the website gave you very little information about the game —just a banner, 3 images, and a 120-character synopsis— just enough to give you that curiosity I wanted.

## What did I consider a small game?

Before doing anything, I defined what for me was considered a small game suitable for that website.

- It couldn't be a demo or a portion of a bigger game. The game had to be small by design; I completely disallowed the promotion of a bigger game.
- No paywall or ads. I wanted that anyone entering that website would be able to play that day's game, and also that ads are usually (not to say always) quite disturbing.
- A 30-minute duration limit. I didn't measure this before accepting the game, but I didn't want that the games that were inside this website to be too long. Therefore, unable to be played during the spare time that any person can have.

With these simple rules, I then gave complete freedom on the topics of the games; some may be more personal and others may be more "creative".

## The infrastructure

The idea for making this work without me having to manually update the page every day was using GitHub actions that would be triggered every day at 12:00 AM. By doing that, I won't have to be paying attention to the "backend" of the website; it _would just work_, plus I would also get the daily surprise of finding a random game to play.

Also, the page is hosted on GitHub Pages, so building the "backend" on top of GitHub Actions made quite a lot of sense.

---

For game submissions, the story was quite similar. In this case, I used the creation of GitHub Issues to trigger an action that processes the issues and creates Pull Requests with that game's information (meaning less work for me). 

But while showing this project to some of my close friends, some of them told me that _maybe_ GitHub Issues were a bit too _techie_ for their goal in this project, and they were right. So then I decided to make use of a common forms app, like Google Forms, and then find a way to somehow connect the forms app with the GitHub Issues. Again, I didn't want to manually do that for each game submitted.

I ended up using a forms app called [Fillout](https://www.fillout.com/) and a service called [Zapier](https://zapier.com/) that enabled me to connect Fillout with GitHub issues. The only problem was that with the free tier on both services I got limited to a maximum of 1000 submissions per month, but given that I'm not a famous person, that wasn't a problem haha!

## Problems & Workarounds

One thing that actually ended up being a problem was the use of GitHub Actions (you weren't expecting that, right?).

For triggering the daily page update action, I used the GitHub cron jobs integrated on the actions, like so:

```yaml
on:
  schedule:
    - cron: "0 0 * * *"
```

That shouldn’t be an issue, but then I found out the incredible imprecision it had; it could be delayed from just a couple of minutes to having such a large delay that it would just not run.

> It doesn't work, but sometimes it does.
> 
> _Someone at GitHub proposing the cron jobs_

I find it quite unacceptable that such a useful service is so much unpredictable.

Then I decided to make use of the GitHub API to trigger actions remotely from a server like AWS, and that’s what I ended up doing. I set up an AWS Lambda that just ran a basic Python script that triggered the action. That ended up being the most reliable way to trigger it.

## Conclusion

This is one of my favorite projects that I’ve done over the past few years. I learned a lot during the process of creating it and also made something that I find quite nice.

If I had to talk about the success of the site, it has no tracking, so it’s impossible for me to give an accurate answer to that. But I think I can say quite confidently that it didn’t succeed at all, but hey, my goal was making something nice, not something successful, and that goal was reached.

---

I never expect a success in popularity for any of my projects; I just make the things I love, and the most I expect is for someone eventually to come and tell me, "Hey, that’s cool".
