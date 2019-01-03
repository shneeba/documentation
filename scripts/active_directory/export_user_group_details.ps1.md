# Export user group details to .csv file

This script can be used to list all users in a specific Active Directory group and export to .csv.

You can edit the "Select" statement to obtain additional informaion if required.

```
#Select the defined group and get all users

Get-ADGroupMember -Identity "VPN-Partner-Dataart" -Recursive |
Where objectclass -eq 'user' |

#For each users get the following properties
Get-ADUser -Properties Displayname,Title,Department |

#Select the following
Select DistinguishedName,samAccountName,Name,Displayname,Title,Department |

#Export it to .csv
Export-CSV c:\temp\dataart.csv
```


