## FTP and Rsync 

FTP ``File Transfer Protocol``

Protocol used for copying entire files over the network from a remote server. Its an older slower protocol. 

Rsync: is a fast and extraordinarily versatile file copying tool. It can copy locally,
to/from another host over any remote shell, or to/from a remote rsync daemon. It offers
a large number of options that control every aspect of its behavior and permit very
flexible specification of the set of files to be copied. It is famous for its deltatransfer algorithm, which reduces the amount of data sent over the network by sending
only the differences between the source files and the existing files in the
destination. Rsync is widely used for backups and mirroring and as an improved copy
command for everyday use.


The main stages of an rsync transfer are the following:
1. rsync establishes a connection to the remote host and spawns another rsync receiver process.
2. The sender and receiver processes compare what files have changed.
3. What has changed gets updated on the remote host.

**DEFAULT PORT = 873**
## Security Flaws
<hr>
Rsync is often misconfigured  to allow anonymous login which can be exploited to access sensitive information on a remote server. 



Anonymous login example 



``rsync --list-only [Target IP]::
``

The above command will list the available shared directories on the target. 
