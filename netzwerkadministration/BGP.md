
![[BgpBasicRouting.svg]]


## Lab setup

## Router configurations

### rt-root

/etc/bird/bird.conf

router id 192.168.56.200;
protocol direct {
        interface "eth2", "eth3", "eth4";
}
protocol kernel {
        persist;                # Don't remove routes on bird shutdown
        scan time 20;           # Scan kernel routing table every 20 seconds
        export all;             # Default is export none
}
protocol device {
        scan time 10;           # Scan interfaces every 10 seconds
}
protocol static {
}
template bgp bgp_tepmlate {
        startup hold time 21;
        hold time 21;
        connect retry time 10;
        error wait time 5, 50;
        error forget time 50;
}
protocol bgp bgp_rt_1 from bgp_tepmlate {
        description "router rt-1";
        local as 65200;
        neighbor 192.168.201.2 as 65200;
        direct;
        import all;
        export all;
}
protocol bgp bgp_rt_2 from bgp_tepmlate {
        description "router rt-2";
        local as 65200;
        neighbor 192.168.202.2 as 65200;
        direct;
        import all;
        export all;
}

### rt-1

/etc/quagga/bgpd.conf

! -*- bgp -*-
!
! BGPd test configuratin file
!
! $Id: BgpRoutingExample.txt,v 1.3 2014/08/14 07:02:42 reves Exp $
!
hostname bgpd
password jahoda
enable password jahoda
!
!bgp mulitple-instance
!
router bgp 65200
 bgp router-id 192.168.56.201
 timers bgp 10 21
! network 192.168.0.0/16
 network 192.168.201.0/24
 network 192.168.211.0/24
 neighbor 192.168.211.2 remote-as 65222
 neighbor 192.168.201.1 remote-as 65200
! neighbor 192.168.211.2 route-map set-nexthop out
! neighbor 10.0.0.2 ebgp-multihop
! neighbor 192.168.211.2 next-hop-self
!
! access-list all permit any
!
!route-map set-nexthop permit 10
! match ip address all
! set ip next-hop 10.0.0.1
!
log file /var/log/quagga/bgpd.log
!
log monitor
log stdout
log syslog

### rt-2

/etc/bird/bird.conf

router id 192.168.56.202;
protocol direct {
        interface "eth2", "eth3";
}
protocol kernel {
        persist;                # Don't remove routes on bird shutdown
        scan time 20;           # Scan kernel routing table every 20 seconds
        export all;             # Default is export none
}
protocol device {
        scan time 10;           # Scan interfaces every 10 seconds
}
protocol static {
}
protocol bgp bgp_rt_root {
        description "BIRD BGP router rt-root";
        local as 65200;
        neighbor 192.168.202.1 as 65200;
        startup hold time 21;
        hold time 21;
        connect retry time 10;
        error wait time 5, 50;
        error forget time 50;
        direct;
        next hop self;
        import all;
        export all;
}
protocol bgp bgp_gateway {
        description "gateway";
        local as 65200;
        neighbor 192.168.212.2 as 65222;
        startup hold time 21;
        hold time 21;
        connect retry time 10;
        error wait time 5, 50;
        error forget time 50;
        direct;
        next hop self;
        import all;
        export all;
}

### gateway

/etc/bird/bird.conf

router id 192.168.56.222;
protocol direct {
        interface "eth2";
}
protocol kernel {
        persist;                # Don't remove routes on bird shutdown
        scan time 20;           # Scan kernel routing table every 20 seconds
        export all;             # Default is export none
}
protocol device {
        scan time 10;           # Scan interfaces every 10 seconds
}
protocol static {
        preference 1000;        # Default preference of routes
}
template bgp bgp_tepmlate {
        startup hold time 21;
        hold time 21;
        connect retry time 10;
        error wait time 5, 50;
        error forget time 50;
}
protocol bgp bgp_rt_1 from bgp_tepmlate {
        description "router rt-1";
        local as 65222;
        neighbor 192.168.211.1 as 65200;
        direct;
        import all;
        export all;
}
protocol bgp bgp_rt_2 from bgp_tepmlate {
        description "router rt-2";
        local as 65222;
        neighbor 192.168.212.1 as 65200;
        direct;
        import all;
        export all;
}

## 1st results

## Route path selection

/etc/bird/bird.conf

router id 192.168.56.222;
protocol direct {
        interface "eth2";
}
protocol kernel {
        persist;                # Don't remove routes on bird shutdown
        scan time 20;           # Scan kernel routing table every 20 seconds
        export all;             # Default is export none
}
protocol device {
        scan time 10;           # Scan interfaces every 10 seconds
}
protocol static {
        preference 1000;        # Default preference of routes
}
template bgp bgp_tepmlate {
        startup hold time 21;
        hold time 21;
        connect retry time 10;
        error wait time 5, 50;
        error forget time 50;
}
protocol bgp bgp_rt_1 from bgp_tepmlate {
        description "router rt-1";
        local as 65222;
        neighbor 192.168.211.1 as 65200;
        direct;
        #import all;
        import filter {
                #bgp_local_pref = 210; #PRIMARY
                bgp_local_pref = 190; #SECONDARY
                accept;
        };
        #export all;
        export filter {
                #bgp_med = 9; #PRIMARY
                bgp_med = 11; #SECONDARY
                accept;
        };
}
protocol bgp bgp_rt_2 from bgp_tepmlate {
        description "router rt-2";
        local as 65222;
        neighbor 192.168.212.1 as 65200;
        direct;
        #import all;
        import filter {
                bgp_local_pref = 200;
                accept;
};
        #export all;
        export filter {
                bgp_med = 10;
                accept;
        };
}