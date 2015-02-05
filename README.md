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

You need to generate hashed password values for the user connecting to
the ZNC servers. Generate these values using "znc --makepass".

You will need to manage the firewall settings for the host you run
this on to expose the ports configured for each client service.

Role Variables
--------------

znc_config_dir

  Base directory for the personal ZNC configuration files. A separate
  subdirectory is created under this path for each server. Must be
  writable by the user running the ZNC services. Must be a full
  path. Defaults to ~/znc.

znc_base_port

  The TCP port for the base ZNC server to listen on. Clients do not
  usually connect to this server, so the port doesn't really matter,
  but it's configurable anyway. Defaults to 6666.

znc_max_buffer_size

  The MaxBufferSize for the server.

znc_user

  Dictionary containing credentials to be used to connect to the ZNC
  bouncer. Generate these values using "znc --makepass".

  name

    The user name.

  hash

    The hashed password.

  method

    The hash encryption method used.

  salt

    The salt value used for the encryption.

znc_nick

  The nickname to use on the IRC server.

znc_alt_nick

  The alternate nickname to fall back to if znc_nick is
  taken. Defaults to znc_nick + "_".

znc_buffer

  The number of lines to buffer. Defaults to 500.

znc_ident

  The confirmed identity on the IRC service. Frequently this is the
  same as the nick, but multiple nicks can be associated with a single
  identity.

znc_quit_msg

  Message to use when ZNC shuts down.

znc_real_name

  A fuller name than the nick or ident.

znc_networks

  The IRC networks to connect to. For each network, specify:

  name

    The unique name of the network. For example, "freenode".

  server_name

    The hostname or IP of the IRC server. For example,
    "chat.freenode.net".

  server_port

    The port on which to connect. For SSL connections, append "+" to
    the port number. For example, "6667" or "6667+".

  server_password

    The password associated with znc_ident on the server.

znc_channels

  List of names of channels to join by default.

Dependencies
------------

None

Example Playbook
----------------

Including an example of how to use your role (for instance, with
variables passed in as parameters) is always nice for users too:

  - hosts: znc
    roles:
      - znc-on-znc
    vars:
      znc_user:
        name: dhellmann
        hash: hashhashhash
        method: SHA256
        salt: "saltsaltsalt"
      znc_ident: dhellmann
      znc_nick: dhellmann
      znc_quit_msg: disconnecting
      znc_real_name: Doug Hellmann
      znc_networks:
        - name: freenode
          server_name: chat.freenode.net
          server_port: 6667
          server_password: supersecretvalue
      znc_channels:
        - "#openstack"
        - "#openstack-dev"
        - "#openstack-infra"
        - "#openstack-meeting"
        - "#openstack-meeting-3"
        - "#openstack-meeting-alt"
        - "#openstack-oslo"
        - "#wsme"

License
-------

BSD

Author Information
------------------

Doug Hellmann
