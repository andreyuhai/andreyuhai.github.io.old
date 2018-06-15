---
layout: post
title: "Version control and deployment practices for SugarCRM"
excerpt: This post will give you ideas on how to do version control with SugarCRM and how to manage multiple environments and deployments.
modified: 2016-04-12 21:40:25 +0200
categories: articles
tags: [sugarcrm, vcs, version control, deployment, configuration management, ansible, php, vagrant]
image:
  feature: 2016-01-14-version-control-and-deployment-practices-for-sugarcrm/cover.jpg
  credit: http://www.kinamu.com
  creditlink: http://www.kinamu.com/sites/default/files/media/pimp_my_sugar_crm_300dpi.png
comments: true
share: true
published: true
aging: true
---

SugarCRM is a web based customer relationship management system written in PHP. I'm fairly new to this piece of software and I started to figure out how to manage changes and multiple environments of SugarCRM. My plan was to have my own local development environment, a staging and a production environment.

## Problems with SugarCRM

In SugarCRM, users are allowed to define their own custom modules. These are essentially new tables in the database. They can define data types and add constraints to new fields. In addition to changes in the database, SugarCRM generates new PHP files as well. In order to keep 2 environments in sync, both file system level changes and database changes need to be transferred. While being very flexible towards its users, SugarCRM is at first glance difficult to handle by operations.

I'm going to rephrase the problem a bit more clearly. Users are able to basically create new database tables and generate new PHP files without them knowing what is happening in the background. If I create a new module in my local environment, I need to have this module be present in the staging environment for testing as well. This means the new PHP files and database changes need to be applied to the staging environment. Without knowing much about SugarCRM, this seems to be an easy task. Source code changes can be handled by version control and database changes could be handled by some kind of database migration tool (e.g. [Flyway](http://flywaydb.org/ "Flyway homepage")). But as usual, life is not as easy as it seems.

## Handling source code changes - version control

When it comes to source code changes, yes, version control is the way to go. But there are some things to keep in mind. You should not add all SugarCRM's files to source control or you'll end up in merge conflict hell. As far as I understand, Sugar does some magic and generates new source files in the `/cache` directory which is used during runtime. So that folder can be excluded from version control. But there are others as well.

![Merge conflict]({{ site.url }}/images/2016-01-14-version-control-and-deployment-practices-for-sugarcrm/meme-merge.jpg "Merge conflict")

I'm using [Git](https://git-scm.com/ "Git source code management homepage") for version control. The following is my `.gitignore` file which instructs Git from not adding the specified files and folders to version control. As you can see, a lot of files can be ignored.

{% highlight bash %}
/config.php
/config_override.php

/.htaccess
/cache/

/custom/history/
/custom/modulebuilder/
/custom/working/
/custom/modules/*/Ext/
/custom/application/Ext/

# The silent upgrade scripts aren't needed.
/silentUpgrade*.php

*.log

/upload/*
/upload_backup/
!/upload/.gitkeep

portal2/config.js

.idea/
{% endhighlight %}

## Handling changes in the database

Handling database changes is more difficult. You need to know which database tables are changed and which ones need to be kept in sync. Sugar does not generate migration files. After doing some testing and googling, I came to a conclusion that at first, I should track the changes in the `fields_meta_data` table. When you create a new custom module and deploy it, SugarCRM creates a new database table for it. Additionally, it is able to recreate that table from the generated PHP files. But when you decide to edit a deployed module, the changes are added to the `fields_meta_data` table.

In addition to `fields_meta_data` table, I'm interested in the workflow related tables as well. Since some workflow information is stored in the database, I need them to be available in other environments too.

I created a `database` folder in the root of the SugarCRM folder and I'm going to add database dumps there. To make sure I'm not going to forget to take a database dump before a commit, I created a git pre-commit hook which does it for me.

{% highlight bash %}
#!/bin/bash
RED=`tput setaf 1`
GREEN=`tput setaf 2`
RESET=`tput sgr0`

exec < /dev/tty

printf "Removing old database dumps... "
#remove old dumps
rm database/fields_meta_data.sql database/workflows.sql
echo "${GREEN}Done${RESET}"

printf "Starting to dump fields_meta_data and workflow related tables... \n"
read -s -p "Enter MySql password: " mysql_password
printf "\n"

#dump important tables
MSG="$((
  mysqldump -u username -h hostname --password=$mysql_password --extended-insert=FALSE --skip-dump-date sugar_db_name fields_meta_data  > database/fields_meta_data.sql
  mysqldump -u username -h hostname --password=$mysql_password --extended-insert=FALSE --skip-dump-date sugar_db_name workflow workflow_actions workflow_actionshells workflow_alerts workflow_alertshells workflow_schedules workflow_triggershells expressions > database/workflows.sql
) 2>&1)"

if [ $? -ne 0 ]; then
  echo "${RED}$MSG"
  echo "Commit failed${RESET}"
  exit 1
fi

echo "${GREEN}Done${RESET}"

printf "Adding database dump to commit... "
git add database/*
echo "${GREEN}Done${RESET}"
{% endhighlight %}

## Automate the creation of a new environment

Before even starting to investigate how to manage multiple environments of Sugar, I decided to first automate the creation of a clean environment. I reckoned I was going to mess up the installation at some point. So having a quick and easy way to recreate the environment was a must for me. There are many configuration management tools out there that can do the job. I decided to use [Ansible](https://github.com/ansible/ansible "Ansible github page") for provisioning since I'm most familiar with it. But you can achieve the same results with [Chef](https://www.chef.io/chef/ "Chef's homepage"), [Puppet](https://puppetlabs.com/ "Puppet's homepage") or [Salt](https://github.com/saltstack/salt "Salt's github page"). In addition, I'm using Vagrant to create a new virtual machine. Later I am able to use [Ansible for deployments]({{ site.url }}/articles/using-ansible-with-sugarcrm-7/ "Overview of using Ansible with SugarCRM").

## Headless SugarCRM installation

If you have ever installed an instance of SugarCRM you have probably used the web based installer. The downside is that it requires user interaction. My goal was to run a command and let Ansible do everything for me. In addition, the web based installer makes it harder to set up and deploy SugarCRM to multiple servers. You would have to go through each installer session separately and this is time consuming. Fortunately I was able to find [a handy SugarCRM silent installer script](https://gist.github.com/sadekbaroudi/f0f3c759df00ce1094f9 "Github gist of a silent installer script") which can be used to automate the installation process.

## Automated deployments

Having Ansible in place helps me to deploy changes to new environments. Previously I showed a pre-commit hook which takes a database dump of important tables. I created an [Ansible deploy playbook](https://github.com/indrekots/sugarcrm-ansible "sugarcrm-ansible Github repository") which checks out the latest commit from my Git repository and applies it to the server I specify. The commit might include source code changes and database changes. If you've ever edited a custom module you know that you must click the repair button in the SugarCRM Administration menu. I don't like human interaction which gets in the way of automation. Luckily I found a [repair script](https://gist.github.com/chicks/6084088#file-repair-php "Github gist of a repair script") which can be used to automate the deployment step.

## Summary

To conclude, I use [Ansible to create a fully functioning SugarCRM environment]({{ site.url }}/articles/using-ansible-with-sugarcrm-7/ "Overview of using Ansible with SugarCRM"). I can create new servers with ease by calling my [Ansible playbook](https://github.com/indrekots/sugarcrm-ansible "sugarcrm-ansible Github repository"). For my local environment I use Ansible together with Vagrant to create a virtual machine. In addition, Ansible is used to do deployments as well. I keep my SugarCRM files in source control using Git. A database dump of relevant tables is taken before each commit by a pre-commit hook. Ansible deploy playbook checks out the latest commit, applies it to the server I specify and calls the repair script. No human interaction with SugarCRM's web interface is needed during installation and deployment.
