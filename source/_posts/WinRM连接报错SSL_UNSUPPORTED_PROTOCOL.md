---
title: WinRM连接报错SSL_UNSUPPORTED_PROTOCOL
date: 2024-04-02
tags: [winrm,https]
---
>  参考：https://docs.ansible.com/ansible/latest/os_guide/windows_winrm.html#tls-1-2-support



使用WinRMl连接主机报错：


```
HTTPSConnectionPool(host='server', port=5986): Max retries exceeded with url: /wsman (Caused by SSLError(SSLError(1, '[SSL: UNSUPPORTED_PROTOCOL] unsupported protocol (_ssl.c:1056)')))
```

报错原因SSL协议版本不支持，需要升级服务器使用TLS v1.2或以上版本。Windows 8和Windows Server 2012默认安装并启用了TLS v1.2，但像Server 2008 R2和Windows 7这样的旧主机必须手动启用(这里待确认，发现Windows Server 2012 R2 也没有开启)。



1. 验证 Windows 主机支持的协议:



```
openssl s_client -connect <hostname>:5986
```

输出将包含有关TLS会话的信息，协议行将显示已经协商的版本:



```
Connecting to 172.16.52.159
CONNECTED(00000003)
Can't use SSL_get_servername
depth=0 CN=NODE-1
verify error:num=18:self-signed certificate
verify return:1
depth=0 CN=NODE-1
verify return:1
---
Certificate chain
 0 s:CN=NODE-1
   i:CN=NODE-1
   a:PKEY: rsaEncryption, 4096 (bit); sigalg: RSA-SHA256
   v:NotBefore: Mar 28 10:09:27 2024 GMT; NotAfter: Mar 28 10:09:27 2027 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
subject=CN=NODE-1
issuer=CN=NODE-1
---
No client certificate CA names sent
---
SSL handshake has read 1488 bytes and written 913 bytes
Verification error: self-signed certificate
---
New, TLSv1.2, Cipher is AES128-SHA256
Server public key is 4096 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : AES128-SHA256
    Session-ID: 384D000039807105789CF5CD1FA146D98784F7EB229076D3411C696F9A312405
    Session-ID-ctx:
    Master-Key: 535B8CA128376927EC549A2A7037686C62726DF9D6C840EDA4A81C9E2D6B54F2A7AF52FBADC22CF73FA95962106440E0
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    Start Time: 1711706775
    Timeout   : 7200 (sec)
    Verify return code: 18 (self-signed certificate)
    Extended master secret: no
---
read:errno=54
```



如果主机返回的是TLSv1，则应配置为启用TLS v1.2。您可以通过运行以下PowerShell脚本来实现这一点：(**注意需要重启**)



```powershell
Function Enable-TLS12 {
    param(
        [ValidateSet("Server", "Client")]
        [String]$Component = "Server"
    )

    $protocols_path = 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols'
    New-Item -Path "$protocols_path\TLS 1.2\$Component" -Force
    New-ItemProperty -Path "$protocols_path\TLS 1.2\$Component" -Name Enabled -Value 1 -Type DWORD -Force
    New-ItemProperty -Path "$protocols_path\TLS 1.2\$Component" -Name DisabledByDefault -Value 0 -Type DWORD -Force
}

Enable-TLS12 -Component Server

# Not required but highly recommended to enable the Client side TLS 1.2 components
Enable-TLS12 -Component Client

Restart-Computer
```



2. 查看服务器SSL/TLS配置

在Windows服务器上，WinRM服务的SSL/TLS配置通常是由Windows操作系统的Schannel组件来处理的，而不是由WinRM服务本身来处理。因此，需要查看Schannel的配置来了解WinRM服务的SSL/TLS配置。

以下是如何查看Schannel的SSL/TLS配置的步骤：

- 打开注册表编辑器（Regedit）。你可以在开始菜单中搜索"regedit"，然后点击"注册表编辑器"应用来打开它。
- 在注册表编辑器中，导航到以下路径：

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols
```

- 在这个路径下，你应该能看到几个文件夹，比如"SSL 2.0"，"SSL 3.0"，"TLS 1.0"，"TLS 1.1"，"TLS 1.2"，"TLS 1.3"等。这些文件夹代表了服务器支持的SSL/TLS版本。

![4392eee5146633841007471659f756d4.png](../../_resources/4392eee5146633841007471659f756d4.png)



3. 代码修改适配ssl2.0协议

配置TLS1.2 需要重启服务器，对于客户环境该要求过于理想，所以需要代码改造。

主要是重写 `requests.HTTPAdapter`类的 `init_poolmanager` 方法，在 `init_poolmanager` 方法中，创建了一个新的 SSL 上下文，并配置该上下文以使用 TLSv1 协议。然后，它通过 `ctx.options |= ssl.PROTOCOL_TLS` 语句启用了对所有版本的 SSL 和 TLS 协议的支持。最后，它创建了一个新的 `PoolManager` 实例，并将创建的 SSL 上下文传递给这个实例:



```Python
import ssl
import requests

from requests.adapters import HTTPAdapter
from requests.packages.urllib3.poolmanager import PoolManager
from requests.packages.urllib3.util import ssl_


class TLSAdapter(HTTPAdapter):
    """
    适配器类，用于处理https请求
    创建了一个新的 SSL 上下文，该上下文默认配置为使用 TLSV1 协议
    options指定为ssl.PROTOCOL_TLS, 处理其他所有版本的 SSL 和 TLS 协议
    使用：
    self.session.mount('https://', TLSAdapter())
    """
    def init_poolmanager(self, *pool_args, **pool_kwargs):
        ctx = ssl_.create_urllib3_context(ssl.PROTOCOL_TLSv1)
        ctx.options |= ssl.PROTOCOL_TLS
        self.poolmanager = PoolManager(*pool_args,
                                       ssl_context=ctx,
                                       **pool_kwargs)
# 这里示范怎么使用TLSAdapter类
session = requests.Session()
session.mount('https://', TLSAdapter())
```



我们对Winrm进行改造：



```python
class WinrmExectuor(ExecutorBase, winrm.Session, metaclass=ExecutorMeta):
    def __init__(self, target, auth, **kwargs):
        ...
    
    def run_cmd(self, command, args=()):

        # 兼容低版本协议ssl2.0
        if not self.protocol.transport.session:
            self.protocol.transport.build_session()
        self.protocol.transport.session.mount('https://', TLSAdapter())
        
        shell_id = self.protocol.open_shell(codepage=54936)  # 54936(gb18030) or 936(gbk)
        ...

    def async_run_cmd(self, command, args=()):

        # 兼容低版本协议ssl2.0
        if not self.protocol.transport.session:
            self.protocol.transport.build_session()
        self.protocol.transport.session.mount('https://', TLSAdapter())

        shell_id = self.protocol.open_shell(codepage=54936)  # 54936(gb18030) or 936(gbk)
        ...


```
