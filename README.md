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

* znc_system_user

  User to use on the system to run znc.  By default it will be the same user as
  what Ansible uses to connect to the system.

* znc_system_group

  Group to use on the system to run znc.  By default it will be the same group
  as what Ansible uses to connect to the system.

* znc_config_dir

  Base directory for the personal ZNC configuration files. A separate
  subdirectory is created under this path for each server. Must be
  writable by the user running the ZNC services. Must be a full
  path. Defaults to ~/znc.

* znc_base_port

  The TCP port for the base ZNC server to listen on. Clients do not
  usually connect to this server, so the port doesn't really matter,
  but it's configurable anyway. Defaults to 6666.

* znc_max_buffer_size

  The MaxBufferSize for the server.

* znc_user

  Dictionary containing credentials to be used to connect to the ZNC
  bouncer. Generate these values using "znc --makepass".

  * name

    The user name.

  * hash

    The hashed password.

  * method

    The hash encryption method used. Defaults to SHA256.

  * salt

    The salt value used for the encryption.

  * password

    The unencrypted password value, used to let the child servers
    connect to the base server.

* znc_nick

  The nickname to use on the IRC server.

* znc_alt_nick

  The alternate nickname to fall back to if znc_nick is
  taken. Defaults to znc_nick + "_".

* znc_buffer

  The number of lines to buffer. Defaults to 500.

* znc_quit_msg

  Message to use when ZNC shuts down.

* znc_real_name

  A fuller name than the nick or ident.

* znc_extra_modules

  A list of module names to be added to the base server. For example,
  include the "log" module to log all conversations and include
  "webadmin" to enable the web UI.

* znc_networks

  The IRC networks to connect to. For each network, specify:

  * name

    The unique name of the network. For example, "freenode".

  * ident

    The confirmed identity on the IRC service. Frequently this is the
    same as the nick, but multiple nicks can be associated with a single
    identity. (Changed from the single value "znc_ident" as part of 2.x.)

  * server_name

    The hostname or IP of the IRC server. For example,
    "chat.freenode.net".

  * server_port

    The port on which to connect. For SSL connections, append "+" to
    the port number. For example, "6667" or "6667+".

* znc_server_passwords

  Mapping of network names to passwords for connecting to them as the
  confirmed identity in the ident field. Replaces the
  "server_password" parameter to allow the passwords to be stored
  separately in a file managed with ansible-vault.

* znc_channels

  List of names of channels to join by default.

* znc_clients

  List of ZNC instances to run for different clients.

  * name

    The name of the client. Avoid spaces and punctuation because the
    name is used to identify the service and name configuration files
    and directories.

  * host

    The host/IP on which the client should listen.

  * port

    The port on which the client should listen. This port needs to be
    exposed through your firewall. The service runs as the the user
    ansible is using, so the port shouldn't be privileged. (See
    znc_firewall_bypass_port.)

  * ipv4

    Enable IPv4. Defaults to true.

  * ipv6

    Enable IPv6. Defaults to false.

  * buffer

    Override znc_buffer for this connection. Optional, defaults to
    value of znc_buffer.

  * use_ssl

    Boolean flag to control whether SSL is used. Defaults to true.

  * ssl_protocols

    String passed to the SSLProtocols variable in the ZNC
    config. Defaults to empty, which does not set the value and uses
    the default for SSLProtocols.

* znc_firewall_bypass_port

  Many corporate and public wifi networks block outgoing connections
  to "arbitrary" or IRC ports. To bypass these, many users configure
  their IRC bouncer to listen on a port that is more likely to be
  open, such as 443. Because this playbook configures services to run
  as a regular non-root user, the services cannot be bound directly to
  port 443. Instead, this option can be used to specify one port
  number that should be mapped to 443 using rinetd.

Configuring Your IRC Client
---------------------------

Configure your client to connect to your server using one of the
settings from the `znc_clients` variable. SSL is always enabled for
all connections.

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
        # By default we will run znc as the same user/group Ansible uses to
        # connect to the system.  If you prefer to specify a different
        # user/group or if Ansible uses the root user you can specify the
        # user/group to run znc.  To run znc as the user 'znc' and group 'znc'
        # uncomment the following two lines:
        #znc_system_user: znc
        #znc_system_group: znc
        # znc_user and znc_server_passwords can go into a vault-encrypted file.
        znc_user:
          name: dhellmann
          hash: hashhashhash
          method: SHA256
          salt: "saltsaltsalt"
          password: unencryptedpass
        znc_server_passwords:
          freenode: supersecretvalue
          tech404: evenmoresecretvalue
        # The remaining values are safe to leave in the playbook in clear text.
        znc_nick: dhellmann
        znc_quit_msg: disconnecting
        znc_real_name: Doug Hellmann
        znc_networks:
          - name: freenode
            server_name: chat.freenode.net
            server_port: 6667
            ident: dhellmann
            channels:
              - "#openstack"
              - "#openstack-dev"
              - "#openstack-infra"
              - "#openstack-meeting"
              - "#openstack-meeting-3"
              - "#openstack-meeting-alt"
              - "#openstack-oslo"
              - "#wsme"
          - name: tech404
            ident: dhellmann
            server_name: tech404.irc.slack.com
            server_port: "+6697"
            channels:
              - "#python"
              - "#openstack"
        znc_firewall_bypass_port: 6672
        znc_clients:
          - name: hubert
            port: 6667
          - name: lrrr
            port: 6672
            auto_clear_chan_buffer: true
          - name: phone
            port: 6673
            buffer: 100
            auto_clear_chan_buffer: true
          - name: ipad
            port: 6677
            buffer: 100
            auto_clear_chan_buffer: true

License
-------

BSD

Author Information
------------------

Doug Hellmann


TODO
----

* Restart ZNC instances when their configuration files change.
* Support using vault for passwords.
