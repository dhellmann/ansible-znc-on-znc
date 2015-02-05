znc-on-znc
==========

This role makes it easy to create a set of connected ZNC bouncer
instances, so that each IRC client has its own scrollback.

This work is based on the work of Sean Dague and Dan Smith, as
described in https://dague.net/2014/09/13/my-irc-proxy-setup/

A base ZNC instance is configured to talk to the IRC servers
upstream. Then separate child servers are created and exposed for
clients to connect to. Each child server has its own self-signed SSL
certificate and ZNC configuration file. A monit service is configured
for each ZNC server to ensure that the bouncer itself stays running.

Requirements
------------

You need accounts on whatever IRC server(s) you use.

You will need to manage the firewall settings for the host you run
this on to expose the ports configured for each client service.

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
