# Export user count in OU

```
Import-Module activeDirectory  

#Populate variables
$output = Read-Host "'Y' for output to file or any key for output in GUI table view" -foreground Cyan
$fqdn = Read-Host "Enter FQDN domain"  
$cred = Get-Credential  

#Contact the DC
Write-Host "Contacting $fqdn domain..." -ForegroundColor Yellow  

#Populate the $domain variable
$domain = (get-addomain $fqdn -Credential $cred | select distinguishedName,pdcEmulator,DNSroot,DomainControllersContainer)  

#Inform user that we're calculating the OUs
Write-Host "Completed. Enumerating OUs.." -ForegroundColor Yellow  

#Populate the $OUlist variable with array data
$OUlist = @(Get-ADOrganizationalUnit -filter * -Credential $cred -SearchBase $domain.distinguishedName -SearchScope Subtree -Server $domain.DNSroot)  

#Inform user that we're now counting the users
Write-Host "Completed. Counting users..." -ForegroundColor Yellow  

#Create a progress bar counting the OUs
for($i = 1; $i -le $oulist.Count; $i++)  
    {write-progress -Activity "Collecting OUs" -Status "Finding OUs $i" -PercentComplete ($i/$OUlist.count*100)}  
$newlist = @{}  
  
  
#Count the number of users in each OU list created from the $OUlist variable
foreach ($_objectitem in $OUlist)  
    {  
    $getUser = Get-ADuser -Filter * -Credential $cred -SearchBase $_objectItem.DistinguishedName -SearchScope OneLevel -Server $domain.pdcEmulator | measure | select Count  
    for($i = 1; $i -le $getUser.Count; $i++)  
    {write-progress -Activity "Counting users" -Status "Finding users $i in $_objectitem" -PercentComplete ($i/$getUser.count*100)}  
      
    $newlist.add($_objectItem.DistinguishedName, $getUser.Count)      
    }  
 
#Either write to file or write to console depending on the $output variable populated at the start
 if ($output -eq "Y")
 {
 $newlist | ft -AutoSize | Out-File .\OUuserCount.txt 
 Write-Host "All done!" -ForegroundColor yellow   
 }
 else 
 {
 $newList | Out-GridView
 }
```
 
