!SLIDE

# Securing and Extending Puppet for World Domination



!SLIDE bullets

# Hi, I&#8217;m Richard Crowley

* Equal opportunity technology hater
* DevStructure&#8217;s operator and UNIX hacker



!SLIDE bullets

# Configuration management<br />Cliff&#8217;s Notes



!SLIDE bullets

# Infrastructure as code

* TODO



!SLIDE bullets
.notes Just as you don't have to think about capacitors to microwave your burrito, you don't have to think about the intermediate steps to know how you want your server to be.

# Declare state,<br />not process

* TODO



!SLIDE bullets
.notes It's a good thing that the Puppet language is not itself Ruby but I'm not here to start a holy war.

# Puppet, Chef,<br />and `~/bin/doit5`

* Limits are good.
* Puppet and Chef are idempotent by default.



!SLIDE bullets

# Architecture

* Master knows best.
* Agents phone home.
* Hostname matching.
* Resources.
* Agents make it so.



!SLIDE bullets

# Resources

## The smallest unit of configuration.

	package { "foo": ensure => "0.0.0" }

	file { "/etc/foo.conf":
		content => template("foo.conf.erb"),
		owner   => "foo",
		mode    => "600",
		ensure  => file,
	}



!SLIDE bullets
.notes Be wary of using `latest` on packages you don't build in-house.

# Classes and definitions

## Compose resources in interesting ways.

	class bar {
		exec { "apt-get update": }
		package { "bar":
			require => Exec["apt-get update"],
			ensure  => latest,
		}
		file { "/etc/bar.conf":
			content => template("bar.conf.erb"),
			ensure  => file,
		}
	}



!SLIDE bullets
.notes The defaults are good except for a few places that break filesystem standards, which I choose to fix.  `pluginsync` will be important later.  You can customize the master and agents separately using `[master]` and `[agent]` sections, INI-style.

# `/etc/puppet/puppet.conf`

	[main]
		logdir=/var/log/puppet
		rundir=/var/run/puppet
		ssldir=$vardir/ssl
		pluginsync=true
		server=puppetmaster.example.com



!SLIDE bullets

# (R)TFM

* `puppet --genconfig`
* <http://bit.ly/puppet-config-ref>



!SLIDE bullets
.notes This should exist on the master, too.  There's a minor memory hit to running `puppet agent` as a daemon but that makes a huge difference to DevStructure users with sometimes just 256 MB of RAM.  This cron calls `bash`, not `sh` to use `$RANDOM` which in effect prevents the thundering herd from taking down your Puppet master every half hour.  This is a good time to point out the obvious - that not all your servers will always be the same - plan accordingly.

# `/etc/cron.d/puppet`

	PATH="/usr/sbin:/usr/bin:/sbin:/bin"

	# Remove the line breaks.
	*/30 * * * * root bash -c '
		sleep $(($RANDOM \% 1800));
		puppet agent --certname=$(cat
		/etc/puppet/certname)
		--no-daemonize --onetime'


!SLIDE bullets

# Hello, world!

* Entire config in `/etc/puppet`
* Entrypoint: `manifests/site.pp`



!SLIDE bullets

# Hello, `manifests/site.pp`!

	import "nodes"

	Exec {
		path => "/usr/sbin:/usr/bin:/sbin:/bin",
	}



!SLIDE bullets
.notes Node and class names may collide.  "default" is special so I use "base" as my most general class name.

# Hello, `manifests/nodes.pp`!

	node default { include base }

	node www inherits default { include www }

	node 'staging.example.com' inherits www {}
	node /\.www\.example\.com$/ inherits www {}



!SLIDE bullets

# Hello, `base`!

## `modules/base/manifests/init.pp`

	class base {
		package {
			"dnsutils": ensure => latest;
			"psmisc": ensure => latest;
			"strace": ensure => latest;
			"sysstat": ensure => latest;
			"telnet": ensure => latest;
		}
	}



!SLIDE bullets

# Hello, `www`!

## `modules/www/manifests/init.pp`

	class www {
		package {
			"nginx":
				ensure => "0.7.65-1ubuntu2";
		}
	}



!SLIDE bullets

# Get yours

* <http://bit.ly/devstructure-puppet-agent>
* <http://bit.ly/devstructure-puppet-master>



!SLIDE bullets
.notes Talk about choosing the right tool for the right job.  For example, scripting `./configure && make && make install` in Puppet is a good sign you should build a package.

# An interlude on<br />package management

* Configuration management is the<br />centralized authority<br />to a package manager&#8217;s<br />local authority.
* (This is my usual answer to<br />&#8220;why do I need this?&#8221;)



!SLIDE bullets

# Security



!SLIDE bullets

# SSL Cliff&#8217;s Notes

* TODO



!SLIDE bullets

# SSL in Puppet

* `/var/lib/puppet/ssl` on agents.
* `puppet cert` on master.
* <http://bit.ly/puppet-security-ref>



!SLIDE bullets

# `autosign.conf`

	foo.example.com
	*.bar.example.com

* Wildcards are a bad idea without some help.
* Help comes in a few slides.



!SLIDE bullets

# `auth.conf`

	path ~ ^/catalog/([^/]+)$
	method find
	allow $1

* Safe by default.  Easy to mistakenly open.
* <http://bit.ly/puppet-security-ref>
* <http://bit.ly/puppet-rest-ref>



!SLIDE bullets

# `fileserver.conf`

	[modulename]
		path /foo/bar/baz
		allow *

* Usually certificates have to be signed.
* <http://bit.ly/puppet-fileserver-ref>



!SLIDE bullets small

# Why to lie to Puppet

## Gather compromising information about other servers

* `iptables` rules.
* Database passwords.
* SSH private keys.
* `/etc/passwd` entries.
* AWS credentials.



!SLIDE bullets

# How to lie to Puppet



!SLIDE bullets

# How to lie to Puppet

## Maybe they&#8217;re `autosign`ing?



!SLIDE bullets

# How to lie to Puppet

## Sneak through regex `node` definitions

	[agent]
		certname=foobarbaz.www.example.com



!SLIDE bullets
.notes TODO VERIFY

# How to lie to Puppet

## Unmatched names fall back to `default`.

	[agent]
		certname=adhadgsdhsfsdhxcb.example.com



!SLIDE bullets

# How to lie to Puppet

## `autosign` disabled?  Try social engineering.

	[agent]
		certname=admin0.example.com
		# certname=dev.example.com
		# certname=test.example.com
		# certname=corp.example.com



!SLIDE bullets small

# How to lie to Puppet

## With a signed certificate, snoop for other catalogs.

	export ssldir=/var/lib/puppet/ssl
	export server=puppetmaster.example.com

	curl --insecure \
		--cert $ssldir/certs/$CERTNAME.pem \
		--key $ssldir/private_keys/$CERTNAME.pem \
		--cacert $ssldir/ca/ca_crt.pem \
		https://$server:8140/$ENV/catalog/db1.example.com



!SLIDE bullets small
.notes If you do have a signed certificate, you can snoop around using it, too.

# How to lie to Puppet

## Snoop around an open file server,<br />no certificates needed.

	export server=puppetmaster.example.com

	curl --insecure \
		https://$server:8140/$ENV/file_content/$MODULE/$FILE



!SLIDE bullets

# How to safely not care

* Don&#8217;t autosign anything, ever.



!SLIDE bullets

# How to safely not care

* <del>Don&#8217;t autosign anything, ever.</del>
* Understand and protect the network.
* Remove special cases.



!SLIDE bullets

# Public and private interfaces

* TODO
* Security aside, use free bandwidth on private interfaces when you can.



!SLIDE bullets

# `iptables`

* TODO



!SLIDE bullets

# `stunnel`

* (Not specific to Puppet.)
* If it&#8217;s on the public Internet,<br />it&#8217;s either public or encrypted.
* Example: use `stunnel`(8)<br />to make `$NoSQL` SSL-aware.



!SLIDE bullets

# TODO Templates

* TODO



!SLIDE bullets

# Use only the `default` node

* Removes advantages of `certname` speculation.
* Use `--config`, `--manifestdir`, or `--manifest` to run different masters listening on different interfaces/ports.



!SLIDE bullets

# Extending puppet

## Where does your code run?

* External node classifier runs on master.
* Plugins run on agents.



!SLIDE bullets

# Example task

* Authorize an SSH key unique to each host.
* By no means impossible in the Puppet language.



!SLIDE bullets

# External node classifier

* Run your own code on the master.
* Make decisions based on<br />more than just the hostname.



!SLIDE bullets

# Configuring an external node classifier

	[master]
		external_nodes=/usr/local/bin/classifier
		node_terminus=exec



!SLIDE bullets

# Input

* Hostname...
* ...which maps to a YAML file full of facts.



!SLIDE bullets

# Facts?

## Key value pairs describing<br />the server in question.

<pre>
--- !ruby/object:Puppet::Node::Facts
  expiration: 2010-09-20 20:27:14.445807 +00:00
  name: &id003 hooah.example.com
  values:
    hardwaremodel: &id002 x86_64
    kernelrelease: 2.6.35.1-rscloud
    selinux: "false"
    sshrsakey: OH HAI
</pre>

* Lots more: `facter | less`



!SLIDE bullets

# Output

* Classes, variables.  No resources.  YAML.

<pre>
---
classes:
  - base
  - www
environment: production
parameters:
  mail_server: mail.example.com
</pre>



!SLIDE bullets small

# Example external node classifier

<pre>
#!/bin/sh
set -e
TMPNAME=$(mktemp "$1.XXXXXXXXXX")
ssh-keygen -q -f "$TMPNAME" -b 2048 -N ""
zomg_post_to_the_api "$1" "$(cat "$TMPNAME.pub")"
cat &lt;&lt;EOF
---
classes:
  - ssh
parameters:
  public_key: "$(cat "$TMPNAME.pub")"
EOF
</pre>



!SLIDE bullets

# Example Puppet class

	class ssh {
		file {
			"/root/.ssh":
				mode   => "700",
				ensure => directory;
			"/root/.ssh/authorized_keys":
				content => "$public_key\n",
				ensure  => file;
		}
	}



!SLIDE bullets

# Puppet plugins

* Because sometimes Puppet won&#8217;t let you.



!SLIDE bullets

# The easy way

* &#8220;It&#8217;s just Ruby.&#8221;



!SLIDE bullets

# The orderly way

* Gather the necessary data.
* Build a Puppet catalog complete with dependency declarations.
* Apply the catalog.



!SLIDE bullets

# Plugin file structure

	modules/ssh/
		manifests/
			init.pp
		lib/puppet/
			type/
				keygen.rb
			provider/keygen/
				posix.rb



!SLIDE bullets

# Types versus providers

* A `package` is a type.<br />`apt` is a provider of packages.
* Types parse and normalize arguments.<br />Providers make it so.



!SLIDE bullets

# Manifest

	class ssh {
		keygen { "name-it-whatever": }
	}



!SLIDE bullets

# Type

	require 'puppet/type'
	Puppet::Type.newtype(:users) do
	  @doc = "ssh-keygen example"
	  newparam(:whatever, :namevar => true) do
	    desc "Name it whatever."
	  end
	  ensurable do
	    self.defaultvalues
	    defaultto :present
	  end
	end



!SLIDE bullets

# Provider

	require 'openssl'
	require 'puppet/resource'
	require 'puppet/resource/catalog'

	Puppet::Type.type(:keygen).
	  provide(:posix) do

	  desc "ssh-keygen example for POSIX"
	  defaultfor :operatingsystem => :debian

	  # Define exists?, create, and destroy.

	end



!SLIDE bullets

	  def exists?
	    File.exists?(
	      "/root/.ssh/authorized_keys")
	  end

	  def destroy
	    Puppet.warning "No turning back."
		raise NotImplementedError
	  end



!SLIDE bullets
.notes TODO See how useful the new-to-2.6 Ruby DSL is to use in this case.

	  def create
	    key = OpenSSL::PKey::RSA.generate(2048)
	    zomg_post_to_the_api TODO hostname, key
	    catalog = Puppet::Resource::Catalog.new
	    catalog.create_resource(:file,
	      :path => "/root/.ssh",
	      :mode => "700",
	      :ensure => :directory
	    )
	    catalog.create_resource(:file,
	      :path => "/root/.ssh/authorized_keys",
	      :content => "#{key.public_key}\n",
	      :ensure => :file
	    )
	    catalog.apply
	  end



!SLIDE bullets

# Which is right - easy or orderly?

* That&#8217;s an exercise for the reader.



!SLIDE bullets

# Thank you

* <richard@devstructure.com> or [@rcrowley](http://twitter.com/rcrowley)
* P.S. use DevStructure.



!SLIDE bullets

# Self quotes

* DevOps is the scalable practice of engineering.
* Version control is not a form of government.

# NOTES

* Break things down into 80/20 rules.
* Go back at the end and fix quotes to look pretty.
* Link to blueprints for client and server.  Actually build them.
* Note when Puppet and Chef are similar and that the only significant difference is in dependency resolution.
* A great example plugin would be generating and authorizing an SSH key for `root` on each server.

* What about fucking with the Puppet REST API via Rack middleware?  Oh yeah!
* What about combining multiple app servers into one?

# Orphaned sections

* Agents run as `root`.  HERE BE DRAGONS.
* (Not really.)
