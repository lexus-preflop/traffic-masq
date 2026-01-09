
# Wireguard + [xt_wgobfs](https://github.com/infinet/xt_wgobfs)

## Server configuration
### build-dependence 

* Alpine: alpine-sdk iptables-dev linux-lts-dev or linux-virt-dev
* CentOS 7: iptables-devel kernel-devel
* Debian 10 to 13 : autoconf libtool libxtables-dev linux-headers pkg-config
* openSUSE 15: autoconf automake gcc kernel-default-devel libtool libxtables-devel make


By default, **openSUSE** does not allow unsupported kernel modeule. To override, create or modify /etc/modprobe.d/10-unsupported-modules.conf, add the following line:
```shell
allow_unsupported_modules 1
```
### For example, I will use Debian 13
#### Package update:
```shell
apt-get update 
apt-get upgrade
```
#
#### Install dependencies:
```shell
apt-get install git make autoconf libtool libxtables-dev linux-headers-$(uname-r) pkg-config
```
#
#### Clone repositories:
```shell
git clone https://github.com/infinet/xt_wgobfs
```
#
#### Build and install:
##### Build:
```shell
./autogen.sh
./configure
make
```
##### Install:
```shell
make install
```
#
#### Load the kernel module.
```sh
depmod -a && modprobe xt_WGOBFS
```
#
#### Configuration iptables rules

This extension takes two parameters.

>--key for a shared secret between client and server. If a key is a long string, it will be cut at 32 characters; if a key is short, then it will be repeated until reaches 32 characters. This 32 characters long string is the key used by chacha8 hash.

>--obfs or --unobfs to indicate the operation mode.

#### For example, get a 32-character* string.
```shell
root@server:~# RAND_STRING_32CH=$(cat /dev/urandom | tr -dc 'A-Za-z0-9' | head -c 32)
Bkt1V1JMFlbk2PiwIfeguiJlWCf1xwr8
```
#### Next, add a new rule to iptables
```shell
iptables -t mangle -I INPUT -p udp -m udp --dport 443 -j WGOBFS --key $RAND_STRING_32CH --unobfs

iptables -t mangle -I OUTPUT -p udp -m udp --sport 443 -j WGOBFS --key $RAND_STRING_32CH --obfs
```
##### If you want the rules to persist across reboots:
```shell
apt-get install iptables-persistent
iptables-save > /etc/iptables/rules.v4
```
#



