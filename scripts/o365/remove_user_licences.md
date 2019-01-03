# Remove O365 user licences

```
#Import msonline module
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

#CSV import command. Use the UserPrincipalName from the CSV and make sure location is "GB" and remove licence
import-csv $path | foreach { 
Set-MsolUser -UserPrincipalName $_.UserPrincipalName -usagelocation “GB” 
Set-MsolUserLicense -UserPrincipalName $_.UserPrincipalName -RemoveLicenses "$AccountSKU" 
} 

#Result report on licenses removed from imported users 
import-csv $path | Get-MSOLUser | out-gridview

Write-Host "Press any key to continue ..."

$x = $host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
```
