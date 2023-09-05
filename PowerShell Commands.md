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
