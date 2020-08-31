# saos_netconf_bfd

## ISIS Overview

[Reference](https://en.wikipedia.org/wiki/IS-IS)

## BFD Overview

Bidirectional Forwarding Detection (BFD) is a network protocol that is used to detect faults between two forwarding engines connected by a link. It provides low-overhead detection of faults even on physical media that doesn't support failure detection of any kind, such as Ethernet, virtual circuits, tunnels and MPLS Label Switched Paths.

BFD establishes a session between two endpoints over a particular link. If more than one link exists between two systems, multiple BFD sessions may be established to monitor each one of them. The session is established with a three-way handshake, and is torn down the same way. Authentication may be enabled on the session. A choice of simple password, MD5 or SHA1 authentication is available.

BFD does not have a discovery mechanism; sessions must be explicitly configured between endpoints. BFD may be used on many different underlying transport mechanisms and layers, and operates independently of all of these. Therefore, it needs to be encapsulated by whatever transport it uses. For example, monitoring MPLS LSPs involves piggybacking session establishment on LSP-Ping packets. Protocols that support some form of adjacency setup, such as OSPF, IS-IS, BGP or RIP may also be used to bootstrap a BFD session. These protocols may then use BFD to receive faster notification of failing links than would normally be possible using the protocol's own keepalive mechanism.

A session may operate in one of two modes: asynchronous mode and demand mode. In asynchronous mode, both endpoints periodically send Hello packets to each other. If a number of those packets are not received, the session is considered down.
In demand mode, no Hello packets are exchanged after the session is established; it is assumed that the endpoints have another way to verify connectivity to each other, perhaps on the underlying physical layer. However, either host may still send Hello packets if needed.
Regardless of which mode is in use, either endpoint may also initiate an Echo function. When this function is active, a stream of Echo packets is sent, and the other endpoint then sends these back to the sender via its forwarding plane. This is used to test the forwarding path on the remote system.

## Ciena BFD implementation

The BFD protocol is used for monitoring the data-path over various transport mechanisms such as IP, VCCV, MPLS LSP etc. The encapsulation of the BFD payload depends on the underlying transport path. BFD sessions are created on both end points of the path to be monitored & BFD CC packets are transmitted at configured intervals. BFD also runs using a 4-state FSM machine according to RFC 5880. During the initial packet exchange process, the BFD protocol executes a 3-way handshake with its peer and the session parameters are negotiated and the session state is established. Thus, the actual time interval used for transmission of a BFD session is the greater of the configured local transmit interval and the configured remote receive interval. When the BFD session misses the reception of 3 (multiplier value) consecutive packets, it raises a fault and notifies its client protocols, thus helping the (usually) slower protocols withdraw routes and converge faster.

### ISIS Single-hop IP BFD

* The ISIS Single-hop IP BFD sessions run between PE & P nodes and P & P nodes.
* Upon detection of a fault in the data-path, the faults are notified to the ISIS protocol, which helps in quicker convergence of the IGP routes.
* The propagation of IP BFD faults to ISIS also helps trigger LDP FRR switchovers.
* The ISIS IP-BFD sessions can be created for both ipv4 and ipv6 interfaces attached.


### ISIS Single-hop IP BFD Configuration

1.  Create a Single-hop IP BFD session through BFD CLI: (Source-address is mandatory)

    bfd ip-sh sessions session <interface> <dest-addr> source-addr <source-addr>

    bfd ip-sh sessions session intf11 11.11.11.2 source-addr 11.11.11.1

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <rpc message-id="1220" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <edit-config>
        <target>
            <running/>
        </target>
        <config>
            <bfd xmlns="http://ciena.com/ns/yang/ciena-bfd">
            <ip-sh xmlns="http://ciena.com/ns/yang/ciena-bfd-ip-sh">
                <sessions>
                <session  xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0" nc:operation="merge">
                    <interface>intf1</interface>
                    <dest-addr>11.11.11.2</dest-addr>
                    <source-addr>11.11.11.1</source-addr>
                </session>
                </sessions>
            </ip-sh>
            </bfd>
        </config>
        </edit-config>
    </rpc>
    ```

2.  Register ISIS as a client for this BFD session:
    isis instance <isis-instance-name> interfaces interface <interface-name> bfd enable <true>

    isis instance CORE interfaces interface intf11 bfd enable true

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <rpc message-id="1230" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <edit-config>
        <target>
            <running/>
        </target>
        <config>
            <isis xmlns="http://ciena.com/ns/yang/ciena-isis">
            <instance>
                <tag>CORE</tag>
                <interfaces>
                <interface>
                    <name>intf1</name>
                    <bfd  xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0" nc:operation="merge">
                    <enable>true</enable>
                    </bfd>
                </interface>
                </interfaces>
            </instance>
            </isis>
        </config>
        </edit-config>
    ```

## Results

Session scaling results (example)

| Interval | Docker Based | Raspberry Pi |
| -------- | ------------ | ------------ |
| 10ms     | 0            | 5            |
| 20ms     | 0            | 30           |
| 50ms     | 30           | 70           |
| 100ms    | 60           | 100          |
| 300ms    | 100          | 100          |
| 1sec     | 100          | 100          |
| 10sec    | 100          | 100          |

## Reference

BFD RFCs

* https://tools.ietf.org/html/rfc5880
* https://tools.ietf.org/html/rfc5881
* https://tools.ietf.org/html/rfc5883
* https://tools.ietf.org/html/rfc5884
* https://datatracker.ietf.org/doc/draft-ietf-bfd-yang/
