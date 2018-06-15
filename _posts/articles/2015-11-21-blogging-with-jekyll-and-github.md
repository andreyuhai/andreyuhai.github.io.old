---
layout: post
title: "Blogging with Jekyll and Github"
excerpt: It is possible to host your blog at Github and version control your blog posts with Git. Here's how.
modified: 2015-11-21 10:42:19 +0300
categories: articles
tags: [jekyll, github, git, blog, blogging, markdown, ruby, YAML, wordpress]
image:
  feature: 2015-11-14-blogging-with-jekyll-and-github/cover.jpg
  credit: https://jekyllrb.com/
  creditlink: https://jekyllrb.com/
comments: true
share: true
published: true
aging: true
---

Starting a blog is easy. Find a blogging service provider or host it yourself. But did you know you can use Github to host your blog and make use of Git to version control your blog posts.

[Github Pages](https://pages.github.com/) helps you host a static webpage for your project for free. Technically you can host any static website there, including your blog. But static you say? That's so 90's. How can I edit my blog post? Write it in HTML?

## Introducing Jekyll

[Jekyll](https://jekyllrb.com/ "Jekyll homepage") is a static site generator designed for blogs and written in Ruby. You can edit your content in [Markdown](https://daringfireball.net/projects/markdown/) and let Jekyll generate HTML pages from it. This is ideal if you're only producing static content. No need to set up a database and no need to install security updates. And most noticeable is the speed. Because there's no database queries and no templating happening, pages are served very fast.

If you have the know-how you can customize every part of your site to your liking. Otherwise you can just start with a freely available [theme](http://jekyllthemes.org/ "Jekyll themes").

## Version control with Git

Since Github Pages are regular Git repositories, all your content is version controlled. Creating or editing a post includes writing Markdown files and committing them to your repository. Publishing is as simple as doing a `git push`.

## Writing in Markdown and YAML

Blog posts are written in Markdown. Each markdown file has a header which includes some metadata about the post (e.g. title, excerpt, published status). For those new to Markdown, check out [this tutorial](https://www.youtube.com/watch?v=6A5EpqqDOdk "Markdown tutorial"). Site configuration and post metadata is written in YAML. All in all, a Jekyll site consist mainly of text files, which is good for version control.

## Having your own domain

Although with Github Pages your page will be accessible via `username|organization.github.io`, you can still configure your own custom domain name. You just need to configure a CNAME record with your DNS provider and add a file called `CNAME` to the root of your repository containing your domain name. For more information, visit [Github Help](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/ "Gihtub Help page").

## Quick start guide

* Head over to [Github Pages](https://pages.github.com/ "Github Pages") and see how to set up a repository to host a website
* Install Jekyll on your local machine (you need to have [Ruby installed](https://www.ruby-lang.org/en/documentation/installation/ "Instructions on how to install ruby"))
{% highlight bash %}
$ gem install jekyll
{% endhighlight %}

* Create a new Jekyll site
<script type="text/javascript" src="https://asciinema.org/a/2j4jw72mql1czu1vdkveywd0q.js" id="asciicast-2j4jw72mql1czu1vdkveywd0q" async></script>

* Create a post (create a new markdown file under the posts directory, use the example file for reference)
* User `jekyll serve` to start a local development server
* Commit and push to Github
* Point your browser to `username|organization@github.io` to view your post

## Conclusion

For those who are fed up with Wordpress being bloated or don't like its WYSIWYG editor, Jekyll is definitely a good alternative. It provides most of the features you need out of the box and is focused on blogging. I must add that it might be a little harder to pick up if you're only familiar with CMS's like Wordpress. Apparently there are a lot of [static site generators](https://www.staticgen.com/ "List of static site generators") out there. So if Jekyll is not for you, there's a lot of choice.
