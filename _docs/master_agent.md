---
layout: architecture
title: Master of Masters
date: 2019-07-09
version: v0.0.1
summary: A complete master/agent architecture with multiple compile masters and load balancing for redundancy.
---

## Intended Audience

This architecture is intended for large infrastructures or dynamic infrastructures
that require the redundancy of multiple compile masters.


<div class="mermaid">
  graph LR;
  git(Git Repository)
  Foreman(The Foreman)
  Webhook(Puppet Webhook Server)
  AllCM((All Compile Masters))

  PuppetDB

  PuppetServerMoM{Master of Masters}

  subgraph CM[Compile Masters]
      Compile1[/Compile Master\]
      Compile2[/Compile Master\]
      Compile3[/Compile Master\]
  end

  LoadBalancer[Load Balancer]

  Agent1(Agent 1)
  Agent2(Agent 2)
  Agents(Agents ...)
  Agent_n(Agent n)

  click Foreman "https://www.theforeman.org" "Foreman is a complete lifecycle management tool for physical and virtual servers."
  click Webhook "https://github.com/voxpupuli/puppet_webhook" "A webhook service that can trigger code deploys from source code repository updates."

  git --webhook--> Webhook
  Webhook --r10k code deploy--> PuppetServerMoM
  Webhook -.r10k code deploy.-> AllCM

  PuppetDB --- PuppetServerMoM
  Foreman --- PuppetServerMoM

  PuppetServerMoM --> Compile1
  PuppetServerMoM --> Compile2
  PuppetServerMoM --> Compile3

  Compile1 --> LoadBalancer
  Compile2 --> LoadBalancer
  Compile3 --> LoadBalancer

  LoadBalancer --> Agent1
  LoadBalancer --> Agent2
  LoadBalancer --> Agents
  LoadBalancer --> Agent_n
</div>

## Setup and Usage

{write a guide on how to deploy, configure, and use this architecture}


### Git Repository

We recommend organizing your code as a Control Repository with branches for
environments. See the [reference repository](https://github.com/puppetlabs/control-repo)
for an example.


### Foreman

[Foreman](https://www.theforeman.org) is a complete lifecycle management tool
for physical and virtual servers. It will provide you with a graphical
classifier, a Hiera data source, and report monitoring. It also includes the
power to easily automate repetitive tasks, quickly deploy applications, and
proactively manage servers, on-premise or in the cloud.


### Puppet Webhook

Configure [Puppet Webhook](https://github.com/voxpupuli/puppet_webhook) to receive
webhook events from your code repository and automate your code deploys. This
service should be installed on the central Master of Masters. You might consider
using the Bolt task from the [puppet-r10k module](https://github.com/voxpupuli/puppet-r10k/blob/master/tasks/deploy.json)
to trigger code deployments on each compile master, or you can also install
Puppet Webhook on each.


### Code Deployment

[r10k](https://github.com/puppetlabs/r10k) is considered the default Puppet code
deployment tool. Install it on each master in your infrastructure and use it to
deploy your control repository as needed.

If you're a Golang shop, you might consider [g10k](https://github.com/xorpaul/g10k) as well.


### Load Balancer

Is current state-of-the-art still Nginx?


### Puppet Stack

We recommend managing each of these components with the supported module.

* PuppetDB
    * [puppetlabs/puppetdb](https://forge.puppet.com/puppetlabs/puppetdb)
    * The default PostgreSQL database is recommended.
* Puppet Server
    * [puppet/puppetserver](https://forge.puppet.com/puppet/puppetserver)
* Puppet Agents
    * [puppetlabs/puppet_agent](https://forge.puppet.com/puppetlabs/puppet_agent)
* Puppet Metrics Dashboard
    * [puppetlabs/puppet_metrics_dashboard](https://forge.puppet.com/puppetlabs/puppet_metrics_dashboard)
