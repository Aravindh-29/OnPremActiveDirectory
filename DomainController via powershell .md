## After creation of Server 

1. open powershell as Admin and run below commands
cmd :
```
   Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```
2 . After finishing above execute below
cmd :
```
Install-ADDSForest `
 -DomainName "goat.com" `
 -DomainNetbiosName "GOAT" `
 -SafeModeAdministratorPassword (ConvertTo-SecureString "Goat@123" -AsPlainText -Force) `
 -Force

```
