# Get unlicenced users

```
#Import the msonline module
import-module msonline 

#Connect to O365
Connect-MsolService

#Get all users, maximum of 10000 and unlicenced only. Export to .csv file.
Get-MsolUser -maxresults 10000 -UnlicensedUsersOnly | export-csv .\UnlicensedUsers.csv

#Script finished and requires user interaction
Write-Host "Script completed, press any key to continue ..."

$x = $host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
```
