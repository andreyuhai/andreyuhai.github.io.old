---
layout: post
title: "Ansible skipping localhost when using Vagrant"
excerpt: Vagrant Ansible provisioner skips plays that are meant to be run on localhost. There's a simple workaround for that.
modified: 2016-05-28 16:01:38 +0300
categories: articles
tags: [ansible, vagrant, provisioning]
image:
  feature: 2016-05-28-ansible-skipping-localhost-when-using-vagrant/cover.jpg
  credit: https://www.vagrantup.com
  creditlink: https://www.vagrantup.com/assets/images/vagrant_header_background-482a12a7.png
comments: true
share: true
published: true
aging: true
---

I have a playbook which will be run on the [Ansible](https://github.com/ansible/ansible "Ansible is a simple IT automation platform") host.

{% highlight yaml %}
---
- name: My play
  hosts: localhost
  connection: local
  roles:
   - list of roles
{% endhighlight %}

Running the play from the command line using `ansible-playbook` I got the result that I expected. But when I use Vagrant to provision a virtual machine using [Ansible provisioner](https://www.vagrantup.com/docs/provisioning/ansible.html "Vagrant Ansible provisioner"), the play that should have been executed on the Ansible host was skipped altogether.

{% highlight yaml %}
PLAY [My play] ****
skipping: no hosts matched
{% endhighlight %}

From what I understand, this is caused by Vagrant Ansible provisioner [always setting](https://github.com/mitchellh/vagrant/pull/2991 "Github pull request") the `--limit` argument. I can reproduce the same issue by executing the playbook using `ansible-playbook` and setting the `--limit` argument explicitly.

{% highlight bash %}
$ ansible-playbook -i inventory/dev/hosts play.yml --limit all
{% endhighlight %}

## Workaround

In your `Vagrantfile` add the following line to your Ansible provisioner configuration

{% highlight ruby %}
ansible.limit = 'all,localhost'
{% endhighlight %}

## Related links

* [https://github.com/mitchellh/vagrant/issues/3140](https://github.com/mitchellh/vagrant/issues/3140)
