# Server Insider lab 17713

## Howto
To create Insider lab, you can reuse your already hydrated lab (DC can be reused), or hydrate all from scratch. You can use both VHD or ISO. VHD can be copied to Parent disks, and then its reused or ISO can be chosen during create parent disk phase (if not present in parent disks)

[Download location](https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewserver)

You can create Win10 VHD with script provided in tools folder. You can then give a try to Windows Admin Center (former project Honolulu). Just uncomment Win10 machine in labconfig and deploy Win Admin Center there.

## Note

If hydrating from scratch, make sure you use latest scripts as DSC needed some adjustments. I also fixed NestedVirt CPU issue, so make sure deploy.ps1 is latest.

## LabConfig for vNext LTSC preview

```PowerShell
$LabConfig=@{ DomainAdminName='LabAdmin'; AdminPassword='LS1setup!'; Prefix = 'WSLabInsider-'; SwitchName = 'LabSwitch'; DCEdition='4' ; PullServerDC=$false ; Internet=$false ;AdditionalNetworksConfig=@(); VMs=@(); ServerVHDs=@()}

1..4 | % {$VMNames="S2D"; $LABConfig.VMs += @{ VMName = "$VMNames$_" ; Configuration = 'S2D' ; ParentVHD = 'Win2019Core_17713.vhdx'; SSDNumber = 0; SSDSize=800GB ; HDDNumber = 12; HDDSize= 4TB ; MemoryStartupBytes= 1GB ; MemoryMinimumBytes=512MB }}
#optional Win10 management machine
#$LabConfig.VMs += @{ VMName = 'WinAdminCenter' ; Configuration = 'Simple' ; ParentVHD = 'Win10RS4_G2.vhdx'  ; MemoryStartupBytes= 1GB ; MemoryMinimumBytes=1GB ; AddToolsVHD=$True ; DisableWCF=$True }
#or with nested virtualization enabled
#1..4 | % {$VMNames="S2D"; $LABConfig.VMs += @{ VMName = "$VMNames$_" ; Configuration = 'S2D' ; ParentVHD = 'Win2019Core_17713.vhdx'; SSDNumber = 0; SSDSize=800GB ; HDDNumber = 12; HDDSize= 4TB ; MemoryStartupBytes= 2GB ; NestedVirt=$True }}

$LabConfig.ServerVHDs += @{
    Edition="4"
    VHDName="Win2019_17713.vhdx"
    Size=60GB
}
$LabConfig.ServerVHDs += @{
    Edition="3"
    VHDName="Win2019Core_17713.vhdx"
    Size=30GB
}
 
```

## LabConfig for SAC (if you reuse DC from 14393 and just copy VHD to parent disks)

```PowerShell
$LabConfig=@{ DomainAdminName='LabAdmin'; AdminPassword='LS1setup!'; Prefix = 'ws2016lab-'; SwitchName = 'LabSwitch'; DCEdition='DataCenter'; AdditionalNetworksConfig=@(); VMs=@(); ServerVHDs=@()}

1..4 | % {$VMNames="S2D"; $LABConfig.VMs += @{ VMName = "$VMNames$_" ; Configuration = 'S2D' ; ParentVHD = 'Windows_InsiderPreview_Server_VHDX_17713.vhdx'; SSDNumber = 0; SSDSize=800GB ; HDDNumber = 12; HDDSize= 4TB ; MemoryStartupBytes= 1GB ; MemoryMinimumBytes=1GB }}
$LabConfig.VMs += @{ VMName = 'PasteScriptsHere' ; Configuration = 'Simple' ; ParentVHD = 'Windows_InsiderPreview_Server_VHDX_17713.vhdx'; MemoryStartupBytes= 1GB ;MemoryMinimumBytes=1GB }
#optional Win10 management machine
#$LabConfig.VMs += @{ VMName = 'WinAdminCenter' ; Configuration = 'Simple' ; ParentVHD = 'Win10RS4_G2.vhdx' ; MemoryStartupBytes= 1GB ; MemoryMinimumBytes=1GB ; AddToolsVHD=$True ; DisableWCF=$True }
 
```

## Result

![](/Insider//Screenshots/cluadmin.png)
 
## Known issues
