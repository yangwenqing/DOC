---
id: oe9isdmukh50o7u8we5rydm
title: Beremiz
desc: ''
updated: 1721887283356
created: 1721887249558
---
-
#loongarch
# 介绍

beremiz 开源地址：https://github.com/beremiz/beremiz

当前使用的为[ Python3 分支(8126841c258a9e490f3157ad66049e9c9f02a339)](https://github.com/beremiz/beremiz/tree/8126841c258a9e490f3157ad66049e9c9f02a339)

## [新世界]debian12 安装说明

```
# 基础编译环境
sudo apt install build-essential automake flex bison mercurial  libgtk-3-dev libgl1-mesa-dev libglu1-mesa-dev libssl-dev virtualenv cmake git mercurial
# python 及其依赖
sudo apt install libpython3.11-dev python3.11 python3-pycountry python3-zeroconf python3-matplotlib python3-mesonpy
# 强制安装一些依赖
pip install gnosis asyncua --break-system-packages
```

对项目依赖进行如下修改

```diff
diff --git a/requirements.txt b/requirements.txt
index 6525420d..7b16ff10 100644
--- a/requirements.txt
+++ b/requirements.txt
@@ -21,13 +21,13 @@ ifaddr==0.2.0
 incremental==22.10.0
 kiwisolver==1.4.4
 lxml==4.9.2
-matplotlib==3.7.1
+# matplotlib==3.7.1
 msgpack==1.0.5
 Nevow @ git+https://git@github.com/beremiz/nevow-py3.git@cb62cc37824361725c0c0a599802b15210e6aaa3#egg=Nevow
 numpy==1.24.3
 packaging==23.1
 Pillow==9.5.0
-pycountry==22.3.5
+# pycountry==22.3.5
 pycparser==2.21
 pyparsing==3.0.9
 Pyro5==5.14
@@ -39,6 +39,6 @@ sortedcontainers==2.4.0
 Twisted==22.10.0
 txaio==23.1.1
 typing_extensions==4.5.0
wxPython==4.2.0
-zeroconf==0.62.0
+# zeroconf==0.62.0
 zope.interface==6.0
```

## 源码错误修改

```diff
diff --git a/bacnet/BacnetSlaveEditor.py b/bacnet/BacnetSlaveEditor.py
index c266c3a0..a30c79b8 100644
--- a/bacnet/BacnetSlaveEditor.py
+++ b/bacnet/BacnetSlaveEditor.py
@@ -575,16 +575,16 @@ class ObjectTable(CustomTable):
                 PropertyName = self.BACnetObjectType.PropertyNames[col]
                 PropertyConfig = self.BACnetObjectType.PropertyConfig[PropertyName]
                 grid.SetReadOnly(row, col, False)
-                GridCellEditorConstructorArgs = \
-                    PropertyConfig["GridCellEditorConstructorArgs"]
-                    if "GridCellEditorConstructorArgs" in PropertyConfig else []
-                grid.SetCellEditor(row, col, PropertyConfig["GridCellEditor"](*GridCellEditorConstructorArgs))
-                grid.SetCellRenderer(row, col, PropertyConfig["GridCellRenderer"]())
-                grid.SetCellBackgroundColour(row, col, wx.WHITE)
-                grid.SetCellTextColour(row, col, wx.BLACK)
+                GridCellEditorConstructorArgs = PropertyConfig["GridCellEditorConstructorArgs"]
+                if "GridCellEditorConstructorArgs" in PropertyConfig:
+                    grid.SetCellEditor(row, col, PropertyConfig["GridCellEditor"](*GridCellEditorConstructorArgs))
+                    grid.SetCellRenderer(row, col, PropertyConfig["GridCellRenderer"]())
+                    grid.SetCellBackgroundColour(row, col, wx.WHITE)
+                    grid.SetCellTextColour(row, col, wx.BLACK)
                 if "GridCellEditorParam" in PropertyConfig:
-                    grid.GetCellEditor(row, col).SetParameters(
-                        PropertyConfig["GridCellEditorParam"])
+                    grid.GetCellEditor(row, col).SetParameters(PropertyConfig["GridCellEditorParam"])
             self.ResizeRow(grid, row)
 
     def FindValueByName(self, PropertyName, PropertyValue):
diff --git a/etherlab/CommonEtherCATFunction.py b/etherlab/CommonEtherCATFunction.py
index 46ff2271..4892ed4c 100644
--- a/etherlab/CommonEtherCATFunction.py
+++ b/etherlab/CommonEtherCATFunction.py
@@ -1555,7 +1555,7 @@ class _CommonSlave(object):
         eeprom.append("01")  # Physical Layer Port info - assume 01
         #  CoE Details; <EtherCATInfo>-<Descriptions>-<Devices>-<Device>-<Mailbox>-<CoE>
         coe_details = 1  # sdo enabled
-        with device.getMailbox() as mb
+        with device.getMailbox() as mb:
             if mb is not None:
                 coe = mb.getCoE()
                 if coe is not None:
diff --git a/etherlab/EthercatCFileGenerator.py b/etherlab/EthercatCFileGenerator.py
index 15941597..00cff6f8 100644
--- a/etherlab/EthercatCFileGenerator.py
+++ b/etherlab/EthercatCFileGenerator.py
@@ -549,9 +549,9 @@ class _EthercatCFileGenerator(object):
                             if (entry_infos["index"] == 0) or (entry_infos["name"] == None):
                                 pass
                             else: 
-                        entry_infos.update(type_infos)
-                        entries_infos.append("    {0x%(index).4x, 0x%(subindex).2x, %(bitlen)d}, /* %(name)s */" % entry_infos)
-                        entry_check_list.append(check_data)
+                                entry_infos.update(type_infos)
+                                entries_infos.append("    {0x%(index).4x, 0x%(subindex).2x, %(bitlen)d}, /* %(name)s */" % entry_infos)
+                                entry_check_list.append(check_data)
                         
                         entry_declaration = slave_variables.get((index, subindex), None)
                         if entry_declaration is not None and not entry_declaration["mapped"]:
diff --git a/etherlab/etherlab.py b/etherlab/etherlab.py
index 383680fa..7a181a1c 100644
--- a/etherlab/etherlab.py
+++ b/etherlab/etherlab.py
@@ -86,7 +86,7 @@ class EntryListFactory(object):
     def AddEntry(self, context, *args):
         index, subindex = map(lambda x: int(x[0]), args[:2])
         if len(args) > 9:
-		    new_entry_infos = {
+            new_entry_infos = {
             key: translate(arg[0]) if len(arg) > 0 else default
             for (key, translate, default), arg
             in zip(ENTRY_INFOS_KEYS, args)}
@@ -449,8 +449,9 @@ for mapping needed location variables
                                 "order": group.getSortOrder(),
                                 "devices": [],
                                 # add jblee for support Moduler Device Profile (MDP)
-                                "modules": []})
-                            })
+                                "modules": []
+                            }
+                        )
 
                     for device in self.devices_xpath(self.modules_infos):
                         device_group = device.getGroupType()
```
