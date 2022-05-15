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

## Cyber kill chain

A cyber kill chain reveals the phases of a cyberattack: from early reconnaissance to the goal of data exfiltration. The kill chain can also be used as a management tool to help continuously improve network defense. According to Lockheed Martin, **threats** must progress through several phases in the model, including:

1. **Reconnaissance**: Intruder selects target, researches it, and attempts to identify vulnerabilities in the target network.
2. **Weaponization**: Intruder creates remote access malware weapon, such as a virus or worm, tailored to one or more vulnerabilities.
3. **Delivery**: Intruder transmits weapon to target (e.g., via e-mail attachments, websites or USB drives)
4. **Exploitation**: Malware weapon's program code triggers, which takes action on target network to exploit vulnerability.
5. **Installation**: Malware weapon installs access point (e.g., "backdoor") usable by intruder.
6. **Command and Control**: Malware enables intruder to have "hands on the keyboard" persistent access to target network.
7. **Actions on Objective**: Intruder takes action to achieve their goals, such as [data exfiltration](https://en.wikipedia.org/wiki/Data_exfiltration "Data exfiltration"), [data destruction](https://en.wikipedia.org/wiki/Data_destruction "Data destruction"), or encryption for [ransom](https://en.wikipedia.org/wiki/Ransomware "Ransomware").

**Defensive** courses of action can be taken against these phases:

1. **Detect**: Determine whether an intruder is present.
2. **Deny**: Prevent information disclosure and unauthorized access.
3. **Disrupt**: Stop or change outbound traffic (to attacker).
4. **Degrade**: Counter-attack command and control.
5. **Deceive**: Interfere with command and control.
6. **Contain**: Network segmentation changes

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
- **iOS trustjacking**: iOS Trustjacking is a vulnerability that can be exploited by an attacker to read messages and emails and capture sensitive information such as passwords and banking credentials from a remote location without a victim’s knowledge. This vulnerability exploits the “iTunes Wi-Fi Sync” feature whereby a victim connects his/her phone to any trusted computer (could be of a friend or any trusted entity) that is already infected by the attacker.
- **Agent Smith attack**: android agent smith malware adwareAgent Smith (detected by Trend Micro as AndroidOS_InfectionAds.HRXA), a new kind of mobile malware, has been found infecting Android devices by exploiting the vulnerabilities found within the operating system (OS) to replace installed apps with malicious versions without the user knowing.
- **Honey trap**: Attackers target a person inside the company online, pretending to be an attractive person. They then begin a fake online relationship to obtain confidential information about the target company
- **aLTEr attack**: Usually performed against LTE. Attacker installs a virtual (fake) communication tower between two authentic endpoints intending to mislead the victim. This virtual tower is used to interrupt the data transmission between the user and real tower attempting to hijack the active session.
- **Dragonblood**: A set of vulnerabilities in the WPA3 security standard that allows attackers to recover keys, downgrade security mechanisms, and launch various information-theft attacks
- **STP attack**: In a Spanning Tree Protocol (STP) attack, attackers connect a rogue switch into the network to change the operation of the STP protocol and sniff all the network traffic. STP is used in LAN-switched networks with the primary function of removing potential loops within the network.

#### h4ck3r t00lz

- **CHNTPW**: chntpw is a software utility for resetting or blanking local passwords used by Windows NT, 2000, XP, Vista, 7, 8, 8.1 and 10. It does this by editing the SAM database where Windows stores password hashes.
- **tcptrace**: Used to analyze the files produced by several packet-capture programs such as tcpdump, WinDump, Wireshark, and EtherPeek
- **Infoga**: Infoga is a tool gathering email accounts informations (ip,hostname,country,...) from different public source (search engines, pgp key servers and shodan) and check if emails was leaked using haveibeenpwned.com API.
- **JXplorer**: LDAP browser and editor.
- **Flowmon**: OT security tool that further protects against security incidents such as cyber espionage, zero-day attacks, and malware.
- **Bluto**: DNS Recon | Brute Forcer | DNS Zone Transfer | DNS Wild Card Checks | DNS Wild Card Brute Forcer | Email Enumeration | Staff Enumeration | Compromised Account Enumeration | MetaData Harvesting [github.com/darryllane/Bluto](https://github.com/darryllane/Bluto)
- **Web-Stat**:  Website Analytics | Full Visitor Details | Free Stats
- **CryptCat**: Cryptcat is the standard netcat enhanced with twofish encryption with ports for WIndows NT, BSD and Linux. Twofish is courtesy of counterpane, and cryptix.
- **Censys**: Censys is a platform that helps information security practitioners discover, monitor, and analyze devices that are accessible from the Internet.

### IPSec

- Modes:
  - **Transport mode (ESP)**: IPsec encrypts only the payload of the IP packet, leaving the header untouched.
  - **Tunnel mode (AH)**: IPsec encrypts both the payload and header

### Glosary

- **Multihomed Firewall**: Multiple (2 or more) interfaces
- **Counter-based authentication system**: An authentication system that creates one-time passwords that are encrypted with secret keys.
- **Hit-list scanning technique**: Mítico uso de los gusanos en su día. La net era lenta nmap tardaba la vida y te cundía un gusano que fuese repartiendo el escaneo entre las víctimas.
- **Untethered jailbreaking**: A type of iOS jailbreak that allows a device to boot up jailbroken every time it is rebooted.
- **Scanner types**:
  - Network-Based Scanner: Network-based scanners are those that interact only with the real machine where they reside and give the report to the same machine after scanning.
  - Agent-Based Scanner: Agent-based scanners reside on a single machine but can scan several machines on the same network.
  - Proxy Scanner: Proxy scanners are the network-based scanners that can scan networks from any machine on the network.
  - Cluster scanner: Cluster scanners are similar to proxy scanners, but they can simultaneously perform two or more scans on different machines in the network.
- **Remote Authentication Dial-In User Service (RADIUS)**: Is an authentication protocol that provides centralized authentication, authorization, and accounting (AAA) for the remote access servers to communicate with the central server.
- **Threat Intelligence**:
  - **Tactical threat intelligence** is the most basic form of threat intelligence. These are your common indicators of compromise (IOCs). Tactical intelligence is often used for machine-to-machine detection of threats and for incident responders to search for specific artifacts in enterprise networks.
  - **Operational threat intelligence** provides insight into actor methodologies and exposes potential risks. It fuels more meaningful detection, incident response, and hunting programs. Where tactical threat intelligence gives analysts context on threats that are already known, operational intelligence brings investigations closer to uncovering completely new threats.
  - **Strategic threat intelligence** provides a big picture look at how threats and attacks are changing over time. Strategic threat intelligence may be able to identify historical trends, motivations, or attributions as to who is behind an attack. Knowing the who and why of your adversaries also provides clues to their future operations and tactics. This makes strategic intelligence a solid starting point for deciding which defensive measures will be most effective.


### Encryption

- **Algorithms**
  - **Twofish**: symmetric key block cipher that is characterized by a 128-bit block size, and its key size can be up to 256 bits


### Viruses

- **Multipartite Virus**: Infects the system boot sector and the executable files at the same time.




### Container security

#### Five-tier container technology architecture

- `Tier-1`: **Developer machines** - image creation, testing and accreditation
- `Tier-2`: **Testing and accreditation systems** - verification and validation of image contents, signing images and sending them to the registries.
- `Tier-3`: **Registries** - storing images and disseminating images to the orchestrators based on requests.
- `Tier-4`: **Orchestrators** - transforming images into containers and deploying containers to hosts.
- `Tier-5`: **Hosts** - operating and managing containers as instructed by the orchestrator.