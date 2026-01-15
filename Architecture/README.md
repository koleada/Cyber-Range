# Home Lab Topology & Architecture 

I spent a pretty good amount of time planning the best approach for building this home lab envionrment including determining what technologies I should use, what sorts of requirements I have for the network, what things I want to learn and expiriment with both now and in the future, general network security best practices, and fundamental routing and networking requirements. 

### Design Goals
1. A full fledged Active Directory envionrment - AD was the main reason I built this lab. I wanted to reasearch Ad security and learn to to administrate and AAD enviornment.
2. No extra hardware requirements beyond my computer - I was not looking to buy any hardware for this, I needed this cyber range to function on a single, desktop (thankfully with a decent amount of RAM).
3. Networking involvement - I wanted my lab to include as many network devices as possible, I am interested in everything IT related, I wanted to set up my first network and get expierence with new technologies.
4. Replicate a true enterprise enviornment - I wanted this to micic a real network I will see at work one day. That was another reason I wanted to have networking devices on there, to better mimic a real network. Additionally, for the security testing, I wanted a realistic enviornment to research on. Nowadays, most companies will have strong firewalls, if not NGFWs, EDR, top of the line anti-virus, strcit group policy, etc, I wanted to mimic that.

### Constraints
1. Money - my goal with this was to not have to spend any additional money. There is so much awesome open source tech out there I knew this would not be a problem. Of course microsoft is pretty strict about there OS, but things worked out just fine. This constraint is why I didn't want to go out and buy hardware, sure it would be cool, but I do not have space for it anyways, maybe one day.
2. Virtualization constraints - Theres a lot of interesting stuff with virtualization as we likely all know. At least for VirtualBox, but I'd bet for VMWare too, switching is performed either at the hypervisor level or just on your phsycial network. What this means is that you cannot have a true switch in that without it traffic would not be able to flow.
   - This was sort of limiting, and made it so the topology is not as realsitic as it could've been otherwise
   - I still did manage to set up one switch, while it was not entirely necessary it was still kind of interesting
   - I've been going through a CCNA exam course and doing a bunch of labs there too, so the main purpose of this lab was not to do switching, I can do that on cisco's packet tracer
   - I did also have to really look into what technologies I could use in the netwrok since everything had to be made into a VM, like you cannot typically make a cisco device into a virtualbox VM (certinaly not legally at least).
  
  
