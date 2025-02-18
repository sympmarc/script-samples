---
plugin: add-to-gallery
---

# Scan for Microsoft 365 Groups created with user's first or last name

## Summary

We can use the group naming policy to enforce a consistent naming strategy for groups created by users in our organization. A naming policy can help us and our users identify the function of the group. We can use the policy to block specific words from being used in group names and aliases. But what if we need to find out the list of Microsoft 365 groups created with user’s givenName or surname as their mail?

This sample script scans the Microsoft 365 groups that may contain user’s first or last name as the group mail.

> [!Note]
> The filter condition can be changed as per your requirement.

# [CLI for Microsoft 365 with PowerShell](#tab/cli-m365-ps)
```powershell
$groupsToFlag = @()

$users = m365 aad user list --properties 'displayName,givenName,surname' -o json | ConvertFrom-Json
$groups = m365 aad o365group list -o json | ConvertFrom-Json

foreach ($user in $users) {
  $userGivenName = $user.givenName
  $userSurname = $user.surname

  if ($userGivenName -and $userSurname) {
    $groupsMatch = $groups | Where-Object { $_.mail -like "*$userGivenName*" -or $_.mail -like "*$userSurname*" }

    foreach ($group in $groupsMatch) {
      $groupObject = New-Object -TypeName PSObject
      $groupObject | Add-Member -MemberType NoteProperty -Name "groupId" -Value $group.id
      $groupObject | Add-Member -MemberType NoteProperty -Name "groupMail" -Value $group.mail
      $groupObject | Add-Member -MemberType NoteProperty -Name "userGivenName" -Value $userGivenName
      $groupObject | Add-Member -MemberType NoteProperty -Name "userSurname" -Value $userSurname
      $groupsToFlag += $groupObject
    }
  }
}

$groupsToFlag | Format-Table -AutoSize
```
[!INCLUDE [More about CLI for Microsoft 365](../../docfx/includes/MORE-CLIM365.md)]
 
# [Microsoft 365 CLI with Bash](#tab/m365cli-bash)
```bash
#!/bin/bash
# requires jq: https://stedolan.github.io/jq/

defaultIFS=$IFS
IFS=$'\n'

groupsToFlag=()
users=`m365 aad user list --properties 'displayName,givenName,surname' -o json`
groups=`m365 aad o365group list  -o json`

for user in `echo $users | jq -c '.[]'`; do
  userGivenName=`echo $user | jq -r '.givenName'`
  userSurname=`echo $user | jq -r '.surname'`

  if [ ! -z "$userGivenName" ] || [ !-z "$userSurname" ] then
    groupsMatch=$(echo $groups | jq -c --arg GivenName "$userGivenName" --arg Surname "$userSurname" 'map(select((.mail|ascii_downcase|contains($GivenName|ascii_downcase)) or (.mail|ascii_downcase|contains($Surname|ascii_downcase))))')

    for group in `echo $groupsMatch | jq -c '.[]'`; do 
      groupId=`echo $group | jq  -r '.id'`
      groupMail=`echo $group | jq  -r '.mail'`
      groupObject=$(jq -n -c \
        --arg GroupId "$groupId" \
        --arg GroupMail "$groupMail" \
        --arg UserGivenName "$userGivenName" \
        --arg UserSurname "$userSurname" \
        '{groupId: $GroupId, groupMail: $GroupMail, userGivenName: $UserGivenName, userSurname: $UserSurname}')

      groupsToFlag+=($groupObject)
    done
  fi
done

echo ${groupsToFlag[@]} | jq -csr '(.[0] |keys_unsorted | @tsv), (.[]|.|map(.) |@tsv)' | column -s$'\t' -t

IFS=defaultIFS
exit 1
```
[!INCLUDE [More about CLI for Microsoft 365](../../docfx/includes/MORE-CLIM365.md)]
***

## Source Credit

Sample first appeared on [Scan for Microsoft 365 Groups created with user's first or last name | CLI for Microsoft 365](https://pnp.github.io/cli-microsoft365/sample-scripts/aad/flag-groups-with-user-names/)

## Contributors

| Author(s) |
|-----------|
| Joseph Velliah |


[!INCLUDE [DISCLAIMER](../../docfx/includes/DISCLAIMER.md)]
<img src="https://telemetry.sharepointpnp.com/script-samples/scripts/aad-flag-groups-with-user-names" aria-hidden="true" />