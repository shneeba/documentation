# Get O365 user licences

```
#Import the O365 modules
import-module msonline 

#Connect to O365
Connect-MsolService 

#CSV file picker module start 
Function Get-FileName($initialDirectory) 
{ 
[System.Reflection.Assembly]::LoadWithPartialName(“System.windows.forms”) | 
Out-Null 

$OpenFileDialog = New-Object System.Windows.Forms.OpenFileDialog 
$OpenFileDialog.initialDirectory = $initialDirectory 
$OpenFileDialog.filter = “All files (*.*)| *.*” 
$OpenFileDialog.ShowDialog() | Out-Null 
$OpenFileDialog.filename 
} 
#CSV file picker module end 

#Variable that holds CSV file location from file picker 
$path = Get-FileName -initialDirectory “c:\” 

#Window with list of available 365 licenses and their names 
$msolaccount = Get-MsolAccountSku | out-gridview -PassThru 

#Input window where you provide the license package’s name 
$AccountSKU = $msolaccount.AccountSkuId 

#CSV import command and get licence service status from each UserPrincipalName in the .csv file.
import-csv $path | foreach { 
Get-MsolUser -UserPrincipalName $_.UserPrincipalName.Licenses[0].ServiceStatus

#Result report on licenses assigned to imported users 
import-csv $path | Get-MSOLUser | out-gridview

Write-Host "Press any key to continue ..."

$x = $host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
```
