# Enable CPU and Memory accounting for docker (or any systemd service)

## Requirements

* enable cpu and memory accouting for docker so its consumation is visible when running ``systemctl status docker``

**Status with accounting disabled**

```
# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2016-05-20 19:54:03 UTC; 2s ago
     Docs: http://docs.docker.com
 Main PID: 11437 (sh)
   CGroup: /system.slice/docker.service
           ├─11437 /bin/sh -c /usr/bin/docker daemon            $OPTIONS            $DOCKER_STORAGE_OPTIONS            $DOCKER_NETWORK_OPTIONS            $INSECURE_REGISTRY ...
           ├─11438 /usr/bin/docker daemon --selinux-enabled --log-driver=journald
           └─11439 /usr/bin/forward-journald -tag docker
```

**Status with accounting enabled**

```
# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/docker.service.d
           └─50-CPUAccounting.conf, 50-MemoryAccounting.conf
   Active: active (running) since Fri 2016-05-20 19:54:03 UTC; 3min 19s ago
     Docs: http://docs.docker.com
 Main PID: 11437 (sh)
   Memory: 0B
      CPU: 902us
   CGroup: /system.slice/docker.service
           ├─11437 /bin/sh -c /usr/bin/docker daemon            $OPTIONS            $DOCKER_STORAGE_OPTIONS            $DOCKER_NETWORK_OPTIONS            $INSECURE_REGISTRY ...
           ├─11438 /usr/bin/docker daemon --selinux-enabled --log-driver=journald
           └─11439 /usr/bin/forward-journald -tag docker
```

Notice ``Memory`` and ``CPU`` fields.

## How to check the current setting

```
# systemctl show docker | grep Accounting
CPUAccounting=no
BlockIOAccounting=no
MemoryAccounting=no
```

## How to enable the accounting

```
# systemctl set-property docker.service MemoryAccounting=yes
# systemctl set-property docker.service CPUAccounting=yes
```

Checking...

```
# systemctl show docker | grep Accounting
CPUAccounting=yes
BlockIOAccounting=no
MemoryAccounting=yes
```

## Literature

* http://www.dsm.fordham.edu/cgi-bin/man-cgi.pl?topic=systemd.resource-control
* https://www.freedesktop.org/software/systemd/man/systemctl.html#set-property%20NAME%20ASSIGNMENT...
* https://www.freedesktop.org/software/systemd/man/systemd.resource-control.html#MemoryAccounting=

