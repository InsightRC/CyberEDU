# shigehira

### Category: Forensics | Difficulty: Easy | Tenant: cyberedu

### Q1. Specificați toate extensiile descărcate care au fost utilizate, în ordinea folosirii lor.

We can use http filter in wireshark and get all extensions in cronological order.

OSCN{x86,mips,mpsl}

### Q2. Care sunt User Agent-urile (în ordinea cronologică a marcajelor de timp) pentru fișierele care au fost instalate de dropper-ul găsit în pcap?

Follow http stream and get the User Agents in cronological order

OSCN{Wget/1.20.3-curl/7.68.0}

### Q3. Identifică cei 2 producători de plăci de rețea (NIC) ai celor două dispozitive implicate în cele mai multe tentative de conexiune.

Look at Layer 2 (Ethernet) headers. Use a MAC address vendor lookup, if you have not set wireshark to automatically show you the vendor.

OSCN{hewlett packard,netgear}

###  Q4. Care sunt cele 2 numere de port utilizate care încearcă să realizeze conexiuni post-infecție?

Look at the many many many TCP SYN flooding packets, at the port they are trying to connect to

OSCN{23,37215}

### Q5. Ce CVE utilizează variația malware-ului principal?

The family of the malware ( see this https://github.com/altilunium/jaws.exploit and this https://www.virustotal.com/gui/file/d0f7a59eb33c351c2d856b9b8153d16552bba1d912d79bcd901127cee891f9b7/detection), the malware family associated with the ports it scanned (Q4) is Satoshi. Search on malpedia for the CVE: https://malpedia.caad.fkie.fraunhofer.de/details/elf.satori

CVE-2014-8361
