# serf-web-load-balancer

original https://github.com/hashicorp/serf/tree/master/demo/web-load-balancer

refs https://github.com/kentaro/serf-hosts

## Usage

### Load Balancer Node

install command.

``` sh
export SERF_ROLE=lb
sh ./setup_load_balancer.sh
sh ./setup_serf.sh
```

stop agent command.

``` sh
stop serf
```

start agent command.

``` sh
start serf
```

### Web Node

require apache or nginx.

install command.

``` sh
export SERF_ROLE=web
export LB_IP="xxx.xxx.xxx.xxx"
sh ./setup_serf.sh
```

As a result,

 1. `web` is now also a member of cluster.
 2. `member-join` event is occurred. 
 3. `web` is added into `/etc/haproxy/haproxy.cfg` on `lb` node.

``` sh
[root@lb001 ~]# cat /etc/haproxy/haproxy.cfg
global
    daemon
    maxconn 256

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

listen stats
    bind *:9999
    mode http
    stats enable
    stats uri /
    stats refresh 2s

listen http-in
    bind *:80
    balance roundrobin
    option http-server-close
    server web001.foo.com 192.168.xxx.xxx check
    server web002.foo.com 192.168.xxx.xxx check
```

stop agent and leave command.

``` sh
sudo stop serf
```

start agent and join command.

``` sh
sudo start serf
sudo start serf-join
```

see the members of the Serf cluster

``` sh
serf members
```

## Web Node Leaves

stop `web`. Then `member-leave` event is propagated to `lb` and the handler script is fired.

``` sh
sudo stop serf
```

see the members of the Serf cluster

``` sh
serf members
```

As a result,

 1. `web` is now not a member of cluster.
 2. `member-leave` event is occurred. 
 3. `web` is removed from `/etc/haproxy/haproxy.cfg` on `lb` node.

## Distro Support

 * CentOS 6.x x86_64
 * Scientific Linux 6.x x84_64
