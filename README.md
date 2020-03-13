# PowerShell Scripts to Migrate User Home Drives to their OneDrive

I am dropping all of the scripts I have been making to migrate users' home drives to their work account's OneDrive. In my scenario, this project wasn't expected to be done in a short amount of time. Due to the concerns of coronavirus (COVID-19) and the probability of our workplace being switched to *Work-From-Home*/*Teach-From-Home*, which has just been officially enforced, we needed to fast track this project from months out to before the end of March 2020.

While we have not migrated users' home drives yet, I have been making these PowerShell scripts to quickly gather data about the users' home drive directory, if the user is licensed, and building a CSV file for bulk migration jobs utilizing the [SharePoint Migration Tool](https://docs.microsoft.com/en-us/sharepointmigration/introducing-the-sharepoint-migration-tool). This recently released tool from Microsoft allows administrators to migrate not just SharePoint sites, but also file shares.

I do want to note that not all of the scripts are ready to be pushed to this public repository, but I do heavily intend to get them on here as fast as I can. I do know that this current situation the world is in of coronavirus (COVID-19) is taxing a lot of IT departments out there because of the transition from on-premise resources to *cloud-based* resources. This is one of those instances where off-loading on-premise resources to the cloud, will alleviate the need for users to connect back with a VPN. In the end I hope this helps other administrators in the future, even after today's current events.

I hope this repository will help anybody out there!

## Status of Repository

- [x] Invoke-UserHomeMigrationBuilder.ps1
- [x] Get-FolderFileSize.ps1
- [ ] Get-AadUserStatus.ps1
    - In the process of removing organization references.
- [ ] Script to run the SharePoint Migration Tool using the [PowerShell cmdlets available](https://docs.microsoft.com/en-us/powershell/spmt/intro?view=spmt-ps).

***Other scripts I create for this process will be added in when necessary.***

## Script Details

### Get-FolderFileSize.ps1

This script is intended to help gather data from a folder and determine when the last time a file was written to it and how large the total size of the folder is. It also converts the total bytes into total gigabytes to make the output easier to read. In my case with our users' home drives, I needed to identify which users had the largest amount of storage. My methodology with doing the migration jobs utilizing the **SharePoint Migration Tool** was to create batches of jobs depending on who has the most data and how many to do based off of that.

To get the total size of a single folder, you can run the script like this:

```powershell
PS \> .\Get-FolderFileSize.ps1 -FolderPath ".\path\to\folder\"
```

The output would be something like this:

```
DirectoryPath          LastWrittenFileDate  SizeInGB
-------------          -------------------  --------
C:\path\to\folder\     9/17/2019 7:43:13 AM   0.0824
```

To get the total size of all the folders in directory, you can run the script like this:

```powershell
PS \> Get-ChildItem -Path "C:\path\to\folder\" | ForEach-Object { .\Get-FolderFileSize.ps1 -FolderPath $PSItem.FullName }
```

The output would be something like this:
```
DirectoryPath          LastWrittenFileDate  SizeInGB
-------------          -------------------  --------
C:\path\to\folder\a    9/17/2019 7:43:13 AM   12.7824
C:\path\to\folder\b    10/20/2016 8:00:43 AM  0.5334
C:\path\to\folder\c    4/21/2019 3:30:04 PM   3.9221
C:\path\to\folder\d    9/01/2012 1:03:19 AM   0.0020
```

### Invoke-UserHomeMigrationBuilder.ps1

This script will utilize the `ActiveDirectory` module that comes installed with the **Remote Server Administration Tools** on Windows and Windows Server to get user objects and create the bulk migration job CSV file that can be imported into the **SharePoint Migration Tool**. You supply the usernames, Office 365 tenant domain name, and the subfolder you want to put into a user's OneDrive directory.

An example of using it would be like this:

```powershell
PS \> .\Invoke-UserHomeMigrationBuilder.ps1 -UserName @("jdoe1", "jwinger", "pryan") -TenantName "contoso.com" -SPOSubFolder "UserHome Migration" -ExportPath ".\UserHomeDir-MigrationJob.csv"
```

This will create a CSV file in the current directory that is formatted to import into the **SharePoint Migration Tool**.

**NOTE**: The way this script works, it only checks the `HomeDirectory` property that is on the user's Active Directory object. If you map their home directory differently, you can re-tool it to fit your needs.