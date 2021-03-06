---
layout: default
title: cf-serverd
categories: [Reference, Components, cf-serverd]
published: true
alias: reference-components-cfserver.html
tags: [Components, cf-serverd]
---

Server - used to distribute policy and/or data files to clients requesting 
them and used to respond to requests from `cf-runagent`.

`cf-serverd` keeps the promises made in `common` and `server` bundles, and is  
affected by `common` and `server` control bodies.

## Command reference

    '--help'
       (-h) - Print the help message
    '--debug'
       (-d value) - Set debugging level 0,1,2,3
    '--verbose'
       (-v) - Output verbose information about the behavior of the
        agent
    '--version'
       (-V) - Output the version of the software
    '--file'
       (-f value) - Specify an alternative input file than the default
    '--define'
       (-D value) - Define a list of comma separated classes to be
        defined at the start of execution
    '--negate'
       (-N value) - Define a list of comma separated classes to be
        undefined at the start of execution
    '--no-lock'
       (-K) - Ignore locking constraints during execution
        (ifelapsed/expireafter) if "too soon" to run
    '--inform'
       (-I) - Print basic information about changes made to the
        system, i.e. promises repaired
    '--diagnostic'
       (-x) - Activate internal diagnostics (developers only)
    '--no-fork'
       (-F) - Run as a foreground processes (do not fork)
    '--ld-library-path'
       (-L value) - Set the internal value of LD\_LIBRARY\_PATH for
        child processes
    '--legacy-output'
       (-l) - Use legacy output format

Debug levels: 1=parsing, 2=running, 3=summary, 4=expression eval


## Control Promises

Settings describing the details of the fixed behavioral promises made by 
`cf-serverd`. Server controls are mainly about determining access policy for 
the connection protocol: i.e. access to the server itself. Access to specific 
files must be granted in addition.

```cf3
    body server control
    {
        allowconnects         => { "127.0.0.1" , "::1" ,  ".*\.example\.org" };
        allowallconnects      => { "127.0.0.1" , "::1" ,  ".*\.example\.org" };

        # Uncomment me under controlled circumstances
        #trustkeysfrom         => { "127.0.0.1" , "::1" ,  ".*\.example\.org" };
    }
```


### allowallconnects

**Type**: `slist`

**Allowed input range**: (arbitrary string)

**Description**: List of IPs or hostnames that may have more than one
connection to the server port

    allowallconnects      => { 
         "127.0.0.1", 
         "::1", 
         "200\.1\.10\..*", 
         "host\.domain\.tld", 
         "host[0-9]+\.domain\.com" 
         };

**Notes**:

This list of regular expressions matches hosts that are allowed to
connect an unlimited number of times up to the maximum connection
limit. Without this, a host may only connect once (which is a very
strong constraint, as the host must wait for the TCP FIN\_WAIT to
expire before reconnection can be attempted).

Note that `127.0.0.1` is a regular expression (i.e., "127 any
character 0 any character 0 any character 1"), but this will only
match the IP address `127.0.0.1`. Take care with IP addresses and
domain names, as the hostname regular expression `www.domain.com`
will potentially match more than one hostname (e.g.,
`wwwxdomain.com`, in addition to the desired hostname
`www.domain.com`).


### allowconnects

**Type**: `slist`

**Allowed input range**: (arbitrary string)

**Description**: List of IPs or hostnames that may connect to the
server port

    allowconnects => { 
         "127.0.0.1", 
         "::1", 
         "200\.1\.10\..*", 
         "host\.domain\.tld", 
         "host[0-9]+\.domain\.com" 
         };

**Notes**:

If a client's identity matches an entry in this list it is granted
to permission to send data to the server port. Clients who are not
in this list may not connect or send data to the server.

See also the warning about regular expressions in
`allowallconnects`.


### allowusers

**Type**: `slist`

**Allowed input range**: (arbitrary string)

**Description**: List of usernames who may execute requests from this
server

    allowusers => { "cfengine", "root" };

**Notes**:

The usernames listed in this list are those asserted as public key
identities during client-server connections. These may or may not
correspond to system identities on the server-side system.


### auditing

**Type**: (menu option)

**Allowed input range**:

    true
    false
    yes
    no
    on
    off

**Default value:** false

**Description**: true/false activate auditing of server connections

    body agent control
    {
    auditing  => "true";
    }

**Notes**:

If this is set, CFEngine will perform auditing on promises in the
current configuration. This means that all details surrounding the
verification of the current promise will be recorded in the audit
database.


### bindtointerface

**Type**: `string`

**Allowed input range**: (arbitrary string)

**Description**: IP of the interface to which the server should bind
on multi-homed hosts

    bindtointerface => "192.168.1.1";

**Notes**:

On multi-homed hosts, the server and client can bind to a specific
interface for server traffic. The IP address of the interface must
be given as the argument, not the device name.


### cfruncommand

**Type**: `string`

**Allowed input range**: `.+`

**Description**: Path to the cf-agent command or cf-execd wrapper for
remote execution

    body server control
    
    {
    cfruncommand => "/var/cfengine/bin/cf-agent";
    }
    

**Notes**:

It is normal for this to point to the location of `cf-agent` but it
could also point to the `cf-execd`, or even another program or
shell command at your own risk.


### call_collect_interval

CFEngine Enterprise only.

**Type**: `int`

**Allowed input range**: `0,99999999999`

**Description**: The interval in minutes in between collect calls to
the CFEngine Server offering a tunnel for report collection.

    call_collect_interval => "5";

**Notes**:

**History**: Was introduced Enterprise 3.0.0 (2012)

If option time is set, it causes the server daemon to peer with a
policy hub by attempting a connection at regular intervals of the
value of the parameter in minutes.

This feature is designed to allow Enterprise report collection from
hosts that are not directly addressable from a hub data-aggregation
process. For example, if some of the clients of a policy hub are
behind a network address translator then the hub is not able to
open a channel to address them directly. The effect is to place a
'collect call' with the policy hub.

If this option is set, the client's `cf-serverd` will "peer" with
the server daemon on a policy hub. This means that, `cf-serverd` on
an unreachable (e.g. NATed) host will attempt to report in to the
`cf-serverd` on its assigned policy hub and offer it a short time
window in which to download reports over the established
connection. The effect is to establish a temporary secure tunnel
between hosts, initiated from the satellite host end. The
connection is made in such a way that host autonomy is not
compromised. Either hub may refuse or decline to play their role at
any time, in the usual way (avoiding DOS attacks). Normal access
controls must be set for communication in both directions.

Collect calling cannot be as efficient as data collection by the
`cf-hub`, as the hub is not able to load balance. Hosts that use this
approach should exclude themselves from the cf-hub data
collection.

The sequence of events is this:

-   The host's `cf-serverd` connects to its registered CFEngine Server
-   The host identifies itself to authentication and access
    control and sends a collect-call pull-request to the server
-   The server might honor this, if the access control grants access.
-   If access is granted, the server has `collect_window` seconds to
    initiate a query to the host for its reports.
-   The server identifies itself to authentication and access
    control and sends a query request to the host to collect the
    reports.
-   When finished, the host closes the tunnel.

The full configuration would look something like this

```cf3
        #########################################################
        # Server config
        #########################################################
        
        body server control 
        {
        allowconnects         => { "10.10.10" , "::1" };
        allowallconnects      => { "10.10.10" , "::1" };
        trustkeysfrom         => { "10.10.10" , "::1" };
        
        call_collect_interval => "5";
        }
        
        #########################################################
        
        bundle server access_rules()
        
        {
        access:
        
          policy_hub::
        
           "collect_calls"
               resource_type => "query",
                     admit   => { "10.10.10" }; # the apparent NAT address of the satellite
        
          satellite_hosts::
         
            "delta"
                     comment => "Grant access to cfengine hub to collect report deltas",
               resource_type => "query",
                     admit   => { "policy_hub" };
        
            "full"
                    comment => "Grant access to cfengine hub to collect full report dump",
              resource_type => "query",
                    admit   => { "policy_hub"  };
        }
```


### collect_window

CFEngine Enterprise only.

**Type**: `int`

**Allowed input range**: `0,99999999999`

**Description**: A time in seconds that a collect-call tunnel remains
open to a hub to attempt a report transfer before it is closed
(Enterprise)

    collect_window => "15";

**Notes**:

**History**: Was introduced in Enterprise 3.0.0 (2012)

The time is measured in seconds, default value 10s.


### denybadclocks

**Type**: (menu option)

**Allowed input range**:

    true
    false
    yes
    no
    on
    off

**Default value:** true

**Description**: true/false accept connections from hosts with clocks
that are out of sync

    body server control
    {
    #..
    denybadclocks => "true";
    }

**Notes**:

A possible form of attack on the fileserver is to request files
based on time by setting the clocks incorrectly. This option
prevents connections from clients whose clocks are drifting too far
from the server clock (where "too far" is currently defined as
"more than an hour off"). This serves as a warning about clock
asynchronization and also a protection against Denial of Service
attempts based on clock corruption.


### denyconnects

**Type**: `slist`

**Allowed input range**: (arbitrary string)

**Description**: List of IPs or hostnames that may NOT connect to the
server port

    body server control
    {
    denyconnects => { "badhost\.domain\.evil", "host3\.domain\.com" };
    }

**Notes**:

Hosts or IP addresses that are explicitly denied access. This
should only be used in special circumstances. One should never
grant generic access to everything and then deny special cases.
Since the default server behavior is to grant no access to
anything, this list is unnecessary unless you have already granted
access to some set of hosts using a generic pattern, to which you
intend to make an exception.

See also the warning about regular expressions in
`allowallconnects`.


### dynamicaddresses

**Type**: `slist`

**Allowed input range**: (arbitrary string)

**Description**: List of IPs or hostnames for which the IP/name
binding is expected to change

    body server control
    {
    dynamicaddresses => { "dhcp_.*" };
    }

**Notes**:

The addresses or hostnames here are expected to have non-permanent
address-name bindings, we must therefor work harder to determine
whether hosts credentials are trusted by looking for existing
public keys in files that do not match the current hostname or IP.

**This feature has been deprecated since 3.1.0.** This is now
handled transparently.


### hostnamekeys

**Type**: (menu option)

**Allowed input range**:

    true
    false
    yes
    no
    on
    off

**Default value:** false

**Description**: true/false store keys using hostname lookup instead
of IP addresses

    body server control
    {
    hostnamekeys => "true";
    }

**Notes**:

Client side choice to base key associations on host names rather
than IP address. This is useful for hosts with dynamic addresses.

**This feature has been deprecated since 3.1.0.** Host
identification is now handled transparently.


### keycacheTTL

**Type**: `int`

**Allowed input range**: `0,99999999999`

**Default value:** 24

**Description**: Maximum number of hours to hold public keys in the
cache

    body server control
    {
    keycacheTTL => "24";
    }

**Notes**:

**History**: Was introduced in version 3.1.0b1,Enterprise 2.0.0b1 (2010)


### logallconnections

**Type**: (menu option)

**Allowed input range**:

    true
    false
    yes
    no
    on
    off

**Default value:** false

**Description**: true/false causes the server to log all new
connections to syslog

    body server control
    {
    logallconnections => "true";
    }

**Notes**:

If set, the server will record connection attempts in syslog.


### logencryptedtransfers

**Type**: (menu option)

**Allowed input range**:

    true
    false
    yes
    no
    on
    off

**Default value:** false

**Description**: true/false log all successful transfers required to
be encrypted

    body server control
    {
    logencryptedtransfers => "true";
    }

**Notes**:

If true the server will log all transfers of files which the server
requires to encrypted in order to grant access (see `ifencrypted`)
to syslog. These files are deemed to be particularly sensitive.


### maxconnections

**Type**: `int`

**Allowed input range**: `0,99999999999`

**Default value:** 30 remote queries

**Description**: Maximum number of connections that will be accepted

    # client side 
    
    body agent control
    {
    maxconnections => "1000";
    }
    
    # server side
    
    body server control
    {
    maxconnections => "1000";
    }

**Notes**:

Watch out for kernel limitations for maximum numbers of open file
descriptors which can limit this.


### port

**Type**: `int`

**Allowed input range**: `1024,99999`

**Default value:** 5308

**Description**: Default port for cfengine server

    body hub control
    {
    port => "5308";
    }
    
    body server control
    {
    specialhost::
     port => "5308";
    
    !specialhost::
     port => "5308";
    }

**Notes**:

The standard or registered port number is tcp/5308. CFEngine does
not presently use its registered udp port with the same number, but
this could change in the future.

Changing the standard port number is not recommended practice. You
should not do it without a good reason.


### serverfacility

**Type**: (menu option)

**Allowed input range**:

    LOG_USER
    LOG_DAEMON
    LOG_LOCAL0
    LOG_LOCAL1
    LOG_LOCAL2
    LOG_LOCAL3
    LOG_LOCAL4
    LOG_LOCAL5
    LOG_LOCAL6
    LOG_LOCAL7

**Default value:** LOG\_USER

**Description**: Menu option for syslog facility level

    body server control
    {
    serverfacility => "LOG_USER";
    }

**Notes**:

See syslog notes.


### skipverify

**Type**: `slist`

**Allowed input range**: (arbitrary string)

**Description**: List of IPs or hostnames for which we expect no DNS
binding and cannot verify

    body server control
    {
    skipverify => { "special_host.*", "192.168\..*" };
    }

**Notes**:

Server side decision to ignore requirements of DNS identity
confirmation.

See also the warning about regular expressions in
`allowallconnects`.


### trustkeysfrom

**Type**: `slist`

**Allowed input range**: (arbitrary string)

**Description**: List of IPs from whom we accept public keys on trust

    body server control
    {
    trustkeysfrom => { "10\.0\.1\.1", "192\.168\..*"};
    }

**Notes**:

If connecting hosts' public keys have not already been trusted, this allows us 
to accept the keys on trust. Normally this should be an empty list except in 
controlled circumstances.

See also the warning about regular expressions in `allowallconnects`.


### listen

**Type**: (menu option)

**Allowed input range**:

    true
    false
    yes
    no
    on
    off

**Default value:** true

**Description**: true/false enable server deamon to listen on defined
port

    body server control
    {
     
      listening_host_context::
        listen => "true";
    
      !listening_host_context::
        listen => "false";
    }

**Notes**:

**History**: Was introduced in 3.4.0, Enterprise 3.0 (2012)

This attribute allows to disable `cf-serverd` from listening on any
port. Should be used in conjunction with `call_collect_interval`.

This setting only applies to CFEngine clients, the policy hub will
not be affected. Changing this setting requires a restart of
`cf-serverd` for the change to take effect.

