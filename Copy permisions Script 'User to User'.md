--Powershell Script--

$sourceUser = "Akhil"
$targetUser = "Ashok"

Get-ADUser -Identity $sourceUser -Properties MemberOf |
Select-Object -ExpandProperty MemberOf |
ForEach-Object {
    Add-ADGroupMember -Identity $_ -Members $targetUser
}

