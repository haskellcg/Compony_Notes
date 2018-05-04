# Transparent Interconnection of Lots of Links (TRILL) Use of IS-IS
# Abstract
  TODO

# 1. Introduction
  The IETF Transparent Interconnection of Lots of Links (TRILL) prrotocol [RFC6325] provides transparent forwarding in multi-hop networks with arbitrary topology and link technologies using a header witth a hop count and link-state routing. TRILL provides optimal pair-wise forwarding without configuration, safe forwarding even during periods of temporary loops, and support for multipathing of both unicast and multicast traffic. Intermediate System (ISs) implementing TRILL are called Routing Bridges (RBridges) or TRILL Switches.

  This document, in conjunction with [RFC6165], specifics the data formats and code points for the IS-IS [RFC1195] extensions to support TRILL. These data formats and code points may also be used by technologies other than TRILL.

  This document obsoletes [RFC6326], which generally corresponded to the base TRILL protocol [RFC6325]. There has been substantial development of TRILL  since the publication of those documents. The main changes from [RFC6326] are summarized below, and a full list is given in section 7.
  1. Added multicast group announcements by IPv4 and IPv6 addres
  1. Added facilities for Announcing capapbilities supported
  1. Added a tree affinity sub-TLV whereby ISs can request distribution tree association
  1. Added multi-topology support
  1. Added control-plane support for TRILL Data frame find-gained labels. This support is independent of the data-plane representation
  1. Fixed the verified erratum [Err2869] in [RFC6326]
## 1.1 Convertions Used in This Document
  * BVL - Bit Vector Length
  * BVO - Bit Vector Offset
  * IHH - IS-IS Hello
  * IS - Intermediate System. Forr this document, all relevant intermediate systems are RBridge [RFC6325]
  * MAC - Media Access Control
  * MT - Multi-Topology
  * NLPID - Network Layer Protocol Identifier
  * SNPA - Subnetwork Point of Attachment (MAC address)

# 2. TLV and Sub-TLV Extensions to IS-IS for TRILL
  This section, in conjunction with [RFC6165], specifies the data formats and code points for the TLVs and Sub-TLVs for IS-IS to support the IETF TRILL protocol. Information as to the number of occurrences allowed, such as for a TLV in a PDU or set of PDUs or for a sub-TLV in a TLV, is summarized in section 5.

## 2.1 Group Address TLV
  The Group Address (GADDR) TLV, IS-IS TLV type 142, is cariied in an LSP PDU and carries sub-TLVs that in turn advertise multicast group listeners. The sub-TLVs that advertise Listeners are specified below. The Sub-TLVs under GADDR constitude a new series of sub-TLV types.

  GADDR has the following format:

            +-+-+-+-+-+-+-+-+
            |Type=GADDR-TLV |                  (1 byte)
            +-+-+-+-+-+-+-+-+
            |   Length      |                  (1 byte)
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-...
            |      sub-TLVs...
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-...

  * Type: TLV type, set to GADDR-TLV 142
  * Length: variable depending on the sub-TLVs carried
  * sub-TLVs: The Group Address TLV value consists of sub-TLVs formatted as described in [RFC5305]

### 2.1.1. Group MAC Address Sub-TLV
  The Group MAC Address (GMAC-ADDR) sub-TLV is sub-TLV type number 1 within the GADDR TLV. In TRILL, it is used to advertised multicast listener by MAC address as specified in Section 4.5.5 of [RFC6325]. It has the following format:

            +-+-+-+-+-+-+-+-+
            |Type=GMAC-ADDR |                  (1 byte)
            +-+-+-+-+-+-+-+-+
            |   Length      |                  (1 byte)
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |  RESV |     Topology-ID       |  (2 bytes)
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |  RESV |     VLAN ID           |  (2 bytes)
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |Num Group Recs |                  (1 byte)
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   GROUP RECORDS (1)                           |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   GROUP RECORDS (2)                           |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   .................                           |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   GROUP RECORDS (N)                           |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  where each group record is of the following form with k=6:

            +-+-+-+-+-+-+-+-+
            | Num of Sources|                  (1 byte)
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   Group Address         (k bytes)             |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   Source 1 Address      (k bytes)             |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   Source 2 Address      (k bytes)             |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                    .....                                      |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   Source M Address      (k bytes)             |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  * Type: GADDR sub-TLV type, set to 1 (GMAC-ADDR)
  * Length: 5 + m + k\*n = 5 + m + 6\*n, where m is the number of group records and n is the sum of the number of group and source addresses
  * RESV: Reserved. 4-bit fields that must be sent as zero and ignored on receipt
  * Topology-ID: This field carries a topology ID [RFC5120] or zero if topologies are not in use.
  * VLAN ID: This carries the 12-bit VLAN identifier for all subsequent MAC addresses in this sub-TLV, or the value zero if no VLAN is specified
  * Num Group Recs: A 1-byte unsigned integer that is the number of group records in this sub-TLV
  * Group Records: Each group record carries the number of sources. If this field is zero, it indicates a listener for (\*, G), that is, a listener not restricted by source. It then has a 6-byte (48-bit) multicast MAC address of followed by 6-byte source MAC addresses. If the sources do not fit in a single sub-TLV, the same group address may be repeated with different source addresses in another sub-TLV of another instance of the Group Address TLV.

  The GMAC-ADDR sub-TLV is carried only within a GADDR TLV.

### 2.1.2. Group IPv4 Address Sub-TLV
  The Group IPv4 Address (GIP-ADDR) sub-TLV is IS-IS sub-TLV type 2 within the GADDR TLV. It has the same format as the Group MAC Address sub-TLV described in Section 2.1.1 except that k=4. The field are as follows:
  * Type: sub-TLV type, set to 2 (GIP-ADDR)
  * Length: 5 + m + k\*n = 5 + m + 4\*n, same as above
  * Topology-ID: TODO
  * RESV: TODO
  * VLAN ID: TODO
  * Num Group Recs: TODO
  * Group Records: TODO

  The GIP-ADDR sub-TLV is carried only within a GADDR TLV.

### 2.1.3. Group IPv6 Address Sub-TLV
  The Group IPv6 Address (GIPv6-ADDR) sub-TLV is IS-IS sub-TLV type 3 within the GADDR TLV. It has format as the Group MAC Address sub-TLV described in Section 2.1.1 except that k=16. The fields are as follows:
  * Type: sub-TLV type, set to 3 (GIPv6-ADDR)
  * Length: TODO
  * Topology-ID: TODO
  * RESV: TODO
  * VLAN ID: TODO
  * Number Group Recs: TODO
  * Group Records: TODO
  
  The GIPv6-ADDR sub-TLV is carried only within a GADDR TLV

### 2.1.4. Group Labeled MAC Address Sub-TLV
  The GMAC-ADDR sub-TLV of the Group Address (GADDR) TLV specified in Section 2.1.1 provides for a VLAN ID. The Group Labeled MAC Address sub-TLV, below, extends this to a fine-grained label.


            +-+-+-+-+-+-+-+-+
            |Type=GLMAC-ADDR|                  (1 byte)
            +-+-+-+-+-+-+-+-+
            |   Length      |                  (1 byte)
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |  RESV |     Topology-ID       |  (2 bytes)
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |        Fine-Grained Label                     | (3 bytes)
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |Num Group Recs |                  (1 byte)
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   GROUP RECORDS (1)                           |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   GROUP RECORDS (2)                           |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   .................                           |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   GROUP RECORDS (N)                           |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  where each group record is of the following form with k=6:

            +-+-+-+-+-+-+-+-+
            | Num of Sources|                  (1 byte)
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   Group Address         (k bytes)             |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   Source 1 Address      (k bytes)             |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   Source 2 Address      (k bytes)             |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                    .....                                      |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |                   Source M Address      (k bytes)             |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  * Type: GADDR sub-TLV type, set to 4 (GLMAC-ADDR)
  * Length: 6 + m + k\*n = 6 + m + 6\*n, TODO
  * RESV: TODO
  * Topology-ID: TODO
  * Label: This carried the fine-gained label [RFC7172] identifier for all subsequent MAC addresses in this sub-TLV, or the value zero if no label is specified
  * Num Group Recs: TODO
  * Group Records: TODO

  The GLMAC-ADDR sub-TLV is carried only within a GADDR TLV.

### 2.1.5. Group Labeled IPv4 Address Sub-TLV
  The Group Labeled IPv4 Address (GLIP) Sub-TLV is IS-IS sub-TLV type 5 within the GADDR TLV. It has the same format as the Group Labeled MAC Address sub-TLV described in Section 2.1.4 xecept that k=4, the field are as follows:
  * Type: sub-TLV type, set to 5 (GLIP-ADDR)
  * Length: TODO
  * Topology-ID: TODO
  * RESV: TODO
  * Label: TODO
  * Number Group Recs: TODO
  * Group Records: TODO

  The GLIP-ADDR sub-TLV is carried only within a GADDR TLV.

### 2.1.6. Group Labeled IPv6 Address Sub-TLV
  The Group Labeled IPv6 Address (GLIPv6-ADDR) sub-TLV is IS-IS sub-TLV type 6 within the GADDR TLV. It has the same format as the Group Labeled MAC Address sub-TLV described in Section 2.1.4 except that k=16. The field are as follows:
  * Type: sub-TLV type, set to 6 (GLIPv6-ADDR)
  * Length: TODO
  * Topology-ID: TODO
  * RESV: TODO
  * Label: TODO
  * Num Group Recs: TODO
  * Group Records: TODO

  The GLIPv6-ADDR sub-TLV is carried only within a GADDR TLV.

## 2.2. Multi-Topology-Aware Port Capability Sub-TLVs
  TRILL makes use of the Multi-Topology-Aware Port Capability TLV (MT-Port-Cap-TLV) as specified in [RFC6165]. The following subsection specify the sub-TLVs transported by the MT-Port-Cap-TLV for TRILL.

### 2.2.1. Special VLANs and Flags Sub-TLV
  In TRILL, a Special VLANs and Flags (VLAN-FLAGS) sub-TLV is carried in every IHH PDU. It has the following format:

            +--+--+--+--+--+--+--+--+
            |    Type               |                         (1 byte)
            +--+--+--+--+--+--+--+--+
            |    Length             |                         (1 byte)
            +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
            |    Port ID                                    | (2 bytes)
            +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
            |    Sender Nickname                            | (2 bytes)
            +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
            |AF|AC|VM|BY|     Outer.VLAN                    | (2 bytes)
            +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
            |TR|R |R |R |     Designated-VLAN               | (2 bytes)
            +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

  * Type: sub-TLV type, set to MT-Port-Cap-TLV VLAN-FLAGS sub-TLV 1
  * Length: 8
  * Port ID: An ID for the port on which the enclosing TRILL IIH PDU is being sent as specified in [RFC6325], Section 4.4.2
  * Sender Nickname: If the sending IS is holding any nickname as discussed in [RFC6325], section 3.7, one must be included here. Otherwise, the field is set to zero. This field is to support intellegent end stations that determine the egress IS (RBridge) for unicast data through a directory service or the link and that need a nickname for their first hop to insert as the ingress nickname to correctly format a TRILL Data frame. It is also referenced in connect with the VLAN appointed Sub-TLV and can be used as the egress on one-hop RBridge Channel messages [RFC7178], for example, those use for BFD over TRILL [RFC7175].
  * Outer.VLAN: A copy of the 12-bit outer VLAN ID of the TRILL IIH frame containing this sub-TLV, as specified in [RFC6325], Section 4.4.5
  * Designated-VLAN: The 12-bit ID of the Designated VLAN for the link, as specified in [RFC6325], section 4.2.4.2
  * AF, AC, VM, BY, and TR: These flag bits have the following meaning when set to one, as specified in the listed section of [RFC6325]:

  Bit|Section|Meaning if bit is one
  ---|-------|---------------------
  AF|4.4.2|Originating IS believes it is Appointed Forwarder for the VLAN and port on which the containing IIH PDU was sent
  AC|4.9.1|Originating port configured as an access port (TRILL traffic disable)
  VM|4.4.5|VLAN mapping detected on this link
  BY|4.4.2|Bypass pseudonode
  TR|4.9.1|Originating port configured as a trunk port

  * R: Reserved Bit. Must be sent as zero and ignored on receipt.

### 2.2.2. Enabled-VLANs Sub-TLV
  The optional Enabled-VLANs sub-TLV specifies the VLANs enabled at the port of the originating IS on which the containing Hello was sent, as specified in [RFC6325], Section 4.4.2. It has the following format:

                +-+-+-+-+-+-+-+-+
                |     Type      |                  (1 byte)
                +-+-+-+-+-+-+-+-+
                |   Length      |                  (1 byte)
                +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                | RESV  |  Start VLAN ID        |  (2 bytes)
                +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                | VLAN bit-map....
                +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  * Type: sub-TLV type, set to MT-Port-Cap-TLV Enabled-VLANs sub-TLV 2
  * Length: Variable, minimum 3
  * RESV: 4 reserved bits that must be sent as zero and ignored on receipt
  * Start VLAN ID: The 12-bit VLAN ID that is represented by the high-order bit of the first byte of the VLAN bit-map.
  * VLAN bit-map: The highest-order bit indicates the VLAN equal to the start VLAN ID, the next highest bit indicates the VLAN equal to start VLAN ID + 1, continuing to be the end of the VLAN bit-map field.

  If this sub-TLV occurs more than once in a Hello, the set of enabled VLAN is the union of the sets of VLANs indicated by each of the Enabled-VLAN sub-TLVs in the Hello.

### 2.2.3. Appointed Forwarders Sub-TLV
### 2.2.4. Port TRILL Version Sub-TLV
### 2.2.5. VLANs Appointed Sub-TLV
## 2.3. Sub-TLVs of the Router Capability and MT-Capability TLVs
### 2.3.1. TRILL Version Sub-TLV
### 2.3.2. Nickname Sub-TLV
### 2.3.3. Trees Sub-TLV
### 2.3.4. Tree Identifiers Sub-TLV
### 2.3.5. Trees Used identifiers Sub-TLV
### 2.3.6. Interested VLANs and Spanning Tree Roots Sub-TLV
### 2.3.7. VLAN Group Sub-TLV
### 2.3.8. Interested Labels and Spanning Tree Roots Sub-TLV
### 2.3.9. RBridge Channel Protocols Sub-TLV
### 2.3.10 Affinity Sub-TLV
### 2.3.11 Label Group Sub-TLV
## 2.4. MTU Sub-TLV for Extended Readability and MT-ISN TLVs
## 2.5. TRILL Neighbor TLV

# 3. MTU PDUs

# 4. Use of Existing PDUs and TLVs
## 4.1. TRILL IIH PDUs
## 4.2. Area Address
## 4.3. Protocols Supported
## 4.4. Link State PDUs (LSPs)
## 4.5. Originating LSP Buffer Size

# 5. IANA Consideration
## 5.1. TLVs
## 5.2. Sub-TLVs
## 5.3. PDUs
## 5.4. Reserved and Capability Bits
## 5.5. TRILL Neighbor Record Flags

# 6. Security Consideration

# 7. Changes from RFC 6326

# 8. References
## 8.1. Normative References
## 8.2. Informative References

# 9. Acknowledgement