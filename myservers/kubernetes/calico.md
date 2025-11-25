root@vmd134720:/etc# calicoctl ipam check
Checking IPAM for inconsistencies...

Loading all IPAM blocks...
Found 6 IPAM blocks.
 IPAM block 10.233.114.64/26 affinity=host:vmd134720.contaboserver.net:
 IPAM block 10.233.121.192/26 affinity=host:vmd132643.contaboserver.net:
 IPAM block 10.233.83.192/26 affinity=host:vmd154798.contaboserver.net:
 IPAM block fd85:ee78:d8a6:8607::1:3240/122 affinity=host:vmd134720.contaboserver.net:
 IPAM block fd85:ee78:d8a6:8607::1:39c0/122 affinity=host:vmd132643.contaboserver.net:
 IPAM block fd85:ee78:d8a6:8607::1:93c0/122 affinity=host:vmd154798.contaboserver.net:
IPAM blocks record 62 allocations.

Loading all IPAM pools...
  10.233.64.0/18
  fd85:ee78:d8a6:8607::1:0/112
Found 2 active IP pools.

Loading all nodes.
Found 3 node tunnel IPs.

Loading all workload endpoints.
Found 56 workload IPs.
Workloads and nodes are using 59 IPs.

Loading all handles
Looking for top (up to 20) nodes by allocations...
  vmd132643.contaboserver.net has 26 allocations
  vmd154798.contaboserver.net has 26 allocations
  vmd134720.contaboserver.net has 10 allocations
Node with most allocations has 26; median is 26

Scanning for IPs that are allocated but not actually in use...
Found 3 IPs that are allocated in IPAM but not actually in use.
Scanning for IPs that are in use by a workload or node but not allocated in IPAM...
Found 0 in-use IPs that are not in active IP pools.
Found 0 in-use IPs that are in active IP pools but have no corresponding IPAM allocation.

Scanning for IPAM handles with no matching IPs...
Found 0 handles with no matching IPs (and 34 handles with matches).
Scanning for IPs with missing handle...
Found 0 handles mentioned in blocks with no matching handle resource.
Check complete; found 3 problems.