---
title: Certified Ethical Hacker
badge: CEH
---

Just do tests and train your inner neural network.

## Osi model

The following table might help you while preparing the cert:

| Layer  Type | Layer | Protocol data unit (PDU) | Common protocols | Function |
| --- | --- | --- | --- | --- |
| Host | 7 - Application | Data | HTTP(S), DNS, FTP | High-level APIs, including resource sharing, remote file access |
| Host | 6 - Presentation | Data | ICA | Translation of data between a networking service and an application; including character encoding, data compression and encryption/decryption |
| Host | 5 - Session | Data | SCP, RPC, H.245 | Managing communication sessions, i.e., continuous exchange of information in the form of multiple back-and-forth transmissions between two nodes |
| Host | 4 - Transport | Segment, Datagram | TCP, UDP, SCTP | Reliable transmission of data segments between points on a network, including segmentation, acknowledgement and multiplexing |
| Media | 3 - Network | Packet | IP | Structuring and managing a multi-node network, including addressing, routing and traffic control |
| Media | 2 - Data link | Frame | Ethernet | Reliable transmission of data frames between two nodes connected by a physical layer |
| Media | 1 - Physical | Bit, Symbol | No protocols just bits | Transmission and reception of raw bit streams over a physical medium |

## Random concepts

### AV, SLE, ARO, ALE and EF

- **AV**: Asset Value - How much does it cost to replace 1 asset?
- **EF**: Exposure factor - Probability.
- **SLE**: Single Loss Expentancy - `AV * EF`.
- **ARO**: Annual Rate of Occurrence.
- **ALE**: Annual Loss Expentancy - `SLE * ARO`.

### h4ck3r attacks

- **Man-in-the-middle-attack**: Self explanatory
- **Meet-in-the-middle attack**: Used against systems that encrypt data multiple times with different keys.
- **rubber-hose**: Extraction of cryptographic secrets through coercion or torture
- **STP manipulation attack**: An STP attack involves an attacker spoofing the root bridge in the topology. You can redirect traffic basically.
- **Bluesmacking, Bluejacking, Bluesnarfing**: Bluetooth attacks.
- **DROWN attack**: against SSLv2. 2 servers using the same private key.

#### h4ck3r t00lz

- **CHNTPW**: chntpw is a software utility for resetting or blanking local passwords used by Windows NT, 2000, XP, Vista, 7, 8, 8.1 and 10. It does this by editing the SAM database where Windows stores password hashes.
- **tcptrace**: Used to analyze the files produced by several packet-capture programs such as tcpdump, WinDump, Wireshark, and EtherPeek
- **Infoga**: Infoga is a tool gathering email accounts informations (ip,hostname,country,...) from different public source (search engines, pgp key servers and shodan) and check if emails was leaked using haveibeenpwned.com API.
- **JXplorer** LDAP browser and editor.

### IPSec

- Modes:
  - **Transport mode (ESP)**: IPsec encrypts only the payload of the IP packet, leaving the header untouched.
  - **Tunnel mode (AH)**: IPsec encrypts both the payload and header

### Glosary

- **Multihomed Firewall**: Multiple (2 or more) interfaces
- **Counter-based authentication system**: An authentication system that creates one-time passwords that are encrypted with secret keys.
- **Hit-list scanning technique**: Mítico uso de los gusanos en su día. La net era lenta nmap tardaba la vida y te cundía un gusano que fuese repartiendo el escaneo entre las víctimas.

### Viruses

- **Multipartite Virus**: Infects the system boot sector and the executable files at the same time.
