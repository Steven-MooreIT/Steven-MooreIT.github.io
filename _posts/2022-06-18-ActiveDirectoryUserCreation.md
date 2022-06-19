---
title: Create Active Directory User
date: 2022-06-18 11:23:50 -0700
categories: [Powershell, Create AD User]
tags: [powershell,scripting,windows]     # TAG names should always be lowercase
---


# Powershell Script to generate Active Directory Users

```powershell

$JsonFile = ".\data\ad_schema.json"
$jsonObject = Get-Content $JsonFile | ConvertFrom-Json
$Global:domain = $jsonObject.domain

function Register-NewADUser() {
    param (
        [Parameter(Mandatory=$true)] $User
    )
    $firstName, $lastName = $User.name.Split(" ")
    $emailAddress = "{0}.{1}@{2}" -f $firstName, $lastName, $Global:domain
    $samAccountName = $emailAddress.Substring(0, $emailAddress.IndexOf('@'))
    $password= -join("!@#$%^*0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ_abcdefghijklmnopqrstuvwxyz".tochararray() | ForEach-Object {[char]$_} | Get-Random -Count 10)
    $Attributes =@{
        Enabled = $true
        ChangePasswordAtLogon = $false
        Path = $User.oupath
        Givenname = $firstName
        Surname = $lastName
        DisplayName = $firstName + " " + $lastName
        Name = $User.name
        SamAccountName = $samAccountName
        UserPrincipalName = $emailAddress
        Description = $User.description
        EmailAddress = $emailAddress
        Office = $User.office
        Company = $User.company
        Department = $User.department
        Title = $User.title
        City = $User.city
        State = $User.state
        AccountPassword = (ConvertTo-SecureString $password -AsPlainText -Force)
    }
    if (Get-ADUser -Filter {SamAccountName -eq $samAccountName}) {
        Write-Warning "User $($samAccountName) already exists, please check the name and try again."
        return
    }
    New-ADUser @Attributes
    foreach ($Groups in $User.groups) {
        Add-ADGroupMember -Identity $Groups -Members $samAccountName
    }
    Write-Output "CREDENTIALS:`n[USERNAME]: $($emailAddress) `n[PASSWORD]: $($password)"
}

foreach ($User in $jsonObject.users) {
    Register-NewADUser $User
}

```

## Here is the base JSON template file to follow

```json
{
    "domain" : "zrytekk.com",
    "users" : [
        {
            "name" : "Bob Yeet",
            "groups" : ["Marketing", "Engineering"],
            "oupath" : "OU=Domain Users,DC=ABC,DC=com",
            "description" : "Marketing and Engineering team user",
            "office" : "Home Office",
            "company" : "ABC",
            "department" : "Engineering",
            "title" : "Engineering I",
            "city" : "City01",
            "state" : "State01"
        },
        {
           "name" : "Test User",
            "groups" : ["IT", "Managers"],
            "description" : "IT Manager",
            "oupath" : "OU=Domain Users,DC=ABC,DC=com",
            "office" : "Remote",
            "company" : "ABC",
            "department" : "Manager",
            "title" : "INF Manager",
            "city" : "City01",
            "state" : "State01"
        }
    ]
}
```