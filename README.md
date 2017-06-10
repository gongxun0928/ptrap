ptrap
=====

Script that lets you find out which process on your system is sending packets to a single `<ip>:<port>`. It supports both TCP/UDP packet monitoring, and the execution of a custom program in response. Depends on `tc`, `ip`, `iptables` and `lsof`.


Examples
--------

    # Monitor DNS requests to host 192.168.1.1 and print out the process tree,
    # highlighting the originating process and his ancestors
    ptrap -u -i 192.168.1.1 -p 53 -e "pstree -p -H"

    # Monitor new connections TCP connections to any port of 192.168.1.1, and
    # print out the originating process' name
    ptrap -t -c -i 192.168.1.1 -e "ps -o comm="


How it works
------------

*TODO*

### Important considerations

* Your current `tc` configuration will be overwritten, at program start.
* A chatty application may exchange many packets per seconds with a single
  destination. To prevent flooding this script with packets that belong to the
  same connection (and, consequently to the same process), you can use the `-n`
  flag to limit the max number of matches per minute for a single connection;
  however, if an application  generates a lot of traffic by opening _new_
  connections (thus using many different source ports in rapid succession)
  flooding could still happen.
* If too many packets are captured in a short time (flooding), this script might
  not read them fast enough, and for some of them the sending process could die
  before the system could be queried for informations about the sending process.
  If this happens, you might want to consider tuning one of these parameters, to
  lower the chances of that happening again. In order of effectiveness:
    1. Increase the `delay_ms` parameter
    2. If using one, optimize your user script execution time (their execution
       is synchronous)
    3. Lower the `TAIL_INTERVAL_S` variable
  Be aware, however, that this program relies on the process being still alive
  and waiting for response, after sending an IP  packet. If a program sends a
  packet and then immediately dies, without even waiting for response, this
  program might not able to gather informations about it.
* As it relies on iptables rules and tc qudisc to capture traffic, root
  privileges are required.
* It expects syslog to write its logs to `/var/log/messages`.
* Currently, only one instance of the program can be executed at once.


Future work
-----------

- Drop explicit dependency on `/var/log/message`, implement parsing syslog
  messages from other sources
- Enhanced match limitation, based on _processes_ instead of _connections_.
- Asynchronous user script
- Safer deletion of iptables rules, instead of relying on rule numbers
- Monitor other IP protocols (e.g. icmp)
- Check for existence of other tc rules at launch time, save&restore them
- Concurrent execution of multiple istances of this program
