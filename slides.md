!SLIDE

# Securing and Extending Puppet for World Domination



!SLIDE bullets

# Hi, I'm Richard Crowley

* Equal opportunity technology hater
* DevStructure's operator and UNIX hacker
* TODO



!SLIDE bullets

# Configuration management Cliff's Notes



!SLIDE bullets

# Infrastructure as code



!SLIDE bullets

# Declare state, not process



!SLIDE bullets

# Puppet, Chef, and `~/bin/doit5`

* Limits are good.  (It's a good thing that the Puppet language is not itself Ruby but I'm not here to start a holy war.)
* Puppet and Chef are idempotent by default.



!SLIDE bullets

# Architecture

* Master knows best.
* Agents phone home.
* Hostname matching.
* Resources.
* Modules and classes or cookbooks and recipes.
* Agents make it so.



!SLIDE bullets

# Master config



!SLIDE bullets

# Agent config



!SLIDE bullets

# Hello, world!



!SLIDE bullets

# An interlude on package management

* Configuration management is the centralized authority to a package manager's local authority.
* This is my usual answer to "why do I need this?"



!SLIDE bullets

# "Internet scale"

* What the fuck did I mean by this?
* HA?



!SLIDE bullets

# Security

* Puppet communicates over SSL.  (So does Chef.)
* Agents run as root.  HERE BE DRAGONS.  (Not really.)



!SLIDE bullets

# SSL Cliff's Notes



!SLIDE bullets

# SSL in Puppet

* `/var/lib/puppet/ssl` on agents.
* `puppet cert` on master.



!SLIDE bullets

# Why to lie to Puppet



!SLIDE bullets

# How to lie to Puppet



!SLIDE bullets

# How to safely not care



!SLIDE bullets

# `iptables`



!SLIDE bullets

# Where does your code run?

* Plugins run on agents.
* External node classifier runs on master.



!SLIDE bullets

# `stunnel`

* Good advise not specific to Puppet.
* If it's on the public Internet, it's either public or encrypted.
* Use `stunnel`(8) over (for example) MySQL's builtin SSL because it's persistent.



!SLIDE bullets

# External node classifier

* Make decisions based on more than just the hostname.



!SLIDE bullets

# Configuring an external node classifier



!SLIDE bullets

# Input

* Hostname...
* ...which maps to a YAML file full of facts.
* Facts?  Key value pairs describing the server in question.



!SLIDE bullets

# Output

* Classes, variables.  No resources.
* YAML.



!SLIDE bullets

# Puppet plugins

* Because sometimes the declarative language won't let you.



!SLIDE bullets

# The easy way

* "It's just Ruby."



!SLIDE bullets

# The orderly way

* Gather the necessary data.
* Build a Puppet catalog complete with dependency declarations.
* Apply the catalog.



!SLIDE bullets

# Plugin file structure



!SLIDE bullets

# Types versus providers

* Concept portable to Chef.



!SLIDE bullets

# A catalog written in Ruby.

* TODO See how useful the new-to-2.6 Ruby DSL is to use in this case.




!SLIDE bullets

# Which is right - easy or orderly?

* That's an exercise for the reader.



!SLIDE bullets

# Pragmatic cloudiness

* Use free bandwidth when you can.



!SLIDE bullets

# Thank you

* P.S. use DevStructure.



!SLIDE bullets

# Self quotes

* DevOps is the scalable practice of engineering.
* Version control is not a form of government.

# NOTES

* Break things down into 80/20 rules.
* Go back at the end and fix quotes to look pretty.
* Link to blueprints for client and server.
* Note when Puppet and Chef are similar and that the only significant difference is in dependency resolution.
* A great example plugin would be generating and authorizing an SSH key for `root` on each server.
