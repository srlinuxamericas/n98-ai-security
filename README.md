# Reference lab for Network for AI Security Blueprint

This is a reference lab providing a blueprint for securing Networks for AI as described in NANOG 98 talk - **A Blueprint for Securing Networks for AI : From Architecture to Operations**


## Lab Topology

![image](lab-topology.jpg)

### Lab Configuration

When the lab is deployed, a leaf-spine fabric is created with Layer2 and Layer3 EVPN configured to allow communication between the clients.

Client1 (192.168.10.50) and Client3 (192.168.10.60) are connected over a Layer2 EVPN.

Client2 (10.80.1.1) and Client4 (10.90.1.1) are connected over a Layer3 EVPN.

## Deploying the lab

Install Containerlab on your VM.

```bash
curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"
```

**Logout and login for the sudo privileges to take effect.**

Clone this Git repo to your VM:

```
git clone https://github.com/srlinuxamericas/n98-ai-security.git
```

Verify that the git repo files are now available on your VM.

```
ls -lrt n98-ai-security/
```

To deploy the lab, run the following:

```
cd n98-ai-security
sudo clab deploy -t n98-ai-sec.clab.yml
```

[Containerlab](https://containerlab.dev/) will deploy the lab and display a table with the list of nodes and their IPs.

```bash
╭─────────┬──────────────────────────────┬─────────┬────────────────────╮
│   Name  │          Kind/Image          │  State  │   IPv4/6 Address   │
├─────────┼──────────────────────────────┼─────────┼────────────────────┤
│ client1 │ linux                        │ running │ 172.20.20.10       │
│         │ ghcr.io/srl-labs/alpine      │         │ 2001:172:20:20::10 │
├─────────┼──────────────────────────────┼─────────┼────────────────────┤
│ client2 │ linux                        │ running │ 172.20.20.11       │
│         │ ghcr.io/srl-labs/alpine      │         │ 2001:172:20:20::11 │
├─────────┼──────────────────────────────┼─────────┼────────────────────┤
│ client3 │ linux                        │ running │ 172.20.20.12       │
│         │ ghcr.io/srl-labs/alpine      │         │ 2001:172:20:20::12 │
├─────────┼──────────────────────────────┼─────────┼────────────────────┤
│ client4 │ linux                        │ running │ 172.20.20.13       │
│         │ ghcr.io/srl-labs/alpine      │         │ 2001:172:20:20::13 │
├─────────┼──────────────────────────────┼─────────┼────────────────────┤
│ leaf1   │ nokia_srlinux                │ running │ 172.20.20.2        │
│         │ ghcr.io/nokia/srlinux:26.7.1 │         │ 2001:172:20:20::2  │
├─────────┼──────────────────────────────┼─────────┼────────────────────┤
│ leaf2   │ nokia_srlinux                │ running │ 172.20.20.4        │
│         │ ghcr.io/nokia/srlinux:26.7.1 │         │ 2001:172:20:20::4  │
├─────────┼──────────────────────────────┼─────────┼────────────────────┤
│ spine   │ nokia_srlinux                │ running │ 172.20.20.3        │
│         │ ghcr.io/nokia/srlinux:26.7.1 │         │ 2001:172:20:20::3  │
╰─────────┴──────────────────────────────┴─────────┴────────────────────╯
```

To display all deployed labs on your VM at any time, use:

```
sudo clab inspect --all
```

### Using Codespaces

This lab can be deployed using GitHub Codespaces. Once you are logged into GitHub, click the below icon and wait for 2 minutes for codespaces to be ready. During this initialization, codespace will install containerlab and gRPC clients so that you are all set to run the use cases immediately.

---
<div align=center>
<a href="https://codespaces.new/srlinuxamericas/n98-ai-security?quickstart=1">
<img src="https://gitlab.com/rdodin/pics/-/wikis/uploads/d78a6f9f6869b3ac3c286928dd52fa08/run_in_codespaces-v1.svg?sanitize=true" style="width:50%"/></a>

**[Run](https://codespaces.new/srlinuxamericas/n98-ai-security?quickstart=1) this lab in GitHub Codespaces for free**.  
[Learn more](https://containerlab.dev/manual/codespaces/) about Containerlab for Codespaces.

</div>

---

### gRPC clients

We will be using the following gRPC clients:

- [gNMIc](https://gnmic.openconfig.net/)
- [gNOIc](https://gnoic.kmrd.dev/)
- [gNSIc](https://github.com/karimra/gnsic)

Follow the below steps to install these clients on your VM or codespace.

```bash
bash -c "$(curl -sL https://get-gnmic.openconfig.net)"
sudo bash -c "$(curl -sL https://get-gnoic.kmrd.dev)"
curl -L -O https://github.com/karimra/gnsic/releases/download/v0.0.4/gnsic_0.0.4_Linux_x86_64.tar.gz
tar -xzvf gnsic_0.0.4_Linux_x86_64.tar.gz
sudo cp gnsic /usr/local/bin/
```

Verify that clients are installed on your VM:

```
gnmic version
sudo gnoic version
gnsic version
```

All gnmic, gnoic and gnsic commands will be executed from the VM.

### Tshark

Based on your host VM, follow the below steps to install tshark.

For RHEL, Rocky Linux:

```bash
sudo dnf install wireshark-cli -y
```

For Debian, Ubuntu and Codespace VM:

```bash
sudo apt update
sudo apt install tshark -y
```

Verify tshark is installed using:

```bash
tshark --version
```

## Connecting to the devices

Find the nodename or IP address of the device from the above output and then use SSH.

Username: `admin`

Password: `admin`

Note: Password less authentication is enabled by Containerlab using SSH keys.

```
ssh leaf1
```

To login to the client, identify the client hostname using the `sudo clab inspect --all` command above and then:

```bash
sudo docker exec -it client1 bash
```

## SR Linux Configuration Mode

To enter candidate configuration edit mode in SR Linux, use:

```srl
enter candidate private
```

To commit the configuration in SR Linux, use:

```srl
commit stay
```

Here's a reference table with some commonly used commands.

| Action | Command |
| --- | --- |
| Enter Candidate mode | `enter candidate {private}` |
| Commit configuration changes | `commit {now\|stay}` |
| | `now` – commits and exits from candidate mode |
| | `stay` – commits and stays in candidate mode |
| Delete configuration elements | `delete` |
| | Eg: `delete interface ethernet-1/5` |
| Discard configuration changes | `discard {now\|stay}` |
| Compare candidate to running | `diff running /` |
| View configuration in current mode & context | `info {flat}` |
| View configuration in another mode & context | `info {flat} from state /interface ethernet-1/1` |
| Output modifiers | `<command> \| as {table\|json\|yaml}` |
| Access Linux shell | `bash` |
| Find a command | `tree flat detail \| grep <keyword>` |

## Security lab

**All commands in this tutorial are executed on leaf1**

### SZTP, Secure and Measured Boot

These items cannot be configured or testing in Containerlab.

For more information, please refer to the below SR Linux Documentation links:

[Secure ZTP](https://documentation.nokia.com/srlinux/26-7/books/software-install/secureztp.html)

[Secure Boot](https://documentation.nokia.com/srlinux/26-7/books/software-install/swinstall-intro.html#secure_boot)

[Measured Boot](https://documentation.nokia.com/srlinux/26-7/books/software-install/swinstall-intro.html#measured_boot)

### Enabling TLS

#### gRPC

To secure gRPC using TLS, we need TLS certificates and associated artifacts.

We will use gRPC gNSI to configure and manage TLS profile and certificates on the switch. This is a much easier way compared to manually generating certificates and transferring them to the switch.

gNSI Certz RPCs are used for this purpose.

With the deployed lab, gRPC is enabled to use insecure communication.

Let's test it out before proceeding by getting the BGP peer status of `spine` from `leaf1` using gNMI. The `insecure` gRPC server is listening on port `57401` on `leaf1`.

To be 100% sure that communication is not encrypted, we will take a live capture of the packets using `tshark`.

On your host VM, start the capture using the below command and leave it running

```bash
sudo tshark -ni any -f "src leaf1 and port 57401" -P -x
```

Open another session on your host VM and run the below command to get the BGP peer session state.

```bash
gnmic -a leaf1:57401 -u admin -p admin --insecure get --path "/network-instance[name=default]/protocols/bgp/neighbor[peer-address=192.168.10.3]/session-state" -e json_ietf
```

Expected output:

```bash
[
  {
    "source": "leaf1:57401",
    "timestamp": 1792199039768093781,
    "time": "2026-10-16T21:03:59.768093781-04:00",
    "updates": [
      {
        "Path": "srl_nokia-network-instance:network-instance[name=default]/protocols/srl_nokia-bgp:bgp/neighbor[peer-address=192.168.10.3]/session-state",
        "values": {
          "srl_nokia-network-instance:network-instance/protocols/srl_nokia-bgp:bgp/neighbor/session-state": "established"
        }
      }
    ]
  }
]
```

Now stop the `tshark` packet capture using `CTRL+c` and check the packets captured by `tshark`. You will be able to see plain text being sent from the switch for the BGP peer session state.

Sample output:

```bash
   14 0.033212042 2001:172:20:20::2 → 2001:172:20:20::1 TCP 419 [TCP Retransmission] 57401 → 41716 [PSH, ACK] Seq=79 Ack=342 Win=64512 Len=331 TSval=791292457 TSecr=2649651251

0000  00 00 00 01 00 06 1a 81 04 00 00 00 00 00 86 dd   ................
0010  68 8f 50 5c 01 6b 06 40 20 01 01 72 00 20 00 20   h.P\.k.@ ..r. . 
0020  00 00 00 00 00 00 00 02 20 01 01 72 00 20 00 20   ........ ..r. . 
0030  00 00 00 00 00 00 00 01 e0 39 a2 f4 8d 38 8a 3a   .........9...8.:
0040  50 48 5a 59 80 18 00 3f 5b ca 00 00 01 01 08 0a   PHZY...?[.......
0050  2f 2a 2a 29 9d ee 78 33 00 00 4e 01 04 00 00 00   /**)..x3..N.....
0060  01 88 40 0c 63 6f 6e 74 65 6e 74 2d 74 79 70 65   ..@.content-type
0070  10 61 70 70 6c 69 63 61 74 69 6f 6e 2f 67 72 70   .application/grp
0080  63 40 14 67 72 70 63 2d 61 63 63 65 70 74 2d 65   c@.grpc-accept-e
0090  6e 63 6f 64 69 6e 67 17 69 64 65 6e 74 69 74 79   ncoding.identity
00a0  2c 20 64 65 66 6c 61 74 65 2c 20 67 7a 69 70 00   , deflate, gzip.
00b0  00 c6 00 00 00 00 00 01 00 00 00 00 c1 0a be 01   ................
00c0  08 bb d5 d2 d8 99 fb ca ef 18 22 b1 01 0a 9d 01   ..........".....
00d0  1a 3e 0a 2b 73 72 6c 5f 6e 6f 6b 69 61 2d 6e 65   .>.+srl_nokia-ne
00e0  74 77 6f 72 6b 2d 69 6e 73 74 61 6e 63 65 3a 6e   twork-instance:n
00f0  65 74 77 6f 72 6b 2d 69 6e 73 74 61 6e 63 65 12   etwork-instance.
0100  0f 0a 04 6e 61 6d 65 12 07 64 65 66 61 75 6c 74   ...name..default
0110  1a 0b 0a 09 70 72 6f 74 6f 63 6f 6c 73 1a 13 0a   ....protocols...
0120  11 73 72 6c 5f 6e 6f 6b 69 61 2d 62 67 70 3a 62   .srl_nokia-bgp:b
0130  67 70 1a 28 0a 08 6e 65 69 67 68 62 6f 72 12 1c   gp.(..neighbor..
0140  0a 0c 70 65 65 72 2d 61 64 64 72 65 73 73 12 0c   ..peer-address..
0150  31 39 32 2e 31 36 38 2e 31 30 2e 33 1a 0f 0a 0d   192.168.10.3....
0160  73 65 73 73 69 6f 6e 2d 73 74 61 74 65 1a 0f 5a   session-state..Z
0170  0d 22 65 73 74 61 62 6c 69 73 68 65 64 22 00 00   ."established"..
0180  0f 01 05 00 00 00 01 40 0b 67 72 70 63 2d 73 74   .......@.grpc-st
0190  61 74 75 73 01 30 00 00 04 08 00 00 00 00 00 00   atus.0..........
01a0  00 00 7d                                          ..}
```

Now, let's make it a secure connection. 

**Post TLS Test**

To verify that the new TLS profile is working, let's repeat the previous section to do a gNMI Get of the BGP peer session state.

Start your `tshark` capture for port `57400`:

```bash
sudo tshark -ni any -f "src leaf1 and port 57400" -P -x
```

Open another session on your host VM and run the below command to get the BGP peer session state. This time we will the secure gRPC server listening on port `57400`.

```bash
gnmic -a leaf1:57400 -u admin -p admin --skip-verify get --path "/network-instance[name=default]/protocols/bgp/neighbor[peer-address=192.168.10.3]/session-state" -e json_ietf
```

Expected output:

```bash
[
  {
    "source": "leaf1:57400",
    "timestamp": 1792200543314925230,
    "time": "2026-10-16T21:29:03.31492523-04:00",
    "updates": [
      {
        "Path": "srl_nokia-network-instance:network-instance[name=default]/protocols/srl_nokia-bgp:bgp/neighbor[peer-address=192.168.10.3]/session-state",
        "values": {
          "srl_nokia-network-instance:network-instance/protocols/srl_nokia-bgp:bgp/neighbor/session-state": "established"
        }
      }
    ]
  }
]
```

Now stop the `tshark` packet capture using `CTRL+c` and check the packets captured by `tshark`. You will not see the plain text data anymore as all communication is now encrypted by TLS.

Sample output:

```bash
   25 0.254211708 2001:172:20:20::2 → 2001:172:20:20::1 TLSv1.3 441 Application Data

0000  00 03 00 01 00 06 1a 81 04 00 00 00 00 00 86 dd   ................
0010  68 88 2c 71 01 81 06 40 20 01 01 72 00 20 00 20   h.,q...@ ..r. . 
0020  00 00 00 00 00 00 00 02 20 01 01 72 00 20 00 20   ........ ..r. . 
0030  00 00 00 00 00 00 00 01 e0 38 93 62 e6 9f 8c 33   .........8.b...3
0040  ef ab 55 70 80 18 00 48 be 4f 00 00 01 01 08 0a   ..Up...H.O......
0050  2f 38 83 ba 9d fc d1 c9 17 03 03 01 5c 02 e4 bb   /8..........\...
0060  51 84 ce b9 64 2e 2f 13 12 60 88 fd 1e bc 23 26   Q...d./..`....#&
0070  d4 29 2d 40 72 1a bb d2 66 7a 18 dd 4a 36 0b e5   .)-@r...fz..J6..
0080  8e 6c 5e 8a 60 07 cd e4 11 9f a6 5f e1 fd 1d 1d   .l^.`......_....
0090  20 9b f4 1a 49 b2 ba ee 1c 45 b6 77 9a c4 c6 17    ...I....E.w....
00a0  75 e6 c6 de 99 7c 8b b9 33 45 47 5a 22 4c da 17   u....|..3EGZ"L..
00b0  03 45 f1 6d 6e 5a 66 d1 f9 39 bb 1b 7a d6 d6 13   .E.mnZf..9..z...
00c0  34 70 7d 26 7f 60 aa 9a 5c 77 5b 69 72 40 47 b7   4p}&.`..\w[ir@G.
00d0  f9 d4 96 b6 9f e2 aa c5 69 33 3f 6f 84 d8 14 02   ........i3?o....
00e0  7b 93 08 8e 3a a8 df 1a e9 1b aa df 61 7d 3c ce   {...:.......a}<.
00f0  50 cd d6 ed 06 bf cc 90 a0 9d b6 5c 3c ae fb 62   P..........\<..b
0100  2f 3f 05 61 8c c2 c8 de 2c 5b 5a f9 b4 51 ad 2e   /?.a....,[Z..Q..
0110  b5 c1 77 b8 7d 08 cd d4 a0 8a b5 f1 10 d8 88 45   ..w.}..........E
0120  d8 16 83 3c 4a fe 36 3d 46 75 5c 82 52 d2 0f 5d   ...<J.6=Fu\.R..]
0130  eb 24 f9 7e 0d 87 d2 2f 2b 80 1e 5a a2 8f 50 02   .$.~.../+..Z..P.
0140  2e 54 68 5c 09 c2 14 77 32 48 8f 86 3d 37 43 cf   .Th\...w2H..=7C.
0150  03 ad ae a4 66 4b e6 d8 00 f6 5c 13 28 fe ee 57   ....fK....\.(..W
0160  04 1a d1 34 09 c0 b6 b1 e1 6e 1e f7 3b 9c d1 5c   ...4.....n..;..\
0170  4e 50 02 b8 67 88 d7 da 60 da 3e 97 31 af e0 f0   NP..g...`.>.1...
0180  f6 bc d2 41 3e ef 6b 82 fe da 3a 15 77 97 4e 9b   ...A>.k...:.w.N.
0190  54 89 93 11 33 00 0e ce 15 f8 9c ab 76 4b 70 18   T...3.......vKp.
01a0  1c bb 96 53 3b 88 ad c9 11 3b ff 18 fc 69 03 eb   ...S;....;...i..
01b0  8b c8 88 f6 a3 af e9 28 17                        .......(.
```

### gRPC Access Control

In an AI cluster, streaming telemetry is a must have and gRPC gNMI is used for this purpose.

Typical deployments enable gNMI on the switch (secure or insecure) and start streaming the stats to a collector.

But this opens an attack surface using gNMI Set RPC.

Let's test this.

On your host VM, start a secure streaming telemetry session from `leaf1` for BGP peer state and leave it running.

```bash
gnmic -a leaf1:57400 -u admin -p admin sub --skip-verify --stream-mode sample --path "/network-instance[name=default]/protocols/bgp/neighbor[peer-address=192.168.10.3]/session-state" -e json_ietf
```

Expected output:

```bash
{
  "source": "leaf1:57400",
  "subscription-name": "default-1792201901",
  "timestamp": 1792201901292975345,
  "time": "2026-10-16T21:51:41.292975345-04:00",
  "updates": [
    {
      "Path": "srl_nokia-network-instance:network-instance[name=default]/protocols/srl_nokia-bgp:bgp/neighbor[peer-address=192.168.10.3]",
      "values": {
        "srl_nokia-network-instance:network-instance/protocols/srl_nokia-bgp:bgp/neighbor": {
          "session-state": "established"
        }
      }
    }
  ]
}
{
  "sync-response": true
}
```

On another session of your host VM, run the below gNMI Set command:

```bash
gnmic -a leaf1:57400 -u admin -p admin set --update-path "/network-instance[name=*]/protocols/bgp/neighbor[peer-address=*]/admin-state" --update-value disable --skip-verify
```

Expected output:

```bash
{
  "source": "leaf1:57400",
  "timestamp": 1792202028653240120,
  "time": "2026-10-16T21:53:48.65324012-04:00",
  "results": [
    {
      "operation": "UPDATE",
      "path": "network-instance[name=*]/protocols/bgp/neighbor[peer-address=*]/admin-state"
    }
  ]
}
```

Check your streaming telemetry session for the BGP peer state. It has now changed from `Established` to `Idle`. This was because gNMI Set was open for the user that was used for streaming telemetry.

```bash
{
  "source": "leaf1:57400",
  "subscription-name": "default-1792201920",
  "timestamp": 1792202036479782622,
  "time": "2026-10-16T21:53:56.479782622-04:00",
  "updates": [
    {
      "Path": "srl_nokia-network-instance:network-instance[name=default]/protocols/srl_nokia-bgp:bgp/neighbor[peer-address=192.168.10.3]",
      "values": {
        "srl_nokia-network-instance:network-instance/protocols/srl_nokia-bgp:bgp/neighbor": {
          "session-state": "idle"
        }
      }
    }
  ]
}
```

There are multiple way to secure this.

Before we move forward, bring UP the BGP peers.

```bash
gnmic -a leaf1:57400 -u admin -p admin set --update-path "/network-instance[name=*]/protocols/bgp/neighbor[peer-address=*]/admin-state" --update-value enable --skip-verify
```

**User Access Control**

Users (created for stats collectors) should only be enabled to access select services.

This can be achieved using Access Control mechanisms on either the switch or remote authentication servers like RADIUS or TACPLUS.

Example on `leaf1`:

Create a user role and assign it to a user:

```bash
set / system aaa authorization role collector-role services [ gnmi ]
set / system configuration role collector-role rule / action read
set / system aaa authentication user collector-user password collector
set / system aaa authentication user collector-user role [ collector-role ]
```

Start a secure streaming telemetry session using this user:

```bash
gnmic -a leaf1:57400 -u collector-user -p collector sub --stream-mode sample --path "/network-instance[name=default]/protocols/bgp/neighbor[peer-address=192.168.10.3]/session-state" -e json_ietf --skip-verify
```

Now try the gNMI command to disable BGP peer using this user and password:

```bash
gnmic -a leaf1:57400 -u collector-user -p collector set --update-path "/network-instance[name=*]/protocols/bgp/neighbor[peer-address=*]/admin-state" --update-value disable --skip-verify
```

Expected output:

```bash
target "leaf1:57400" set request failed: target "leaf1:57400" SetRequest failed: rpc error: code = PermissionDenied desc = Error in /network-instance[name=default]/protocols/bgp/neighbor[peer-address=2.2.2.2]/admin-state:
    No authorization to execute command: 'set / network-instance default protocols bgp neighbor 2.2.2.2 admin-state disable'
Error: one or more requests failed
```

**Enabling Select gRPC Services only**

It is possible to enable select gRPC service RPCs while defining a gRPC server. This allows for the creation of a separate gRPC server that will interact with an external device only for those select gRPC services.

For example, we do not want the telemetry collector to push a corrupted image to our switch using `gNOI` services.

So let's create a gRPC server instance on `leaf` that allows only gNMI services and uses a non-default port `57411`.

```bash
set / system grpc-server collector-gnmi admin-state enable
set / system grpc-server collector-gnmi rate-limit 65000
set / system grpc-server collector-gnmi network-instance mgmt
set / system grpc-server collector-gnmi port 57411
set / system grpc-server collector-gnmi services [ gnmi ]
set / system grpc-server collector-gnmi unix-socket admin-state enable
```

After committing this config, let's use this new gRPC server for a streaming telemetry session.

```bash
gnmic -a leaf1:57411 -u collector-user -p collector sub --stream-mode sample --path "/network-instance[name=default]/protocols/bgp/neighbor[peer-address=192.168.10.3]/session-state" -e json_ietf --insecure
```

Expected output:

```bash
{
  "source": "leaf1:57411",
  "subscription-name": "default-1792347350",
  "timestamp": 1792347350107945150,
  "time": "2026-10-18T14:15:50.10794515-04:00",
  "updates": [
    {
      "Path": "srl_nokia-network-instance:network-instance[name=default]/protocols/srl_nokia-bgp:bgp/neighbor[peer-address=192.168.10.3]",
      "values": {
        "srl_nokia-network-instance:network-instance/protocols/srl_nokia-bgp:bgp/neighbor": {
          "session-state": "established"
        }
      }
    }
  ]
}
{
  "sync-response": true
}
```

**Enabling Select gRPC Service RPCs only**

It is also possible to restrict certain gRPC RPCs within a gRPC service.

This can be achieved while creating the gRPC server instance.

Here's an example of configuring this in the gRPC server instance. In this example, rather than allowing all gRPC gNOI services, we are only allowing the use of the `gNOI File` service. This service can be used for config backups.

```bash
set / system grpc-server collector-gnmi admin-state enable
set / system grpc-server collector-gnmi rate-limit 65000
set / system grpc-server collector-gnmi network-instance mgmt
set / system grpc-server collector-gnmi port 57411
set / system grpc-server collector-gnmi services [ gnmi gnoi.file ]
set / system grpc-server collector-gnmi unix-socket admin-state enable
```

Let's check if we can list a file using the `file stat` RPC using this gRPC server.

```bash
gnoic -a leaf1:57411 -u gnoic1 -p gnoic1 --insecure file stat --path /etc/opt/srlinux/config.json
```

Expected output:

```bash
+-------------+------------------------------+---------------------------+------------+------------+--------+
| Target Name |             Path             |       LastModified        |    Perm    |   Umask    |  Size  |
+-------------+------------------------------+---------------------------+------------+------------+--------+
| leaf1:57411 | /etc/opt/srlinux/config.json | 2026-10-16T20:19:50-04:00 | -rw-rw-r-- | -----w--w- | 134669 |
```

Also, verify if this gRPC server allows other gNOI services like `System`.

```bash
gnoic -a leaf1:57411 -u admin -p admin --insecure system time
```

Expected output:

```bash
ERRO[0000] "leaf1:57411" System Time failed: rpc error: code = Unimplemented desc =  
```

**Enabling Select gRPC Service APIs only**

In the previous example, we saw how we can allow `gNOI File` RPCs and deny any other RPCs.

It is also possible to allow specific RPC calls within `gNOI File` service and deny others.

This can be achieved using the `gRPC gNSI Authz` service and RPC.

Authz RPC can be used to configure an Authorization policy for access to gRPC services.

We will be using gNSI to configure an authorization policy on leaf1 that will prevent the user called `gnoic1` (used for config file backup) from writing files on the switch.

Let's start by verifying that the user `gnoic1` has access to list on leaf1.

```bash
gnoic -a leaf1:57411 -u gnoic1 -p gnoic1 --insecure file stat --path /etc/opt/srlinux/config.json
```

Expected output:

```bash
+-------------+------------------------------+---------------------------+------------+------------+--------+
| Target Name |             Path             |       LastModified        |    Perm    |   Umask    |  Size  |
+-------------+------------------------------+---------------------------+------------+------------+--------+
| leaf1:57411 | /etc/opt/srlinux/config.json | 2026-10-16T20:19:50-04:00 | -rw-rw-r-- | -----w--w- | 134669 |
```

Create a file on your host VM:

```bash
echo "this is a corrupted image file" > corrupt.img
```

Now transfer this file to the switch.

```bash
gnoic -a leaf1:57411 -u gnoic1 -p gnoic1 --insecure file put --file corrupt.img --dst /var/log/srlinux/corrupt.img
```

Expected output for Put file:

```bash
INFO[0000] "leaf1:57400" sending file="corrupt.img" hash 
INFO[0000] "leaf1:57400" file "corrupt.img" written successfully 
```

Verify on leaf1 that the file transferred exists.

```bash
gnoic -a leaf1:57411 -u gnoic1 -p gnoic1 --insecure file stat --path /var/log/srlinux/corrupt.img
```

Expected output:

```
+-------------+------------------------------+---------------------------+------------+------------+------+
| Target Name |             Path             |       LastModified        |    Perm    |   Umask    | Size |
+-------------+------------------------------+---------------------------+------------+------------+------+
| leaf1:57401 | /var/log/srlinux/corrupt.img | 2025-05-20T03:06:16-04:00 | -rwxrwxrwx | -----w--w- | 31   |
+-------------+------------------------------+---------------------------+------------+------------+------+
```

**4.1.4 Delete the transferred file from the device using `File Remove` RPC.**

```bash
gnoic -a leaf1:57411 -u gnoic1 -p gnoic1 --insecure file remove --path /var/log/srlinux/corrupt.img
```

Expected output:

```bash
INFO[0000] "leaf1:57400" file "/var/log/srlinux/corrupt.img" removed successfully 
```

At this time, the user `gnoic1` has permissions to transfer a file over to leaf1.

Let's block that by pushing an authorization policy using gNSI Authz service.

This is the Authz policy payload that we will push. This gives access to gNOI File Get & Stat to user `gnoic1` and will deny gNOI File Put for this user.

<details>
<summary>Authz Payload</summary>
<br>
<pre>
{
  "name": "Ext-clients",
  "allow_rules": [
    {
      "name": "backup-access",
      "source": {
        "principals": [
          "gnoic1", 
		  "gnoi-clients"
		  
        ]
      },
      "request": {
        "paths": [
          "/gnoi.file.File/Get",
          "/gnoi.file.File/Stat"
        ]
      }
    }
  ],
  "deny_rules": [
    {
      "name": "backup-access",
      "source": {
        "principals": [
          "gnoic1", 
		  "gnoi-clients"
        ]
      },
      "request": {
        "paths": [
          "/gnoi.file.File/Put"
        ]
      }
    }
  ]
}
</pre>
</details>

**4.1.5 Let's push the policy using gNSIc.**

```bash
gnsic -a leaf1 -u admin -p admin --skip-verify authz rotate --policy "{\"name\":\"Ext-clients\",\"allow_rules\":[{\"name\":\"backup-access\",\"source\":{\"principals\":[\"gnoic1\",\"gnoi-clients\"]},\"request\":{\"paths\":[\"/gnoi.file.File/Get\",\"/gnoi.file.File/Stat\"]}}],\"deny_rules\":[{\"name\":\"backup-access\",\"source\":{\"principals\":[\"gnoic1\",\"gnoi-clients\"]},\"request\":{\"paths\":[\"/gnoi.file.File/Put\"]}}]}"
```

Expected output:

```
INFO[0000] targets: map[leaf1:57400:0xc0002c6040]       
INFO[0000] "leaf1:57400": got UploadResponse            
INFO[0001] "leaf1:57400": sending finalize request      
INFO[0001] "leaf1:57400": closing stream 
```

Verify that the authz policy was applied on the system. We will use gNMI Get RPC for this purpose.

```bash
gnmic -a leaf1 -u admin -p admin --skip-verify get --path "/system/aaa/authorization/authz-policy" --encoding json_ietf
```

Expected output:

```bash
[
  {
    "source": "leaf1",
    "timestamp": 1747726096665629489,
    "time": "2025-05-20T03:28:16.665629489-04:00",
    "updates": [
      {
        "Path": "srl_nokia-system:system/srl_nokia-aaa:aaa/authorization/srl_nokia-gnsi-authz:authz-policy",
        "values": {
          "srl_nokia-system:system/srl_nokia-aaa:aaa/authorization/srl_nokia-gnsi-authz:authz-policy": {
            "counters": {
              "rpc": [
                {
                  "access-accepts": "4",
                  "access-rejects": "0",
                  "last-access-accept": "2025-05-20T07:28:16.661Z",
                  "name": "/gnmi.gNMI/Get"
                },
                {
                  "access-accepts": "1",
                  "access-rejects": "0",
                  "last-access-accept": "2025-05-20T07:23:00.183Z",
                  "name": "/gnoi.file.File/Get"
                },
                {
                  "access-accepts": "0",
                  "access-rejects": "2",
                  "last-access-reject": "2025-05-20T07:23:35.004Z",
                  "name": "/gnoi.file.File/Put"
                },
                {
                  "access-accepts": "1",
                  "access-rejects": "1",
                  "last-access-accept": "2025-05-20T07:25:09.619Z",
                  "last-access-reject": "2025-05-20T07:24:36.303Z",
                  "name": "/gnoi.file.File/Remove"
                },
                {
                  "access-accepts": "7",
                  "access-rejects": "0",
                  "last-access-accept": "2025-05-20T07:25:09.538Z",
                  "name": "/gnoi.file.File/Stat"
                },
                {
                  "access-accepts": "5",
                  "access-rejects": "0",
                  "last-access-accept": "2025-05-20T07:28:13.277Z",
                  "name": "/gnsi.authz.v1.Authz/Rotate"
                }
              ]
            },
            "created-on": "2179-09-19T07:42:44.972Z",
            "policy": "{\"name\":\"Ext-clients\",\"allow_rules\":[{\"name\":\"backup-access\",\"source\":{\"principals\":[\"gnoic1\",\"gnoi-clients\"]},\"request\":{\"paths\":[\"/gnoi.file.File/Get\",\"/gnoi.file.File/Stat\"]}}],\"deny_rules\":[{\"name\":\"backup-access\",\"source\":{\"principals\":[\"gnoic1\",\"gnoi-clients\"]},\"request\":{\"paths\":[\"/gnoi.file.File/Put\"]}}]}",
            "version": ""
          }
        }
      }
    ]
  }
]
```

Now, test the list and put file operations again.

Refer to the previous steps for the command.

Put operation will be denied with the below output.

```bash
INFO[0000] "leaf1:57400" sending file="corrupt.img" hash 
ERRO[0000] "leaf1:57400" File Put failed: rpc error: code = PermissionDenied desc = User 'gnoic1' is not authorized to use rpc '/gnoi.file.File/Put' 
Error: there was 1 error(s)
```

### AAA Access Control

User access permissions can be controlled locally using local authorization profiles.

For example, a NOC user should not have `write` permissions to the `system` context.

```bash
set / system aaa authorization role noc-role services [ cli ]
set / system configuration role noc-role rule / action write
set / system configuration role noc-role rule system action read

set / system configuration role noc-role rule system action read
set / system aaa authentication user noc-user password noc-user
set / system aaa authentication user noc-user role [ noc-role ] first
set / system aaa authorization role noc-role services [ cli ]
```

Login as this user on `leaf1` and try to create a new user.

```bash
set / system aaa authentication user test password test
```

Expected output:

```bash
Error: No authorization to execute command: 'set / system aaa authentication user test'
```

#### SNMP

Let's user SNMPv2 to get the uptime of `leaf1`.

SNMPv2 config on `leaf1`:

```bash
set / system snmp access-group SNMPv2-RO-Community security-level no-auth-no-priv
set / system snmp access-group SNMPv2-RO-Community community-entry RO-Community community private
set / system snmp network-instance mgmt admin-state enable
```

Start a packet capture using `tshark`.

```bash
sudo tshark -ni any -f "src leaf1 and port 161" -P -x
```

From another session on your host, run:

```bash
snmpget -v2c -c private leaf1  1.3.6.1.2.1.1.3.0
```

Expected output:

```bash
iso.3.6.1.2.1.1.3.0 = Timeticks: (15582600) 1 day, 19:17:06.00
```

Stop the packet capture using `CTRL+c`. We can see the community value in plain text in the capture.

```bash
    1 0.000000000  172.20.20.2 → 172.20.20.1  SNMP 92 get-response 1.3.6.1.2.1.1.3.0

0000  00 03 00 01 00 06 1a 81 04 00 00 00 00 00 08 00   ................
0010  45 88 00 4c 05 fc 40 00 40 11 b3 f1 ac 14 14 02   E..L..@.@.......
0020  ac 14 14 01 00 a1 de f0 00 38 2f 97 30 2e 02 01   .........8/.0...
0030  01 04 07 70 72 69 76 61 74 65 a2 20 02 04 56 5d   ...private. ..V]
0040  85 a6 02 01 00 02 01 00 30 12 30 10 06 08 2b 06   ........0.0...+.
0050  01 02 01 01 03 00 43 04 00 f2 7b 00               ......C...{.
```

This is where SNMPv3 helps.

SNMPv3 configuration:

```bash
set / system snmp access-group snmpv3-ag1 admin-state enable
set / system snmp access-group snmpv3-ag1 security-level auth-priv
set / system snmp access-group snmpv3-ag1 security-entry entry1 user snmpv3-user
set / system snmp access-group snmpv3-ag1 security-entry entry1 authentication protocol hmac-sha2-256
set / system snmp access-group snmpv3-ag1 security-entry entry1 authentication password snmpv3-user
set / system snmp access-group snmpv3-ag1 security-entry entry1 privacy protocol cfb128-aes-128
set / system snmp access-group snmpv3-ag1 security-entry entry1 privacy password snmpv3-user
set / system snmp network-instance mgmt admin-state enable
```

After committing the config, let's test using SNMPv3.

Start the packet capture using `tshark`.

```bash
sudo tshark -ni any -f "src leaf1 and port 161" -P -x
```

From another session on your host, run:

```bash
snmpget -v 3 -u snmpv3-user -l authPriv -a SHA-256 -A snmpv3-user -x AES -X snmpv3-user leaf1 1.3.6.1.2.1.1.3.0
```

Stop the capture and check the packets. We can see that all the data is now encrypted.

```bash
    1 0.000000000  172.20.20.2 → 172.20.20.1  SNMP 177 report 1.3.6.1.6.3.15.1.1.4.0

0000  00 03 00 01 00 06 1a 81 04 00 00 00 00 00 08 00   ................
0010  45 88 00 a1 f4 13 40 00 40 11 c5 84 ac 14 14 02   E.....@.@.......
0020  ac 14 14 01 00 a1 83 48 00 8d 20 b9 30 81 82 02   .......H.. .0...
0030  01 03 30 11 02 04 5a cf fa 6f 02 03 00 ff e3 04   ..0...Z..o......
0040  01 00 02 01 03 04 2a 30 28 04 19 80 00 19 7f 05   ......*0(.......
0050  e2 fa 01 6b 0f 6b e5 e1 cf 0d ea 4e 97 5e 27 24   ...k.k.....N.^'$
0060  1c e3 6b c4 02 01 03 02 02 00 dd 04 00 04 00 04   ..k.............
0070  00 30 3e 04 19 80 00 19 7f 05 e2 fa 01 6b 0f 6b   .0>..........k.k
0080  e5 e1 cf 0d ea 4e 97 5e 27 24 1c e3 6b c4 04 00   .....N.^'$..k...
0090  a8 1f 02 04 2b 73 9f 13 02 01 00 02 01 00 30 11   ....+s........0.
00a0  30 0f 06 0a 2b 06 01 06 03 0f 01 01 04 00 41 01   0...+.........A.
00b0  02   
```

### Protocol Authentication

Our current deployment does not have BGP authentication enabled.

Before we enabled authentication, shutdown the BGP peering between `leaf1` and `spine`.

On `leaf1`:

```bash
network-instance default protocols bgp neighbor 192.168.10.3 admin-state disable
```

Enable BGP authentication on `leaf1` towards `spine` neighbor.

```bash
insert / system authentication keychain auth-keychain type tcp-md5
insert / system authentication keychain auth-keychain key 0 algorithm hmac-sha-256
insert / system authentication keychain auth-keychain key 0 authentication-key mib26
insert / network-instance default protocols bgp neighbor 192.168.10.3 authentication keychain auth-keychain
```

Repeat the same on `spine` towards `leaf1` neighbor.

```bash
insert / system authentication keychain auth-keychain admin-state enable
insert / system authentication keychain auth-keychain type tcp-md5
insert / system authentication keychain auth-keychain key 0 algorithm hmac-sha-256
insert / system authentication keychain auth-keychain key 0 authentication-key mib26
insert / network-instance default protocols bgp neighbor 192.168.10.2 authentication keychain auth-keychain
```

Bring up the BGP peering session on `leaf1`:

```bash
network-instance default protocols bgp neighbor 192.168.10.3 admin-state enable
```

Verify the state of the peering session and confirm that authentication is enabled.

On `leaf1`:

```bash
info from state flat network-instance default protocols bgp neighbor 192.168.10.3 | grep -E 'session-state|authentication'
```

Expected output:

```bash
/ network-instance default protocols bgp neighbor 192.168.10.3 session-state established
/ network-instance default protocols bgp neighbor 192.168.10.3 authentication keychain auth-keychain
/ network-instance default protocols bgp neighbor 192.168.10.3 authentication transmit-active false
```

### Dyanmic Peering

When dynamic peering is used between inter-switch links, it is a best practise to allow only required AS numbers and also enable dynamic peering only on required interfaces and not all interfaces in general.

Here's a sample config for dynamic peering on `leaf1` towards `spine`.

```bash
set / interface ethernet-1/1 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv6 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv6 router-advertisement router-role admin-state enable
set /network-instance default interface ethernet-1/1.0
set / network-instance default protocols bgp group ebgp
set / network-instance default protocols bgp dynamic-neighbors interface ethernet-1/1.0 peer-group spines
set / network-instance default protocols bgp dynamic-neighbors interface ethernet-1/1.0 allowed-peer-as [ 64500 ]
```

### ICMP Storm Control

Ethernet storm control can be enabled on interface to rate limit BUM traffic.

In this example, we are enabling it on the client facing interface of `leaf1`.

```bash
set / interface ethernet-1/10 ethernet storm-control units kbps
set / interface ethernet-1/10 ethernet storm-control broadcast-rate 100
set / interface ethernet-1/10 ethernet storm-control multicast-rate 10
set / interface ethernet-1/10 ethernet storm-control unknown-unicast-rate 10
set / interface ethernet-1/10 ethernet storm-control rising-threshold-action trigger-event
```

### Microsegmentation

Micro-segmentation is a security capability for data centers and enterprises intended to prevent lateral movement of security threats within the network. It divides networks into smaller, isolated zones (known as micro-segments) and establishes rules to restrict traffic between them. It is much more scalable than traditional ACL based filtering.

Micro-segmentation uses group-based policies (GBPs) to control access between segments. A GBP defines a set of groups, with a tag value assigned to each group. The GBP associates the groups with a subinterface or IP prefix and specifies an ACL to enforce access rules based on group tagging.

In our lab, we allow `client1` to communicate to both `client3` and `client4`.

Login to `client1`:

```bash
docker exec -it client1 bash
```

Ping `client3` and `client4`:

```bash
ping -c 3 172.16.10.60
ping -c 3 10.90.1.1
```

Both pings are successful.

Now, using Group Based Policies (GBP) we will restrict `client1` from talking to `client3`.

Configure GBP on both `leaf1` and `leaf2`.

```bash
set / platform resource-management group-based-policy mac-lookup true
set / network-instance mac-vrf-1 group-based-policy group client1 tag-value 10
set / network-instance mac-vrf-1 group-based-policy group client2 tag-value 20
set / network-instance mac-vrf-1 group-based-policy group client3 tag-value 30
set / network-instance mac-vrf-1 group-based-policy group client4 tag-value 40
set / network-instance mac-vrf-1 group-based-policy interface ethernet-1/10.0 group [ clieset
set / network-instance mac-vrf-1 protocols bgp-evpn bgp-instance 1 group-based-policy add-community true
set / network-instance mac-vrf-1 protocols bgp-evpn bgp-instance 1 group-based-policy install-from-communityset
set / network-instance mac-vrf-1 group-based-policy acl entry 10 match source-group [ client1 ]
set / network-instance mac-vrf-1 group-based-policy acl entry 10 match destination-group [ client1 ]
set / network-instance mac-vrf-1 group-based-policy acl entry 10 action drop
```

Check the pings again from `client1`.

Login to `client1`:

```bash
docker exec -it client1 bash
```

Ping `client3` and `client4`:

```bash
ping -c 3 172.16.10.60
ping -c 3 10.90.1.1
```

We see that `client1` is no longer able to reach `client3` while `client4` is still reachable.

This is because the GBP policy ACL blocks this communication.

### Disk Encryption

Disk encryption can be enabled on real hardware using the below command. This is not supported in the SRLinux docker image.

```bash
tools platform trust disk-encryption control A activate
```

### ACL

ACLs can be configured for control or data plane protection.

ACLs were configured when this lab was deployed.

A sample ACL for SNMP is shown below.

```bash
set / acl acl-filter cpm type ipv4 entry 180 description "Accept incoming SNMP GET/GETNEXT messages from servers"
set / acl acl-filter cpm type ipv4 entry 180 match ipv4 protocol udp
set / acl acl-filter cpm type ipv4 entry 180 match transport destination-port operator eq
set / acl acl-filter cpm type ipv4 entry 180 match transport destination-port value 161
set / acl acl-filter cpm type ipv4 entry 180 action accept
```

