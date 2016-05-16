---
layout: post
title:  Ruby on Rails Development with RVM, PostgreSQL and Redis using Vagrant
categories:
  - Heroku
  - Ruby on Rails
  - Vagrant
---

My Ruby on Rails development environment runs locally on a Mac and for a long time I used the
built-in web server and SQLite database system. The setup works for simple applications but I
always got to a point where development, staging and ultimately production have little differences
that end up costing time to fix. Mostly centered around database.

It is possible to install everything on your own computer. There are even some great solutions out
there like [Postgres.app]. But, I've found the best tool to use is Vagrant which allows you to build a local
virtual machine to your own design.

In this article I'll show you how to build a Vagrant with RVM, PostgreSQL and Redis. First, create
a new Ruby on Rails application or clone an existing one and `cd` into the root of it.

## Installing Vagrant and Chef
You will need to install Vagrant and VirtualBox first.

  1. Install [Vagrant]
  2. Install [VirtualBox]

The next step involves adding the Chef plugin to Vagrant that allow us to automatically build
the virtual machine with [Chef Solo]. We also add a plugin called [vagrant-vbguest] that manages
the VirtualBox guest additions automatically.

    % vagrant plugin install vagrant-librarian-chef-nochef
    % vagrant plugin install vagrant-vbguest

## Configuring Chef

Create a file called Cheffile in the root of the Rails app.

    % touch Cheffile

Add this configuration to the Cheffile, it tells Chef Solo which cookbooks to use to provision our virtual server:

``` Ruby
site 'http://community.opscode.com/api/v1'

cookbook 'apt'
cookbook 'build-essential'
cookbook 'nodejs'
cookbook 'postgresql'
cookbook 'redisio', '~> 2.3.0'
cookbook 'rvm', github: 'fnichol/chef-rvm', ref: 'v0.9.4'
```

## Configuring Vagrant

Now you need initialize Vagrant for this project. Just run:

    % vagrant init

It creates a file called Vagrantfile, you can safely replace it with this:

```Ruby

VAGRANTFILE_API_VERSION = '2'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = 'ubuntu/trusty64'

  config.vm.network :forwarded_port, guest: 3000, host: 3000 # Rails
  config.vm.network :forwarded_port, host: 1080, guest: 1080 # MailCatcher
  config.vm.network :forwarded_port, host: 5000, guest: 5000 # Foreman

  config.ssh.forward_agent = true

  config.vm.provider 'virtualbox' do |vb|
    # Customize the amount of memory on the VM:
    vb.memory = '2048'
  end

  # Use Chef Solo to provision our virtual machine
  config.vm.provision :chef_solo do |chef|
    chef.cookbooks_path = ['cookbooks']

    chef.add_recipe :apt
    chef.add_recipe :nodejs
    chef.add_recipe 'postgresql::server'
    chef.add_recipe 'rvm::system'
    chef.add_recipe 'rvm::vagrant'
    chef.add_recipe 'redisio'
    chef.add_recipe 'redisio::enable'

    # Install Redis, PostgreSQL and Ruby with RVM
    chef.json = {
      postgresql: {
        config: {
          listen_addresses: '*',
          port: '5432'
        },
        password: {
          postgres: 'password'
        }
      },
      rvm: {
        default_ruby: '2.2.2',
        global_gems: [
          { name: 'bundler' },
          { name: 'nokogiri' }
        ],
        vagrant: {
          system_chef_solo: '/opt/chef/bin/chef-solo'
        }
      }
    }
  end

  config.vm.provision 'shell', inline: "echo 'yes' | sudo apt-get autoremove"
end

```

## Using the Environment

Now that everything is configured all you need to do is run the command to build your server.

    % vagrant up

You'll see quite a bit of activity during the build. After it completes SSH into it:

    % vagrant ssh
    % cd /vagrant

Now do whatever you need to start using your application. In my case I have a setup script for
most applications and I use [foreman] to start up a web server and worker process:

    % bin/setup
    % foreman start

[foreman]: https://github.com/ddollar/foreman
[Heroku]: http://heroku.com/
[Postgres.app]: http://postgresapp.com/
[Vagrant]: https://www.vagrantup.com/
[Chef Solo]: http://docs.vagrantup.com/v2/provisioning/chef_solo.html
[vagrant-vbguest]: https://github.com/dotless-de/vagrant-vbguest
[VirtualBox]: http://www.virtualbox.org/
