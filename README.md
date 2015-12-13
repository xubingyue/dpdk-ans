####TCP/IP stack for dpdk
--------------
Netdp is porting from [FreeBSD](http://freebsd.org) TCP/IP stack, and provide a userspace TCP/IP stack for use with the Intel [dpdk](http://dpdk.org/). 

- librte_netdp: TCP/IP stack static library. netdp use dpdk mbuf, ring, memzone, mempool, timer, spinlock. so zero copy mbuf between dpdk and netdp. 

- librte_netdpsock: Netdp socket lib for application, zero copy between netdp and application.

- netdp_cmd: Command for configure netdp tcp/ip stack.
 
- netdp_test: Example application with netdp for testing netdp tcp/ip stack

Support environment
  - EAL is based on dpdk-2.1.0
  - Development enviroment is based on x86_64-native-linuxapp-gcc
  - TCP/IP stack is based on FreeBSD 10.0-RELEASE
  - linux version：
3.16.0-30-generic
  - gcc version：
gcc version 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04)

Support feature:
 - Netdp initialize
 - Ether, zero copy between NIC and netdp TCP/IP stack.
 - ARP, ARP timeout
 - IP layer, IP fragmentation and reassemble
 - Routing
 - ICMP
 - Commands for adding, deleting, showing IP address
 - Commands for adding, deleting, showing static route
 - Commands for showing ARP table
 - UDP protocol
 - Socket layer, share memory.
 - Socket API, socket/close/send/recv/epoll.
 - TCP protocol, no lock, hash table, per tcp stack per lcore.(currently only run in one lcore, will enhance it)
 - 
Next Planning
- Enhance socket API
- Support multi core for TCP
- Performance testing.


####Performance Testing
--------------
- TCP server performance testing
 
 ENV: CPU- intel xeon 2.3G, NIC- 10G, Test tool:ab 

 Procedure: ab establish tcp connection to netdp tcp server, ab download one data, netdp tcp server close socket.

 Command: ab -n 1000000 -c 500  2.2.2.2:8089/
 
 Notes: shall increase test linux PC local port range (net.ipv4.ip_local_port_range = 1024 65000).
```
    |--------------------------------------| 
    |      TCP Server accept performance   |
    |--------------------------------------| 
    | Linux with epoll | NETDP with epoll  | 
    |    (Multi core)  |    (one core)     |
    |--------------------------------------|
    | 53k connection/s | 43k connection/s  | 
    |--------------------------------------| 
```
- TCP socket data transmission performance
 
One socket receive 190Mbyte tcp payload, one socket send 130Mbyte tcp payload
```
Communication(synchronization)  0 runtime:	 0.734931 s
Communication(synchronization)  1 runtime:	 0.469566 s
Communication(synchronization)  2 runtime:	 0.449729 s
Communication(synchronization)  3 runtime:	 0.648432 s
Communication(synchronization)  4 runtime:	 0.449422 s
Communication(synchronization)  5 runtime:	 0.647259 s
Communication(synchronization)  6 runtime:	 0.457027 s
Communication(synchronization)  7 runtime:	 0.457691 s
Communication(synchronization)  8 runtime:	 0.67568 s
Communication(synchronization)  9 runtime:	 0.736285 s
```

- L3 forwarding performance testing

  ENV: CPU- intel xeon 2.3G, NIC- 10G, one lcore rx packets-->l3 forwarding --> tx packets,  Test tool:pktgen-DPDK
```
    |--------------------------------------| 
    |      L3 forwarding performance       |
    |             (one lcore)              |
    |--------------------------------------| 
    | Packet size(byte)| Throughput(Mpps)  | 
    |--------------------------------------|
    |     64           |      3.682        | 
    |--------------------------------------| 
    |     128          |      3.682        | 
    |--------------------------------------| 
    |     256          |      3.683        | 
    |--------------------------------------| 
    |     512          |      2.35         |
    |--------------------------------------| 
    |     1024         |      1.197        | 
    |--------------------------------------| 
    |     1500         |      0.822        | 
    |--------------------------------------| 
```
- dpdk-redis performance testing
```
====ENV=== 
CPU:Intel(R) Xeon(R) CPU E5-2430 0 @ 2.20GHz.
NIC:Intel Corporation 82576 Gigabit Network Connection (rev 01) 
OPENDP run on a lcore.

root@h163:~/dpdk-redis# ./src/redis-benchmark -h 2.2.2.2  -p 6379 -n 100000 -c 50 -q
PING_INLINE: 76687.12 requests per second
PING_BULK: 77459.34 requests per second
SET: 74183.98 requests per second
GET: 75815.01 requests per second
INCR: 76687.12 requests per second
LPUSH: 74794.31 requests per second
LPOP: 74349.44 requests per second
SADD: 75757.57 requests per second
SPOP: 76569.68 requests per second
LPUSH (needed to benchmark LRANGE): 75075.07 requests per second
LRANGE_100 (first 100 elements): 44385.27 requests per second
LRANGE_300 (first 300 elements): 19267.82 requests per second
LRANGE_500 (first 450 elements): 13757.05 requests per second
LRANGE_600 (first 600 elements): 9047.32 requests per second
MSET (10 keys): 61538.46 requests per second

```
- http server connection performance
```
CPU:Intel(R) Xeon(R) CPU E5-2430 0 @ 2.20GHz.
NIC:Intel Corporation 82576 Gigabit Network Connection (rev 01) 
OPENDP run on a lcore.
examples/http_server run as server.

root@h163:~# ab -n 40000 -c 500  2.2.2.2:8089/
This is ApacheBench, Version 2.3 <$Revision: 1528965 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 2.2.2.2 (be patient)
Completed 4000 requests
Completed 8000 requests
Completed 12000 requests
Completed 16000 requests
Completed 20000 requests
Completed 24000 requests
Completed 28000 requests
Completed 32000 requests
Completed 36000 requests
Completed 40000 requests
Finished 40000 requests


Server Software:
Server Hostname:        2.2.2.2
Server Port:            8089

Document Path:          /
Document Length:        63 bytes

Concurrency Level:      500
Time taken for tests:   0.867 seconds
Complete requests:      40000
Failed requests:        0
Total transferred:      6040000 bytes
HTML transferred:       2520000 bytes
Requests per second:    46124.40 [#/sec] (mean)
Time per request:       10.840 [ms] (mean)
Time per request:       0.022 [ms] (mean, across all concurrent requests)
Transfer rate:          6801.55 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    4   1.0      4       7
Processing:     2    7   1.7      7      16
Waiting:        2    5   1.9      5      15
Total:          4   11   1.5     11      18

Percentage of the requests served within a certain time (ms)
  50%     11
  66%     11
  75%     12
  80%     12
  90%     12
  95%     13
  98%     15
  99%     15
 100%     18 (longest request)
```

####Examples
-------
- opendp_tcp_server, tcp server run on opendp tcp/ip stack.
- [dpdk-nginx](https://github.com/opendp/dpdk-nginx), nginx was porting to run on opendp tcp/ip stack.

####[Wiki Page](https://github.com/dpdk-net/netdp/wiki)
-------
You can get more information and instructions from [wiki page](https://github.com/dpdk-net/netdp/wiki).

####Notes
-------
- Netdp socket application run as a secondary dpdk process, If you got below log, shall execute below commands to disable ASLR.
```
EAL: WARNING: Address Space Layout Randomization (ASLR) is enabled in the kernel.
EAL: This may cause issues with mapping memory into secondary processes
$ sudo sysctl -w kernel.randomize_va_space=0
```
####Support
-------
BSD LICENSE, you may use netdp freely.

For free support, please use netdp team mail list at zimeiw@163.com. or QQ Group:86883521
