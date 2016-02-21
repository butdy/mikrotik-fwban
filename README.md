# Mikrotik-fwban

* Mikrotik-fwban acts as a syslog receiver and tries to extract an IP address
  out of messages received. It then adds the IPs to the banlist on the
  configured Mikrotiks. In essence it is a Fail2Ban done the lazy way.
  It leverages the filtering mechanisms of rsyslog to do the pre-filtering
  so even if you have a large set of publicly accessable machines, it
  should be able to handle them (famous last words, I know).
* It handles both IPv4 and IPv6 addresses and banlists.
* It can handle multiple Mikrotiks, keeping the banned IPs in their
  respective banlists in sync.

## Config file

Seems kind of self explanatory so I'm not going to explain every item
in it.

## Installation

I presume you have a working experiance with go, a system with systemd
and rsyslogd and in general some sys admin knowledge as I am not able
to support you with questions on every conveivable way to build, install
and start this daemon at startup.

### Building the binary

* Clone, download, copy/paste the source files onto your local disk.
* Execute `go build .` to create the mikrotik-fwban binary.
* Copy the binary to /usr/local/sbin.

### Mikrotik changes

* Create a group (`apis`) on your mikrotik (system > users; groups) and
  give it at least the `read`, `write` and `api` policies.
* Create a user on your mikrotik (system > users; users) and have it
  belong to the group you just created.
* Make sure you have rules in your mikrotik (input AND forward) to drop
  traffic coming from src ips in the `banlist` addresslist.

### Setup your system.

* Copy mikrotik.gcfg to /etc/mikrotik-fwban.cfg and edit to your liking.
* Copy mikrotik-fwban.service to /etc/systemd/system
* Execute `systemctl daemon-reload`.
* Execute `systemctl enable mikrotik-fwban` to enable the daemon at startup.
* Execute `systemctl start mikrotik-fwban` to start the daemon right now.
* Check your /var/log/messages for possible errors and fix them.
* (If you want to receive off machine, don't forget to open your firewall
  on the configured port.)

### Sending syslog information its way.

* Add a snippet to /etc/rsyslog.d to (re)send interesting messages to the
  mikrotik port, best thing is to filter on error conditions containing an
  IP you want to block. Example below:
  ```
  if re_match($msg, "failed for '[0-9a-f:.]*' - Wrong password") then
	action(type="omfwd" target="<mikrotik-fwban-ip>" port="<mikrotik-fwban-port>" template="RSYSLOG_SyslogProtocol23Format")
  ```
* You can do this on every Unix system in your network if you feel so
  inclined. Again, don't forget to open the firewall on the Mikroik-FW's
  host if you do.

## Credits

Mikrotik-fwban uses
[go-gcfg](https://github.com/go-gcfg/gcfg/tree/v1),
[syslogparser](github.com/jeromer/syslogparser),
[routeros-api-go](https://github.com/Netwurx/routeros-api-go)