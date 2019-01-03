# Simple list of all services and their service names (to be used in scripts ect)

```
#Gets all services and sorts them by DisplayName, format the table in x order

Get-Service | Sort-Object -Property DisplayName | Format-Table DisplayName, Name -AutoSize 
```
