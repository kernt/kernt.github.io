# lscpu


## arsing lscpu Output

```sh
#!/bin/bash

# Get the extended CPU information
lscpu -e | awk 'NR>1 {print "CPU "$1" is in NUMA node "$2}'
```

https://www.golinuxcloud.com/lscpu-command-in-linux/
https://www.baeldung.com/linux/lscpu-command-tutorial