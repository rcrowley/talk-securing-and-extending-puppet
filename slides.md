!SLIDE

# Securing and Extending Puppet for World Domination



!SLIDE bullets

# Hi, I&#8217;m Richard Crowley

* Equal opportunity technology hater
* DevStructure&#8217;s operator and UNIX hacker



!SLIDE bullets

# Configuration management<br />Cliff&#8217;s Notes



!SLIDE bullets
.notes Your infrastructure is never an immutable black box.  It is one step in a long iteration.

# Infrastructure is code

* You can reason about code in ways you can&#8217;t about a tarball or AMI.



!SLIDE bullets
.notes Just as you don't have to think about capacitors to microwave your burrito, you don't have to think about the intermediate steps to know how you want your server to be.

# Declare state,<br />not process

* More precise.  Less verbose.
* The state of a server is easier to unambiguously describe.



!SLIDE bullets

# Why manage configuration?



!SLIDE bullets
.notes Talk about choosing the right tool for the right job.  For example, scripting `./configure && make && make install` in Puppet is a good sign you should build a package.

# An interlude on<br />package management

* Configuration management is the<br />centralized authority<br />to a package manager&#8217;s<br />local authority.



!SLIDE bullets
.notes `~/bin/doit5` may never work again.  It's a good thing that the Puppet language is not itself Ruby but I'm not here to start a holy war.  Limits in configuration management serve the same purpose as in a templating language: they enforce separation of concerns.  In this case, the limits separate process from desired state.

# Puppet, Chef,<br />and `~/bin/doit5`

* Limits are good.
* Puppet and Chef are idempotent by default.



!SLIDE bullets
.notes Puppet and Chef work basically the same way from this altitude.

# Architecture

* Master knows best.
* Agents phone home.
* Hostname matching.
* Resources.
* Agents make it so.



!SLIDE bullets

# Resources
.notes Puppet and Chef again work the same way here.  The single difference is in the granularity of dependency declarations.  Puppet's are at the resource level.  Chef's are at the cookbook (module) level.

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

# A basic configuration<br />for Puppet itself



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

# (R)TFM

* `puppet --genconfig`
* <http://bit.ly/puppet-config-ref>



!SLIDE bullets

# Hello, world!

* Entire thing in `/etc/puppet`
* Entrypoint is `manifests/site.pp`



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
		package { "nginx":
			ensure => "0.7.65-1ubuntu2",
		}
	}



!SLIDE bullets

# Get yours

* <http://bit.ly/devstructure-puppet-agent>
* <http://bit.ly/devstructure-puppet-master>



!SLIDE bullets

# Security



!SLIDE bullets

# SSL Cliff&#8217;s Notes



!SLIDE bullets

# SSL handshake

* Server certificate authenticates<br />server to client.
* Clients may verify a server&#8217;s certificate against trusted certificate authority.
* Client and server compute matching<br />secret keys.



!SLIDE bullets

# Client certificates

* Client certificate authenticates<br />client to server.
* Not used by browsers.
* Used by Puppet because you can&#8217;t lie here.



!SLIDE bullets

# SSL in Puppet

* `/var/lib/puppet/ssl` on agents.
* `puppet cert` on master.
* <http://bit.ly/puppet-security-ref>



!SLIDE bullets

# A tour of secondary Puppet config files



!SLIDE bullets

# `autosign.conf`

	foo.example.com
	*.www.example.com
	*

* A bad idea when there are untrusted clients.



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

* Maybe they&#8217;re `autosign`ing?



!SLIDE bullets

# How to lie to Puppet

## Sneak through regex `node` definitions.

	[agent]
		certname=foo.www.example.com



!SLIDE bullets

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

# The network

* Understand who can contact your<br />Puppet master.
* Private networks may be shared.



!SLIDE bullets small

# `iptables`

	@@@ sh
	iptables -P INPUT ACCEPT
	iptables -P OUTPUT ACCEPT
	iptables -P FORWARD ACCEPT

	iptables -F

	iptables -A INPUT -m conntrack \
		--ctstate RELATED,ESTABLISHED -j ACCEPT

	iptables -A INPUT -i eth1 -p tcp \
		-s 10.47.0.0/16 --dport 8140 -j ACCEPT
	iptables -A INPUT -i eth1 -p udp \
		-s 10.47.0.0/16 --dport 8140 -j ACCEPT

	iptables -A INPUT -i lo -j ACCEPT

	iptables -A INPUT -j DROP



!SLIDE bullets

# `stunnel`

* (Not specific to Puppet.)
* If it&#8217;s on the Internet,<br />it&#8217;s either public or encrypted.
* Example: use `stunnel`(8)<br />to make `$NoSQL` SSL-aware.



!SLIDE bullets small

# `stunnel` Upstart config

	@@@ sh
	description	"stunnel-redis-client"
	start on runlevel [2345]
	stop on runlevel [!2345]
	respawn
	exec /usr/bin/stunnel -f -c -d localhost:6379 \
		-r redis.example.com:6381

	description	"stunnel-redis-server"
	start on runlevel [2345]
	stop on runlevel [!2345]
	respawn
	exec /usr/bin/stunnel -f -d 6381 -r localhost:6382



!SLIDE bullets

# Special cases<br />aren&#8217;t that special



!SLIDE bullets
.notes Hostname-specific file paths can mitigate this risk.  Leak example: malicious server grabbing the database config or secrets file.

# Use templates

<pre>
file { "/foo/bar/baz":
	<del>source  => "puppet://foo/bar/baz",</del>
	content => template("foo/bar/baz"),
	ensure  => file,
}
</pre>

* File and catalog serving use different ACLs.
* Template content is included in the catalog so they inherit the catalog&#8217;s ACL.



!SLIDE bullets

# Use only the `default` node

* Mitigate the consequences of<br />successful `certname` speculation.
* Use `--config`, `--manifestdir`, or `--manifest` to run different masters listening<br />on different interfaces/ports.



!SLIDE bullets

# Extending Puppet



!SLIDE bullets

# Maybe don&#8217;t

	$extlookup_datadir = "/etc/puppet/extdata"
	$extlookup_precedence = [
		"%{fqdn}", "%{domain}", "base"]

	file { "/foo/bar/baz":
		content => extlookup("foobarbaz"),
		ensure  => file,
	}

* `extlookup` function can retrieve data from external CSV files.



!SLIDE bullets

# Then again, maybe<br />extend Puppet



!SLIDE bullets

# Where does<br />your code run?

* External node classifier runs on master.
* Plugins run on agents.



!SLIDE bullets

# Example task

* Authorize an SSH key unique to each host.
* (This is by no means impossible in the Puppet language.)



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
* ...which maps to facts.

<br />
<br />
<pre>
/usr/local/bin/classifier foo.example.com

# Facts are available as YAML in
# $vardir/yaml/facts/foo.example.com.yaml
</pre>


!SLIDE bullets

# Facts?

## Key value pairs describing<br />the server in question.

	--- !ruby/object:Puppet::Node::Facts
	  expiration: 2010-09-20 20:27:14.445807
	  name: &id003 foo.example.com
	  values:
	    hardwaremodel: &id002 x86_64
	    kernelrelease: 2.6.35.1-rscloud
	    selinux: "false"
	    sshrsakey: OH HAI

* Lots more: `facter | less`



!SLIDE bullets

# Output

	---
	classes:
	  - base
	  - www
	environment: production
	parameters:
	  mail_server: mail.example.com

* Classes, variables.  No resources.  YAML.



!SLIDE bullets small

# Example external node classifier

	@@@ sh
	#!/bin/sh
	set -e
	TMP=$(mktemp -d "$1.XXXXXXXXXX")
	ssh-keygen -q -f "$TMP/id_rsa" -b 2048 -N ""
	zomg_post_to_the_api "$1" "$(cat "$TMP/id_rsa")"
	cat <<EOF
	---
	classes:
	  - ssh
	parameters:
	  public_key: $(cat "$TMP/id_rsa.pub")
	EOF



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

<pre>
modules/ssh/
	manifests/
		<strong>init.pp</strong>
	lib/puppet/
		type/
			<strong>keygen.rb</strong>
		provider/<strong>keygen</strong>/
			<strong>posix.rb</strong>
</pre>



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

	@@@ ruby
	require 'puppet/type'
	Puppet::Type.newtype(:keygen) do
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

	@@@ ruby
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

	@@@ ruby
	  def exists?
	    File.exists?(
	      "/root/.ssh/authorized_keys")
	  end

	  def destroy
	    Puppet.warning "No turning back."
		raise NotImplementedError
	  end



!SLIDE bullets

	@@@ ruby
	  def create
	    key = OpenSSL::PKey::RSA.generate(2048)
	    zomg_post_to_the_api \
	      Facter.value(:certname), key
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

# Which is right?<br />Easy or orderly?

* That&#8217;s an exercise for the reader.



!SLIDE bullets

# Extending Puppet

## (One more thing.)



!SLIDE bullets

# Rack middleware

* Puppet master is Rack application.
* Rack allows pre- and post- processing.



!SLIDE bullets

# `config.ru`

	@@@ ruby
	$0 = "master"
	ARGV << "--rack"
	ARGV << "--certname=#{
	  File.read("/etc/puppet/certname").chomp}"

	require 'puppet/application/master'

	# TODO Middleware.

	run Puppet::Application[:master].run



!SLIDE bullets smaller

# No-op middleware

	@@@ ruby
	require 'base64'
	require 'json'
	require 'rack/utils'
	require 'yaml'
	require 'zlib'

	class StuckInTheMiddleWithYou

	  def initialize(app)
	    @app = app
	  end

	  def call(env)
		# TODO Preprocessing.
	    status, headers, body = @app.call(env)
		# TODO Postprocessing.
	    [status, headers, body]
	  end

	end

	use StuckInTheMiddleWithYou



!SLIDE bullets smaller

# Preprocessing

	@@@ ruby
	    params = Rack::Utils.parse_query(env["QUERY_STRING"], "&")
	    facts = case params["facts_format"]
	    when "b64_zlib_yaml"
	      YAML.load(Zlib::Inflate.inflate(Base64.decode64(
	        Rack::Utils.unescape(params["facts"]))))
	    end

	    # Change facts.
	    if Puppet::Node::Facts === facts
	      facts.values["foo"] = "bar"
	    end

	    params["facts"] = case params["facts_format"]
	    when "b64_zlib_yaml"
	      Rack::Utils.escape(Base64.encode64(Zlib::Deflate.deflate(
	        YAML.dump(facts), Zlib::BEST_COMPRESSION)))
	    end if facts
	    env["QUERY_STRING"] = Rack::Utils.build_query(params)
	    env["REQUEST_URI"] =
	      "#{env["PATH_INFO"]}?#{env["QUERY_STRING"]}"



!SLIDE bullets smaller

# Postprocessing

	@@@ ruby
	    object = case headers["Content-Type"]
	    when /[\/-]pson$/ then JSON.parse(body.body.join)
	    when /[\/-]yaml$/ then YAML.load(body.body.join)
	    when "text/marshal" then Marshal.load(body.body.join)
	    else body.body.join
	    end

	    # Change catalog.
	    if Hash === object && "Catalog" == object["document_type"]
	      object["data"]["resources"].unshift({
	        "exported" => false,
	        "title" => "apt-get update",
	        "parameters" => {"path"=>"/usr/sbin:/usr/bin:/sbin:/bin"},
	        "type" => "Exec",
	      })
	    end

	    body = case headers["Content-Type"]
	    when /[\/-]pson$/ then [JSON.generate(object)]
	    when /[\/-]yaml$/ then [YAML.dump(object)]
	    when "text/marshal" then [Marshal.dump(object)]
	    else [object]
	    end
		headers["Content-Length"] = Rack::Utils.bytesize(body.first)



!SLIDE bullets
.notes Mention this is a last resort and that I've fixed bugs in Puppet itself via this method.

# Rack middleware

* Drink responsibly.
* <http://gist.github.com/602922>



!SLIDE bullets

# Thank you

* <richard@devstructure.com> or [@rcrowley](http://twitter.com/rcrowley)
* <http://rcrowley.org/talks/strange-loop-2010>
* P.S. use DevStructure.
