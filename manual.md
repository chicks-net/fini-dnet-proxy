distributed.net Personal Proxy Manual {align="center"}
=====================================

 

This document contains the necessary instructions to set up your
[distributed.net](http://www.distributed.net/) personal proxy on both
UNIX and Windows computers. If you have any additional questions, please
refer to the [FAQ-o-matic](http://faq.distributed.net/) or e-mail
[help@distributed.net](mailto:help@distributed.net).

 

[Introduction](#introduction)

[Proxy build numbers and client versions supported](#proxyver)

[Proxy buffer files](#proxybuffer)

[Installation and operation of the Personal Proxy](#install)

[Configuration](#configure)

[Buffer Sizes](#config-buffersizes)

[Settings](#config-settings)

-   [upstream server](#config-settings-keyserver)
-   [listing ports](#config-settings-ports)
-   [miscellaneous](#config-settings-misc)
-   [console logging](#config-settings-console)
-   [rc5-72 project](#config-settings-rc5-72)
-   [Other Projects](#config-settings-other-projects)
-   [Access Control](#config-settings-access-control)

[Generic proxy mode](#config-generic-proxy-mode)

[Consolelog and keyblocklog output](#output)

[Operations and signals](#opsandsigs)

[Running on windows 95/98/NT](#runonwin)

[Command line options](#clioptions)

[Running on Solaris](#runonsolaris)

[CPU/OS numbers](#cpuosnumbers)

[Additional support](#additionalsupport)

### INTRODUCTION {align="left"}

This release of the v2 Personal Proxy will allow you to serve all recent
v2 distributed.net Bovine clients. It allows you to establish one of
your own machines as a buffer between your clients and one of the "full
servers" (keyservers) that are officially run by the distributed.net
organizers. This will allow your clients to waste less time trying to
connect to one of the main proxies, since the personal proxy will
already have more key blocks waiting for your clients when they're ready
and need more. Running a personal proxy is definitely not needed by
everyone, nor is it recommended. You should only run a personal proxy if
you are very confident of your abilities, and you have a very weak,
unreliable, or intermittent connection to the Internet to directly
contact one of the real proxies.

 

### PROXY BUILD NUMBERS AND CLIENT VERSIONS SUPPORTED {align="left"}

Personal proxies are distinguished by the "build number" that is
associated with each new version.

-   OGR-26 and up (OGR-NG) requires proxy build 344 or greater.
-   RC5-72 requires proxy build 330 or greater. Large-size RC5-72 blocks
    requires proxy build 346 or greater.
-   Proxy build 335 or higher will only communicate with clients
    v2.8015.469 or higher.
-   Proxy build 341 or higher will only communicate to upstream servers
    of build 341 or higher.

 

### PROXY BUFFER FILES {align="left"}

The personal proxy uses buffer files to store all blocks that are
pending distribution to client, or awaiting transfer to an upstream
server. Every effort has been made to keep the format of these buffers
consistent across proxy versions, platforms, and operating systems.
There are currently 3 main types of proxy buffers that you need to keep
in mind before you considering transporting buffers:

-   ~~Format A: up to and inclusive of proxy build 283.~~(obsolete)
-   ~~Format B: proxy build 300 through 304, inclusive.~~(obsolete)
-   Format C: proxy build 305 to current, inclusive. (current)

Format A is completely obsolete. Format B used an experimental
filesystem-like buffering system, but was abandoned due to stability
issues. Format C uses a less resource demanding buffering format more
appropriate for personal proxy use.

Buffers from any operating system or cpu architecture from a proxy that
creates the same "format" above should be transportable. Note that
"transportable" does not mean that you can "share" buffers between two
*simultaneously* running copies of the proxy!!

If you are upgrading to a new proxy version that uses a file format that
is incompatible with the last version you were using, there is no way to
convert them. You must flush all blocks from the existing proxy buffers.
To do this, set maxkeysdone=1, then start the proxy. It should flush all
blocks to the keyservers. If you don't want clients to connect to your
proxy during this process, set the "acceptincoming"-flag in the [ports]
section to 0 (zero).

After you see a logentry with "rc5-72 r=x/x, d=0/1" you know your
buffers are empty. Shutdown the proxy and delete all bufferfiles
(ppdes\*.des and pprc5\*.rc5) It is safe to simply delete the inbuff
files, even if they have keys left in them. They will eventually be
reissued.

If you have blocks within an "out" buffer, and cannot flush them, please
rename the file to something else or delete it.

 

### INSTALLATION AND OPERATION OF THE PERSONAL PROXY {align="left"}

Since you have this file, you probably have already downloaded and
unpacked the proper distribution for your machine. If not, see
[ftp://ftp.distributed.net/pub/dcti/proxyper/](ftp://ftp.distributed.net/pub/dcti/proxyper/)
and download the proxy package for your machine. Once the proxy is
unpacked, you will need to configure it. You may rename the binary to
anything you like, but there must be a corresponding config file with
".ini" as a suffix. Then edit the .ini file with your favorite text
editor (configuration options are listed in the next section), and then
start the personal proxy in the directory you have installed it in. The
log and dump files will be created in this directory. Hint: If you
specify a relative path for a logfile entry in the .ini, the logs will
be created in a directory relative to the current directory.

 

### CONFIGURATION {align="left"}

Edit the ini file with a standard text editor and verify the settings.
Depending upon the number of clients you are serving, you may want to
increase or decrease the number of keyblocks that are kept in memory.
Note that the proxy uses a sliding average that prevents you from
buffering more than 4 days worth of blocks. These numbers will be
adjusted according to your keyrate and should be reasonable stable after
30 minutes after you started the proxy.

Note that the ini file is only read once (as the proxy starts up).
Modifications made to the ini file will not take effect until you
terminate and restart the proxy. On most platforms, there is a method to
trigger the ini file to be reloaded at will. See the "operations and
signals" section to determine the method that is applicable for your
operating system.

None of the settings in the ini file are actually required to be
specified. All of the settings have default values that should work for
most people. However, it is still recommended that you verify and
explicitly set all of the options.

Although we try to keep the format of the ini file consistent between
different builds of the proxy, there are occasionally changes made
between builds that affect the options that are available. Build 300 of
the proxy introduced a fair number of significant differences in the ini
file, so be aware of these differences and compare your current ini to
the one that comes with the new proxy-tarball.

 

### CONFIGURATION: Buffer sizes {align="left"}

The personal proxy requires you to specify the minimum and maximum
number of "blocks" that it should try to keep ready at all times for
clients. However, when you specify the numbers for these two values,
please be certain that you only enter values that are reasonable for the
number of clients that you are planning to support.

Specifying numbers that are too large may cause your proxy to buffer a
more blocks than your installed client-base can use in a realistic
amount of time. It is important to avoid fetching too many extra blocks
to reduce the likelihood of a large number of blocks getting wasted if
your proxy crashes, or you decide to discontinue running a proxy.
Additionally by buffering a large number of blocks, you increase the
number of outstanding blocks that must be tracked and managed by the key
distribution system, which limits many aspects of distributed.net's
performance. On the other hand, specifying too few blocks will make it
more likely that your clients will run out of legitimate blocks should
short-term network outages occur, forcing them to compute randomly
generated blocks, which would have a greater possibility of being
duplicated by other machines.

In general, it is recommended that you buffer no more than about 3 or 4
days worth of blocks, so that you could potentially withstand a network
outage that long in the worst case. There is typically no reason to want
to buffer any more blocks than that. The appropriate number is of course
dependent upon the number and speed of clients that you have connecting
to your personal proxy, so you will have to determine this value
experimentally.

Builds prior to 305 of the personal proxy placed somewhat restrictive
maximum ready block limit of 10000, for the purpose of preventing people
from accidentally specifying an extremely large number of blocks before
realizing that it was too great to support their number of clients.
Unfortunately, this also limited the effectiveness for people who were
legitimately using the personal proxy to support an extremely large
number of clients.

Starting with build 305, the maximum ready buffer limit has been
increased to 200000 blocks, however additional logic has been added to
prevent accidentally fetching an inappropriate number of blocks:

1.  If you specify a buffer limit of 1000 or fewer, the proxy will
    respect this value, just like previous versions.
2.  For values larger than 1000, the proxy will internally treat the
    maximum ready value as if 1000 had been entered, until the proxy has
    been running for a period 30 minutes total.
3.  Beginning after 30 minutes of uptime, the proxy will periodically
    evaluate your average keyrate and allow maxready values as large as
    that sustained rate for 4 days to be specified. If the value entered
    within the ini file is less than the computed sustained usage count,
    then the value you specified will be obeyed.

Additionally, when your maxready value is being suppressed by the proxy,
it will simultaneously adjust the minready in an equivalent proportion
so that the ratio that was originally specified is maintained. As you
can see, the complexity has increased slightly, but the flexibility has
also been improved at the same time. You should keep this behavior in
mind when trying to configure your ready buffer thresholds.

This automatic behavior is recommended for almost all people. The proxy
will adjust its buffer so there's always enough blocks for a couple of
days. People who use the proxies buffers to sneakernet need be to able
to download large numbers of blocks. For this purpose a ini-setting
[rc5-72]/expertmode and [ogr-ng]/expertmode is added. If this option is
set to 1 (one), the proxy will obey your settings of maxkeysready and
minkeysready.

 

### CONFIGURATION: Settings {align="left"}

### The [KeyServer] Setting {align="center"}

**[KeyServer]/ipaddress:** the IP address or hostname of the keyserver
from which the proxy will retrieve and send keyblocks from.

**[KeyServer]/port:** the port on the keyserver to communicate to.

**[KeyServer]/connectperiod:** determines the minimum delay (in seconds)
between sequential attempted network connections to the keyserver. This
value should typically \*not\* be adjusted to more than a couple of
minutes, but it should also not be made too short, or else your proxy
might attempt to make a connection as frequently as this amount, which
may prevent it from handling incoming client connections if your proxy
is unable to successfully make outgoing server connections.

**[KeyServer]/connectivity:** these modes are very much like the normal,
offline, lurk, and lurkonly modes that are available on the windows
clients. This option only has effect on the win32 version of the proxy.

-   "normal" : The proxy will attempt to connect to a server as needed.
-   "offline" : The proxy will not attempt to make any connections to a
    server automatically.
-   "lurk" : The proxy will automatically send/receive blocks when it
    detects a network connection, but it may also initiate connection by
    itself if it needs to send or receive blocks. If you have autodial
    configured, then your modem will auto-dial.
-   "lurkonly" : Same as lurk, however the proxy will not attempt to
    make connections unless it can explicitly detect that your modem is
    currently connected, so it won't ever cause auto-dial.

If you are behind a firewall and must use a proxy to make a connection
on either a telnet or http port, the following options will be useful.

* **[KeyServer]/uuehttpmode:** `0=no special encoding, 1=uue encoding (non
8-bit clean redirect proxies), 2=http encoding, 3=uue+http encoding,
4=socks4 encoding, 5=socks5 encoding, 6=generic proxy, 7=generic proxy
with uue encoding.` For an explanation on generic proxy support, [see
below.](#config-generic-proxy-mode)

* **[KeyServer]/httpproxy:** The name of your site's http or telnet proxy.

* **[KeyServer]/httpport:** The port to connect to the above proxy on.

* **[KeyServer]/httpid:** User and password authentication needed to
connect to the above proxy. This field must be set to the \*encrypted\*
value of the username and password information. In order to get this
value, use the client's configuration option to specify the username and
password, then cut and paste that field from the client .ini to your
proxy .ini. Remember to restore the client configuration when you are
all done with this. For socks4 httpid is the "username" in plaintext.
For socks5 httpid is the "username:password" in plaintext. For generic
proxy, this is the connection negotiation string.

* **[KeyServer]/bindip:** Specifies the local interface address that
should be bound to when creating outgoing server connections. This might
be useful if your machine has multiple interfaces and you wish to listen
for incoming clients on an internal network, and make outgoing internet
connections on a public network interface. If you leave this option
unspecified, it allows the network stack to do default interface and
routing rules to create connections.

* **[KeyServer]/maxservers:** Determines the maximum number of
simultaneous upstream server connections will be used to fetch or flush.
It might be useful to increase this number if you frequently need to
upload or download a large number of blocks at a single time, and it
seems that individual server connections are slow or have high latency.
The default value is 1 and the maximum number you can specify is 4.

### The [ports] Settings {align="center"}

**[ports]/listenaddress:** The IP address of the machine this personal
proxy is running on. This is optional and only needed if the machine has
multiple IP addresses. If you use one ip address for local connections,
and you have a dialup account that you use to fetch and flush blocks,
you do \*not\* need to set this.

**[ports]/port:** the port on \*your\* machine to listen for incoming
clients on.

**[ports]/port2, port3, port4:** these allow you to specify additional
listening ports on \*your\* machine to listen simultaneously for
incoming clients on. Typically these additional ports aren't needed, but
this ability is here if you need it.

**[ports]/testport:** the port on \*your\* machine to listen for
incoming clients on and distributed test blocks to. This value can be
unspecified, or set to 0 to disable this functionality.

**[ports]/acceptincoming:** if this is set to 0, clients will not be
able to connect to the proxy at all. Defaults to 1.

**[ports]/advertisedaddress:** This value is normally not needed and is
only needed if you are having problems serving HTTP clients and your
personal proxy machine has a different address by which it is reached on
than what it listens on. (This is usually the case when you are
attempting to serve clients with a machine behind a NAT firewall, or a
port-redirected environment.) This value should be the public IP address
that is usable by clients to connect back to your personal proxy with.
If you do not specify this value, the value you specified for the
listenaddress is used, otherwise no address is used.

**[ports]/timeout:** This value can be used to set maximum idle timeout
for client connections. Default is 30 seconds. Change of this value
normally is not required. You can try to increase timeout if default
value is not enough for your client (e.g. it having a long
time-consuming benchmark on unknown cpu.)

### The [misc] Settings {align="center"}

**[misc]/statusperiod:** this is the number of seconds between the
status reports from each of the current contests are printed out to the
console log. The contest status reports indicate the number of current
ready and done blocks, their set values, and also the sliding window
average keyrate, and total number of completed blocks since startup.

**[misc]/periodicperiod:** this is the number of seconds between the
rewriting of the proxy buffers to disk. If you have a machine with a
very slow disk or has a bus with high I/O contention, you may want to
increase this value to trigger writes only every few minutes. Note that
when you allow the proxy to shutdown gracefully, it will automatically
rewrite it buffers out again if necessary (regardless of when the last
periodic write occurred).

**[misc]/logfilecompressor:** the name of a script or program to run
once per log file rotation with one parameter, the name of the just
completed log file. This function is enabled since build 305.

**[misc]/proxymessage:** an optional text message displayed to all
connecting clients. If you leave this value blank or unspecified, a
default message including the build number will be displayed to clients.

**[misc]/pidfile:** if specified, this is the filename that will have
the process id (pid) of the proxy when it is started up. This is only
for Unix perproxies.

**[misc]/scheduledupdate:** this is an internal value is is
automatically updated and maintained by the proxy itself each time it
connects to the keyserver. You should not modify or adjust this value.

### The [console] Settings {align="center"}

**[console]/logfileconsole**: name of console log file

**[console]/logfileconsolerotation:** interval to rotate console log at,
options are none, hourly, daily, monthly, yearly, or startup

**[console]/consoleverbosity:** control verbosity of the console log
output. It is specified with a list of space-separated keywords:

-   all : display all message types (default)
-   none : no console output at all
-   general : General logging (Status: Slot X LISTENING, and others)
-   stats : Periodic statistics reports
-   keyblock : Block numbers
-   server : Upstream keyserver communications
-   client : Downstream client communications
-   buffers : statistics (ready=X/X, done=X, Xd XX:XX:XX, X.X
    Mkeys/sec)
-   timestamp : Include UTC timestamps on each line (screen only)
-   attention : Keyspace change, time change, dialup events
-   errwarn : Unrecognized opcodes/version
-   errlow : Invalid settings
-   errsevere : Operation inhibiting problems

If the very first keyword is "not", the verbosity will be "all", minus
the keywords specified. For example, "not keyblock client" will prevent
messages in the keyblock or in the client categories from displayed; all
other messages are displayed.

**[console]/timestampflags:** controls the format that is used to
display timestamps on screen, logged in the console log, and logged in
keyblock logs. Note that timestamps in the onscreen console log can be
hidden entirely with the "consoleverbosity" option above. The integer
that is specified for this value is actually the result of adding one of
the format mode values, plus optionally one or more of the additional
flags. The format mode values available are:

 

 

old-style MM/DD/YY HH:MM:SS

1

 

new-style YYYY-MM-DD HH:MM:SS

2

 

client-style Mmm DD HH:MM:SS

3

 

And the multiple flags that you can choose are:

 

show timezone name

64

 

timestamps should be in UTC

128

 

 

Note that the "show timezone name (64)" only has an effect if timestamps
are being "displayed in UTC format (128)". When that is the case, the
letters "UTC" will be appended to all timestamps that are displayed
on-screen (not in console log files, or keyblock files).

The default format is 193, and will provide output that is identical to
that used by personal proxies before build 313.

Also notice that the timestampflags will not affect the numbering
sequence used to generate filenames for automatically rotated logfiles.
Rotated log filenames are always generated from the current UTC time.

Be aware that changing the timestamp format may affect your ability to
use log parsing or log processing utilities. You should contact the
authors of your utilities to obtain a version that is able to parse the
new 4-digit year format, and accommodate localtime timestamps.

### RC5-72 Related Settings. {align="center"}

**[rc5-72]/logfilekeyblock:** name of log file of all completed
keyblocks

**[rc5-72]/logfilekeyblockrotation:** interval to rotate keyblock log
at, options are none, hourly, daily, monthly, yearly, or startup

**[rc5-72]/maxkeysready:** approximate maximum number of keys to keep
ready for serving clients.

**[rc5-72]/minkeysready:** minimum number of keys to keep ready for
serving clients. This differs from the maxkeysready in that once the
available keys drops below this value, the proxy will "try harder" to
fetch additional keys from the server.

**[rc5-72]/maxbufferdays:** the personal proxy by default will buffer as
much blocks needed for 4 days. If you want to change this default
behavior, this option specifies the number of days the proxy should
buffer blocks for.

**[rc5-72]/minkeysdone:** number of completed key blocks that will
always be kept resident and not transmitted. There is probably no reason
for this to be anything other than 1.

**[rc5-72]/maxkeysdone:** number of completed key blocks that must be
done before an outgoing connection to the keyserver is made to flush the
completed blocks.

**[rc5-72]/timeaverage:** number of chunks to use for the
sliding-average window. The default is 288 (1 week in 5 min chunks), the
valid range is 6-2016 (5 hour - 1 weeks).

**[rc5-72]/timeavgchunk:** number of seconds to use for a chuck in the
sliding-average window. The default is 300 (5 minutes). Valid range is
between 1 minute and 30 minutes. This option is added for people who
desperately need a window bigger than 1 week. Be aware that changing the
defaults of "timeaverage" and "timeavgchunks" is not recommended. Too
high values can lead to an increase in resources needed to track the
stats.

**[rc5-72]/contestclosed:** participation in the contest can be entirely
disabled by setting a negative value

**[rc5-72]/blocksize:** the preferred size of blocks when requesting
additional workunits from the upstream server. The default is 256. If
clients need smaller block sizes, then your proxy will automatically
split large blocks into the needed size, however clients will not be
able to request blocks larger than the size your proxy currently has
available, since the proxy cannot combine smaller blocks together. Be
aware that when your proxy requests additional workunits from the
upstream server, this blocksize value may cause your proxy to
temporarily exceed the "maxkeysready" since requests will always be done
in multiples of this value.

 

### Other Projects {align="center"}

****[ogrng]/[ogrp2]/[desII]/[csc]/[ogr]/[rc564]****
 see the [rc5-72] section for the entries that you can put here, and
what they do....

 

### Access Control {align="center"}

**[ignoredip]/[allowedip]**
 These two sections provide functionality for allowing and denying
certain ipaddresses or ipaddress ranges.
 The rules are:
 a. if there are entries in [allowedip], only allow clients in that
section, else
 b. if there are entries in [ignoredip], deny all clients in that
section, else
 c. allow the connection

This is equivalent with the behavior of the tcpd wrapper in Unix
systems.

After the section header, ip(-ranges) are listed in the form:

  --- -----------------------------
      `anything=1.2.3.4,maskbits`
  --- -----------------------------

one line for each block of IP addresses you want to ignore. To ignore a
class C subnet (255 addresses, x.x.x.\*, /24), maskbits will be 24. For
an entire B network (65536 addresses, x.x.\*.\*, /16) you will want to
use a maskbits of 16. Note that this format is different from the format
that was used in build 280 and prior of the personal proxy.

Additionally, you can enter masks using any of these forms:

  --- ---------------------------------------------------------------------------
      ` anything=1.2.3.*           anything=1.2.*.*           anything=1.*.*.*`
  --- ---------------------------------------------------------------------------

 

### CONFIGURATION: Generic proxy mode {align="left"}

Build 306 of the personal proxy introduced a new method of firewall
compatibility communications, known as "generic proxy" (a name also used
by the programs CRT and SecureCRT). This new method of communication
allows you to establish redirected TCP/IP connections through an
intermediate machine/proxy that requires a login or negotiation
sequence. This includes connecting to a gateway machine to log in and
then telnetting to the destination keyserver, and also connecting
through a WinGate "telnet proxy" that requires you to specify the
destination hostname and port.

In the generic proxy modes, the interpretation of the "httpid" value is
changed to represent the login sequence string, which is merely a
vertical-bar ("|") delimited string of alternating things to transmit,
and things to wait for. Note that the first sequence in the string is
assumed to be a send string, so if you don't need to send anything first
just include a vertical-bar as the first thing in the httpid. The tags
\\n, \\t, \\r, \\1 (keyserver address), and \\2 (keyserver port) can
only be used in send strings. Be aware that the maximum length of the
entire httpid string is very limited, so you cannot include sequence
strings that are too complicated currently. Also understand that since
the sequence string is of course stored in plaintext in your ini file,
if you include connection passwords, they could be compromised if you do
not protect your ini file.

One example for connecting to a gateway machine is:

  --- ----------------------------------------------------------
      `"|login:|bovine\n|Password:|moocow\n|%|telnet \1 \2\n"`
  --- ----------------------------------------------------------

An example for a WinGate telnet proxy is:

  --- -------------------------
      `"|WinGate>|\1 \2\r\n"`
  --- -------------------------

 

### OUTPUT: Console log {align="left"}

While the proxy is running, it displays countless numbers of messages to
indicate its current status. Because of the potentially large volume of
messages that can be generated, a console verbosity option has been
added to the proxy ini files to allow you to select only specific
classifications of messages to be displayed.

There are also methods for causing the console output to be saved to a
file, either for remote log viewing purposes or record keeping. Although
there exist a number of third-party utilities that permit the user to
analyze console logs to produce statistics regarding your proxy's
operations, though console logs were not really intended for any type of
mechanical analysis. The format and style of the console messages are
purely intended to be human readable and will tend to be modified
slightly between proxy builds.

Some messages explained:

-   `contest <contest> r=<a>/<b>, d=<c>/<d>, <avg>Mkeys/s,   total=<blocks>`

    a/b = a blocks currently in inbuffer, b is threshold.
     c/d = c blocks done, d is threshold

-   `Server xx.xx.xx.xx changed state from x to x`

    state 0 is a no connection
     state 1 is waiting for connection to be set up
     state 50 is a connected connection
     state 51 is a dead connection

-   `Contesthandler <contest> didn't handle a packet `

    For some reason the proxy couldn't process a clientrequest for a
    packet (block). Most of the time due to empty proxybuffers

-   `Client: rejecting unscrambled communication`

    Client tries to send a packet that doesn't comply. Most of the time
    due to too old clients

 

### OUTPUT: Keyblock log {align="left"}

The personal proxy will also output so-called "key logs" which record
details of each completed block that is reported to it by a client.
These files are intended for automated processing and so they have a
very regular, comma delimited formatting. Each line of a key log has the
following fields, in order:

1.  date/timestamp of format, such as "mm/dd/yy hh:mm:ss". The actual
    format can be changed via the [console]/timestampflags option
2.  IP address of the reporting client
3.  email address of the client
4.  unique identifier representing the completed packet. For RC5-72,
    this is a hexadecimal string with embedded colons in the format
    "xx:xxxxxxxx:xxxxxxxx". For RC5-64/DES/CSC, this is is a hexadecimal
    string (14 to 16 hexadecimal digits). For OGR, this is a
    variable-length string resembling the format "dd/dd-dd-dd-dd-dd"
5.  size of the packet. For OGR, this is the number of nodes within the
    stub. For RC5-72, this is the number of number of "blocks" in the
    packet (where each block is 2\^32 keys). For RC5-64/DES/CSC this was
    also the number of "blocks" in the packet, but each block was 2\^28
    keys.
6.  operating system identifier of client (0=unknown).
7.  cpu type of client (0=unknown).
8.  build/version number of client. For clients released before rc5-72,
    this is only the middle segment of version number. For rc5-72
    enabled clients, this contains the full fractional version number
    and build of the client.
9.  project cruncher core number.

`01/15/99 07:39:16,0.0.0.0,rc5@distributed.net,6404AF4970000000,10,0,0,6400`

See bottom of this document for a list of [OS and CPU types and
numbers](#cpuosnumbers).

 

### OPERATIONS AND SIGNALS

On UNIX, sending a TERM or INT signal will cause the personal proxy to
shut down nicely, re-saving all keyblocks that are still in memory to
disk, and closing all open network connections.

On UNIX, sending a HUP signal will cause the personal proxy to reload
its ini configuration file and reread most of the settings specified
within it.

On UNIX, sending an ALRM signal will trigger the personal proxy to
create an outgoing server connection. This is done even if your
connectivity mode is set to offline. This behavior is intentional, and
permits clever scripters to automate periodic dial-ups and forced server
flushes.

On Win95/98/NT, when running the proxy in a window you can press
Ctrl-Break at any time to perform the equivalent of a HUP+ALRM signal on
UNIX. To actually close the proxy, press Ctrl-C or close window.

On OS/2 you can press Ctrl-C to force a reload of the ini file and
create an outgoing server connection. Ctrl-Break is used to close the
proxy.

 

### RUNNING ON WINDOWS 95/98/ME/NT/2000/XP

The personal proxy can run on successfully on a Windows 95/98/ME
workstation, however it is discouraged that this be done on a machine
that is being heavily used as a normal user workstation, due to the
frequent crashes and relative instability of Win95/98/ME when it is
actively being used. If you really need to run a proxy on a Windows
machine, it is recommended that you use an NT machine, if possible.

On Windows NT, the proxy can be run either as a foreground console
application or installed as a service and run transparently in the
background, unaffected by user logins. (See the "command line options"
section for instruction on service installation and removal).

Under Windows 95/98/ME, the proxy can only be run as a foreground
console application. There are no plans to support running the proxy as
a "hidden" or "service" application in this environment.

Please be aware that if you are running the proxy as a foreground
console (and not as an NT Service), you should avoid simply closing the
console window directly. Instead, when you want to shut down the proxy,
you need to focus the window and press Ctrl-C to initiate a shutdown.
Failure to do this will cause the proxy to not have an opportunity to
shutdown nicely, possibly corrupting your buffers or causing some blocks
to be lost.

Notice that the personal proxy requires Winsock 2 services to be
available. Most original distributions of Windows 95 (but not Win98 or
WinNT4) come with only Winsock 1.1, so you must install the update from
Microsoft. You can download [this
update](http://www.microsoft.com/windows95/downloads/contents/wuadmintools/s_wunetworkingtools/w95sockets2/default.asp)
from Microsoft's website.

 

### COMMAND LINE OPTIONS

The personal proxy allows several command line options that tell it to
enter one of several exclusive and session-lasting modes of operation.

On Windows NT only, you can install the proxy as a service by first
running it with the "-install" option to register it as a service. You
can then start and stop the proxy from the standard Windows NT services
Control Panel. You can use the "-uninstall" option to remove it from the
services registry.

Previously a detached mode option existed in the ini file to allow the
proxy to be started as a daemon-like service on your UNIX machine. This
has now been made available only as a command-line option, and must be
invoked by starting the proxy with "-detach" on the command line.

 

### RUNNING ON SOLARIS

The personal proxy requires Solaris 2.6 or greater to run.

Solaris 2.6 requires Sun Patch ID \#105591 which is available from
[http://sunsolve.sun.com/](http://sunsolve.sun.com/) in order to run the
personal proxy. Other Solaris versions may require a similar patch. If
you encounter the error "ld.so.1: proxyper: fatal: libCrun.so.1: open
failed: No such file or directory", you must install the patch.

 

### CPU/OS NUMBERS

Here is the list of CPU and OS identifiers previously mentioned in the
keyblock log section above.

   **CPU types:                  OS types:**
   UNKNOWN             0         UNKNOWN             0
   X86                 1         WIN32               1
   POWERPC             2         DOS                 2
   MIPS                3         FREEBSD             3
   ALPHA               4         LINUX               4
   PA_RISC             5         BEOS                5
   68K                 6         MACOS               6
   SPARC               7         IRIX                7
   SH4                 8         VMS                 8
   POWER               9         DEC_UNIX            9
   VAX                 10        UNIXWARE            10
   ARM                 11        OS2                 11
   88K                 12        HPUX                12
   IA64                13        NETBSD              13
   S390                14        SUNOS               14
   UNUSED              15        SOLARIS             15
   DESCRACKER          16        UNUSED              16
   AMD64               17        UNUSED              17
   CELLBE              18        BSDOS               18
   CUDA                19        NEXTSTEP            19
   AMD_STREAM          20        SCO                 20
                                 QNX                 21
                                 UNUSED              22
                                 UNUSED              23
                                 UNUSED              24
                                 AIX                 25
                                 UNUSED              26
                                 OSX                 27
                                 AMIGAOS             28
                                 OPENBSD             29
                                 NETWARE             30
                                 MVS                 31
                                 ULTRIX              32
                                 UNUSED              33
                                 RISCOS              34
                                 DGUX                35
                                 WIN32S              36
                                 SINIX               37
                                 DYNIX               38
                                 OS390               39
                                 UNUSED              40
                                 WIN16               41
                                 DESCRACKER          42
                                 MAC OSX             43
                                 PS2 LINUX           44
                                 MORPHOS             45
                                 WIN64               46
                                 NETWARE6            47
                                 DRAGONFLY           48
                                 HAIKU               49



### Additional Support {align="left"}

If you need further assistance with the distributed.net personal proxy,
please take a look at [FAQ-o-matic](http://faq.distributed.net/) then if
you still need assistance, e-mail
[help@distributed.net](mailto:help@distributed.net)

 
