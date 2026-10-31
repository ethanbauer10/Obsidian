# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn 10.129.238.35 -vv

Host is up, received user-set (0.014s latency).
Scanned at 2026-10-31 19:29:56 GMT for 197s
Not shown: 65508 filtered tcp ports (no-response)
PORT      STATE SERVICE       REASON
22/tcp    open  ssh           syn-ack
25/tcp    open  smtp          syn-ack
80/tcp    open  http          syn-ack
111/tcp   open  rpcbind       syn-ack
135/tcp   open  msrpc         syn-ack
139/tcp   open  netbios-ssn   syn-ack
443/tcp   open  https         syn-ack
445/tcp   open  microsoft-ds  syn-ack
1063/tcp  open  kyoceranetdev syn-ack
2049/tcp  open  nfs           syn-ack
3389/tcp  open  ms-wbt-server syn-ack
5985/tcp  open  wsman         syn-ack
6160/tcp  open  ecmp          syn-ack
6161/tcp  open  patrol-ism    syn-ack
6162/tcp  open  patrol-coll   syn-ack
6169/tcp  open  unknown       syn-ack
6170/tcp  open  unknown       syn-ack
6190/tcp  open  unknown       syn-ack
6210/tcp  open  unknown       syn-ack
6290/tcp  open  unknown       syn-ack
10001/tcp open  scp-config    syn-ack
10002/tcp open  documentum    syn-ack
10003/tcp open  documentum_s  syn-ack
10005/tcp open  stel          syn-ack
10006/tcp open  netapp-sync   syn-ack
11731/tcp open  unknown       syn-ack
49669/tcp open  unknown       syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 197.42 seconds
```

## Nmap
```python

```

# SSH (22)

![](Pasted%20image%2020261031193222.png)

Password based auth

