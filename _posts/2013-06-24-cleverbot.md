---
title: Cleverbot and Friends
layout: post
---

Over the weekend I took a detour from my main project. [Simon Weber](http://github.com/simon-weber), a fellow Hacker Schooler, created a bridge between Omegle and IRC. I sensed opportunity, and quickly lost my weekend to shenanigans.

I forked the project into my own repo, as I intended on divergent functionality. In a process that favored only speed to completion, I tore out his control mechanism for his friendly bot, so that the bot could be controlled via private message (for better control), I removed the "piping" feature that he has enabled, thus allowing anyone to pipe messages to Omegle by saying the bot's name, added some command line features for choosing a network and channel, and set the bot to automatically reconnect by default.

Then I stuck two copies of the bot in a channel and expected to sit and eat popcorn while I and my friends on IRC man-in-the-middled the Omegle conversations.

The trouble is, the average Omegle conversation is really boring. Most of them end after three or four lines.

So a friend in channel who goes by Tekk, quickly whipped up a bot to talk to the Cleverbot API. Since all he needed to do was prepend messages with my bot's name to have them talk, it didn't take long before we were man-in-the-middle-ing conversations between Cleverbot and people on Omegle who didn't realize that they were talking to a bot. Cleverbot did a pretty good job and convincing people, and among lewd and hilarious scenes, managed to harvest a certain amount of personal information (which we didn't keep, but come on people, guard your identity!) and apparently fooled many participants.

It wasn't as rigorous as a Turing Test, surely, but I think Cleverbot did pretty well in our impromptu Turing Quiz.


	09:09:49       person-on-omegle │ why do u come to omegle
	09:10:00        Cleverbot │ person-on-omegle: Because I am the master of the universe. Didn't you know that?
	09:10:27       person-on-omegle │ i know -  u want to eat brains
	09:10:32        Cleverbot │ person-on-omegle: You can have mine.
	09:10:41       person-on-omegle │ good joke
	09:10:45        Cleverbot │ person-on-omegle: It was not a joke.

