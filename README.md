# Ruby Quick Start Guide

This guide will walk you through deploying a Ruby application to Amazon EC2 using OpDemand.

## Prerequisites

* An [OpDemand account](http://www.opdemand.com/nodejs/) that is [linked to your GitHub account](http://www.opdemand.com/docs/about-github-integration/)
* An [OpDemand environment](http://www.opdemand.com/how-it-works/) that contains valid [AWS credentials](http://www.opdemand.com/docs/adding-aws-creds/)

## Setup your workstation

* Install [Ruby](http://www.ruby-lang.org/en/downloads/) if you haven't already (we recommend Ruby 1.9.3)
* Install [RubyGems](http://rubygems.org/pages/download) to get the `gem` command on your workstation
* Install [Foreman](http://ddollar.github.com/foreman/) with `gem install foreman`

## Clone your Application

The simplest way to get started is by forking OpDemand's sample application located at <https://github.com/opdemand/example-ruby-sinatra>.  After forking the project, clone it to your local workstation using the SSH-style URL:

	$ git clone git@github.com:mygithubuser/example-ruby-sinatra.git
    $ cd example-ruby-sinatra

If you want to use an existing application instead, no problem.

## Prepare your Application

To use a Ruby application with OpDemand, you will need to conform to 3 basic requirements:

 1. Use [Bundler](http://gembundler.com/) to manage dependencies
 2. Use [Foreman](http://ddollar.github.com/foreman/) to manage processes
 3. Use [Environment Variables](https://help.ubuntu.com/community/ EnvironmentVariables) to manage configuration inside your application

If you're deploying the example application, it already conforms to these requirements.

#### 1. Use Bundler to manage dependencies

Every time you deploy, OpDemand will run a `bundle --deployment` on all application instances to ensure dependencies are up to date.  Bundler requires that you explicitly declare your dependencies using a [Gemfile](http://gembundler.com/v1.3/gemfile.html).  Here is a very basic example:

    source 'http://rubygems.org'
    ruby '1.9.3'
    gem 'rack'
    gem 'sinatra'

Install your dependencies on your local workstation using `bundle install`:

    $ bundle install
    Fetching gem metadata from http://rubygems.org/..........
    Fetching gem metadata from http://rubygems.org/..
    Resolving dependencies...
    Installing rack (1.5.2) 
    Installing rack-protection (1.5.0) 
    Using tilt (1.3.6) 
    Installing sinatra (1.4.2) 
    Using bundler (1.3.4) 
    Your bundle is complete!
    Use `bundle show [gemname]` to see where a bundled gem is installed.

If your dependencies require any system packages, you can install those later by specifying a list of custom packages in the Instance configuration or by customizing the deploy script to install your own packages.

#### 2. Use Foreman to manage processes

OpDemand uses [Foreman](http://ddollar.github.com/foreman/) to manage the processes that serve up your application.  Foreman relies on a `Procfile` that lives in the root of your repository.  This is where you define the command(s) used to run your application.  Here is an example `Procfile`:

    web: bundle exec ruby web.rb -p $PORT

This tells OpDemand to run web application workers using the command `ruby web.rb -p $PORT` wrapped in a `bundle exec` (highly recommended).  You can test this locally by running `foreman start`.

    $ foreman start
    10:06:14 web.1  | started with pid 63945
    10:06:15 web.1  | [2013-04-06 10:06:15] INFO  WEBrick 1.3.1
    10:06:15 web.1  | [2013-04-06 10:06:15] INFO  ruby 1.9.3 (2012-02-16) [x86_64-darwin12.3.0]
    10:06:15 web.1  | == Sinatra/1.4.2 has taken the stage on 5000 for development with backup from WEBrick
    10:06:15 web.1  | [2013-04-06 10:06:15] INFO  WEBrick::HTTPServer#start: pid=63945 port=5000

#### 3. Use Environment Variables to manage configuration

OpDemand uses environment variables to manage your application's configuration.  For example, your application listener must use the value of the `PORT` environment variable.  The following code snippet demonstrates how this can work inside your application:

    require 'sinatra'
    set :port, ENV["PORT"] || 5000

The same is true for external services like databases, caches and queues.  Here is an example of how to connect to a MySQL database using environment variables:

    ActiveRecord::Base.establish_connection(
      adapter: "mysql2", 
      host: env["MYSQL_HOST"] || settings.db_host,
      database: env["MYSQL_DBNAME"] || settings.db_name,
      username: env["MYSQL_USERNAME"] || settings.db_username,
      password: env["MYSQL_PASSWORD"] || settings.db_password)

## Add a Ruby Stack to your Environment

We now have an application that is ready for deployment, along with an [OpDemand environment](http://www.opdemand.com/how-it-works/) that includes [AWS credentials](http://www.opdemand.com/docs/adding-aws-creds/).  Let's add a basic Ruby stack to host our example application:

* Click the **Add/Discover Services** button
* Select the **Ruby** stack and press **Save**

A typical application stack includes:

* An **EC2 Load Balancer** used to route traffic to your EC2 instances
* An **EC2 Instance** used to host the application behind [Nginx](http://wiki.nginx.org/Main)
* An **EC2 Security Group** used as a virtual firewall inside EC2
* An **EC2 Key Pair** used for deployment automation

## Deploy the Environment

To deploy this application stack to EC2, press the green deploy button on the environment toolbar.

![Deploy your environment](http://www.opdemand.com/wp-content/uploads/2013/03/Screen-Shot-2013-03-27-at-1.04.35-PM.png)

### Specify Required Configuration

OpDemand provides reasonable defaults, but you'll want to review a few configuration values.  For a Ruby application, you'll be prompted for:

###### EC2 Instance

 * EC2 Region, Zone, & Instance Type
 * SSH Authorized Keys (optional, used for accessing Instances over SSH)
 * Git Repository URL (defaults to the example project, or point it to your own)
 * Git Repository Revision (defaults to master)
 * Git Repository Key (required for private repositories)

If your application resides in a private GitHub repository, click **Create Deploy Key** to have OpDemand automatically install a secure deploy key using the GitHub API.

###### EC2 Load Balancer, EC2 Security Group & EC2 Key Pair

 * EC2 Region
 * Name (auto-generated by default)

### Save & Continue

Once you've reviewed and modified the required configuration, press **Save & Continue** to save the configuration and initiate your first deploy.

### Wait until Active

OpDemand will now orchestrate the deployment of your application stack.  Once the environment has an **Active** status, your application should be good to go.  However, this can take a while depending on the cloud provider, service type and size, and the build/deploy scripts (are you compiling something?).  Grab some coffee and:

* Watch the Key Pair and Security Group build, deploy and become **Active**
* Watch the Instance build, deploy and become **Active** (this takes a few minutes)
* Watch the Load Balancer build, deploy and become **Active**

While you wait for the Instance to become active, click into the Instance to watch real-time log feedback.

### Troubleshooting

It's not uncommon to experience errors when provisioning new stacks from scratch.  As you work on customizing configuration, you may need to **Destroy** and **Deploy** the environment multiple times before the automation works reliably.

* For *Cloud Provider API* Errors, check the service's primary configuration fields
* For *SSH Key* Errors, make sure Deployment configuration sections contain valid SSH private keys
* For *SSH Return Code* Errors, SSH into an instance and make sure the Build & Deploy scripts execute successfully

###### SSH Access

Click the **SSH** button on the toolbar to SSH into Instances.  If you didn't add your SSH key initially, you can always modify SSH keys later, Save the new configuration and **Deploy** again to update the Instance.

![SSH into your Instance](http://www.opdemand.com/wp-content/uploads/2013/03/Screen-Shot-2013-03-27-at-1.10.19-PM.png)

If you get stuck, on any error message you can click **Report This** to [open a ticket](https://desk.opdemand.com/) with the OpDemand help desk.

## Access your Application

Once your application is active, you can access its [published URLs](http://www.opdemand.com/how-it-works/monitor/) on the Environment's **Monitor** tab.  If you're looking at a service that publishes something, you can jump to the published URL in the upper-right corner of the service:

![Access your application](http://www.opdemand.com/wp-content/uploads/2013/03/Screen-Shot-2013-03-27-at-2.43.09-PM.png)

For the example application you should see: *Powered by OpDemand*

## Update your Application

As you make changes to your application or deployment automation:

1. **Push** the code to GitHub
2. **Deploy** the environment

OpDemand will use the latest deployment automation to update configuration, pull down source code from GitHub, install dependencies, re-package your application and restart services where necessary.

If you want to integrate OpDemand into your command-line workflow, `opdemand deploy` can also be used to trigger deploys.  See [Using the OpDemand Command-Line Interface](http://www.opdemand.com/docs/) more details.

## Additional Resources

* [OpDemand Documentation](http://www.opdemand.com/docs/)
* [OpDemand - How It Works](https://www.opdemand.com/how-it-works/)