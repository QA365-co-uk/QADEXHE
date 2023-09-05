# Install Windows Features
```
Install-WindowsFeature Server-Media-Foundation, NET-Framework-45-Core, NET-Framework-45-ASPNET, NET-WCF-HTTP-Activation45, NET-WCF-Pipe-Activation45, NET-WCF-TCP-Activation45, NET-WCF-TCP-PortSharing45, RPC-over-HTTP-proxy, RSAT-Clustering, RSAT-Clustering-CmdInterface, RSAT-Clustering-Mgmt, RSAT-Clustering-PowerShell, WAS-Process-Model, Web-Asp-Net45, Web-Basic-Auth, Web-Client-Auth, Web-Digest-Auth, Web-Dir-Browsing, Web-Dyn-Compression, Web-Http-Errors, Web-Http-Logging, Web-Http-Redirect, Web-Http-Tracing, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Lgcy-Mgmt-Console, Web-Metabase, Web-Mgmt-Console, Web-Mgmt-Service, Web-Net-Ext45, Web-Request-Monitor, Web-Server, Web-Stat-Compression, Web-Static-Content, Web-Windows-Auth, Web-WMI, Windows-Identity-Foundation, RSAT-ADDS
```

# Lab 3 - Configure hostnames and URLs for Exchange
```
Get-OutlookAnywhere | Select Server,ExternalHostname,Internalhostname
Get-OutlookAnywhere | Set-OutlookAnywhere -ExternalHostname AdatumQAxxxx.QAExHybrid.com -InternalHostname AdatumQAxxxx.QAExHybrid.com -ExternalClientsRequireSsl $true -InternalClientsRequireSsl $true -DefaultAuthenticationMethod NTLM
Get-MAPIVirtualDirectory | Set-MAPIVirtualDirectory -ExternalUrl https://AdatumQAxxxx.QAExHybrid.com/mapi -InternalUrl https://AdatumQAxxxx.QAExHybrid.com/mapi
Get-OwaVirtualDirectory | Set-OwaVirtualDirectory -ExternalUrl https://AdatumQAxxxx.QAExHybrid.com/owa -InternalUrl https://AdatumQAxxxx.QAExHybrid.com/owa
Get-EcpVirtualDirectory | Set-EcpVirtualDirectory -ExternalUrl https://AdatumQAxxxx.QAExHybrid.com/ecp -InternalUrl https://AdatumQAxxxx.QAExHybrid.com/ecp
Get-ActiveSyncVirtualDirectory | Set-ActiveSyncVirtualDirectory -ExternalUrl https://AdatumQAxxxx.QAExHybrid.com/Microsoft-Server-ActiveSync -InternalUrl https://AdatumQAxxxx.QAExHybrid.com/Microsoft-Server-ActiveSync
Get-WebServicesVirtualDirectory | Set-WebServicesVirtualDirectory -ExternalUrl https://AdatumQAxxxx.QAExHybrid.com/EWS/Exchange.asmx -InternalUrl https://AdatumQAxxxx.QAExHybrid.com/EWS/Exchange.asmx -Force
Get-OabVirtualDirectory | Set-OabVirtualDirectory -ExternalUrl https://AdatumQAxxxx.QAExHybrid.com/OAB -InternalUrl https://AdatumQAxxxx.QAExHybrid.com/OAB
Get-ClientAccessService | Set-ClientAccessService -AutoDiscoverServiceInternalUri https://AdatumQAxxxx.QAExHybrid.com/Autodiscover/Autodiscover.xml
Set-OutlookProvider EXPR -CertPrincipalName:*.QAExHybrid.com
Get-ClientAccessService | Select Name,AutoDiscoverServiceInternalURI
```
# Lab 3 - Import Exchange Server certificate
```
Import-ExchangeCertificate -FileData ([System.IO.File]::ReadAllBytes('\\QAExHybridEx\Cert\QADEXHE.pfx')) -Password (ConvertTo-SecureString -String 'Pa$$w0rd' -AsPlainText -Force) -PrivateKeyExportable $true
```
# Lab 5a - Create Users, OUs and Exchange Objects
```
Import-Module ActiveDirectory

$Sid = Read-Host -Prompt 'Enter your student ID (AdatumQAxxxx)'
 
if ($Sid -notlike 'AdatumQA????') {Throw 'Invalid student ID'}

Get-ADForest | Set-ADForest -UPNSuffixes @{Add="$Sid.QAExHybrid.com"}


$OUs = 'Executive','Finance','Marketing','Operations','IT','Sales','Research'

$OUs | foreach {New-ADOrganizationalUnit $_}

$UsersCSV = `
 'FirstName,LastName,OrganizationalUnit' ,
 'Kim,Abercrombie,Executive',
 'Josh,Barnhill,Executive',
 'David,Campbell,Executive',
 'Brenda,Diaz,Finance',
 'Micheal,Emanuel,Finance',
 'Charles,Fitzgerald,Finance',
 'Jon,Ganio,Marketing',
 'Don,Hall,Marketing',
 'Lisa,Jacobson,Marketing',
 'John,Kelly,Marketing',
 'Jolie,Lenehan,Finance',
 'Sandra,Martinez,IT',
 'Lorraine,Nay,IT',
 'Harrold,Ortiz,IT',
 'Jeff,Price,IT',
 'Randy,Reeves,Sales',
 'Megan,Sherman,Sales',
 'Danielle,Tiedt,Sales',
 'Garrett,Vargas,Research',
 'Bryan,Walton,Research',
 'Tom,Young,Research'

 $Users = $UsersCSV | ConvertFrom-Csv
 $sp= 'Pa$w0rd'| ConvertTo-SecureString -AsPlainText -Force


new-aduser -GivenName Adam -Surname Able -UserPrincipalName "Adam@adam@$sid.qaexhybrid.com" -Name 'Adam Able' -AccountPassword $sp

new-aduser -GivenName Fred -Surname Flintstone -UserPrincipalName "FredF@$sid.qaexhybrid.local" -Name 'Fred Flintstone' -AccountPassword $sp

 $testcmd = Get-Command New-Mailbox -ErrorAction SilentlyContinue
if (! $testcmd)
 {
 Write-Verbose 'No Session Exists, connecting to Server'
 $Session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri http://QAEXHybridEX/powershell -Authentication Kerberos
 Import-PSSession $session
 }

New-AcceptedDomain -Name "$Sid.QAExHybrid.com" -DomainName "$Sid.QAExHybrid.com" -DomainType 'Authoritative'

Get-EmailAddressPolicy | Set-EmailAddressPolicy -EnabledEmailAddressTemplates @("SMTP:%m@$sid.QAExHybrid.com") 

Get-EmailAddressPolicy | Update-EmailAddressPolicy

 foreach ($i in $users)
{
 $upn=$i.FirstName + '@' + $Sid + '.QAExHybrid.com'
 $name = $i.FirstName + ' ' + $i.LastName
 New-Mailbox -Password $sp -DisplayName $name -UserPri $upn -Alias $i.FirstName -Name $name -OrganizationalUnit $i.OrganizationalUnit -LastName $i.LastName -FirstName $i.FirstName
}

New-SendConnector -Name 'Internet Send Connector' -Usage 'Internet' -DNSRoutingEnabled:$true -AddressSpaces @('SMTP:*;1') -IsScopedConnector:$false -SourceTransportServers @('QAEXHYBRIDEX')

New-Mailbox -Name PFMailbox1 -PublicFolder -OrganizationalUnit IT

'Executive','Finance','IT','Marketing','Operations','Research','Sales' | % {New-PublicFolder -Name $_} 

Get-PublicFolder -Recurse | where name -NotLike IPM_SUBTREE | Enable-MailPublicFolder

New-RetentionPolicy -Name 'QA Retention Policy' -RetentionPolicyTagLinks @('1 Month Delete','1 Year Delete','5 Year Delete','Default 2 year move to archive','Never Delete','Recoverable Items 14 days move to archive')


```
