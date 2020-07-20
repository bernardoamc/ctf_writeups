# Kerberos 
#pentest/tryhackme

### What
Kerberos is the authentication system for Windows and Active Directory networks. It’s a network authentication protocol based on tickets. It Allows parties, client/server for example, to authenticate each other over an insecure network channel, provided that both parties trust a third party, the Kerberos server.

### Main components
* KDC (Kerberos Distribution Center or Key Distribution Center)
	* It’s our domain control
* The client requesting access
* The service the client is attempting to obtain access to 

### How does it work
Kerberos uses shared secrets for authentication. In a Windows domain there is only one, the NTLM Hash.

**Steps:**
![](Kerberos/57AFCBDD-C2AA-4E1F-B5BA-D4A0FA458C9B.png) -> ![](Kerberos/A3439AA9-729C-4642-BF8C-FF0E6F9FD3F6.png) -> ![](Kerberos/0C4F17F3-38F5-4E7A-B1EA-49B0B48CF90C.png)

### Steps in depth

![](Kerberos/Screen%20Shot%202020-07-08%20at%204.13.45%20PM.png) -> ![](Kerberos/Screen%20Shot%202020-07-08%20at%204.15.20%20PM.png) -> ![](Kerberos/CA9C70A3-9D48-4471-9BB7-E2E13E73583C.png) -> 
-> ![](Kerberos/31BFCB62-16ED-47D7-A3C9-02D25891DE33.png) 

**So what is a PAC?**
It stands for privilege attribute certificate and it’s something that’s inside the Server half of the ticket and is also inside the TGT. It contains information about myself like my account name and so on. It has two signatures:
1. The service account hash
2. The KDC hash

![](Kerberos/3B2534E3-AF4B-4D3A-A9A8-5E1608CC05E8.png)

**Ticket breakdown:**
![](Kerberos/E96503ED-D5A1-4133-B013-D438E16E2CBC.png)

**Cool, but how do the KDC knows which service I’m talking about?**
The SPN or Service Principal Name takes care of this! It’s just a mapping between service and account.

![](Kerberos/61FFF32D-5E4B-4B69-9F18-FA14204A1219.png)  ->  ![](Kerberos/725B7534-6E50-4B0E-99F5-0741C0787521.png)

### Attack vectors
There are three long term keys in play in this protocol. If we can exploit any of them we can pretty much gain entry in certain parts of the system.
![](Kerberos/31838EBF-D94B-40C7-8673-2C856EFA03EE.png)

### Attacks

**Golden ticket**
![](Kerberos/D8D80D75-91EC-4FEB-B4A4-C161E72C3A79.png) -> ![](Kerberos/73295773-0C5C-4085-8AE7-67B48C367FC7.png)
![](Kerberos/BF6C14F1-AF97-46D4-8D16-C9FC42C8C437.png) -> ![](Kerberos/C71341E4-CB4A-4D10-9596-CE044CA58028.png)

**Skeleton key**
![](Kerberos/582B511B-E7D9-489C-B2DF-DFEA057B951B.png)

**Kerberoasting**
![](Kerberos/6281B301-15C8-4495-9DD8-07F21D74A7C0.png) -> ![](Kerberos/85ED4C46-D5CC-46D0-A470-7A8B40B0B7D9.png)

**Silver key**
![](Kerberos/E513AC4D-0659-41C0-B810-CD6685491534.png) -> ![](Kerberos/B3F5F516-2443-4CB8-94C8-9AFD3C08E5AB.png)

**When to use each?**
![](Kerberos/C7144088-9267-4796-9C68-8836C1DDC4CE.png)

### Resources
* [Kerberos 101](https://www.youtube.com/watch?time_continue=82&v=LmbP-XD1SC8&feature=emb_logo)
* [GitHub - nidem/kerberoast](https://github.com/nidem/kerberoast)
