ProxyChains is a UNIX program, that hooks network-related libc functions in dynamically linked programs via a preloaded DLL and redirects the connections through SOCKS4a/5 or HTTP proxies.

WARNING: this program works only on dynamically linked programs. also both proxychains and the program to call must use the same dynamic linker (i.e. same libc)

https://github.com/haad/proxychains