# Availability sets vs Availability zones in Azure

_Initial setup_:
4 VMs: malzonka01 malzonka02 malzonka03 malzonka04 Standard D4as_v4 (4 vcpus, 16 GiB memory)

Linux (ubuntu 20.04)

The same subscription and vnet

Accelerated networking: Enabled

Proximity groups: NOT Enabled

Location: WEUR

No kernel nor network tuning


## Test 1 sockperf
_malzonka01 created in **Zone1**
malzonka02 created in **Zone3**_

Installed iperf3 and sockperf via apt.

```
root@malzonka02:/home/azureuser# sockperf ping-pong -i 10.0.1.4 --tcp -m 1400 -t 101 -p 12345 --full-rtt
sockperf: == version #3.6-no.git ==
sockperf[CLIENT] send on:sockperf: using recvfrom() to block on socket(s)

[ 0] IP = 10.0.1.4        PORT = 12345 # TCP
sockperf: Warmup stage (sending a few dummy messages)...
sockperf: Starting test...
sockperf: Test end (interrupted by timer)
sockperf: Test ended
sockperf: [Total Run] RunTime=101.000 sec; Warm up time=400 msec; SentMessages=1064751; ReceivedMessages=1064750
sockperf: ========= Printing statistics for Server No: 0
sockperf: [Valid Duration] RunTime=100.549 sec; SentMessages=1060186; ReceivedMessages=1060186
sockperf: ====> avg-rtt=94.790 (std-dev=9.300)
sockperf: # dropped messages = 0; # duplicated messages = 0; # out-of-order messages = 0
sockperf: Summary: Round trip is 94.790 usec
sockperf: Total 1060186 observations; each percentile contains 10601.86 observations
sockperf: ---> <MAX> observation = 2129.252
sockperf: ---> percentile 99.999 =  773.712
sockperf: ---> percentile 99.990 =  338.588
sockperf: ---> percentile 99.900 =  249.029
sockperf: ---> percentile 99.000 =  106.059
sockperf: ---> percentile 90.000 =   98.485
sockperf: ---> percentile 75.000 =   95.800
sockperf: ---> percentile 50.000 =   93.556
sockperf: ---> percentile 25.000 =   92.464
sockperf: ---> <MIN> observation =   86.803
```



Reran with smaller packet size: 350

```
root@malzonka02:/home/azureuser# sockperf ping-pong -i 10.0.1.4 --tcp -m 350 -t 101 -p 12345 --full-rtt
sockperf: == version #3.6-no.git ==
sockperf[CLIENT] send on:sockperf: using recvfrom() to block on socket(s)

[ 0] IP = 10.0.1.4        PORT = 12345 # TCP
sockperf: Warmup stage (sending a few dummy messages)...
sockperf: Starting test...
sockperf: Test end (interrupted by timer)
sockperf: Test ended
sockperf: [Total Run] RunTime=101.000 sec; Warm up time=400 msec; SentMessages=1175132; ReceivedMessages=1175131
sockperf: ========= Printing statistics for Server No: 0
sockperf: [Valid Duration] RunTime=100.550 sec; SentMessages=1169871; ReceivedMessages=1169871
sockperf: ====> avg-rtt=85.902 (std-dev=7.425)
sockperf: # dropped messages = 0; # duplicated messages = 0; # out-of-order messages = 0
sockperf: Summary: Round trip is 85.902 usec
sockperf: Total 1169871 observations; each percentile contains 11698.71 observations
sockperf: ---> <MAX> observation = 2513.221
sockperf: ---> percentile 99.999 =  363.586
sockperf: ---> percentile 99.990 =  275.689
sockperf: ---> percentile 99.900 =  229.613
sockperf: ---> percentile 99.000 =   96.321
sockperf: ---> percentile 90.000 =   89.789
sockperf: ---> percentile 75.000 =   87.895
sockperf: ---> percentile 50.000 =   84.389
sockperf: ---> percentile 25.000 =   83.477
sockperf: ---> <MIN> observation =   77.406
```

## TEST 2 ping test
20 packets transmitted, 20 received, 0% packet loss, time 19235ms
rtt min/avg/max/mdev = 0.718/0.969/2.137/0.299 ms

## TEST 3 iperf3
On malzonka01 ```# iperf3 -s -p 5001```

```
root@malzonka02:/home/azureuser# iperf3 -c 10.0.1.4 -p 5001
Connecting to host 10.0.1.4, port 5001
[  5] local 10.0.1.5 port 40502 connected to 10.0.1.4 port 5001
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   461 MBytes  3.87 Gbits/sec    0   1.28 MBytes
[  5]   1.00-2.00   sec   454 MBytes  3.81 Gbits/sec    0   1.35 MBytes
[  5]   2.00-3.00   sec   454 MBytes  3.81 Gbits/sec    0   1.35 MBytes
[  5]   3.00-4.00   sec   454 MBytes  3.81 Gbits/sec    0   1.35 MBytes
[  5]   4.00-5.00   sec   454 MBytes  3.81 Gbits/sec    0   1.35 MBytes
[  5]   5.00-6.00   sec   454 MBytes  3.81 Gbits/sec    0   1.35 MBytes
[  5]   6.00-7.00   sec   454 MBytes  3.81 Gbits/sec    0   1.35 MBytes
[  5]   7.00-8.00   sec   455 MBytes  3.82 Gbits/sec    0   1.35 MBytes
[  5]   8.00-9.00   sec   452 MBytes  3.80 Gbits/sec    0   1.35 MBytes
[  5]   9.00-10.00  sec   454 MBytes  3.81 Gbits/sec    0   1.35 MBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  4.44 GBytes  3.81 Gbits/sec    0             sender
[  5]   0.00-10.00  sec  4.44 GBytes  3.81 Gbits/sec                  receiver

iperf Done.
```

_malzonka03 and malzonka04 in the **same availability set**._

## Test 4 sockperf on as
```
root@malzonka04:/home/azureuser# sockperf ping-pong -i 10.0.1.6 --tcp -m 1400 -t 101 -p 12345 --full-rtt
sockperf: == version #3.6-no.git ==
sockperf[CLIENT] send on:sockperf: using recvfrom() to block on socket(s)

[ 0] IP = 10.0.1.6        PORT = 12345 # TCP
sockperf: Warmup stage (sending a few dummy messages)...
sockperf: Starting test...
sockperf: Test end (interrupted by timer)
sockperf: Test ended
sockperf: [Total Run] RunTime=101.000 sec; Warm up time=400 msec; SentMessages=1850855; ReceivedMessages=1850854
sockperf: ========= Printing statistics for Server No: 0
sockperf: [Valid Duration] RunTime=100.550 sec; SentMessages=1842588; ReceivedMessages=1842588
sockperf: ====> avg-rtt=54.516 (std-dev=8.896)
sockperf: # dropped messages = 0; # duplicated messages = 0; # out-of-order messages = 0
sockperf: Summary: Round trip is 54.516 usec
sockperf: Total 1842588 observations; each percentile contains 18425.88 observations
sockperf: ---> <MAX> observation = 2205.026
sockperf: ---> percentile 99.999 =  615.469
sockperf: ---> percentile 99.990 =  245.213
sockperf: ---> percentile 99.900 =  219.133
sockperf: ---> percentile 99.000 =   67.660
sockperf: ---> percentile 90.000 =   59.034
sockperf: ---> percentile 75.000 =   56.709
sockperf: ---> percentile 50.000 =   53.433
sockperf: ---> percentile 25.000 =   51.118
sockperf: ---> <MIN> observation =   42.431
```
root@malzonka04:/home/azureuser
```
# sockperf ping-pong -i 10.0.1.6 --tcp -m 350  -t 101 -p 12345 --full-rtt
sockperf: == version #3.6-no.git ==
sockperf[CLIENT] send on:sockperf: using recvfrom() to block on socket(s)

[ 0] IP = 10.0.1.6        PORT = 12345 # TCP
sockperf: Warmup stage (sending a few dummy messages)...
sockperf: Starting test...
sockperf: Test end (interrupted by timer)
sockperf: Test ended
sockperf: [Total Run] RunTime=101.000 sec; Warm up time=400 msec; SentMessages=2214075; ReceivedMessages=2214074
sockperf: ========= Printing statistics for Server No: 0
sockperf: [Valid Duration] RunTime=100.550 sec; SentMessages=2203938; ReceivedMessages=2203938
sockperf: ====> avg-rtt=45.570 (std-dev=8.102)
sockperf: # dropped messages = 0; # duplicated messages = 0; # out-of-order messages = 0
sockperf: Summary: Round trip is 45.570 usec
sockperf: Total 2203938 observations; each percentile contains 22039.38 observations
sockperf: ---> <MAX> observation = 2137.585
sockperf: ---> percentile 99.999 =  659.135
sockperf: ---> percentile 99.990 =  234.544
sockperf: ---> percentile 99.900 =  210.156
sockperf: ---> percentile 99.000 =   57.249
sockperf: ---> percentile 90.000 =   49.526
sockperf: ---> percentile 75.000 =   47.100
sockperf: ---> percentile 50.000 =   44.215
sockperf: ---> percentile 25.000 =   43.063
sockperf: ---> <MIN> observation =   36.641
```

## TEST 4 iperf3 on the availability set VMs
```
root@malzonka04:/home/azureuser# iperf3 -c 10.0.1.6 -p 5001
Connecting to host 10.0.1.6, port 5001
[  5] local 10.0.1.7 port 43202 connected to 10.0.1.6 port 5001
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   461 MBytes  3.87 Gbits/sec    0   1.32 MBytes
[  5]   1.00-2.00   sec   454 MBytes  3.81 Gbits/sec    0   1.32 MBytes
[  5]   2.00-3.00   sec   454 MBytes  3.81 Gbits/sec    0   1.32 MBytes
[  5]   3.00-4.00   sec   454 MBytes  3.81 Gbits/sec    0   1.32 MBytes
[  5]   4.00-5.00   sec   454 MBytes  3.81 Gbits/sec    0   1.32 MBytes
[  5]   5.00-6.00   sec   454 MBytes  3.81 Gbits/sec    0   1.32 MBytes
[  5]   6.00-7.00   sec   454 MBytes  3.81 Gbits/sec    0   1.32 MBytes
[  5]   7.00-8.00   sec   454 MBytes  3.81 Gbits/sec    0   1.32 MBytes
[  5]   8.00-9.00   sec   455 MBytes  3.82 Gbits/sec    0   1.32 MBytes
[  5]   9.00-10.00  sec   452 MBytes  3.80 Gbits/sec    0   1.32 MBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  4.44 GBytes  3.81 Gbits/sec    0             sender
[  5]   0.00-10.00  sec  4.44 GBytes  3.81 Gbits/sec                  receiver

iperf Done.
```

## TEST5 ping avset
20 packets transmitted, 20 received, 0% packet loss, time 19161ms
rtt min/avg/max/mdev = 0.728/1.902/10.265/2.507 ms


## Conclusion
In the sockperf test latency was lower on the availabiliy set test comparing to the zonal VMs but overall no dramatical differences.
Worth to check additionally how latency problematic application: ActiveMQ or Artemis on the NFS4 share (HA mode) would behave in such situation when comparing as, az and proximity groups.

