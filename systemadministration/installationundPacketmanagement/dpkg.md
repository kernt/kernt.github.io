---
tags:
  - packet-management
  - dpkg
  - lowlevel
---
**`dpkg` command to find out which installed package owns a file**

`dpkg -S $(realpath $(which <command>))`

```sh
dpkg-query -W -f='Package: ${Package}\nProvides: ${Provides}\n' | grep -B 1 -E "^Provides: .*editor"
```

**How To Find The Package That Provides A Specific File In Linux**

`dpkg -S $(which alisp.h)`

or

`dpkg -S `which alisp.h``

`dpkg -S /bin/ls`

