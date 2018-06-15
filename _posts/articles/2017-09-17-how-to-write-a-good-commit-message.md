---
layout: post
title: "How to write a good commit message"
excerpt: Version control systems play a vital role in tracking the history of changes in a codebase. Knowing how to write good commit messages is an essential skill if you wish to have your project's history well documented.
modified: 2017-09-17 19:47:17 +0300
categories: articles
tags: [vcs, git, version control]
image:
  feature: 2017-09-03-how-to-write-a-good-commit-message/cover.jpg
  credit: Cathryn Lavery
  creditlink: https://unsplash.com/@cathrynlavery?photo=fMD_Cru6OTk
comments: true
share: true
published: true
aging: false
---
A [version control system](https://en.wikipedia.org/wiki/Version_control "Version Control") is an essential collaboration tool used in software engineering.
I think for most developers, it is a tool for sharing work in a co-operative manner.
For the most part, this is correct.
Multiple developers can work on the same codebase without overwriting each other's work.
But VCSs also play an important role—something that cannot be understated—in tracking the history of changes in a codebase.
But a history is only good if it's well documented.
This leads me to the topic of this post—commit messages and how to write a good one.

This post centered around [Git](https://git-scm.com/ "Git"), but the basic concepts apply to other VCSs as well.

## Bad commit messages

To understand what's a good commit message, let's start with a few examples of bad ones.
I dug through some old git repositories and found some gems that I have written.
I'm sure you've seen similar messages.

* `started working on feature X`
* `some testing stuff`
* `little fix`
* `TICKET-66`
* very big commit but a short commit message

All of them would receive a high ranking in [the commit messages hall of shame](http://www.codelord.net/2015/03/16/bad-commit-messages-hall-of-shame/ "Bad Commit Messages Hall of Shame").
In general, they're vague and don't say anything about why the particular change was introduced.
Looking at them now, they're almost as good as having no commit messages at all.
To understand what's wrong, let's turn our attention to what to look for in a good commit message.

## Understanding good commit messages

To understand how to write a good commit message, let's put ourselves in the place of a maintainer or
a future developer of the project. After all, we should write commit messages not for our pleasure but
for others to read.

Imagine yourself as a developer who is tasked to maintain a relatively large codebase.
You need to hunt down a bug and while doing that, you come across a line of code that looks interesting to say the least.
You start to wonder why the original author implemented it like that. Maybe there was a very specific and rational reason.
Or maybe—just maybe—this is the source of that dreaded bug that you've been trying to catch for hours.

Being a good [software archeologist](https://en.wikipedia.org/wiki/Software_archaeology "Software Archeology"), you whip out your handy Swiss Army knife—Git—like MacGyver and proceed to unravel the mystery.
Using `git blame` you try to understand who was the author of the line of code in question
and what was the original intent. After some time spent on [reading how `git blame` works](https://git-scm.com/docs/git-blame "Git blame documentation") you read the commit message
of the revision the line of code was part of.

If this were a *choose your own adventure* book, you would be presented with the following options:

1. The commit message is vague. You look at other changes in the same commit in hopes of understanding
what the original author tried to achieve but they're all over the place. It seems that the original author
bundled many logical changes into one commit.

2. The commit message is vague. Luckily the commit is fairly small with only 2 files being changed in total.
Although the message is not useful, other changes in the commit help you deduce the original intent of the
author.

3. The commit message is well structured and describes clearly the intent of the author. The message
includes why the change was introduced and also explains the peculiarities of the line of code that prompted you to take this journey.

If you're like me, you'd choose the third option. The main takeaway is that commit messages should be
written for other developers—or for the future you—so they could understand why you introduced these changes.

Writing code is a team effort.
Even your hobby project has at least two developers working on it—you and the future you.
Software projects can last years and see many developers joining and leaving.
A lot of times it is impossible for a new developer to go and ask a previous developer what was on his/her mind when a piece of code was written.
Fortunately, good commit messages can help here.
They're like a trail of breadcrumbs you leave behind for others who might stumble on the same twisted path you once took, hopefully leading them out of the dark forest.

## What makes up a good commit message?

Hopefully by now I have convinced you of the importance of good commit messages.
Without further ado, let's move on and look at what a commit message should contain.  

Overall, a commit message should explain what was changed and why.
There's no need for a detailed description on how a patch was implemented.
That's what source code is for.
But what if the code is not clear enough?
Then I think you might have bigger problems.
Read [Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882) by Robert C. Martin.

It's also good to compare the current implementation to how things were before the change.
This usually gives enough context so the reader can understand why the change was necessary.

[Peter Hutterer writes](https://who-t.blogspot.com.ee/2009/12/on-commit-messages.html "On commit messages") that a good commit message should answer three questions about a patch:

* **Why is it necessary?** It may fix a bug, it may add a feature, it may improve performance, reliabilty, stability, or just be a change for the sake of correctness.
*My boss told me to do it* is not a good reason.

* **How does it address the issue?** For short obvious patches this part can be omitted, but it should be a high level description of what the approach was.

* **What effects does the patch have?** (In addition to the obvious ones, this may include benchmarks, side effects, etc.)

Put yourself in the position of a future developer or maintainer and try to think of what they should know about your patch.
In addition, try to avoid large commits.
Checking in a day's worth of work at the end of the day is usually not a good practice.
If you don't know what to write, your commit might be too large.
[Try to keep your commits atomic](https://www.freshconsulting.com/atomic-commits/).

Don't be too verbose.
Writing a good commit summary is difficult.
Keeping it short and meaningful can be a challenge.
It's like naming a variable or a method.
You don't want it to be too long, but you want to convey a meaning that's easy to understand.

## Commit message formatting

An equally important aspect of commit messages is formatting.
To make your commits easy to read, there's a few things to keep in mind.
First of all, [the very first line of the commit message is the subject line](https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project).
Use it to briefly describe the changeset.
The subject line should be followed with a blank line and starting from the third line, you can add a more detailed description of the patch.

Don't use `git commit -m "my commit message"` to write commit messages.
It won't allow you to write a more detailed description on the third line.
[Configure Git to open a text editor](https://stackoverflow.com/a/2596835/2928051) to write a commit message.

Git tooling expects that the first line of a commit message is the title.
Therefore, the first letter should be capitalized and it should not end with a period.
You would not want to have a lower case heading that ends with a period, would you?

For example, when running `git log --oneline`, Git prints out only the commit hash and the first line of the commit message, giving you a quick overview of the commit history.
The following is an example of `git log --oneline` when applied on the [Spring Framework](https://github.com/spring-projects/spring-framework "Spring Framework") project.

{% highlight bash %}
9aa369f402 Data class construction supports field default/marker parameters
18f42f9667 Fix typo
47a7475898 Resolve remaining nullability warnings
ac5e2599f7 Consistent overriding for all variants of init/destroy method inheritance
b94302b5bd Enforce non-null value from getBean and at injection points
10dcaa9bf6 Update introduction to Spring Web MVC
{% endhighlight %}

But if you're interested in a more detailed view of the history, you could run `git log` and the output will include the full commit message.

{% highlight bash %}
commit da8d50f91fed1470c69baa0cbaeeffc4b31a0e8b
Author: Sam Brannen <sam@sambrannen.com>
Date:   Thu Aug 3 23:19:57 2017 +0300

    Revise SpringExtension based on recent changes in JUnit Jupiter

    This commit revises the implementation of the SpringExtension to use
    the getRequired*() methods in the ExtensionContext which are now built
    into JUnit Jupiter thanks to inspiration from the initial "convenience"
    methods implemented here.

commit fabc9c28d7e115431e25da81a0dca47787a683aa
Author: Juergen Hoeller <jhoeller@pivotal.io>
Date:   Thu Aug 3 14:21:00 2017 +0200

    Align new FileSystemUtils NIO implementation with original behavior

    Issue: SPR-15845
    Issue: SPR-15846
{% endhighlight %}

[Multiple](https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project) [resources](https://medium.com/@preslavrachev/what-s-with-the-50-72-rule-8a906f61f09c "What’s with the 50/72 rule?") advise to follow the 50/72 rule.
It says that your title line should be 50 characters or less and you should wrap lines in the message body around the 72 character mark.
Although, I don't think this is a hard requirement, it has its merits.
A short title forces you to give a brief overview.
Limiting the line length in your body [greatly improves readability of text](https://baymard.com/blog/line-length-readability "Readability: the Optimal Line Length").

[Use imperative style for commit summary](https://chris.beams.io/posts/git-commit/#imperative).
Instead of writing *Adding/added support for markdown*, use *Add support for markdown*.
Git's generated commits use this convention too.

Think of the subject line not as what you did, but what this commit will do if it is applied to my history.
For example, this commit will *add support for markdown*.
When reviewing a pull request or merging branches, you can have a look at the summary of the commits and read them as follows: if I accept commit `bc43a2d`, then this will *remove unused variables in controller*.
The same is true if you're cherry picking commits.
This would not work if the commit summary was written as something that has happened in the past.

## Other benefits

In addition to leaving behind a trail of intent, well written commit messages have other benefits as well.
If you do [code reviews](https://en.wikipedia.org/wiki/Code_review), an informative commit message can be of great help to the reviewer.
Instead of trying to understand the committers intentions by reading the changes in source code and possibly asking further questions, dedicating a bit of your time to write a good commit message can save the reviewer's time.

When a team has a consistent commit message style, the commit log could be used as a source for release notes.
Without much effort you'd have a quick way to generate release notes for your upcoming release.


## Summary

I like how Peter Hutterer put in a [blog post](https://who-t.blogspot.com.ee/2009/12/on-commit-messages.html)

> A commit message shows whether a developer is a good collaborator.

A commit message should make it easy to understand why a change was introduced in the first place.
There's no need to describe how the change was implemented.
Leave that for the source code.

Treat the first line in your commit message as the title.
After that, leave a blank line and write a more thorough description starting from the third line.

As developers, we agree that source code should follow a similar style.
Consistency is better than having [each participant use their own style](https://chris.beams.io/posts/git-commit/).
This should apply to the commit log as well.
Project's commit history is a valuable artifact, so let's treat it as we treat our code.

If you're interested in Git, how to use it and its internals, then I suggest you to read [Pro Git](https://git-scm.com/book/en/v2).
