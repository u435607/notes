# ssh
to change/remove passphrase on the private key
```$ ssh-keygen -p```
# winrm
```ansible -i hostwin.txt -m raw -a 'Get-CimInstance -ClassName Win32_LogicalDisk' all```
<pre>
DeviceID DriveType ProviderName VolumeName Size         FreeSpace
-------- --------- ------------ ---------- ----         ---------
C:       3                                 53040115712  33999831040
D:       3                                 107355303936 107251564544
H:       5
</pre>
# chrony
when ```cmdport 0 is set in /etc/chrony.conf``` then chronyc can not connect to the daemon when running as normal user. Works for root as he has access to the ```/var/run/chrony/chronyd.sock``` And  ```# chronyc tracking |grep Stratum``` shows distance from stratum

https://en.wikipedia.org/wiki/Network_Time_Protocol

https://en.wikipedia.org/wiki/NTP_server_misuse_and_abuse
