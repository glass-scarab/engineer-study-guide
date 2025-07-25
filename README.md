# Security Engineer Study Guide
## Revision By: glass-scarab
### Original By: [nolang](https://twitter.com/__nolang)


### Foreword
Nolang and his contributors did an awesome job of compiling this information and adding relevant interviewing tips for prospective security engineers. If you want to see those tips, please check out Nolang's original [guide](https://github.com/gracenolan/Notes/blob/master/interview-study-notes-for-security-engineering.md). I'm going to be jumping straight to content in this guide. This revision will also be adding detailed explanations and examples of code relevant to the original topics and new topics. If you don't think I did something right, cool just let me know and I'll fix it. If there's something you want to add, cool let me know and I'll add it. I'll try to make this as much of a living document as possible. Just remember to credit anyone that you borrow from. It's not a graduate thesis, but it is the nice thing to do when people work so hard to supply knowledge. Anyway, hope this helps you land a job somewhere. If it does, tweet me because I love hearing that shit! [@glass-scarab](https://x.com/GlassScara89125)

![Alt text](https://github.com/glass-scarab/engineer-study-guide/blob/main/Images/bing_chilling_kitty.png "a title")

### Contents
- [Networking](#networking)
- [Web Application](#web-application)
- [Infrastructure (Prod / Cloud) Virtualisation](#infrastructure-prod--cloud-virtualisation)
- [OS Implementation and Systems](#os-implementation-and-systems)
- [Mitigations](#mitigations)
- [Cryptography, Authentication, Identity](#cryptography-authentication-identity)
- [Malware & Reversing](#malware--reversing)
- [Exploits](#exploits)
- [Attack Structure](#attack-structure)
- [Threat Modeling](#threat-modeling)
- [Detection](#detection)
- [Digital Forensics](#digital-forensics)
- [Incident Management](#incident-management)
- [Coding & Algorithms](#coding--algorithms)
- [Security Themed Coding Challenges](#security-themed-coding-challenges)

# Networking 

- OSI Model [1](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)
	- Application; layer 7
 		- Human - Computer interaction layer
   		- Software applications rely on the application layer to communications.
     		- Software apps are not IN layer 7, but rely on its protocols.
       		- Protocols: HTTP, SMTP
 	- Presentation; layer 6
  		- Responsible for preparing data to be used by the application layer
    		- Includes encryption, translation, and compression.
  	- Session; layer 5
  		- Responsible for opening and closing communications between two systems.
  	 	- Also manages checkpointing and recovery of session.
  	  	- Includes RPC and establishment calls for NetBIOS and SQL.
	- Transport; layer 4 (TCP/UDP).
	- Network; layer 3 (Routing).
	- Datalink; layer 2 (Error checking and frame synchronisation).
	- Physical; layer 1 (Bits over fibre).	
- Firewalls [2](https://www.cisco.com/site/us/en/learn/topics/security/what-is-a-firewall.html)
	- A firewall is a network security device that separates a trusted internal network from an untrusted, external network.
	- Will contain rules blocking or allowing based on filtering and traffic type.
 	- **Packet filtering** firewalls search each packet and filter based on IP, port, and protocol types; however, they do not inspect packet contents.
  	- **Proxy firewalls** serves as a gateway from one network to another for a specific application.
  		- Can also provide conent caching an additional security, by preventing direct connections from external users/systems.
  	- **Web application firewalls (WAF)** act as a middleman for communication requests between an internal and external network.
  		- Robust security through content inspection.
  	 	- May be known as Layer 7 firewalls.
  	- **Next-generation firewalls (NGFW)** provide advanced capabilities over a traditional firewall by offering application awareness/control, IPS functions, URL filtering, and threat intelligence integrations.
  	- **Cloud-native firewalls** operate in the cloud and support elastic security workflows, multi-tenancy, and smart load balancing.
- NAT [3](https://www.cisco.com/site/us/en/learn/topics/networking/what-is-network-address-translation-nat.html)
	- Translates a single public IP to multiple internal IP addresses (i.e. 8.8.8.8 --> 192.168.0.0/16).
 	- Introduced to address IPv4 exhaustion.
 	- Usually implemented on WAN edge routers to connect private networks to public networks like the Internet.
  	- NAT64 allows IPv4-only devices and IPv6-only devices to communicate.
- DNS [4](https://www.cloudflare.com/learning/dns/what-is-dns/)
	- Port 53
	- DNS is the "phonebook" of the Internet -- resolving domain names to IP addresses and vice-versa.
 	- In a reverse DNS lookup, PTR might contain- 2.152.80.208.in-addr.arpa, which will map to  208.80.152.2. DNS lookups start at the end of the string and work backwards, which is why the IP address is backwards in PTR.
 	- How does it work? / "What happens when you type *insert website* into the address bar?

            1. A user types ‘example.com’ into a web browser and the query travels into the Internet and is received by a DNS
               recursive resolver.
            2. The resolver then queries a DNS root nameserver (.).
            3. The root server then responds to the resolver with the address of a Top Level Domain (TLD) DNS server (such as
               .com or .net), which stores the information for its domains. When searching for example.com, our request is
               pointed toward the .com TLD.
            4. The resolver then makes a request to the .com TLD.
            5. The TLD server then responds with the IP address of the domain’s nameserver, example.com.
            6. Lastly, the recursive resolver sends a query to the domain’s nameserver.
            7. The IP address for example.com is then returned to the resolver from the nameserver.
            8. The DNS resolver then responds to the web browser with the IP address of the domain requested initially.
            9. The browser makes a HTTP request to the IP address.
            10. The server at that IP returns the webpage to be rendered in the browser
	      
    ![Alt text](https://github.com/glass-scarab/engineer-study-guide/blob/main/Images/dns-lookup.png "Complete DNS Lookup and Webpage Query")
	     
	
- DNS tunneling / DNS exfiltration [5](https://www.paloaltonetworks.com/cyberpedia/what-is-dns-tunneling)
	- This is a tactic used to send information (commands, data, etc.) between an internal system and external command & control (C2) systems by hiding it in DNS queries and responses. 
	- For example: 26856485f6476a567567c6576e678.badguy.com
 		- If an internal system sent this, then it is likely data exfiltration
   	- For example: CNAME d2hvYW1p.badguy.com
   		- If an external system responds like this, then it is likely some sort of acknowledgement or instruction set.
	- DNS tunneling doesn’t show up in http logs.
- DNS configs
	- Start of Authority (SOA). [6](https://www.cloudflare.com/learning/dns/dns-records/dns-soa-record/)
 		- DNS record about the domain or zone such as the admin's email, when the last update occured, and the server's refresh time.
	- IP addresses (A and AAAA). [7](https://www.cloudflare.com/learning/dns/dns-records/dns-a-record/) [8](https://www.cloudflare.com/learning/dns/dns-records/dns-aaaa-record/)
 		- A reocrds:
   			- Stands for **address record**.
      			- Maps an IPv4 address to a given domain, and includes the record's origin (i.e. "example.com") and time to live (TTL).
         	- AAAA records
          		- Still stands for address record, but the bit size is 4 times larger than IPv4 (32 x 4 = 128) hence 4 A's.
            		- Maps an IPv6 address to a domain rather than an IPv4 address.
	- SMTP mail exchangers (MX). [9](https://www.cloudflare.com/learning/dns/dns-records/dns-mx-record/)
 		- Stands for **mail exchange**, and directs email to a mail server by indicating how mail should be routed using the SMTP.
   		- Contains domain origin and TTL like A/AAAA records, but also includes priority numbers email server domains to indicate the mail server preference.
	- Name servers (NS). [10](https://www.cloudflare.com/learning/dns/dns-records/dns-ns-record/)
 		- Stands for **nameserver** and indicates the authoratative DNS server for the domain.
   		- Tells systems where to go to find the domain's IP address.
     		- Includes origin, record type, NS value, and TTL.
	- Pointers for reverse DNS lookups (PTR). [11](https://www.cloudflare.com/learning/dns/dns-records/dns-ptr-record/)
 		- Stands for **pointer** and correlates domain names with IP addresses
   		- **Opposite of A/AAAA records** which corelate **IPs -> Domains** (i.e. What IP is associated with this domain I have?).
     		- A/AAAA records are used in web browsing and other forward lookups.
   		- PTR == Domains -> IPs** (i.e. What domain is associated with this IP I have?).
     		- PTR records are used in reverse DNS lookup
	- Domain name aliases (CNAME). [12](https://www.cloudflare.com/learning/dns/dns-records/dns-cname-record/)
 		- Stands for **canonical name** and points from an alias domain to a "canonical" domain.
   		- **All** CNAME records must point to a domain and **never** to an IP address.
     		- Example: "blog.example.com" (alias) -> example.com (domain)
       		- Only points clients to the same IP as the root domain. Once there, the server will handle the URL accordingly.
- ARP
	- **A**ddress **R**esolution **P**rotocol
 	- Pair MAC address with IP Address for IP connections. 
- DHCP
	- Dynamic Host Control Protocol
	- UDP (67 - Server, 68 - Client)
	- Dynamic address allocation (allocated by router).
	- `DHCPDISCOVER` -> `DHCPOFFER` -> `DHCPREQUEST` -> `DHCPACK`
- Multiplex 
	- Way of transmitting multiple signals over a single medium.
 		- Timeshare: signals multiplexed based on timeslots
   		- Frequency spectrum: signals multiplexed based on different frequencies
     			- Electromagnetic signals (radio, electric, light) travel in waves and can be split into a spectrum based on frqueny (i.e. Hz), so signlas can be organized across this spectrum to allow for multiplexing of communications 
- Traceroute 
	- Usually uses UDP, but might also use ICMP Echo Request or TCP SYN. TTL, or hop-limit.
	- Initial hop-limit is 128 for windows and 64 for *nix. Destination returns ICMP Echo Reply. 
- Nmap 
	- Network scanning tool.
 	- Can run traceroute, ping, and port scans.
- Intercepts (PitM - Person in the middle)
	- Traffic can and will be intercepted if a network listening device is inserted between two points of communication. For example, Wi-Fi sniffing.
 	- If anyone can intercept information in transit how can anything be confidential? Enter, Public Key Infrastructure or PKI.
  	- PKI is critical for managing the issuance and management of digital certificates.
  	- We'll get into cryptography specifics later in this guide, but for now remember that PKI is a system of systems verifying one another. With PKI, you can take a certificate/key to an authoritative server to verify the intended recipient and their digital signature. This helps to mitigate potential abuse from PitM attacks.
- VPN 
	- Hide traffic from ISP but expose traffic to VPN provider.
 	- An encrypted tunnel between you, a VPN provider, and your target endpoint.
  	- If someone were to intercept traffic between you and the VPN server, then they would only see one side of the encrypted conversation.
  	- If someone were to intercept traffic between the VPN and your target, then they again would only see traffic to and from the VPN server.
  	- This masks the true source and destination (i.e. you -> target). It does open you up to the VPN provider though. They manage the session keys, so they could decrypt your entire traffic.
- Tor 
	- Traffic is very obvious on a network.
 		- Tor nodes are very well-documented and there are plenty of lists and application filters that can be used to alert on and reject Tor traffic
   			- Many enterprises will even block the installation of the Onion browser altogether.
	- Tor is not a foolproof solution!
 		- Honey nodes are real and a bad exit node can reveal your traffic to threat actors or law enforcement.
   		- Timing attacks and traffic analysis can be used if you can monitor both the entry and exit nodes.
     			- This is pretty sneaky, but is also a dead giveaway if you know your target's connection habits.
- Proxy  
	- Layering multiple proxies won't help you stay anonymous.
 		- Sending traffic through 7 proxies would be so incredibly slow.
   		- You may mask the source of your traffic to the endpoint, but the proxies in between are more than likely going to log your activity and cache the content you search.
     		- Plus, there is no guarantee of end-to end encryption like a VPN.
       		- Also, your ISP can definitely see the entire stream of communication between you and the 1st proxy.
- BGP
	- Border Gateway Protocol.
	- Holds the internet together.
 	- Kind of like the post-office system of the Internet. BGP is responsible for looking at all the paths and determining the best route for data to take.
- Network traffic tools
	- Wireshark
 		- GUI packet analysis/capture tool
	- Tcpdump
 		- CLI packet analysis/capture tool
	- Burp suite
 		- Suite of tools used for penetration testing and vulnerability scanning
- HTTP/S 
	- Hyper Text Transport Protocol (HTTP): port 80
 	- Hyper Text Transport Protocol Secure (HTTPS): port 443
  		- Uses transport layer security (TLS) to encrypt communications 
- SSL/TLS
	- Secure Socet Layer: port 443
 	- TLS: port 443
  		- TLS replaced SSL
    	- TLS Handshake: [13](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/)
     		- The 'client hello' message: The client initiates the handshake by sending a "hello" message to the server. The message will include which TLS version the client supports, the cipher suites supported, and a string of random bytes known as the "client random."
       		- The 'server hello' message: In reply to the client hello message, the server sends a message containing the server's SSL certificate, the server's chosen cipher suite, and the "server random," another random string of bytes that's generated by the server.
         	- Authentication: The client verifies the server's SSL certificate with the certificate authority that issued it. This confirms that the server is who it says it is, and that the client is interacting with the actual owner of the domain.
          	- The premaster secret: The client sends one more random string of bytes, the "premaster secret." The premaster secret is encrypted with the public key and can only be decrypted with the private key by the server. (The client gets the public key from the server's SSL certificate.)
          	- Private key used: The server decrypts the premaster secret.
          	- Session keys created: Both client and server generate session keys from the client random, the server random, and the premaster secret. They should arrive at the same results.
          	- Client is ready: The client sends a "finished" message that is encrypted with a session key.
          	- Server is ready: The server sends a "finished" message encrypted with a session key.
          	- Secure symmetric encryption achieved: The handshake is completed, and communication continues using the session keys
	- Super important to learn this, includes learning about handshakes, encryption, signing, certificate authorities, trust systems. A good [primer](https://english.ncsc.nl/publications/publications/2021/january/19/it-security-guidelines-for-transport-layer-security-2.1) on all these concepts and algorithms is made available by the Dutch cybersecurity center.
	- POODLE, BEAST, CRIME, BREACH, HEARTBLEED.
- TCP/UDP
	- TCP is great for **stateful** data transmissions (i.e. web browsing, IM, etc.)
 		- For when you need to know the client received the message.
   		- Slower, but less prone to dropped packets.
   	- UDP is great for **stateless** connections (i.e. VOIP, streaming, etc.)
   		- For when you need to give the best effort to send connent, but you don't want to wait for the reply.
   	 	- Faster, but prone to lost data (aka buffering).
	- TCP will throttle back if packets are lost but UDP doesn't. 
	- Streaming can slow network TCP connections sharing the same network.
- ICMP 
	- Ping and traceroute.
- Mail
	- SMTP ports (25, 587, 465)
	- IMAP ports (143, 993)
	- POP3 ports (110, 995)
- SSH 
	- Port (22)
	- Handshake uses asymmetric encryption to exchange symmetric key.
 	-  SSH handshake:
  		- TCP 3-way handshake (syn, syn-ack, ack)
    		- SSH Protocol Version Exchange: Both the client and the server exchange messages to agree on a compatible SSH protocol version to use for the connection.
    		- Key Exchange (KEX): This is where the client and server exchange public information to derive a shared secret key. This process happens without actually transmitting the key itself over the network, ensuring the confidentiality of the session key.
      		- Elliptic Curve Diffie-Hellman Initialization (ECDH Init): The client generates a temporary key pair and sends its public key to the server.
        	- Elliptic Curve Diffie-Hellman Reply (ECDH Reply): The server generates its own temporary key pair and uses the client's public key (along with its own key pair) to derive the shared secret key. The server then generates an exchange hash and signs it with its host private key, and sends this signed exchange hash to the client along with its public key (or certificate). This ensures the client can verify the server's identity and detect any potential Man-in-the-Middle (MITM) attacks. The client also verifies the server's host key against a local database of known hosts to further prevent MITM attacks.
         	- New Keys: Both the client and the server use the shared secret derived during the Key Exchange process to generate session keys. These session keys are then used for symmetric encryption of all subsequent communication in the SSH session, guaranteeing confidentiality and integrity. 
- Telnet
	- Ports (23, 992)
	- Allows remote communication with hosts.
- ARP  
	- Who is 0.0.0.0? Tell 0.0.0.1.
	- Linking IP address to MAC, Looks at cache first.
- DHCP 
	- server;client Ports (67;68) (546;547)
	- Dynamic mode (leases IP address, not persistent).
	- Automatic mode (leases IP address and remembers MAC and IP pairing in a table).
	- Manual mode (static IP set by administrator).
- IRC 
	- IRC is a simple text-based protocol used to send messages
 	- It can allow a botmaster to send commands to bots.
- FTP/SFTP 
	- File transfer protocol (FTP)
 		- Port 21
 	- Secure File Transfer Protocol (SFTP)
  		- Port 22
- Remote procedure call (RPC)
	- Predefined set of tasks that remote clients can execute.
	- Used inside orgs. 
- Service ports
	- 0 - 1023: Reserved for common services - sudo required. 
	- 1024 - 49151: Registered ports used for IANA-registered services. 
	- 49152 - 65535: Dynamic ports that can be used for anything. 
- HTTP Header
	- | Verb | Path | HTTP version |
	- Domain
	- Accept
	- Accept-language
	- Accept-charset
	- Accept-encoding(compression type)
	- Connection- close or keep-alive
	- Referrer
	- Return address
	- Expected Size?
- HTTP Response Header
	- HTTP version
	- Status Codes: 
		- 1xx: Informational Response
		- 2xx: Successful
		- 3xx: Redirection
		- 4xx: Client Error
		- 5xx: Server Error
	- Type of data in response 
	- Type of encoding
	- Language 
	- Charset
- UDP Header
	- Source port
	- Destination port
	- Length
	- Checksum
- Broadcast domains
	- Logical areas within a network where a broadcast by one device is received by all others in the domain.
- Collision domains
	- a network segment where data packets can collide with each other when multiple devices try to transmit simultaneously.
- Root stores
	- Collection off trusted root certificates.
- CAM table overflow (MAC flood)
	- The idea is that you flood a switch with fake MAC addresses and overflow the finite amount of table space. When this happens, the switch fails open and begins acting like an old network hub broadcasting packets to everyone.


# Web Application 

- Same origin policy
	- Policy that only accepts requests from a script/docuiment from the same origin domain.  
- CORS 
	- Cross-Origin Resource Sharing. Can specify allowed origins in HTTP headers. Sends a preflight request with options set asking if the server approves, and if the server approves, then the actual request is sent (eg. should client send auth cookies).
- HSTS 
	- HTTP Strict-Transport-Security. Policy that informs browsers that the host should only be access using HTTPS, and any future attempts to access it over HTTP will automatically be upgraded to HTTPS.
- Cert transparency 
	- Can verify certificates against public logs 	
- HTTP Public Key Pinning
	- (HPKP)
 	- Allowed browsers to pin keys that the server sent in the HPKP header-- a list of public key hashes that could be expected in the future. The problem was if there was a misconfiguration, the policy could deny legitimate users access.
	- Deprecated by Google Chrome
- Cookies 
	- httponly - cannot be accessed by javascript.
- CSRF
	- Cross-Site Request Forgery.
	- Cookies.
- XSS
	- Reflected XSS.
	- Persistent XSS.
	- DOM based /client-side XSS.
	- `<img scr=””>` will often load content from other websites, making a cross-origin HTTP request. 
- SQLi 
	- Person-in-the-browser (flash / java applets) (malware).
	- Validation / sanitisation of webforms.
- POST 
	- Form data. 
- GET 
	- Queries. 
	- Visible from URL.
- Directory traversal 
	- Find directories on the server you’re not meant to be able to see.
	- There are tools that do this.
- APIs 
	- Think about what information they return. 
	- And what can be sent.
- Beefhook
	- Get info about Chrome extensions.
- User agents
	- Is this a legitimate browser? Or a botnet?
- Browser extension take-overs
	- Miners, cred stealers, adware.
- Local file inclusion
- Remote file inclusion (not as common these days)
- SSRF 
	- Server Side Request Forgery.
- Web vuln scanners. 
- SQLmap.
- Malicious redirects.


# Infrastructure (Prod / Cloud) Virtualisation 

- Hypervisors.
- Hyperjacking.
- Containers, VMs, clusters.
- Escaping techniques.
	- Network connections from VMs / containers.  
- Lateral movement and privilege escalation techniques.
	- Cloud Service Accounts can be used for lateral movement and privilege escalation in Cloud environments.
	- GCPloit tool for Google Cloud Projects.
- Site isolation.
- Side-channel attacks.
	- Spectre, Meltdown.
- Beyondcorp 
	- Trusting the host but not the network.
- Log4j vuln. 


# OS Implementation and Systems

- Privilege escalation techniques, and prevention.
- Buffer Overflows.
- Directory traversal (prevention).
- Remote Code Execution / getting shells.
- Local databases
	- Some messaging apps use sqlite for storing messages.
	- Useful for digital forensics, especially on phones.
- Windows
	- Windows registry and group policy.
	- Active Directory (AD).
		- Bloodhound tool. 
		- Kerberos authentication with AD.
	- Windows SMB. 
	- Samba (with SMB).
	- Buffer Overflows. 
	- ROP. 
	
- *nix 
	- SELinux.
	- Kernel, userspace, permissions.
	- MAC vs DAC.
	- /proc
	- /tmp - code can be saved here and executed.
	- /shadow 
	- LDAP - Lightweight Directory Browsing Protocol. Lets users have one password for many services. This is similar to Active Directory in windows.
- MacOS
	- Gotofail error (SSL).
	- MacSweeper.
	- Research Mac vulnerabilities.

## Mitigations 
- Patching 
- Data Execution Prevention
- Address space layout randomisation
	- To make it harder for buffer overruns to execute privileged instructions at known addresses in memory.
- Principle of least privilege
	- Eg running Internet Explorer with the Administrator SID disabled in the process token. Reduces the ability of buffer overrun exploits to run as elevated user.
- Code signing
	- Requiring kernel mode code to be digitally signed.
- Compiler security features
	- Use of compilers that trap buffer overruns.
- Encryption
	- Of software and/or firmware components.
- Mandatory Access Controls
	- (MACs)
	- Access Control Lists (ACLs)
	- Operating systems with Mandatory Access Controls - eg. SELinux.
- "Insecure by exception"
	- When to allow people to do certain things for their job, and how to improve everything else. Don't try to "fix" security, just improve it by 99%.
- Do not blame the user
	- Security is about protecting people, we should build technology that people can trust, not constantly blame users. 


# Cryptography, Authentication, Identity 

- Encryption vs Encoding vs Hashing vs Obfuscation vs Signing
	- Be able to explain the differences between these things. 
	- [Various attack models](https://en.wikipedia.org/wiki/Attack_model) (e.g. chosen-plaintext attack).

- Encryption standards + implementations
	- [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) (asymmetrical).
	- [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) (symmetrical).
	- [ECC](https://en.wikipedia.org/wiki/EdDSA) (namely ed25519) (asymmetric).
	- [Chacha/Salsa](https://en.wikipedia.org/wiki/Salsa20#ChaCha_variant) (symmetric).

- Asymmetric vs symmetric
	- Asymmetric is slow, but good for establishing a trusted connection.
	- Symmetric has a shared key and is faster. Protocols often use asymmetric to transfer symmetric key.
	- Perfect forward secrecy - eg Signal uses this.

- Cyphers
	- Block vs stream [ciphers](https://en.wikipedia.org/wiki/Cipher).
	- [Block cipher modes of operation](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation).
	- [AES-GCM](https://en.wikipedia.org/wiki/Galois/Counter_Mode).

- Integrity and authenticity primitives
	- [Hashing functions](https://en.wikipedia.org/wiki/Cryptographic_hash_function) e.g. MD5, Sha-1, BLAKE. Used for identifiers, very useful for fingerprinting malware samples.
	- [Message Authentication Codes (MACs)](https://en.wikipedia.org/wiki/Message_authentication_code).
	- [Keyed-hash MAC (HMAC)](https://en.wikipedia.org/wiki/HMAC).

- Entropy
	- PRNG (pseudo random number generators).
	- Entropy buffer draining.
	- Methods of filling entropy buffer.

- Authentication
	- Certificates 
		- What info do certs contain, how are they signed? 
		- Look at DigiNotar.
	- Trusted Platform Module 
		- (TPM)
		- Trusted storage for certs and auth data locally on device/host.
	- O-auth
		- Bearer tokens, this can be stolen and used, just like cookies.
	- Auth Cookies
		- Client side.
	- Sessions 
		- Server side.
	- Auth systems 
		- SAMLv2o.
		- OpenID.
		- Kerberos. 
			- Gold & silver tickets.
			- Mimikatz.
			- Pass-the-hash.	  
	- Biometrics
		- Can't rotate unlike passwords.
	- Password management
		- Rotating passwords (and why this is bad). 
		- Different password lockers. 
	- U2F / FIDO
		- Eg. Yubikeys.
		- Helps prevent successful phishing of credentials.
	- Compare and contrast multi-factor auth methods.

- Identity
	- Access Control Lists (ACLs)
		- Control which authenicated users can access which resources.
	- Service accounts vs User accounts
		- Robot accounts or Service accounts are used for automation.
		- Service accounts should have heavily restricted priviledges.
		- Understanding how Service accounts are used by attackers is important for understanding Cloud security.  
	- impersonation
		- Exported account keys.
		- ActAs, JWT (JSON Web Token) in Cloud.
	- Federated identity


# Malware & Reversing

- Interesting malware
	- Conficker.
	- Morris worm.
	- Zeus malware.
	- Stuxnet.
	- Wannacry.
	- CookieMiner.
	- Sunburst.

- Malware features
	- Various methods of getting remote code execution. 
	- Domain-flux.
	- Fast-Flux.
	- Covert C2 channels.
	- Evasion techniques (e.g. anti-sandbox).
	- Process hollowing. 
	- Mutexes.
	- Multi-vector and polymorphic attacks.
	- RAT (remote access trojan) features.

- Decompiling/ reversing 
	- Obfuscation of code, unique strings (you can use for identifying code).
	- IdaPro, Ghidra.

- Static / dynamic analysis
	- Describe the differences.
	- Virus total. 
	- Reverse.it. 
	- Hybrid Analysis.


# Exploits

- Three ways to attack - Social, Physical, Network 
	- **Social**
		- Ask the person for access, phishing. 
		- Cognitive biases - look at how these are exploited.
		- Spear phishing.
		- Water holing.
		- Baiting (dropping CDs or USB drivers and hoping people use them).
		- Tailgating.
	- **Physical** 
		- Get hard drive access, will it be encrypted? 
		- Boot from linux. 
		- Brute force password.
		- Keyloggers.
		- Frequency jamming (bluetooth/wifi).
		- Covert listening devices.
		- Hidden cameras.
		- Disk encryption. 
		- Trusted Platform Module.
		- Spying via unintentional radio or electrical signals, sounds, and vibrations (TEMPEST - NSA).
	- **Network** 
		- Nmap.
		- Find CVEs for any services running.
		- Interception attacks.
		- Getting unsecured info over the network.

- Exploit Kits and drive-by download attacks

- Remote Control
	- Remote code execution (RCE) and privilege.
	- Bind shell (opens port and waits for attacker).
	- Reverse shell (connects to port on attackers C2 server).

- Spoofing
	- Email spoofing.
	- IP address spoofing.
	- MAC spoofing.
	- Biometric spoofing.
	- ARP spoofing.

- Tools
	- Metasploit.
	- ExploitDB.
	- Shodan - Google but for devices/servers connected to the internet.
	- Google the version number of anything to look for exploits.
	- Hak5 tools.


# Attack Structure

Practice describing security concepts in the context of an attack. These categories are a rough guide on attack structure for a targeted attack. Non-targeted attacks tend to be a bit more "all-in-one".

- Reconnaissance
	- OSINT, Google dorking, Shodan.
- Resource development
	- Get infrastructure (via compromise or otherwise).
	- Build malware.
	- Compromise accounts.
- Initial access
	- Phishing.
	- Hardware placements.
	- Supply chain compromise.
	- Exploit public-facing apps.
- Execution
	- Shells & interpreters (powershell, python, javascript, etc.).
	- Scheduled tasks, Windows Management Instrumentation (WMI).
- Persistence
	- Additional accounts/creds.
	- Start-up/log-on/boot scripts, modify launch agents, DLL side-loading, Webshells.
	- Scheduled tasks.
- Privilege escalation
	- Sudo, token/key theft, IAM/group policy modification.
	- Many persistence exploits are PrivEsc methods too.
- Defense evasion
	- Disable detection software & logging.
	- Revert VM/Cloud instances.
	- Process hollowing/injection, bootkits.
- Credential access
	- Brute force, access password managers, keylogging.
	- etc/passwd & etc/shadow.
	- Windows DCSync, Kerberos Gold & Silver tickets.
	- Clear-text creds in files/pastebin, etc.
- Discovery
	- Network scanning.
	- Find accounts by listing policies.
	- Find remote systems, software and system info, VM/sandbox.
- Lateral movement
	- SSH/RDP/SMB.
	- Compromise shared content, internal spear phishing.
	- Pass the hash/ticket, tokens, cookies.
- Collection
	- Database dumps.
	- Audio/video/screen capture, keylogging.
	- Internal documentation, network shared drives, internal traffic interception.
- Exfiltration
	- Removable media/USB, Bluetooth exfil.
	- C2 channels, DNS exfil, web services like code repos & Cloud backup storage.
	- Scheduled transfers.
- Command and control
	- Web service (dead drop resolvers, one-way/bi-directional traffic), encrypted channels.
	- Removable media.
	- Steganography, encoded commands.
- Impact
	- Deleted accounts or data, encrypt data (like ransomware).
	- Defacement.
	- Denial of service, shutdown/reboot systems.


# Threat Modeling

- Threat Matrix
- Trust Boundries
- Security Controls
- STRIDE framework
	- **S**poofing
	- **T**ampering
	- **R**epudiation
	- **I**nformation disclosure
	- **D**enial of service
	- **E**levation of privilege 
- [MITRE Att&ck](https://attack.mitre.org/) framework
- [Excellent talk](https://www.youtube.com/watch?v=vbwb6zqjZ7o) on "Defense Against the Dark Arts" by Lilly Ryan (contains *many* Harry Potter spoilers)


# Detection Engineering

- IDS
	- Intrusion Detection System (signature based (eg. snort) or behaviour based).
	- Snort/Suricata/YARA rule writing
	- Host-based Intrusion Detection System (eg. OSSEC)

- SIEM
	- Security Information and Event Management.

- IOC 
	- Indicator of compromise (often shared amongst orgs/groups).
	- Specific details (e.g. IP addresses, hashes, domains)
 	- Often indicative of **post-exploit** activity
 - IOA
 	- Indocator of attack (often shared amongst orgs/groups).
  	- Specific details (e.g. IP addresses, hashes, domains)
   	- More focused on patterns of behavior.
   	- "Shifting left" to detect indicators of **pre-exploit** behavior.

- Things that create signals
	- Honeypots, snort.

- Things that collect/triage signals
	- SIEM, eg splunk.

- Things that will alert a human 
	- Automatic triage of collated logs, machine learning.
	- Notifications and analyst fatigue.
	- Systems that make it easy to decide if alert is actual hacks or not.

- Signatures
	- Host-based signatures
		- Eg changes to the registry, files created or modified.
		- Strings in found in malware samples appearing in binaries installed on hosts (/Antivirus).
	- Network signatures
		- Eg checking DNS records for attempts to contact C2 (command and control) servers. 

- Anomaly / Behaviour based detection 
	- IDS learns model of “normal” behaviour, then can detect things that deviate too far from normal - eg unusual urls being accessed, user specific- login times / usual work hours, normal files accessed.  
	- Can also look for things that a hacker might specifically do (eg, HISTFILE commands, accessing /proc).
	- If someone is inside the network- If action could be suspicious, increase log verbosity for that user.

- Firewall rules
	- Brute force (trying to log in with a lot of failures).
	- Detecting port scanning (could look for TCP SYN packets with no following SYN ACK/ half connections).
	- Antivirus software notifications.
	- Large amounts of upload traffic.

- Honey pots
	- Canary tokens.
	- Dummy internal service / web server, can check traffic, see what attacker tries.

- Things to know about attackers
	- Slow attacks are harder to detect.
	- Attacker can spoof packets that look like other types of attacks, deliberately create a lot of noise.
	- Attacker can spoof IP address sending packets, but can check TTL of packets and TTL of reverse lookup to find spoofed addresses.
	- Correlating IPs with physical location (is difficult and inaccurate often).

- Logs to look at
	- DNS queries to suspicious domains.
	- HTTP headers could contain wonky information.
	- Metadata of files (eg. author of file) (more forensics?).
	- Traffic volume.
	- Traffic patterns.
	- Execution logs.

- Detection related tools
	- Splunk.
	- Arcsight.
	- Qradar.
	- Darktrace.
	- Tcpdump.
	- Wireshark.
	- Zeek.
 	- Sigma Rules
 - 

- A curated list of [awesome threat detection](https://github.com/0x4D31/awesome-threat-detection) resources


# Digital Forensics

 - Evidence volatility (network vs memory vs disk)

 - Network forensics
	- DNS logs / passive DNS
	- Netflow
	- Sampling rate

 - Disk forensics
	- Disk imaging
	- Filesystems (NTFS / ext2/3/4 / AFPS)
	- Logs (Windows event logs, Unix system logs, application logs)
	- Data recovery (carving)
	- Tools
	- plaso / log2timeline
	- FTK imager
	- encase

 - Memory forensics
	- Memory acquisition (footprint, smear, hiberfiles)
	- Virtual vs physical memory
	- Life of an executable
	- Memory structures
	- Kernel space vs user space
	- Tools
	- Volatility
	- Google Rapid Response (GRR) / Rekall
	- WinDbg

  - Mobile forensics
	- Jailbreaking devices, implications
	- Differences between mobile and computer forensics
	- Android vs. iPhone

  - Anti forensics
	- How does malware try to hide?
	- Timestomping

  - Chain of custody
  	- Handover notes 


# Incident Management

- Privacy incidents vs information security incidents
- Know when to talk to legal, users, managers, directors.
- Run a scenario from A to Z, how would you ...

- Good practices for running incidents 
	- How to delegate.
	- Who does what role.
	- How is communication managed + methods of communication.
	- When to stop an attack.
	- Understand risk of alerting attacker.
	- Ways an attacker may clean up / hide their attack.
	- When / how to inform upper management (manage expectations).
	- Metrics to assign Priorities (e.g. what needs to happen until you increase the prio for a case)
	- Use playbooks if available

- Important things to know and understand
	- Type of alerts, how these are triggered.
	- Finding the root cause.
	- Understand stages of an attack (e.g. cyber-killchain)
	- Symptom vs Cause.
	- First principles vs in depth systems knowledge (why both are good).
	- Building timeline of events.
	- Understand why you should assume good intent, and how to work with people rather than against them.
	- Prevent future incidents with the same root cause

  - Response models
  	- SANS' PICERL (Preparation, Identification, Containement, Eradication, Recovery, Lessons learned)
   	- Google's IMAG (Incident Management At Google)


# Coding & Algorithms

- The basics
	- Conditions (if, else).
	- Loops (for loops, while loops).
 	- Dictionaries.
 	- Slices/lists/arrays.
 	- String/array operations (split, contaings, length, regular expressions).
 	- Pseudo code (concisely describing your approach to a problem).

- Data structures
	- Dictionaries / hash tables (array of linked lists, or sometimes a BST).
	- Arrays.
	- Stacks.
	- SQL/tables. 
	- Bigtables.

- Sorting
	- Quicksort, merge sort.

- Searching 
	- Binary vs linear.

- Big O 
	- For space and time.

- Regular expressions
	- O(n), but O(n!) when matching.
	- It's useful to be familiar with basic regex syntax, too.

- Recursion 
	- And why it is rarely used.

- Python
	- List comprehensions and generators [ x for x in range() ].
	- Iterators and generators.
	- Slicing [start:stop:step].
	- Regular expressions.
	- Types (dynamic types), data structures.
	- Pros and cons of Python vs C, Java, etc.
	- Understand common functions very well, be comfortable in the language.


## Security Themed Coding Challenges

These security engineering challenges focus on text parsing and manipulation, basic data structures, and simple logic flows. Give the challenges a go, no need to finish them to completion because all practice helps.

- Cyphers / encryption algorithms 
	- Implement a cypher which converts text to emoji or something.
	- Be able to implement basic cyphers.

- Parse arbitrary logs 
	- Collect logs (of any kind) and write a parser which pulls out specific details (domains, executable names, timestamps etc.)

- Web scrapers 
	- Write a script to scrape information from a website.

- Port scanners 
	- Write a port scanner or detect port scanning.

- Botnets
	- How would you build ssh botnet?

- Password bruteforcer
	- Generate credentials and store successful logins. 

- Scrape metadata from PDFs
	- Write a mini forensics tool to collect identifying information from PDF metadata. 

- Recover deleted items
	- Most software will keep deleted items for ~30 days for recovery. Find out where these are stored. 
	- Write a script to pull these items from local databases. 
 
- Malware signatures
	- A program that looks for malware signatures in binaries and code samples.
	- Look at Yara rules for examples

 ## Sources
1. Cloudflare (2025). "*What is the OSI Model?*". https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/
2. Cisco (2025). "*What is a Firewall?*". https://www.cisco.com/site/us/en/learn/topics/security/what-is-a-firewall.html
3. Cisco (2025). "*What is Network Address Translation (NAT)?*". https://www.cisco.com/site/us/en/learn/topics/networking/what-is-network-address-translation-nat.html
4. Cloudflare (2025). "*What is DNS?*". https://www.cloudflare.com/learning/dns/what-is-dns/
5. Palo Alto Networks (2025). "*What is DNS Tunneling? \[\+Examples and Protection Tips\]*". https://www.paloaltonetworks.com/cyberpedia/what-is-dns-tunneling
6. Cloudflare (2025). "*What is a DNS SOA Record?*". https://www.cloudflare.com/learning/dns/dns-records/dns-soa-record/
7. Cloudflare (2025). "*DNS A Record*". https://www.cloudflare.com/learning/dns/dns-records/dns-a-record/
8. Cloudflare (2025). "*DNS AAAA Record*". https://www.cloudflare.com/learning/dns/dns-records/dns-aaaa-record/
9. Cloudflare (2025). "*What is a DNS MX Record?*". https://www.cloudflare.com/learning/dns/dns-records/dns-mx-record/
10. Cloudflare (2025). "*What is a DNS NS Record?*". https://www.cloudflare.com/learning/dns/dns-records/dns-ns-record/
11. Cloudflare (2025). "*What is a DNS PTR Record?*". https://www.cloudflare.com/learning/dns/dns-records/dns-ptr-record/
12. Cloudflare (2025). "*What is a DNS CNAME record?*". https://www.cloudflare.com/learning/dns/dns-records/dns-cname-record/
13. Cloudflare (2025). "*What is a TLS Handshake?*". https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/
 
