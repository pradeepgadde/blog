---
layout: single
title:  "Python for Network Engineers Week#2"
categories: Programming
tags: Python
show_date: true
toc: true
toc_sticky: true
header:
  teaser: /assets/images/python.png
author:
  name     : "Python"
  avatar   : "/assets/images/python.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar
---
# Python for Network Engineers Week 2

## Reading from a File

Open file for reading, use the `.read()` method to read in the entire file as a string. Finally, close the file. Note the file is in the same directory.

```python
(Python4Networking) (base) pradeep:~$cat file-reading-1.py 
f1=open("junos-config.txt")
print (type(f1))
my_config=f1.read()
print(type(my_config))
f1.close()
print(my_config)
(Python4Networking) (base) pradeep:~$
```

```sh
(Python4Networking) (base) pradeep:~$python3 file-reading-1.py 
<class '_io.TextIOWrapper'>
<class 'str'>
root@srx1> show configuration | display set 
set system host-name srx1
set system root-authentication encrypted-password "$1$nq.N1UsY$JxA/ESAj3KuXseXE597gg0"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDS8VmjJwq5I7swH3/Uh6UIecpz4AXF3WGMWTZQUP2TpKxA6cRfMbjnS3WgMpsjWwiM67n5/5ONfeLRwMerFyLTEPRZZhUOpk7PRInUHXOQ9n25eeOwxXA2RyD5FHmlmBnr1zf38Yq2Za+zR0PmY2k7KBb4EILfAeOcJnIj4IurQWJvf0mt1dQzqkRj6nT7c3q5849NUlL+S2sDjCBI/0UqZbX1dic1i1IwXALuvNz2ZWhROQRzQf21+ZiwKFqfI3A5asMrTZDI7Cfy9VNrK4BeCJOAcZJHGTgleusYZGnq05ZQQY0U/htw3FQ2Rkj+nKgAGSza6/2Mi6mBuIyzk/0P vagrant"
set system login user vagrant uid 2000
set system login user vagrant class super-user
set system login user vagrant authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"
set system services ssh root-login allow
set system services netconf ssh
set system services web-management http interface ge-0/0/0.0
set system syslog user * any emergency
set system syslog file messages any any
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set system license autoupdate url https://ae1.juniper.net/junos/key_retrieval
set interfaces ge-0/0/0 unit 0 family inet dhcp
set interfaces ge-0/0/1 unit 0 family inet address 192.168.1.1/24
set security forwarding-options family inet6 mode packet-based
set security forwarding-options family mpls mode packet-based

root@srx1> 

(Python4Networking) (base) pradeep:~$
```

Note the type of  input and output, at the top. Defaults to text file and read mode. These settings can be changed.

Another way, using `readline()` method, to read each line, line by line.

```py
(Python4Networking) (base) pradeep:~$python3
Python 3.9.12 (main, Apr  5 2022, 01:53:17) 
[Clang 12.0.0 ] :: Anaconda, Inc. on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> f1 = open("junos-config.txt")
>>> f1
<_io.TextIOWrapper name='junos-config.txt' mode='r' encoding='UTF-8'>
>>> f1.readline()
'root@srx1> show configuration | display set \n'
>>> f1.readline()
'set system host-name srx1\n'
>>> f1.readline()
'set system root-authentication encrypted-password "$1$nq.N1UsY$JxA/ESAj3KuXseXE597gg0"\n'
>>> f1.readline()
'set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDS8VmjJwq5I7swH3/Uh6UIecpz4AXF3WGMWTZQUP2TpKxA6cRfMbjnS3WgMpsjWwiM67n5/5ONfeLRwMerFyLTEPRZZhUOpk7PRInUHXOQ9n25eeOwxXA2RyD5FHmlmBnr1zf38Yq2Za+zR0PmY2k7KBb4EILfAeOcJnIj4IurQWJvf0mt1dQzqkRj6nT7c3q5849NUlL+S2sDjCBI/0UqZbX1dic1i1IwXALuvNz2ZWhROQRzQf21+ZiwKFqfI3A5asMrTZDI7Cfy9VNrK4BeCJOAcZJHGTgleusYZGnq05ZQQY0U/htw3FQ2Rkj+nKgAGSza6/2Mi6mBuIyzk/0P vagrant"\n'
>>> 

```

To go to the beginning of the file, use the `seek` method with a value of `0`.

To read all of the lines of a file as a list, use the `readlines()`method. Note the plural usage here.

```py
>>> f1.seek(0)
0
>>> my_config=f1.readlines()
>>> type(my_config)
<class 'list'>
>>> print(my_config)
['root@srx1> show configuration | display set \n', 'set system host-name srx1\n', 'set system root-authentication encrypted-password "$1$nq.N1UsY$JxA/ESAj3KuXseXE597gg0"\n', 'set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDS8VmjJwq5I7swH3/Uh6UIecpz4AXF3WGMWTZQUP2TpKxA6cRfMbjnS3WgMpsjWwiM67n5/5ONfeLRwMerFyLTEPRZZhUOpk7PRInUHXOQ9n25eeOwxXA2RyD5FHmlmBnr1zf38Yq2Za+zR0PmY2k7KBb4EILfAeOcJnIj4IurQWJvf0mt1dQzqkRj6nT7c3q5849NUlL+S2sDjCBI/0UqZbX1dic1i1IwXALuvNz2ZWhROQRzQf21+ZiwKFqfI3A5asMrTZDI7Cfy9VNrK4BeCJOAcZJHGTgleusYZGnq05ZQQY0U/htw3FQ2Rkj+nKgAGSza6/2Mi6mBuIyzk/0P vagrant"\n', 'set system login user vagrant uid 2000\n', 'set system login user vagrant class super-user\n', 'set system login user vagrant authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"\n', 'set system services ssh root-login allow\n', 'set system services netconf ssh\n', 'set system services web-management http interface ge-0/0/0.0\n', 'set system syslog user * any emergency\n', 'set system syslog file messages any any\n', 'set system syslog file messages authorization info\n', 'set system syslog file interactive-commands interactive-commands any\n', 'set system license autoupdate url https://ae1.juniper.net/junos/key_retrieval\n', 'set interfaces ge-0/0/0 unit 0 family inet dhcp\n', 'set interfaces ge-0/0/1 unit 0 family inet address 192.168.1.1/24\n', 'set security forwarding-options family inet6 mode packet-based\n', 'set security forwarding-options family mpls mode packet-based\n', '\n', 'root@srx1> \n']
>>> 

```

As you can see now, `my_config` is a list now, with each line of the config as a list item.

```python
>>> len(my_config)
21
>>> 
```

```py
>>> dir(f1)
['_CHUNK_SIZE', '__class__', '__del__', '__delattr__', '__dict__', '__dir__', '__doc__', '__enter__', '__eq__', '__exit__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__lt__', '__ne__', '__new__', '__next__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '_checkClosed', '_checkReadable', '_checkSeekable', '_checkWritable', '_finalizing', 'buffer', 'close', 'closed', 'detach', 'encoding', 'errors', 'fileno', 'flush', 'isatty', 'line_buffering', 'mode', 'name', 'newlines', 'read', 'readable', 'readline', 'readlines', 'reconfigure', 'seek', 'seekable', 'tell', 'truncate', 'writable', 'write', 'write_through', 'writelines']
>>> 
```

Third method is looping over the lines in the file.

```py
>>> f1=open("junos-config.txt")
>>> for line in f1:
...     print(line)
... 
root@srx1> show configuration | display set 

set system host-name srx1

set system root-authentication encrypted-password "$1$nq.N1UsY$JxA/ESAj3KuXseXE597gg0"

set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDS8VmjJwq5I7swH3/Uh6UIecpz4AXF3WGMWTZQUP2TpKxA6cRfMbjnS3WgMpsjWwiM67n5/5ONfeLRwMerFyLTEPRZZhUOpk7PRInUHXOQ9n25eeOwxXA2RyD5FHmlmBnr1zf38Yq2Za+zR0PmY2k7KBb4EILfAeOcJnIj4IurQWJvf0mt1dQzqkRj6nT7c3q5849NUlL+S2sDjCBI/0UqZbX1dic1i1IwXALuvNz2ZWhROQRzQf21+ZiwKFqfI3A5asMrTZDI7Cfy9VNrK4BeCJOAcZJHGTgleusYZGnq05ZQQY0U/htw3FQ2Rkj+nKgAGSza6/2Mi6mBuIyzk/0P vagrant"

set system login user vagrant uid 2000

set system login user vagrant class super-user

set system login user vagrant authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"

set system services ssh root-login allow

set system services netconf ssh

set system services web-management http interface ge-0/0/0.0

set system syslog user * any emergency

set system syslog file messages any any

set system syslog file messages authorization info

set system syslog file interactive-commands interactive-commands any

set system license autoupdate url https://ae1.juniper.net/junos/key_retrieval

set interfaces ge-0/0/0 unit 0 family inet dhcp

set interfaces ge-0/0/1 unit 0 family inet address 192.168.1.1/24

set security forwarding-options family inet6 mode packet-based

set security forwarding-options family mpls mode packet-based



root@srx1> 

>>> 

```

