## Unbound

Print cache to stdout

`unbound-control dump_cache`

In case some invalid records are cached, it may be required to flush everything

```sh
unbound-control flush_zone .
ok removed 19 rrsets, 8 messages and 3 key entries
```

This entry was posted in DNS, Linux and tagged BIND, CentOS, EX300, RHCE, RHEL, Unbound. Bookmark the permalink. If you notice any errors, please contact us.