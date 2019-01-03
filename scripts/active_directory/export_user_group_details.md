<#

This script can be used to list all users in a specific Active Directory group and export to .csv.

You can edit the "Select" statement to obtain additional informaion if required.

#>

Raw .ps1 code below

```
Get-ADGroupMember -Identity "VPN-Partner-Dataart" -Recursive |
Where objectclass -eq 'user' |
Get-ADUser -Properties Displayname,Title,Department |
Select DistinguishedName,samAccountName,Name,Displayname,Title,Department |
Export-CSV c:\temp\dataart.csv
```


