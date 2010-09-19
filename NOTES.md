# `intro`

## Securing and Extending Puppet for World Domination

## Hi, I'm Richard Crowley

# `basics`

## Configuration management Cliff's Notes

## Infrastructure as code

## Declare state, not process

## Puppet, Chef, and `~/bin/doit5`

## Architecture

* Master knows best.
* Agents phone home.
* Hostname matching.
* Resources.
* Modules and classes or cookbooks and recipes.
* Agents make it so.

## Master config

## Agent config

## Hello, world!

## An interlude on package management

* Configuration management is the centralized authority to a package manager's local authority.
* This is my usual answer to "why do I need this?"

## "Internet scale"

* What the fuck did I mean by this?
* HA?

# `security`

## Security

* Puppet communicates over SSL.  (So does Chef.)
* Agents run as root.  HERE BE DRAGONS.  (Not really.)

## SSL Cliff's Notes

## SSL in Puppet

* `/var/lib/puppet/ssl` on agents.
* `puppet cert` on master.

## Why to lie to Puppet

## How to lie to Puppet

## How to safely not care

## `iptables`

## Where does your code run?

* Plugins run on agents.
* External node classifier runs on master.

## `stunnel`

* Good advise not specific to Puppet.
* If it's on the public Internet, it's either public or encrypted.
* Use `stunnel`(8) over (for example) MySQL's builtin SSL because it's persistent.

# `external_nodes`

## External node classifier

* Make decisions based on more than just the hostname.

## Configuring an external node classifier

## Input

* Hostname...
* ...which maps to a YAML file full of facts.
* Facts?  Key value pairs describing the server in question.

## Output

* Classes, variables.  No resources.
* YAML.

# `plugins`

## Puppet plugins

* Because sometimes the declarative language won't let you.

## The easy way

* "It's just Ruby."

## The orderly way

* Gather the necessary data.
* Build a Puppet catalog complete with dependency declarations.
* Apply the catalog.

## Plugin file structure

## Types versus providers

* Concept portable to Chef.

## A catalog written in Ruby.

* TODO See how useful the new-to-2.6 Ruby DSL is to use in this case.

## Which is right - easy or orderly?

* That's an exercise for the reader.

# `outro`

## Pragmatic cloudiness

* Use free bandwidth when you can.

## Thank you

* P.S. use DevStructure.



# Self quotes

* DevOps is the scalable practice of engineering.
* Version control is not a form of government.

# NOTES

* Break things down into 80/20 rules.
* Go back at the end and fix quotes to look pretty.
* Link to blueprints for client and server.
* Note when Puppet and Chef are similar and that the only significant difference is in dependency resolution.
* A great example plugin would be generating and authorizing an SSH key for `root` on each server.
