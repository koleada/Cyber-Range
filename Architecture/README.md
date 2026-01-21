# Home Lab Topology & Architecture 

I spent a pretty good amount of time planning the best approach for building this home lab envionrment including determining what technologies I should use, what sorts of requirements I have for the network, what things I want to learn and experiment with both now and in the future, general network security best practices, and fundamental routing and networking requirements. 

### Design Goals
1. A full-fledged Active Directory environment - AD was the main reason I built this lab. I wanted to research AD security and learn to to administrate and AAD environment.
2. No extra hardware requirements beyond my computer - I was not looking to buy any hardware for this; I needed this cyber range to function on a single desktop (thankfully with a decent amount of RAM).
3. Networking involvement - I wanted my lab to include as many network devices as possible. I am interested in everything IT related, I wanted to set up my first network and get experience with new technologies.
4. Replicate a true enterprise environment - I wanted this to mimic a real network I will see at work one day. That was another reason I wanted to have networking devices on there, to better mimic a real network. Additionally, for the security testing, I wanted a realistic environment to research on. Nowadays, most companies will have strong firewalls, if not NGFWs, EDR, top-of-the-line anti-virus, strict group policy, etc I wanted to mimic that.

### Constraints
1. Money - my goal with this was to not have to spend any additional money. There is so much awesome open source tech out there, I knew this would not be a problem. Of course microsoft is pretty strict about their OS, but things worked out just fine. This constraint is why I didn't want to go out and buy hardware, sure it would be cool, but I do not have space for it anyways, maybe one day.
2. Virtualization constraints - There's a lot of interesting stuff with virtualization, as we likely all know. At least for VirtualBox, but I'd bet for VMWare too, switching is performed either at the hypervisor level or just on your physical network. What this means is that you cannot have a true switch in that, without it, traffic would not be able to flow.
   - This was sort of limiting, and made it so the topology is not as realistic as it could've been otherwise
   - I still did manage to set up one switch, while it was not entirely necessary, it was still kind of interesting
   - I've been going through a CCNA exam course and doing a bunch of labs there too, so the main purpose of this lab was not to do switching, I can do that on Cisco's Packet Tracer
   - I also had to really think carefully about what technologies I could use in the network since everything had to be made into a VM, like you cannot typically make a Cisco device into a VirtualBox VM (certainly not legally, at least).

***

### High-Level Topology Overview
The environment consists of two firewall routers connected via a point-to-point (/30) transit network, with each managing distinct trust zones. 

- pfSense Firewall - Handles routing to the internet via NAT network adapter, has an attacker subnet, which is where I spend most of my time remotely managing, monitoring, and performing exploits, it also has a subnet exclusively for standalone vulnerable machines and labs (mostly from Vulnhub).
- OPNSense Firewall - This models an internal enterprise firewall/router, connected to it is the Active Directory network, the monitoring network, and a subnet on which VPN clients land. We host an OpenVPN server on this firewall, and the corresponding subnet is able to communicate with both the AD and the monitoring subnet. We also created a switch virtual machine that while not necessary to allow traffic to flow, is still useful for visualization, realism, and expierimentation.  

I chose this architecture to best mimic a modern enterprise network, whereby there are deliberate trust boundaries/security zones separated by internal routed firewalls. 

***

### Security Zones and Trust Boundaries 

1. pfSense
   - The attacker machine is largely free to communicate with any machines in the vulnerable subnet
   - Attacker machine must be able to communicate with the AD and monitoring subnets
The pfSense environment models an external or semi-trusted security zone, representing attacker-controlled systems, unmanaged/legacy hosts, and test infrastructure in an enterprise network. This zone would be intentionally assumed to be hostile and would be subject to strict access controls when communicating with internal enterprise networks. The pfSense firewall is the first boundary between our internal network and the internet. 
   
2. OPNSense
   - This is the core internal network. The Active Directory subnet is the center of this entire lab, just as in an enterprise, this is where all of the business critical services and infrastructure lives, it is also the area of highest trust. 
   - We allow external access into the network through an OpenVPN server hosted on our firewall.
   - We have a dedicated monitoring subnet where we host a Wazuh instance that watches over the AD environment.
      - All AD machines have the Wazuh agent installed on them, which provides EDR.
      - I chose to isolate the monitoring machine into its own network so we can craft very specific firewall rules to control the flow of traffic to this network. Unauthorized access to a machine controlling our EDR/SIEM would be bad news, so I wanted some practice securing these critical systems.
   - Separating the monitoring machine from the AD network also allows us to add much more specific inbound rules for the AD network, too. This separation gives us much more control over what traffic gets into either network, instead of trying to create a single rule set that works for both networks.

The trust zones are as follows: 
   - VPN Subnet - conditional trust, users must have a valid configuration file with a certificate from our CA, and they must also have a valid credential on our firewall. With that said, users may be connecting via personal devices, which could inadvertently bring security risks onto the network. We want to be careful of exactly what traffic is flowing from this subnet into the AD subnet; this subnet really should never communicate with the monitoring subnet (unless we want to eventually put EDR on VPN client devices).
   - AD Subnet - internal, trusted, but also highly sensitive. Again, this subnet would hold the majority of critical infrastructure and services in a real environment. We want to heavily monitor all devices and netwok traffic on this network, limit east-west traffic, minimize/heavily restrict inbound traffic, have very strcit policys regarding passwords, elevated privledges, protocol usage, etc, we also want to heavily collect all authentication and security related logs. 
   - Monitoring - internal, potentially the most trusted subnet, but also potentially the most highly restricted. We want very specific firewall rules involving this submet, we really only want to allow EDR communication from machines on the AD network (takes place on one specific port for Wazuh), HTTP connections again on a specific port to access the Wazuh web GUI, and SSH to allow remote management of the monitoring VM only from a minimal number of static IPs. 
    
3. The Transit Network
   - This is the boundary from our untrusted DMZ network and the true internal network that holds the keys to the kingdom.
   - This transit network is a key trust security demarcation, trust boundary, and policy enforcement point. If we had our AD network and monitoring subnets connect directly to the pfSense firewall, I would argue that would make things much more unrealistic. This boundary was a core piece of the overall topology design.
   - The transit network is where we want to convert untrusted traffic into trusted traffic. We do this through very strict firewall rules on the OPNSense firewall/router, depending on the source, destination, and protocol.
   - We assume traffic coming into the OPNSense network through this boundary is malicious and use our firewall rules to only allow what we deem as being benign.
   - This subnet should be minimal, only two addresses here, should not host any services, because its traffic is untrusted.   

***

### Traffic Flow Philosophy

Network traffic in this lab is governed by a deny-by-default approach. Explicit allow rules are created only where required for specific tasks, such as monitoring capability or remote management. Traffic is generally expected to flow as follows:

- From lower-trust zones to higher-trust zones ONLY for certain permissible tasks, controlled by specific firewall rules
- From monitored assets toward the monitoring subnet for logging and telemetry, one single port is used for this data transmission
- From VPN clients to specific internal services on a least-privilege basis

East–west traffic within internal subnets is strictly controlled to reduce lateral movement opportunities.

***

### Layer 3–Focused Security Design

Due to virtualization constraints (switching is done automatically at the hypervisor or host network level) and modern enterprise best practices, security enforcement in this lab is performed primarily at Layer 3. Subnets are routed through firewall appliances rather than relying solely on Layer 2 segmentation.

This design prioritizes clear trust boundaries, explicit routing decisions, and centralized policy enforcement, mirroring how modern enterprise networks are commonly secured.

As mentioned earlier, we did include a switch, but it is not necessary for traffic flow and is just done for deeper inspection and experimentation. 

***

### Scalability and Future Use

I wanted this lab enviornment to be something I can grow in to and can adapt to my learning needs and desires as I progress in my carerr. 

There are ways to add more network adapters to VMs than those offered in the VirtualBox GUI using some basic PowerShell commands. 

The network is designed to be added to and has a lot of flexibility. So if I one day and studying for my OSCP or CEH in the future, and just want to quickly work on exploiting some labs that are recommended when studying for these, I can very quickly download the ISO, throw it in the Cyber Range, and am immediately able to work on exploiting it. Similarly, if I want to test out or learn a new monitoring software, deploy some Linux management machines for the AD, deploy more AD machines, and so forth, I can do that extremely easily since everything is virtualized and subnets are configured. 

The use of firewalls also allows for a ton of cool experiumentation and scalability type stuff. Just on the firewall alone, I can learn about and configure routing, static routes, DHCP configuration, dynamic routing protocols, port forwarding, NAT, VPNs, and a lot more. 

***

### Note

Again, this lab is built largely to perform exploits in an AD envionrment, so I change these rules a lot. The security design architecture laid out above is often broken to give myself the ability to perform various attacks, and eventually learn how to secure my network against them, and detect exploit attempts on the network. 

I did design the network in a security-focused way, both to gain expierence in creating a security focused network, and also to see the benefits and limitations of these network security principles.

***

### Topology Image (for reference, same as in repo README)
![Image of the cyberr range home lab topology](https://raw.githubusercontent.com/koleada/Cyber-Range/refs/heads/main/Diagrams/Cyber%20Range%20Network%20Diagram.drawio%20(3).png)
