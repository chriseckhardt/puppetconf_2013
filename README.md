Thurs 22 Aug 2013
# Notes from PuppetConf 2013
## San Francisco
## Keynote
Fairmont ballroom looks like the Apple 1984 commercial.

### Luke Kanies 9:00 am
http://bit.ly/luke-QnA

Identified challenges in cultural shifts needed at companies who need config
management but can't figure out how to implement.

PuppetConf 2012 largest complaint was "not enough time to see all the talks."
* 1.2m forge downloads.

#### Contributors to Puppet Highlighted
* Derek Elan of Spotify
* Kris Buytaert PuppetCamp
* Greg Baker ask.puppetlabs.com
* Dustin Mitchell of Mozilla
* List of companies using Puppet, Go Daddy conspicuously missing from slide


Nod to VMWare
"We're not a VMWare company, not an OpenStack company..."


* Cisco and Juniper support for Puppet
(Thanks to Bruce Figaro) sp?

#### Description of a new kind of administrator
Moving from vertical silos to horizontal abstractions

Level up from technology to applications

Whole community needs to get better about DRY

2 major problems with adopting companies, want agility & culture change.

* X-Wing strategy
1. Making the core better
2. Geppetto IDE
3. Increasing on perf, stability, better documented API
4. Enterprise Puppet 3
5. Building towards things like continuous delivery

#### Joe Wagner - Puppet demo
* A new reporting tool with insight on how puppet is configuring your system
* Say you're a sysadmin converting from debian to Red Hat and you
want to make sure mongodb is going to deploy on new Red Hat systems
* Puppet Enterprise console
* Classes Nodes Resources reporting elements

#### Puppet Enterprise Demo available for test drives
Tell Puppet Labs how it sucks, how to improve
Questions? <http://bit.ly/luke-QnA>
  * They are working on more granular ACLs but no ETA
  * One of bigger goals is to simplify experience for clustering Puppet
  * Can contribute to Puppet by participating
  * Publish, run a local Puppet Group, contribute to the Forge

## Google uses Puppet
New head of engineering defining how long they can afford to maintain
backwards compatibility.

$2000 Google cloud credit for on-site attendees - talk to Puppet volunteers.
Follow @puppetconf with hashtag #puppetconf

<http://youtube.com/puppetlabsinc>

## Why did we think large scale distributed systems would be easy?
### Gordon Rowell
Google
<gordonr@google.com>
#### SRE Manager
Makes Google run from the inside
Focus is on externalizing a lot of internalized feed
SRE runs many services
  * make the service scale
  * make the deployment consistent
  * understand all layers of the system
  * monitor everything
      * how many requests
      * how long does request take
      * what's the effect on the db
      * which one is expensive
      * which is a x-system call
      * when that network crawls
      * when that switch is busy...
      * put data inflection monitoring points at any point in code you can
      * operations want to see 'this piece of data went into the db at this time and took this long...'
  * plan for future
      * Make sure things break and break them
      * Take system apart and see what happens
        * slow db down, use slow disks and see what happens
        * load the disk with a cronjob with arbitrary proc
  * break things, under controlled conditions

In order to engineer reliable systems, you need to understand all layers
of the system.  Unless you know how all the parts work together you won't
be able to support the system.

* Scaling is fun
  * "We don't deploy 'a server'"
    * servers break, power fails
    * clients DNS need to be reconfigured

  * Don't deploy " a cluster"
    * network breaks
    * a single cluster isn't good enough
    * clients/dns need to be reconfigured

  * Deploy redundant clusters
    * attempt to send clients to nearest serving cluster
    * anycast allows for unified client configuration

Must have redundancy in every layer of the stack
Anycast has same ip addr injected into route tables of any available route structure
Everything has to be identical.
If you get routed to wrong cluster you get the wrong answer
nice thing about using anycast is it's a requirement of application deploy
that any cluster can answer any question for any user
consistency at every point


* Client DOS is not fun
  Poorly written code on a small number of clients is annoying

* Poorly written code on a huge number of clients can cause serious infrastructure pain

* Write good code and stage releases
    Work with service owners
    Stage rollouts, allow soak time
    Have a rollback plan and test it
    Have DoS limits for services, test them

Professionalism of change management; roll out as quickly as possible but
if you haven't given soak time to test "canary" clusters that can go wrong,
you're not doing the right thing for application owners

Stage rollouts, allow soak time.

Does it look good odn an almost-prod cluster 3 days/week later on prod traffic?

#### Load Balancing is fun
  * do you have enough capacity?
  * how many backends do you need
  * what happens if half lose power?
  * what happens wheen half are out for maint?

  * how do you send clients to right cluster?
    * client configuration
    * dns rr
    * dns views
    * dns anycast
    * consier dns views plus anycast
      * normally send all to anycast but have a switch for emergencies

Make sure there's enough headroom but don't go crazy (overprovision)

* Monitor everything
* Test everything
* Learn from outages
  * write postmortems; not about blame
  * focus on the facts

### Thundering herds are not fun
#### Puppet at Google
* Lots of desktops
* What if they want to do a puppet run? every hour? every 5 mins?
* Randomize cronjobs
* How can you shed load on the server end?
    anycast is coarse-grain load balancing
    network break - physical/routing/config/load balancer issues
    anycast monitoring is hard

#### You need to measure anycast metrics from the user
* include load calculations early in health checks
* consider DNS views w/ Anycast to manage traffic
* dropped traffic better than cascading failures

#### Be ruthless against platform diversity
1. If you can't automate it, don't do it.
2. Anycast helps be consistent; traffic could go anywhere
3. Every OS upgrade is a time to refactor and clean

* clients need to be able to handle RR DNS
* he's never had a need to use SRV records

They can turn live machines into canary machines to soak in newly deployed
code.  Then they turn an entire cluster into canaries, if that's good then
a complete rollout.


## Coffee Break
Perfect weather on the balcony of the Fairmont today.
Warm sun with a light, cool ocean breeze.


## 11:10am - Puppet Enterprise at Scale
### Ben Irizarry VP/Architect/Bank of America
Lead config management SME for engineering team providing and architecting
cloud services for BoA/Merrill Lynch
@beenybeenz

* not touching into db replication, performance, passenger tuning, etc.
* focus more on the Puppet product itself, how to handle different components

### Agenda
* puppet components
* basic deployment
* master pair
* master deployment behind a proxy/load balancer

#### Current deployment @ BoA
* Global
  * Americas
  * EMEA
  * APAC
* Nodes under management 7200+
* Linux, working on Windows CFM
* Separate External Node Classifier drives data
* Internally developed Web UI serves as request system
* Puppet provides configuration and life cycle management

#### Puppet Enterprise
puppet enterprise
puppet master + puppet ca
puppet agent
mcollective -> activemq/mcollective server
-> console gui/report url + mysql/pg/puppetdb

Single master can handle ~ 1k nodes
  * How did Puppet Labs come up with this calculation?
  * Number of cores x Memory / Avg manifest comipilation time / (check-ins/day)

#### Load balancer configuration
* All puppet enterprise components installed on 2 servers
* Agents still point to 1 destination
* RR DNS distributes load across 2 servers
* Allows for one entire server to go down but still have ability to manage and provision

* CA ensure your serial numbers have different start ranges if not you'll have certificate conflicts.  This file should NOT be synced.

    /etc/puppetlabs/puppet/ssl/ca/serial

* ensure inventory file is merged, not overwritten

    /etc/puppetlabs/puppet/ssl/ca/inventory.txt

* Mcollective
  * Ensure both mcollective credential files are identical (md5sum)
  * SCP file from one server to other after it is generated by module (rsync?)

    /etc/puppetlabs/mcollective/credentials

  * ActiveMQ network broker config needs to be added to activemq.xml

    /etc/puppetlabs/activemq/activemq.xml


### Puppet Masters behind Proxy
* Natural progression, build upon prev installation
* Proxy server round robin config requests across puppet workers which only have puppet-server installed.
* Distributes primary load across commodity compute (VMs)
* Central inventory for node classification and reports.
* CA performed centrally
* Central orchestration


### Orchestration servers
* CA - add to /etc/puppetlabs/puppet/auth.conf

### Puppet config workers
* update [main] section, [master] sections
* Update path of new CRL file that will be downloaded to httpd vhost file
* Add the proxy match string for CA requests within same file

    SSLProxyEngine On
    ProxyPassMatch ^/([^/]+certificate.*)$ ...

* sync over mcollective required files to configure new agents
    rsync -av
    orchestration.mydomain.com:/etc/puppetlabs/mcollective/{credentials, certs,
      keys}

* On worker VMs, only need mcollective not activemq
* each worker needs to accept dns name going to be used

#### Install HAProxy and configuration is easy peezy:

    listen puppet_00 192.168.1.2:8140
        mode tcp
        server mypuppetworker1 192.168.1.3:8140 check
        server mypuppetworker2 192.168.1.4:8140 check

* Use Zack's HAProxy module

#### Things to consider with Proxy
* Logs on puppet master access.log will always come from haproxy ip
* Can use "optionforwardfor" but then you must terminate ssl and use http mode (not tcp mode)
* Filebucket should be centralized to orchestration servers.
* Use puppet to maintain everything in sync

agent ->
haproxy -> puppet config workers (vms) -> orchestration (mcollective) -> agent

### Q & A
* GlusterFS for PuppetCA - how to replicate /var/lib/puppet/ssl across masters?

_rsync works_

* Proxied ssl request from master nodes to CA; not have agent config to ca_server?

_can do what works_

* Why go from FOSS to PE?

_nice to have everything pre-packaged + Puppet Labs' support_


## Puppet Labs Powered OpenShift
### Matt Hicks Director of Engineering, OpenShift|Red Hat
@matthicksj

1:30pm

### The tech wizardry behind OpenShift
Puppet, MCollective

When he started building OpenShift, his background was IT

Red Hat has been using Puppet since the beginning.

#### What is OpenShift?
* Platform as a Service

If you're on the app dev side, they try to make it magically easy

One of the differences it's not only hosted.

Tries to make managing systems easier as well.

```ruby
    class openshift {
      package { 'openshift-origin-broker':
        ensure => installed,
      }
    }
```


<https://github.com/openshift/>

* Focus on linux containers instead of virtual machines.
* Needed something volatile that wasn't going to kill operations.

#### OpenShift specific terms
* Gears == "Containers"
* Cartridges == "Code running in a container"

### Container to OpenShift
* Kernel namespace segmentation
* Control group workload management
* SELinux protection


#### Kernel Namespaces
* Better at segmentation but not for security
* Mount namespaces
* UTS namespaces
* IPC/PID
* Network
* User namespaces

#### Control Groups
Workload management
* blkio
* cpu
* memory
* net_cls

#### SELinux
* whitelist approach to security Mandatory Access Control
* effective in controlled use cases
* Critical to control malicious activity

### Containers?
* Not exclusive from VMs, good complement

Virtualization is still relatively heavyweight

Containers powerful for driving density up

OpenStack's challenge was to use AWS business model, but most cost-effective

MCollective is agent driven
Puppet -> Setup new VM
MCollective -> Setup new container

### Primary Functions
* Query: Find the best matching machine to install something upon
* Execute: Find the perfect machine and runn the command
* Control: Limit the applicable commands.

Don't want a distributed root account to expose control.

Puppet -> Setup new VM

MCollective -> Setup new container

MCollective -> Interact w/ Container


### Why use MCollective?
* Easy agent development; faster to build an agent with MCollective.
* Good Docs, quick starting point
* Built on Messaging: Utilizes ActiveMQ
* Security: Message signing, auditing, etc.

Kernel Namespaces <http://bit.ly/WuXpZm>


### Q & A
* What point will it tip over and fall?
_Broadcast paradigm consuming more latency than needed_


## Managing Windows with Puppet
2:20pm

### James Sweeny
Professional Services Engineer| Puppet Labs
@jsween_y

<james.sweeny@puppetlabs.com>

supercow on irc.freenode.net


### Introduction
* Windows Agent overview
* Puppet resource model overview
* Managing Linux vs. managing Windows
* Windows specific challenges and solutions
* Windows oddities that will bite you


### Supported Platforms
* Server 2003 and 2003 R2
* Server 2008
* Windows 7/Server 2008 R2
* Windows Server 2012


Basic Windows .msi installation

    msiexec /qn /l*v install.log /i pupet-3.2.4.msi

    INSTALLDIR=...
    PUPPET_MASTER_SERVER=...

Puppet on Windows works out of a basic service.

* C:\Program Files (x86)\Puppet Labs\Puppet\{sys,bin}
    * Don't mess with those
* Docs&Settings\all users\app data\puppetlabs
    * puppet\var
      * cached data
      * plugins
    * puppet\etc
      * puppet.conf
      * ssl data <- directory to delete for puppet cert regen

* Goes over Puppet architecture, RAL
* Basic puppet code example

### What makes Windows special?

```ruby
  service {'TerminalService':
    ensure => running,
    enable => true,
  }
```

#### Cron resource
* Windows doesn't have cron

```ruby
    scheduled_task{'run_job':
      command => 'C:\jobs\run_job.bat',
      user => 'SYSTEM',
      trigger => {
	schedule => 'MONTHLY',
      }
    }
```

#### Files
* Line endings
* Paths

Need a linux master with linux line endings/paths (no windows masters)

* Forward slashes are safer (Puppet will translate to \)...
...except when a Windows program will read them:

```ruby
    scheduled_task {'something':
      command => 'C:\something\something',
    }
```

* Think about where code is being evaluated (client or server)
* File resource in Puppet will copy everything in binary
* Source engine will copy it down perfectly
* If you have code generating newlines, it will use Linux style line endings
    * Can use the unix2dos utility
    * use separate files between linux and windows machines for templates

#### File Permissions
* Still specified with Unix-style modes
* Mode must be specified if owner/group are


```ruby
    file { 'C:\Blah\blah.txt':
      ensure => file,
      owner  => 'Administrator',
      mode   => '0644',
    }
```

Pretend WIndows is case sensitive with your Puppet code

No way to set SID

#### Exec Resource

```ruby
    exec {'iisreset':
      path => 'C:/WINDOWS/System32',
      refreshonly => true,
    }
```

* Execs run without a shell
* If you need shell built-ins

```ruby
    command => 'cmd \c ...',
```

#### Puppet runs in 32-bit on Windows
Workaround is sysnative directory

    %WINDIR%\Sysnative
    %WINDIR%\System32

```ruby
    $psh_cmd = 'powershell.exe -ExecutionPolicy RemoteSigned'
    exec { 'my_script.ps1':
      path    => 'C:/WINDOWS/System32/WindowsPowerShell/v1.0',
      command => "${psh_cmd},
      provider => powershell,
    }
```

#### Modules
Modules best way to organize code and extend core Puppet
forge.puppetlabs.com

    puppet module search keyword
    puppet module install author-module

* puppetlabs/registry

```ruby
    registry_key {'HKLM\System\TestKey':
      ensure => present,
    }
```

* adenning/winntp

```ruby
    class {'winntp':
      ntp_server => 'ntp.internaldomain.com',
    }
```

* trlinkin/domain_membership
Good to change Computer password

opentable/iis

sql server module
windows facts module (good for desktops)
The more we contribute, teh better Windows on Puppet becomes.

### MSI Package Provider
package {'mysql':
  install_options => {},
}


* deprecates msi provider (don't use provider attribute)
* puppet 3
* backports avail for puppet 2.7

### Chocolatey
"apt-get for Windows"

<http://chocolatey.org/>

rismoney/chocolatey

```ruby
    package { 'sysinternals':
      ensure   => installed,
      provider => chocolatey,
    }
```

puppetlabs/dism

Check OpenTable namespace on PuppetForge for modules that use native PowerShell

```ruby
    exec { 'reboot':
      path    => 'C:/WINDOWS/System32',
      command => 'shutdown.exe /r /t 300',
    }

    exec {'pending_reboot':
      command => 'cmd.exe /c "echo Reboot Pending"',
      refreshonly => true,
      noop => true,
      Package['driver_update'],
    }
```

or

#### Windows Reboot Resource

```ruby
    reboot {'SQL_Server_Install':
      message => 'blah',
      prompt  => true,
    }
```

<http://docs.puppetlabs.com/windows>

<http://puppetlabs.com/blog/part-top-questions-on-puppet-and-windows/>

What Windows really needs most is more Puppet providers contributors.

### Q & A
* Support for Complex ACLs?
  - Separate Puppet type might be the way to do that, have auto-repliers on the type.
* Able to run both linux clients and windows from same tree?
  - Yes.
* Windows admin has problems with Puppet relgating puppet with classes & groups
  Would like Puppet to read what's already in AD groups that the Computer
  belongs to.  "If you're a member of that group, do this."  Active Directory
  Security Groups
    Classification hiera-ldap backend?  One could be easily written.
    Have Puppet talk to LDAP and get that information out of it.
* Users resource, use UID for SID, what about AD for shared environment
    Puppet will not modify AD for resources
* For Windows Services, for service resources, eg. RabbitMQ on a Windows service
    Feature request currently open.
* Way to set .net runtime for which PowerShell executes under?
    Have to set up xml file inside directory.
    <http://projects.puppetlabs.com>
* Does Puppet apply work in windows?
    Yes
* No way to install Windows Agent with the cloud provider
    Not sure if there's something in projects. (there is)


## Managing Cisco Devices Using Puppet
### Jason Pfeifer - Cisco
Technical difficulties with the mic

Technical Marketing Engineer | Cisco

Cisco Automation with Puppet and onePK

#### Leon Zachary, Sunil GV

Evolution of Operational Maturity
from minimized cost/best effort
to more of a managed approach (basic SLAs)

fixes from days to hours

tiered domain experts

business operations -> virtual overlay networks

Business today requries
self service, on demand
on premise, remote, hybrid cloud
with sla

good solution with Puppet

Let business operations drive the network

* Type 1 automating existing tasks
* Type 2 automating new tasks
* Type 3 automation as integral solution

Opening up the API, "What if the 'User' is a Software app?"

* Simplified Operations
* Enhanced Agility
* New Business Opportunities

APIs Cisco is opening up

* Try to run layers 2-4 through a consistent API cross platform
* Write the same program that behaves the same way
* APIs in C, Java, Python, will support Ruby
* "Cisco ONE Platform Kit" (onePK)

Architecture slide

API presentation layer ->

API infrastructure {
  catalyst
    nexus
    asr/isr
}

Choose the hosting model that suits your app/platform

allow you to run your applications in multiple locations

run apps directly on blade or router

allows cisco to put applications (agents) on boxes

### Puppet Agents running directly on Cisco boxes



## Life and Times of PuppetDB
### Deepak Giridharagopal
<deepak@puppetlabs.com>

@grim_radical

Puppet Master takes facts, generates catalogue, sends to Puppet Agent,
Puppet Agent applies catalogue and sends Report
but PuppetDB does not throw facts away

storedconfigs previously used w/ ActiveRecord into MySQL

Didn't work very well

Now you can query PuppetDB

    /nodes/foo.com/resources/User/deepak

In order to make PuppetDB super fast (1800 node catalogues on his laptop)

Design decisions:
1. Asynchrony
    * PuppetDB is supposed to store stuff, and let you query it.
#### CQRS
* Command
* Query
* Responsibility
* Separation

Use a different model to update information than the model you use to read

    CQRS Write Pipeline
    async, parallel, MQ-parallel

```clojure
{
  :command "replace catalog"
  :version 2
  :payload {...}
}
```


    /commands -> mq -> parse -> process -> catalog
                          \      /
                      Dead Letter Office

Dead Letter Office is a bit-for-bit copy of wire communications

Deepak can use data from this directory to replay

#### Command processors must be retry-aware
expect failure because it will happen

### Why the JVM?
PuppetDB needed to be:
* Fast
* Free
* Portable
* Multi-core
* Popular

Tons of high quality libraries for 
web servers, concurrency frameworks, databases, fast parsing/lexing, clustering
debugging, profiling, etc.

Can ship an uberjar, makes straightforward with few moving pieces.

Nobody cares what runtime is used, they just want it to work.

### The query language is just an Abstract Syntax Tree
* Queries are expressed in their own language
domain specific, AST-based query language:

["and",
  ["=", "type", "User"]]

PuppetDB walks the AST to compile efficient SQL

AST-based API lets users write their own languages

Erik Dalen of Spotify

<http://github.com/erikdalen/puppet-puppetdbquery>

AST-based API lets us more safely manipulate queries

daenny, Puppetboard:
<http://github.com/nedoap/puppetboard>


#### Foreman Integration (CERN)
<http://github.com/cernops>

<http://github.com/dm-puppetdb-adapter> ORM for Ruby talks directly to PuppetDB via Ruby code

hiera backend so they can use hiera lookups to yank data out of PuppetDB

ripienaar/ruby-puppetdb

Bindings for Java, Go, Scala, CoffeeScript, Node.js, MCollective, Rundeck

OpenStack bodepd/puppet-openstack_puppetdb

grahamgilbert/vagrant-puppetmaster

evenup/evenup-pdns

### Boring technology
* Relational Database backend
* embedded or PostgreSQL
* arrays, recursive queries, indexing inside complex structures

### Weird Alien Technology
* Wrote it in Clojure
* Functional language
* No locking
* 0 bugs related to deadlocks, 0 bugs involving mutable state
* companion Ruby code has ~10x the defect rate.

7k lines of Clojure code.

### Conjectures about performance
resource often exists across multiple hosts

single instance resource storage

#### Posit
We'll often receive the same catalog for a host

In the field, we almost always see Resource and catalog duplication rates of 85%

Should be able to get metrics on how volatile your environment is.

Users want easy ways to consume metrics and analyze performance.

* jasonhancock/nagios-puppetdb
* puppetdb-external-naginator
* munin_puppetdb
* puppetdb-muninplugins
* mfournier collectd

### Thousands of Production Deployments

### Number of changes since last PuppetConf
* Available in PE 3
* basis for reporting and analytics
* 20% faster storage
* improvements to memoization, caching, eliminate double-serialization,
  superfluous indexes nuked
* Much faster terminus
* Death to keystores
    can now use PEM certs directly eliminating one of the largest sources of
    configuration problems.
* Configurable HTTPS
* Automated recovery from MQ corruption, compression of the Dead Letter Office, purging of inactive node data, DB connection recycling

* Backup and restore now integrated into daemon, can restore while PuppetDB is running.

#### V2 API
* no need to ask for only active nodes
* full fact queries instead of just a list of facts
* node metadata
* exploration-friendly

#### Wildcard Accept Headers

```bash
    curl localhost:8080/v2/nodes
```

#### Sub-queries
Report storage
* comes with a report processing plugin
* store report-level metadata
* can do queries on events that span reports
* basis for PE's event Inspector


#### Streaming Queries
stream results to clients on the fly as they come in from the database

Will be developing tools to replicate data from one puppetdb daemon to another,
to help with HA and DR

Can also later optimize the process to lower latency, but preserve eventual
consistency.

More flexible routing is coming, allowing soft failures and read/write splits:
* log errors and continue
* write to one puppetdb, read from another


# PuppetConf 2013
## Friday 23 Aug 2013
Remembered my recharge cables today.

### Design Pattern
modules are atomic
role, library
use puppet librarian
cooperative modules
  build loosely coupled odules that can work together if installed together
  but can work on their own

will try to push deps into puppet instead of rpm
rpm packaging works fine for deploying to puppet masters
modules are good for 4am provisioning

transparency and simplicity if everything is in the Puppet manifest

Composition trumps inheritance

Same Puppet code used from dev -> test -> staging -> prod
everybody works off same modules

Cisco is hiring


### Stan Hsu Sr dev manager PayPal
Puppet at scale study of paypal's learnings
With Harendra Narayan

* PayPal part of eBay Inc
132 million active registered accounts
25 currencies in 193 markets
net tootal payment vol $43B in Q2
7.6 million payments per day

$5,277 in total payment volume every second

### The challenge
* build anew system for deploying app/sys software
* massive scale in production across multiple data centers
* 3000 devs QE in 10+ offices across time zones and geographic regions


Stores in MongoDB
created a web API in REST any tool in the data center can leverage their tool
they integrated with openstack

cloud portal -> openstack -> flame icon -> 

1. enter label, application, size (20, 50, 1000 deps on what type of app)
2. Click Go, key orchestration stack provisions VM, configures load balancer,
   adds to DNS, verifiecation checks
3. Stored in hiera database (Mongo)
4. Register with Puppet
6. Deploy API will download appropriate packages, install and configure.
Fully automated.  User enters info on step 1, all the infrastructure is spun up
within a minute.  "Project Velocity"

#### Challenges
* traditional 1 app 1 module does not scale
* high velocity env with increasing speed of change
  * new pkgs sunset pkgs dep changes dev staff opperating 24x5
* Lack of puppet expertise to complement 3000+ technology

Is it possible to create a system where Puppet coding is not required for
deploying applications?

You give a list of applications you want to install
* get list of packages
* autodiscover dependencies
* generate necessary puppet resources looking at the dependency graph
* execute deployment

To optimize it, they cache this, so next time they install the application all
the code is available in memory for quick lookup.

Break out sessions with more detail later today

### Roles & Labels
* Role
1 role per pool
define a set of packages to install

* Label
a set of versioned packages
backed by a yum repository

Label gets placed in line for deployment.
Pools are just a list of VMs servicing functions.
abWeb role pool receives aWeb and bWeb packages
abSvc role pool receives aSvc and bSvc package

Hosts receive versioned packages based on Label

### System Hierarchy
Harendra Narayan

PayPal uses hierarchy

env {
  geo {
    dc {
      pool {
	host
      }
    }
  }
}

They store classes on each of the nodes.  Whenever they want to look for a 
Puppet class applied to a host, they can traverse the tree.

### Scaling ActiveMQ
activemq clients -> load balancer -> activemq cluster

MCollective wanted long connection, load balancer would time out and terminate
connections.

now they connect activemq clients directly to activemq cluster

### MCollective at Scale
#### query systems through facts, agents, or regex
#### Verify package versions in all systems for simple audits
MCollective has replaced all the SSH scripts, now if they want to query boxes
for what version of a package it has on all packages, they can run a simple mco
query and check state of packages.  Same w/ facts "how may machines have this
or that" easily retrieved via MCollective.  OPs loves it.
* SSH script replacement

####  kick off puppet runs
Also use MCollective to kick off Puppet on demand.
* REST API enables MCollective to web and other tools

Large number of apps/services -> real time status updates of puppet runs

"Why is it taking so long?  ETA?
Using MCollective to monitor deployment status
* They want to make sure system packages are in sync
* PayPal worked with Puppet Labs to create Puppet Progress module
Every time puppet processes a resource it will send a message back to AMQ and
tell if the process failed or if in sync.

Sample update message 
```json
{
  "host" "blah",
    "time": "Zulu",
    "type": "puppet_run",
    "catalog_version": 12345,
    "puppet_run_status": "running":,
    "package: {
      "status": "successful",
      "name": "axis"
    }
}
```

progress_mq on GitHub


## VMWare vCHS Puppet and Project Zombie
Nick Weaver
Cloud Automation Architect, Hybrid Cloud Service | VMWare
@lynxbat
nickapedia.com
linkedin.com/in/nicholasweaver

What is vCloud Hybrid Service?
IaaS cloud owned and operated by VMWare based on VMWare software

Any application... No Changes


Launching a public infrastructure service
will give network connectivity
give common management
if you have stuff in your infr that runs in vmware, moving to cloud will run
same SLA, same tools

Nick's job == Automation

"Automation is Effort Evolution"

Why is automation important for vCHS?

### Project Zombie
Needed an automation to scale out cloud offerings at VMWare;
what key principles were needed?

* Scalability
need to build something for scale, horizontal all the way across
* Extensibility
got to live with this solution need to expose it so they can enable more than
just ops to use that tool.  Multiplies the benefit for everyone.
* Simplicity
cannot build on automation platform that takes 10 hours of manual configuration

"I'll just do it myself."

* Resiliency
Reliability is highly tuned athlete.
Betting on based on behaviors observed
Difference with resiliency, you expect things to fail.
Plan for Failure.  Never make assumptions things will ever stay in place.

#### "Automation framework you cannot kill."
Puppet had critical pieces w/ MCollective and VMWare support

"Puppet was the right choice for what VMWare wanted to do."

Zombie runs on Cassandra, JRuby, RabbitMQ

Built a distributed resource management system

Built distributed locking system.


Rez is the refridgerator, their engine is "the chef"

"Needed Puppet Runs at concurrency scale."

#### Zombie Engine DSL = ZED
Zed lets an ops person say "do this concurrently then wait for..."

Gives management of different types of execution

Call OVFTool via Puppet -> check report/wait report

Can distribute work across multiple nodes/datacenters globally

Modularized in a way, can add Compute cluster to cloud.

talks to Rez, gets compute, talks to Razor, comes back, talks to Puppet
take this build server from Razor and add it.

Created an operational language without needing to know Ruby/Java

Load up your Zed code into engine, action can be called immediately from API

REST API endpoint can be called dynamically. Horizontally scales.

### Why VMWare uses Puppet
13 in-house Puppet Modules so far
vCloud Director
vShield networking
vSphere

total of 47 modules for everything
puppet modules for installing zombie in production, integration, and
development (including vagrant + puppet use for laptops)
project zombie itself uses Puppet to push and do stuff, puppet outside zombie
to push out zombie/update.  "A Puppet sandwich"

Can stage environments with MCollective.
update that env through job execution
can apply env against target at a time

To a fully built stack, take 72 man hours, 6 days delivery

Project Zombie launched and moved it to 1.5 man hours to 2.5 hour delivery

Primarily testing, minor configs

entire job run from call API to complete = 1 hour

turned 4-5 days of human effort to 1hr of hands off

#### details for nerds
120 tasks (pluign calls)
3000 config points
1400 managed resources
dynamically sized (pick the # of compute and storage)
controls: vCloud director, vCenter, ESXi, EMC VX, Razor, vShield Manager,
  vShield Edge, Linux, Windows OS all in the right order, concurrent,
  distributed, in 1 hr.

Puppet will work just as well outside vCHS as well as inside data center

VMware believes its something that should be available to all web stacks.

### Matt Lodge VMWare with Luke Kanies
key thing vmware wanted to do is make it easy to bring existing apps and build
it together.

LK: Why Puppet?
ML: because it solves an incredibly difficult problem of config management
Puppet helps SLA to market
What they wanted to do is move quickly
Very rapid expansion plans


## More Logstash Awesome
Jordan Sissel
@jordansissel

Majority maintainer of LogStash
Continuation of PuppetConf 2012 Logstash talk

"I get angry at computers."

### Hate-driven development
Take that sour feeling that computers have ruined everything

'#hugops'
amazon had an outage, github ddossed, instead of being angry at these things,
express "you're having a crappy day, let's hug it out"

#### Agenda
* open source and community
* what is a log?
* what's new in logstash?
* logstash ecosystem news

"open [in open source] means community"

One of the things you shouldn't do is tell new users "You're doing it wrong, RTFM"
#### If a new user has a bad time, it's a bug.

Documentation isn't perfect.

Goes over example of apache logs, variances with syslog, ntpd log...etc.

    time + data = log

### What is LogStash?
* takes logs from anywhere (input)
* parse, process, or combine logs (filter)
* ship them somewhere else (ouptut)
...in real time


Logstash should be:
* fast
* easy to learn
* easy to deploy
  If it feels slow, there's something wrong.
* easy to operate
* easy to extend


If you find it isn't fast and easy to use, then it's a bug (and we can fix it).

logstash 1.2 release Sep 1

125 plugins so far

For any given task, if there is a need to transmit logs from 1 location to
another there's a good chance logstash has a plugin.

Speed is way up
* 3.5x higher event rate
* dramatically faster startup time

### New Features
* Conditionals

```ruby
  output {
    # notify nagios on http 5x's
    # for events with a 'status' field
    if [status ] == '500' {
     email { to => "ops_loves_email@example.com" } 
    }
```

* new json schema
  smaller in size, easier to integrate with

LogStash web, first attempt at a web UI
"It's ok, not good."

### Kibana
* LogStash web is dead and gone
* Kibana comes with LogStash 1.2

### Ecosystem
Tools that LogStash uses or built around LogStash
* elasticsearch
can take very high data rates and scalable, just throw another machine in.
strong community (leslie)

Smaller memory footprint, enables compression by default.


* Kibana
  * What logstash web should have been
  * Different types of charts, themes, maps
  * Filter in logstash to do a lookup to find coordinates for data

Start with input data from logs, then visualize it in Kibana.

### LogStash with Puppet
LogStash puppet module is umbrella'ed under LogStash project
* versioned
* tested

Puppet DSL lines up really well with logstash data model

<http://grokdebug.herokuapp.com>

    %{SYSLOGBASE} <(?<level>\w_)> *%{GREEDYDATA:message}

```json
    {
      "timestamp": "Aug 16 13:01:51",
      "logsource": "pork",
      "program": "NetworkManager"
      "pid": "820",
      "level": "info",
      "message": "address 192.168.1.109"
    }
```

Can start with a sample log and have grok figure out a pattern to watch.

* expiring old logs
Can do time-based or disk-storage-based arguments for data retention

    # Quickly delete logs older than 90 days
    

### Quick demo of Kibana


### Q & A
* Support for Thrift?
_Nobody has submitted one._

* Metrics filter labeled beta?
_the label chosen for plugin status was poorly chosen.  In the next version of
logstash, they're just numbers, with definitions for what the numbers mean.
Benefits of numbers is. There are no tests for it yet, so he's not comfortable
moving out of beta title._

* Multiline support in grokdebug?
_If your question is 'can we make it do something' I will respond, 'if it's a
computer we can make it do what we want.'_

Kibana not intended for highly technical people primarily, so it has a web
interface.

* How can we keep logstash cookbook up to date better?
_"wiki means to me, where documentation goes to die."_
Why it's called a cookbook instaed of a wiki
_Would like to see more examples._
<http://cookbook.logstash.net>

<http://logstash.jira.com/>

#### Kibana Demo
"Show me the top IRC talkers in the past 2 days"

Business folks don't want to see raw logs, they want to see visualization of
trends.

We should make systems that are easier to use that don't depend on meatspace.

Learning a query language is a b.s. proposal
If you're writing a complex search query, that's a bug or a feature that should
exist.


## Automating VMware Vsphere with Puppet
* Got to conference room too late, standing room only
* Still working on proof of concept
* Not available for public consumption yet
Hard to see slides or hear presentation from back of room.  Went to CERN
presentation on OpenStack


## A One Stop Solution for Puppet and OpenStack
### Daniel Lobato Garcia | CERN
CERN using Puppet Foreman
"Foreman is our tool for building servers"

    Foreman Proxy {
      IPMI {
        Physical Box
      }
    }

CERN uses an API wrapper internally that may be open sourced eventually

### OpenStack VM Creation
new()
{
  set up compute instance
}

#### Scalability experiences
* Split up services
* Puppet critical vs noncritical

    12 backend nodes Batch
    4 backend nodes interactive

* autoscale via alarms (heat)
* define situations (load threshold...)
* spin up VMs as necessary

Do not server large files over Puppet.

That's what repositories are for!

### Q & A
* Using their own monitoring system based on Ganglia and Nagios


## Windows: Having Its Ass Kicked by Puppet and Paurshell since 2012
* PaurShell
the northern irish pronunciation of the windows based task orchestration tool
Developer for OpenTable
Member of Jetbrains Dev Academy
DevOps Extremist

### Agenda
* classic infr mgmt
* snowflakes
* infrastructure
* powershell as a way to manage Windows
* How Puppet kicks its ass

"The Run book"
RHEL 5 Installation Guide 416 pages

People are generally rubbish at performing manual repetitive tasks.


### Snowflake Servers
Martin Fowler said that snowflake servers are things you have no control over
been in production a long time, don't know how they'll react.

Machines are much more reliable at performing repetitive tasks.

Very important we don't run our machines at 100%

### Infrastructure at Code
If we can automate things, we can code them.

### Phoenix Serves
completely destroyed and brought back from Scratch

### Chaos Monkey

Chad Fowler wrote about 
### Immutable infrastructure
Comes from the idea of running servers in prod for a long time.
If they make a deployment push, they bring it up from scratch.
Concept comes from functional programming.

Puppet has Geppetto, RubyMine, IntelliJ
Testing with puppet lint, rspec, huge community
Generally better you can store your infrastructure in the same place you store
your application.  Can version it, tie it to app versions, code is generally
better.

### What can we do with Windows?
CFEngine, Puppet, Chef, Ansible, Salt, all built on Linux

Windows has Powershell

Powershell 1 was complete horsecrap.

Powershell 2 was a lot better, RPC control.

Powershell 3 came along.

Powershell 4 destroyed it with "desired state configuration."  Another layer of
abstraction over something that's simple to write.

### Managing Windows Server 2008 with PowerShell
Presentation demo run on his macbook.

When PowerShell 3 came out, MS came out with a clean API.

If you try and add a feature that is already on a server, it doesn't do
anything.  Just says the server's there.

Not difficult problems to solve, up until the 2 years ago Windows community
hasn't done a lot to solve them.

Goes over Chocolatey for a bit

Live Demo: destroyed and rebuilt a qa server using PowerShell 3 in seconds

### Puppet on Windows
Been around since Puppet 2.7.6

Getting better, but still nascient.

Not all the types from linux side are available in Windows.

Limited in functionality.

Lots of exec statements, not the best way to write modules.

Need to convert execs to custom Types.

#### Puppet + PowerShell = Windows Tap Out
OpenTable will be open sourcing a lot of their Windows Puppet Modules to the
Forge.

Going to run from a base, newly provisioned Windows Server 2008 with just
Puppet installed, via a VPN connection to his box in London.

Somebody mentioned SCCM, crowd laughs.



## BOXEN
Will Farrengton
@wfarr

### Inventing on Principle
Presentation to look up

Throwing out preconceptions on how things work now, and how things could work
back at the beginning.

_What in the hell is my principle?_

Developing Software is hard

Painful software creates friction

Friction gets between product and user

"Software should get out of the way."

GitHub statement = _Help ppl design, build, and ship software better, together._

Need more porcelain to enable ppl to ship

Shipping isn't just for software.  Legal team ships a takedown, HR ships a
policy, can't just treat it as a software design problem exclusively.

### Enable
How?

"We need a mission statement."

TAFT - Test All The Fucking Time

Need to be able to throw your code spike away and go back to TDD

_"Whatever you do, make sure you are testing, because if you aren't, all you
are doing is making it harder for yourself when you revisit the code and making
it even harder..."


"Whatever you do make sure you are automating because if you aren't all you are
doing is making it harder for yourself when you revisit hte problem and making
it even harder for the next person..."

AUTOMATE PROBLEM REPRODUCE SOLUTION

AUTOMATING SOLUTIONS TO PROBLEMS LEADS TO REPRODUCIBLE SOLUTIONS WHICH ARE
EASIER THAN SOVLING PROBLEMS UNIQUELY EACH TIME FOR EVERYONE

AUTOMATE ALL THE FUCKING THINGS

### Work in Environments
There are ppl in GitHub who don't work in Rails.

There are ppl who do work in Rails.

Aiming for a higher level of abstraction.

How do you describe a workflow for an environment?

We know a thing about development environments.

But we are clueless when it comes to empoweirng ppl outside developer space.

"non-technical" ppl are very technical ppl who are specialized in their own
skills.  Devs cannot do what lawyers/HR does as well as they do.

We need to understand different kinds of environments and then make them better
with automation.

Saying they should use "our tools" is a cop-out.

It's about finding tools that enable a better workflow.

### Shippers work on projects
github/github
* long-lived piece of software

```bash
    $ boxen github
```
    
...
now Jill developer can work on GitHub.

What about everybody in finance?

class projects::financial_audit {
  # ???
}

### Shippers need different things
Arrive at a better level of abstraction for everybody.

### Shippers deserve to be happy


### Shippers really like...
How are we going to improve Boxen

* OS X 10.9 "Mavericks" Support
* No manual XCode installation required
* 10.9 has shims installed for CLI dev tools

Hiera everywhere

All boxens are designed with Hiera

* Updating modules to get a new version of X sucks.
* Trying to run a service on a different port sucks.
* YAML changes are more approachable than Puppet

More core modules will get support over time
* DNS mask, MySQL, PG
Trying to figure out how to make some things, making multiple instances of
e.g., Redis server running.

* PuppetMaster Support
Use PuppetMaster in context of their Boxen environment.

### HENSON
librarian-puppet is not the greatest piece of software.
author is co-writing Henson.

librarian-puppet "works"ish (sort of)

Librarian makes a lot of assumptions that don't fit Puppet's use cases.

Not extensible.  Not a very good library.

Henson is not a dead project.

Rodjek is also working on it.

Ruby, stdlib only, no dependencies.  Local caching and block file work still
needed.

Some other priorities needed their attn first.

### Closing Thoughts
* Boxen is not perfect
* made lives easier at GitHub
* might make your life easier
* Don't use it because there's a big name attached.

### Q & A
* Why did they go with Puppetfile instead of Modulefile?
_They want boxen to be used by the Modulefile itself eventually, avoid
different behavior between Puppet and Boxen with Modulefile_

Puppet module will be split out of Puppet core and its own tool again according
to Puppet Labs.

Rumor Modulefile may be going away, eventually replaced by metadata.json file.


