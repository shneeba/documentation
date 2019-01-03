# Export users logins residing in an OU and outputs to .csv file using csvde


'''
#Define the OU we're looking in (-d) and output to file (-f)
csvde -d "ou=Finance,ou=users,ou=Kingston,dc=meridian,dc=local" -f .\users.csv

#Import the previous .csv file, select the UserPrincipalName and export to a new .csv file. -NoTypeInformation is included for backwards compatibility
Import-Csv .\users.csv | select UserPrincipalName | Export-Csv -Path .\output.csv –NoTypeInformation

#Simple "press any key to continue" prompt
Write-Host "Press any key to continue ..."
$x = $host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
'''
