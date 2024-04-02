---
title: Windows开启WinRM服务
date: 2024-04-02
tags: [winrm,Windows]
---
## 配置 WinRM（HTTP）服务

1 登录 Windows 服务器，并以管理员身份运行 PowerShell。

2 配置完成后, 可以通过以下命令确认是否配置成功


   ```powershell
   winrm enumerate winrm/config/listener
   ```

3 运行以下命令配置 WinRM 服务：


   ```powershell
   winrm quickconfig -quiet
   winrm set winrm/config/service/auth '@{Basic="true"}'
   winrm set winrm/config/service '@{AllowUnencrypted="true"}'
   Set-Item WSMan:\localhost\Shell\MaxShellsPerUser 100
   Set-Item WSMan:\localhost\Shell\MaxMemoryPerShellMB 300
   Set-item wsman:/localhost/listener/listener*/port 5985
   ```

## 配置 WinRM（HTTPS）服务

1 下载 `enable_winrm_ssl.ps1` 脚本并上传到服务器的C盘根目录。


   ```powershell
   $SubjectName = $env:COMPUTERNAME
   $CertValidityDays = 1095
   
   Function New-LegacySelfSignedCert {
       Param (
           [string]$SubjectName,
           [int]$ValidDays = 1095
       )
   
       $hostnonFQDN = $env:computerName 
       $hostFQDN = [System.Net.Dns]::GetHostByName(($env:computerName)).Hostname
       $SignatureAlgorithm = "SHA256"
   
       $name = New-Object -COM "X509Enrollment.CX500DistinguishedName.1"
       $name.Encode("CN=$SubjectName", 0)
   
       $key = New-Object -COM "X509Enrollment.CX509PrivateKey.1"
       $key.ProviderName = "Microsoft Enhanced RSA and AES Cryptographic Provider"
       $key.KeySpec = 1
       $key.Length = 4096
       $key.SecurityDescriptor = "D:PAI(A;;0xd01f01ff;;;SY)(A;;0xd01f01ff;;;BA)(A;;0x80120089;;;NS)"
       $key.MachineContext = 1
       $key.Create()
   
       $serverauthoid = New-Object -COM "X509Enrollment.CObjectId.1"
       $serverauthoid.InitializeFromValue("1.3.6.1.5.5.7.3.1")
       $ekuoids = New-Object -COM "X509Enrollment.CObjectIds.1"
       $ekuoids.Add($serverauthoid)
       $ekuext = New-Object -COM "X509Enrollment.CX509ExtensionEnhancedKeyUsage.1"
       $ekuext.InitializeEncode($ekuoids)
   
       $cert = New-Object -COM "X509Enrollment.CX509CertificateRequestCertificate.1"
       $cert.InitializeFromPrivateKey(2, $key, "")
       $cert.Subject = $name
       $cert.Issuer = $cert.Subject
       $cert.NotBefore = (Get-Date).AddDays(-1)
       $cert.NotAfter = $cert.NotBefore.AddDays($ValidDays)
   
       $SigOID = New-Object -ComObject X509Enrollment.CObjectId
       $SigOID.InitializeFromValue(([Security.Cryptography.Oid]$SignatureAlgorithm).Value)
   
       [string[]] $AlternativeName += $hostnonFQDN
       $AlternativeName += $hostFQDN
       $IAlternativeNames = New-Object -ComObject X509Enrollment.CAlternativeNames
   
       foreach ($AN in $AlternativeName) {
           $AltName = New-Object -ComObject X509Enrollment.CAlternativeName
           $AltName.InitializeFromString(0x3, $AN)
           $IAlternativeNames.Add($AltName)
       }
   
       $SubjectAlternativeName = New-Object -ComObject X509Enrollment.CX509ExtensionAlternativeNames
       $SubjectAlternativeName.InitializeEncode($IAlternativeNames)
   
       [String[]]$KeyUsage = ("DigitalSignature", "KeyEncipherment")
       $KeyUsageObj = New-Object -ComObject X509Enrollment.CX509ExtensionKeyUsage
       $KeyUsageObj.InitializeEncode([int][Security.Cryptography.X509Certificates.X509KeyUsageFlags]($KeyUsage))
       $KeyUsageObj.Critical = $true
   
       $cert.X509Extensions.Add($KeyUsageObj)
       $cert.X509Extensions.Add($ekuext)
       $cert.SignatureInformation.HashAlgorithm = $SigOID
       $CERT.X509Extensions.Add($SubjectAlternativeName)
       $cert.Encode()
   
       $enrollment = New-Object -COM "X509Enrollment.CX509Enrollment.1"
       $enrollment.InitializeFromRequest($cert)
       $certdata = $enrollment.CreateRequest(0)
       $enrollment.InstallResponse(2, $certdata, 0, "")
   
       # extract/return the thumbprint from the generated cert
       $parsed_cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
       $parsed_cert.Import([System.Text.Encoding]::UTF8.GetBytes($certdata))
   
       return $parsed_cert.Thumbprint
   }
   
   $thumbprint = New-LegacySelfSignedCert -SubjectName $SubjectName -ValidDays $CertValidityDays
   Write-Host "Self-signed SSL certificate generated; thumbprint: $thumbprint"
   
   $valueset = @{
       CertificateThumbprint = $thumbprint
       Hostname = $SubjectName
   }
   
   $selectorset = @{
       Address   = "*"
       Transport = "HTTPS"
   }
   
   New-WSManInstance -ResourceURI 'winrm/config/Listener' -SelectorSet $selectorset -ValueSet $valueset
   
   Set-Item WSMan:\localhost\Service\Auth\Basic true
   Set-Item WSMan:\localhost\Shell\MaxShellsPerUser 100
   Set-Item WSMan:\localhost\Shell\MaxMemoryPerShellMB 300
   
   ```

2 以管理员身份运行 PowerShell，执行以下命令：


   ```
   Type c:\enable_winrm_ssl.ps1 | PowerShell.exe -noprofile -
   ```
