
模拟服务端：

```bash
pip install snmpsim
```

模拟数据(使用官方示例数据)：

```bash
mkdir -p ./data
wget https://github.com/etingof/snmpsim/raw/master/data/public.snmprec -O ./data/public.snmprec
```

启动：

```bash
/usr/python/bin/snmpsim-command-responder  --data-dir=./data --agent-udpv4-endpoint=0.0.0.0:161
```

注意观察输出日志，其中 `community name: public`：

```

...

--- SNMP Engine configuration 
SNMPv3 EngineID: 0x80004fb805656530306339613061383931086a9fc0 
  --- Simulation data recordings configuration 
  SNMPv3 Context Engine ID: 0x80004fb805656530306339613061383931086a9fc0 
  Scanning "./data" directory for  *.dump, *.MVC, *.sapwalk, *.snmpwalk, *.snmprec, *.snmprec.bz2 data files... 
    Index /tmp/snmpsim/._data_public.dbm out of date 
    Building index /tmp/snmpsim/._data_public.dbm for data file ./data/public.snmprec (open flags "nfu")... 
    ...3881 entries indexed 
    Configuring ./data/public.snmprec controller 
    SNMPv1/2c community name: public
    SNMPv3 Context Name: 4c9184f37cff01bcdc32dc486ec36961 or public 
  --- SNMPv3 USM configuration 
  SNMPv3 USM SecurityName: simulator 
  SNMPv3 USM authentication key: auctoritas, authentication protocol: MD5 
  SNMPv3 USM encryption (privacy) key: privatus, encryption protocol: DES 
  Maximum number of variable bindings in SNMP response: 64 
  --- Transport configuration 
  Listening at UDP/IPv4 endpoint 0.0.0.0:161, transport ID 1.3.6.1.6.1.1.0
```

另开一个终端连接测试：

| OID                 | 含义   |
| :------------------- | :---- |
| `1.3.6.1.2.1.1.1.0` | 系统描述 |
| `1.3.6.1.2.1.1.3.0` | 运行时间 |
| `1.3.6.1.2.1.1.5.0` | 设备名称 |
执行命令：

```
snmpwalk -v2c -c public 127.0.0.1 1.3.6.1.2.1.1

# 如果snmpwalk命令不存在，可能未安装工具包：
# yum install -y net-snmp-utils
```

输出：

```
SNMPv2-MIB::sysDescr.0 = STRING: Linux zeus 4.8.6.5-smp #2 SMP Sun Nov 13 14:58:11 CDT 2016 i686
SNMPv2-MIB::sysObjectID.0 = OID: NET-SNMP-MIB::netSnmpAgentOIDs.10
DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (124041433) 14 days, 8:33:34.33
SNMPv2-MIB::sysContact.0 = STRING: SNMP Laboratories, info@snmplabs.com
SNMPv2-MIB::sysName.0 = STRING: zeus.snmplabs.com (you can change this!)
SNMPv2-MIB::sysLocation.0 = STRING: San Francisco, California, United States
SNMPv2-MIB::sysServices.0 = INTEGER: 72
SNMPv2-MIB::sysORLastChange.0 = Timeticks: (124041433) 14 days, 8:33:34.33
SNMPv2-MIB::sysORID.1 = OID: SNMP-FRAMEWORK-MIB::snmpFrameworkMIBCompliance
SNMPv2-MIB::sysORID.2 = OID: SNMP-MPD-MIB::snmpMPDCompliance
SNMPv2-MIB::sysORID.3 = OID: SNMP-USER-BASED-SM-MIB::usmMIBCompliance
SNMPv2-MIB::sysORID.4 = OID: SNMPv2-MIB::snmpMIB
SNMPv2-MIB::sysORID.5 = OID: TCP-MIB::tcpMIB
SNMPv2-MIB::sysORID.6 = OID: IP-MIB::ip
SNMPv2-MIB::sysORID.7 = OID: UDP-MIB::udpMIB
SNMPv2-MIB::sysORID.8 = OID: SNMP-VIEW-BASED-ACM-MIB::vacmBasicGroup
SNMPv2-MIB::sysORDescr.1 = STRING: The SNMP Management Architecture MIB.
SNMPv2-MIB::sysORDescr.2 = STRING: The MIB for Message Processing and Dispatching.
SNMPv2-MIB::sysORDescr.3 = STRING: The management information definitions for the SNMP User-based Security Model.
SNMPv2-MIB::sysORDescr.4 = STRING: The MIB module for SNMPv2 entities
SNMPv2-MIB::sysORDescr.5 = STRING: The MIB module for managing TCP implementations
SNMPv2-MIB::sysORDescr.6 = STRING: The MIB module for managing IP and ICMP implementations
SNMPv2-MIB::sysORDescr.7 = STRING: The MIB module for managing UDP implementations
SNMPv2-MIB::sysORDescr.8 = STRING: View-based Access Control Model for SNMP.
SNMPv2-MIB::sysORUpTime.1 = Timeticks: (0) 0:00:00.00
SNMPv2-MIB::sysORUpTime.2 = Timeticks: (0) 0:00:00.00
SNMPv2-MIB::sysORUpTime.3 = Timeticks: (0) 0:00:00.00
SNMPv2-MIB::sysORUpTime.4 = Timeticks: (0) 0:00:00.00
SNMPv2-MIB::sysORUpTime.5 = Timeticks: (0) 0:00:00.00
SNMPv2-MIB::sysORUpTime.6 = Timeticks: (0) 0:00:00.00
SNMPv2-MIB::sysORUpTime.7 = Timeticks: (0) 0:00:00.00
SNMPv2-MIB::sysORUpTime.8 = Timeticks: (0) 0:00:00.00
```

Python客户端连接实例：

```python
"""
yum install -y net-snmp-devel
pip install python3-netsnmp
"""
import netsnmp

sess = netsnmp.Session(Version=2, DestHost='127.0.0.1', Community='public')

print('===========')
vars = netsnmp.VarList(
    netsnmp.Varbind('sysUpTime', 0),
    netsnmp.Varbind('sysContact', 0),
    netsnmp.Varbind('sysLocation', 0)
)
vals = sess.get(vars)
for var in vars:
    print(var.tag, var.iid, "=", var.val, '(',var.type,')')


print('===========')
vars = netsnmp.VarList(netsnmp.Varbind('.1.3.6.1.2.1.1.1', 0))
vals1 = sess.get(vars)
for var in vars:
    print(var.tag, var.iid, "=", var.val, '(',var.type,')')

# 等同shell执行命令：snmpwalk -v2c -c public 127.0.0.1 1.3.6.1.2.1.1
print('以下两种调用一致：\n---------')
print('（1）===========')
vars = netsnmp.VarList(netsnmp.Varbind('system'))
vals = sess.walk(vars)
for var in vars:
    print("  ",var.tag, var.iid, "=", var.val, '(',var.type,')')

print('（2）===========')
vars = netsnmp.VarList(netsnmp.Varbind('.1.3.6.1.2.1.1'))
vals1 = sess.walk(vars)
for var in vars:
    print("  ",var.tag, var.iid, "=", var.val, '(',var.type,')')
```