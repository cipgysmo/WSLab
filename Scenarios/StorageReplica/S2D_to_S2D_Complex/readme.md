<!-- TOC -->

- [About the scenario](#about-the-scenario)
    - [Description](#description)
    - [Scenario requirements](#scenario-requirements)
- [LabConfig.ps1 for Windows Server 2016](#labconfigps1-for-windows-server-2016)
- [Labconfig.ps1 for Windows Server Insider](#labconfigps1-for-windows-server-insider)
- [The lab](#the-lab)
- [Planned failover](#planned-failover)
- [Unplanned failover](#unplanned-failover)
- [Test failover in Windows Server 2019 (insider preview)](#test-failover-in-windows-server-2019-insider-preview)

<!-- /TOC -->

# About the scenario

## Description
* In this scenario will be 2 s2d clusters created
* it is "complex" as it will create 4 volumes (2x Data and 2x Log volumes in each cluster), create VMs on each site and replicate VMs from first to second site and second to first.
* Nanoservers are used as its just smaller and faster
* Labscript takes 30-50 minutes to finish (dependins what hardware is used)
* Aditionally you can give a try to Planned and unplanned failover.

## Scenario requirements

* Windows 10 1511 with enabled Hyper-V or Windows 10 1607+ 
* 20+ GB RAM is required for this scenario
* SSD (with HDD it is really slow, barely usable)


# LabConfig.ps1 for Windows Server 2016

in following labconfig you can see, that 4 machines are created. There is also additional network (ReplicaNet1), that will be used as network for Storage Replica.

```PowerShell
$LabConfig=@{ DomainAdminName='LabAdmin'; AdminPassword='LS1setup!'; Prefix = 'WSLab-'; SwitchName = 'LabSwitch'; DCEdition='4'; AdditionalNetworksConfig=@(); VMs=@(); ServerVHDs=@(); Internet=$false ; CreateClientParent=$true}

$LABConfig.AdditionalNetworksConfig += @{ NetName = 'ReplicaNet1'; NetAddress='172.16.2.'; NetVLAN='0'; Subnet='255.255.255.0'}

1..2 | % { $VMNames="Site1-S2D"     ; $LABConfig.VMs += @{ VMName = "$VMNames$_" ; Configuration = 'S2D'      ; ParentVHD = 'Win2016Core_G2.vhdx'   ; SSDNumber = 0; SSDSize=800GB ; HDDNumber = 4 ; HDDSize= 4TB ; MemoryStartupBytes= 4GB ;NestedVirt=$True;AdditionalNetworks=$True } }
1..2 | % { $VMNames="Site2-S2D"     ; $LABConfig.VMs += @{ VMName = "$VMNames$_" ; Configuration = 'S2D'      ; ParentVHD = 'Win2016Core_G2.vhdx'   ; SSDNumber = 0; SSDSize=800GB ; HDDNumber = 4 ; HDDSize= 4TB ; MemoryStartupBytes= 4GB ;NestedVirt=$True; AdditionalNetworks=$True } }

#or with NanoServer
<#
1..2 | % { $VMNames="Site1-S2D"     ; $LABConfig.VMs += @{ VMName = "$VMNames$_" ; Configuration = 'S2D'      ; ParentVHD = 'Win2016NanoHV_G2.vhdx'   ; SSDNumber = 0; SSDSize=800GB ; HDDNumber = 4 ; HDDSize= 4TB ; MemoryStartupBytes= 4GB ;NestedVirt=$True;AdditionalNetworks=$True } }
1..2 | % { $VMNames="Site2-S2D"     ; $LABConfig.VMs += @{ VMName = "$VMNames$_" ; Configuration = 'S2D'      ; ParentVHD = 'Win2016NanoHV_G2.vhdx'   ; SSDNumber = 0; SSDSize=800GB ; HDDNumber = 4 ; HDDSize= 4TB ; MemoryStartupBytes= 4GB ;NestedVirt=$True; AdditionalNetworks=$True } }
#>
 
```

# Labconfig.ps1 for Windows Server Insider

```PowerShell
$LabConfig=@{ DomainAdminName='LabAdmin'; AdminPassword='LS1setup!'; Prefix = 'WSLabInsider17713-'; SwitchName = 'LabSwitch'; DCEdition='4'; CreateClientParent=$false ; ClientEdition='Enterprise'; PullServerDC=$false ; Internet=$false ;AdditionalNetworksConfig=@(); VMs=@(); ServerVHDs=@()}

$LABConfig.AdditionalNetworksConfig += @{ NetName = 'ReplicaNet1'; NetAddress='172.16.1.'; NetVLAN='0'; Subnet='255.255.255.0'}

#$LabConfig.VMs += @{ VMName = 'WAC' ; Configuration = 'Simple' ; ParentVHD = 'Win10_G2.vhdx'  ; MemoryStartupBytes= 1GB ; MemoryMinimumBytes=1GB ; AddToolsVHD=$True ; DisableWCF=$True ; EnableWinRM=$True }

1..2 | % { $VMNames="Site1-S2D"     ; $LABConfig.VMs += @{ VMName = "$VMNames$_" ; Configuration = 'S2D'      ; ParentVHD = 'Win2019Core_17713.vhdx'   ; SSDNumber = 0; SSDSize=800GB ; HDDNumber = 4 ; HDDSize= 4TB ; MemoryStartupBytes= 4GB ;NestedVirt=$True;AdditionalNetworks=$True } }
1..2 | % { $VMNames="Site2-S2D"     ; $LABConfig.VMs += @{ VMName = "$VMNames$_" ; Configuration = 'S2D'      ; ParentVHD = 'Win2019Core_17713.vhdx'   ; SSDNumber = 0; SSDSize=800GB ; HDDNumber = 4 ; HDDSize= 4TB ; MemoryStartupBytes= 4GB ;NestedVirt=$True; AdditionalNetworks=$True } }

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

**Deploy.ps1 result**

![](/Scenarios/StorageReplica/S2D_to_S2D_Complex/Screenshots/lab.png)

# The lab

The lab begins with setting up two s2d clusters. It may not follow all best practices (for all best practices see [S2D Hyperconverged Scenario page](https://github.com/Microsoft/WSLab/tree/master/Scenarios/S2D%20Hyperconverged) )

Continue with [Scenarip.ps1](/Scenarios/StorageReplica/S2D_to_S2D_Complex/scenario.ps1) script while reading comments.

**Scenario script finished in ~35 minutes**

![](/Scenarios/StorageReplica/S2D_to_S2D_Complex/Screenshots/scenarioscriptfinished.png)

**Cluster disks**

![](/Scenarios/StorageReplica/S2D_to_S2D_Complex/Screenshots/clusterdisks.png)

**Cluster networks**

![](/Scenarios/StorageReplica/S2D_to_S2D_Complex/Screenshots/clusternetworks.png)

**Cluster resources**

![](/Scenarios/StorageReplica/S2D_to_S2D_Complex/Screenshots/VMs.png)

# Planned failover 

To flip data1 to Site1, following script will shut down VMs on volume data1 and flip replication to second datacenter. The prerequisite is to have virtual  machine config in folder "Virtual Machines" and to be able to successfully import, you will need to have disks in "Virtual Hard Disks" folder and the same name of virtual switch. Notice that the very same script is used all the time (just with small modifications)

**Warning:** If you use SCVMM, remove-vm deletes also VHD! Therefore make sure you are using Hyper-V module.

```PowerShell
#Initial test
    if ((Get-Command Remove-VM).commandtype -eq "Alias"){
        Write-Host "You are using virtual machine manager module. Please dont use it! (different remove-vm behavior...)" -ForegroundColor Yellow
        $exit=Read-Host "Press enter to Exit"
        exit
    }

#Variables
    $NewSourceClusterName="Site2-SR-Clus" #name of cluster, that will become source cluster
    $NewDestinationClusterName="Site1-SR-Clus" #name of cluster, that will become destination cluster
    $SourceRGName="Data1-Site2"
    $DestinationRGName="Data1-Site1"
    $VolumePath="C:\ClusterStorage\Data1"

#Shut down VMs on defined volume (Data1 in this case)
    $VMs=Get-VM -Cimsession (Get-ClusterNode -Cluster $NewDestinationClusterName).Name | where path -like "$VolumePath*"
    $VMs | Stop-VM

#Remove VMs from cluster resources
    Foreach ($VM in $VMs){
        Remove-ClusterGroup -Cluster $NewDestinationClusterName -Name $VM.name -RemoveResources -Force
    }

#remove VMs and keep VM config
    Foreach ($VM in $VMs){
        invoke-command -computername $VM.ComputerName -ArgumentList $VM -scriptblock {
            param($VM);
            Copy-Item -Path "$($VM.Path)\Virtual Machines" -Destination "$($VM.Path)\Virtual Machines Bak" -recurse
            Get-VM -Id $VM.id | Remove-VM -force
            Copy-Item -Path "$($VM.Path)\Virtual Machines Bak\*" -Destination "$($VM.Path)\Virtual Machines" -recurse
            Remove-Item -Path "$($VM.Path)\Virtual Machines Bak" -recurse
        }
    }

#Flip replication
    Set-SRPartnership -NewSourceComputerName $NewSourceClusterName -SourceRGName $SourceRGName -DestinationComputerName $NewDestinationClusterName -DestinationRGName $DestinationRGName -confirm:$false

#import VMs
    Invoke-Command -ComputerName (get-clusternode -cluster $NewSourceClusterName).Name[0] -ArgumentList $VolumePath -ScriptBlock{
        param($VolumePath);
        get-childitem $VolumePath -Recurse | Where-Object {($_.extension -eq '.vmcx' -and $_.directory -like '*Virtual Machines*') -or ($_.extension -eq '.xml' -and $_.directory -like '*Virtual Machines*')} | ForEach-Object -Process {
            Import-VM -Path $_.FullName
        }
    }

#Add VMs as Highly available and Start
    $VMs=Get-VM -CimSession (Get-ClusterNode -Cluster $NewSourceClusterName).Name | where path -like "$VolumePath*"
    $VMs.Name | ForEach-Object {Add-ClusterVirtualMachineRole -VMName $_ -Cluster $NewSourceClusterName}
    $VMs | Start-VM
 
```

To flip it back, we will use the very same script as above, but with different variables.

```PowerShell
#Initial test
    if ((Get-Command Remove-VM).commandtype -eq "Alias"){
        Write-Host "You are using virtual machine manager module. Please dont use it! (different remove-vm behavior...)" -ForegroundColor Yellow
        $exit=Read-Host "Press enter to Exit"
        exit
    }

#Variables
    $NewSourceClusterName="Site1-SR-Clus" #name of cluster, that will become source cluster
    $NewDestinationClusterName="Site2-SR-Clus" #name of cluster, that will become destination cluster
    $SourceRGName="Data1-Site1"
    $DestinationRGName="Data1-Site2"
    $VolumePath="C:\ClusterStorage\Data1"

#Shut down VMs on defined volume (Data1 in this case)
    $VMs=Get-VM -Cimsession (Get-ClusterNode -Cluster $NewDestinationClusterName).Name | where path -like "$VolumePath*"
    $VMs | Stop-VM

#Remove VMs from cluster resources
    Foreach ($VM in $VMs){
        Remove-ClusterGroup -Cluster $NewDestinationClusterName -Name $VM.name -RemoveResources -Force
    }

#remove VMs and keep VM config
    Foreach ($VM in $VMs){
        invoke-command -computername $VM.ComputerName -ArgumentList $VM -scriptblock {
            param($VM);
            Copy-Item -Path "$($VM.Path)\Virtual Machines" -Destination "$($VM.Path)\Virtual Machines Bak" -recurse
            Get-VM -Id $VM.id | Remove-VM -force
            Copy-Item -Path "$($VM.Path)\Virtual Machines Bak\*" -Destination "$($VM.Path)\Virtual Machines" -recurse
            Remove-Item -Path "$($VM.Path)\Virtual Machines Bak" -recurse
        }
    }

#Flip replication
    Set-SRPartnership -NewSourceComputerName $NewSourceClusterName -SourceRGName $SourceRGName -DestinationComputerName $NewDestinationClusterName -DestinationRGName $DestinationRGName -confirm:$false

#import VMs
    Invoke-Command -ComputerName (get-clusternode -cluster $NewSourceClusterName).Name[0] -ArgumentList $VolumePath -ScriptBlock{
        param($VolumePath);
        get-childitem $VolumePath -Recurse | Where-Object {($_.extension -eq '.vmcx' -and $_.directory -like '*Virtual Machines*') -or ($_.extension -eq '.xml' -and $_.directory -like '*Virtual Machines*')} | ForEach-Object -Process {
            Import-VM -Path $_.FullName
        }
    }

#Add VMs as Highly available and Start
    $VMs=Get-VM -CimSession (Get-ClusterNode -Cluster $NewSourceClusterName).Name | where path -like "$VolumePath*"
    $VMs.Name | ForEach-Object {Add-ClusterVirtualMachineRole -VMName $_ -Cluster $NewSourceClusterName}
    $VMs | Start-VM
 
```

# Unplanned failover

Simulate the failure of Site1 by pausing VMs on Host

```PowerShell
#run from Hyper-V host to pause VMs in site1
Suspend-VM -Name *site1*
 
```
**Result**

![](/Scenarios/StorageReplica/S2D_to_S2D_Complex/Screenshots/pausedVMs.png)

Now we will let Site2 know, that it is source, and we will import and start all VMs. Notice, that you will see warning about Site1 not reachable.

![](/Scenarios/StorageReplica/S2D_to_S2D_Complex/Screenshots/flipWarning.png)

```PowerShell
#Initial test
    if ((Get-Command Remove-VM).commandtype -eq "Alias"){
        Write-Host "You are using virtual machine manager module. Please dont use it! (different remove-vm behavior...)" -ForegroundColor Yellow
        $exit=Read-Host "Press enter to Exit"
        exit
    }

#Variables
    $NewSourceClusterName="Site2-SR-Clus" #name of cluster, that will become source cluster
    $NewDestinationClusterName="Site1-SR-Clus" #name of cluster, that will become destination cluster
    $SourceRGName="Data1-Site2"
    $DestinationRGName="Data1-Site1"
    $VolumePath="C:\ClusterStorage\Data1"

#Flip replication
    Set-SRPartnership -NewSourceComputerName $NewSourceClusterName -SourceRGName $SourceRGName -DestinationComputerName $NewDestinationClusterName -DestinationRGName $DestinationRGName -confirm:$false

#import VMs
    Invoke-Command -ComputerName (get-clusternode -cluster $NewSourceClusterName).Name[0] -ArgumentList $VolumePath -ScriptBlock{
        param($VolumePath);
        get-childitem $VolumePath -Recurse | Where-Object {($_.extension -eq '.vmcx' -and $_.directory -like '*Virtual Machines*') -or ($_.extension -eq '.xml' -and $_.directory -like '*Virtual Machines*')} | ForEach-Object -Process {
            Import-VM -Path $_.FullName
        }
    }
    
#Add VMs as Highly available and Start
    $VMs=Get-VM -CimSession (Get-ClusterNode -Cluster $NewSourceClusterName).Name | where path -like "$VolumePath*"
    $VMs.Name | ForEach-Object {Add-ClusterVirtualMachineRole -VMName $_ -Cluster $NewSourceClusterName}
    $VMs | Start-VM
 
```

As you can see, you run all VMs in Site2 now.

![](/Scenarios/StorageReplica/S2D_to_S2D_Complex/Screenshots/flipresultVMs.png)

Now we will turn first datacenter on again. If you do this action in real world, make sure noone will access VMs. If the reason of outage was power failure, you may configure VMs not to autostart.

```PowerShell
#Disable VMs Autostart example
$ClusterName="MyCluster"
Get-VM -CimSession (Get-ClusterNode -Cluster $ClusterName).Name | Set-VM -AutomaticStartAction Nothing
 
```

To resume first datacenter run following command on host machine

```PowerShell
#run from Hyper-V host
Resume-VM -VMName *site1*
 
```

Notice, that now you have Active-Active datacenters. Again, make sure if your primary datacenter comes up, it is isolated, so all users will be accessing Site2.

![](/Scenarios/StorageReplica/S2D_to_S2D_Complex/Screenshots/firstDConwithoutisolation.png)

The next step would be to cleanup machines on first site. This is necesarry, because without this step, you would have "zombie" VMs after making this cluster destination. The VMs objects may/may not disappear, while VMs still present in registry. What you could see after import is duplicate VM object. Note: keeping Config is not necesarry as these changes will be rewritten when replication will be enforced.

```PowerShell
#Initial test
    if ((Get-Command Remove-VM).commandtype -eq "Alias"){
        Write-Host "You are using virtual machine manager module. Please dont use it! (different remove-vm behavior...)" -ForegroundColor Yellow
        $exit=Read-Host "Press enter to Exit"
        exit
    }

#Variables
    $NewSourceClusterName="Site2-SR-Clus" #name of cluster, that will become source cluster
    $NewDestinationClusterName="Site1-SR-Clus" #name of cluster, that will become destination cluster
    $SourceRGName="Data1-Site2"
    $DestinationRGName="Data1-Site1"
    $VolumePath="C:\ClusterStorage\Data1"

#Shut down VMs on defined volume (Data1 in this case)
    $VMs=Get-VM -Cimsession (Get-ClusterNode -Cluster $NewDestinationClusterName).Name | where path -like "$VolumePath*"
    $VMs | Stop-VM

#Remove VMs from cluster resources
    Foreach ($VM in $VMs){
        Remove-ClusterGroup -Cluster $NewDestinationClusterName -Name $VM.name -RemoveResources -Force
    }

#remove VMs and keep VM config
    Foreach ($VM in $VMs){
        invoke-command -computername $VM.ComputerName -ArgumentList $VM -scriptblock {
            param($VM);
            Copy-Item -Path "$($VM.Path)\Virtual Machines" -Destination "$($VM.Path)\Virtual Machines Bak" -recurse
            Get-VM -Id $VM.id | Remove-VM -force
            Copy-Item -Path "$($VM.Path)\Virtual Machines Bak\*" -Destination "$($VM.Path)\Virtual Machines" -recurse
            Remove-Item -Path "$($VM.Path)\Virtual Machines Bak" -recurse
        }
    }

#Flip replication
    Set-SRPartnership -NewSourceComputerName $NewSourceClusterName -SourceRGName $SourceRGName -DestinationComputerName $NewDestinationClusterName -DestinationRGName $DestinationRGName -confirm:$false
 
```

You can notice now, that Site1 is now destination for all volumes

![](/Scenarios/StorageReplica/S2D_to_S2D_Complex/Screenshots/allinDC2.png)

To flip replication, you can use script for planned failover (following example is the same as script from planned failover)

```PowerShell
#Initial test
    if ((Get-Command Remove-VM).commandtype -eq "Alias"){
        Write-Host "You are using virtual machine manager module. Please dont use it! (different remove-vm behavior...)" -ForegroundColor Yellow
        $exit=Read-Host "Press enter to Exit"
        exit
    }

#Variables
    $NewSourceClusterName="Site1-SR-Clus" #name of cluster, that will become source cluster
    $NewDestinationClusterName="Site2-SR-Clus" #name of cluster, that will become destination cluster
    $SourceRGName="Data1-Site1"
    $DestinationRGName="Data1-Site2"
    $VolumePath="C:\ClusterStorage\Data1"

#Shut down VMs on defined volume (Data1 in this case)
    $VMs=Get-VM -Cimsession (Get-ClusterNode -Cluster $NewDestinationClusterName).Name | where path -like "$VolumePath*"
    $VMs | Stop-VM

#Remove VMs from cluster resources
    Foreach ($VM in $VMs){
        Remove-ClusterGroup -Cluster $NewDestinationClusterName -Name $VM.name -RemoveResources -Force
    }

#remove VMs and keep VM config
    Foreach ($VM in $VMs){
        invoke-command -computername $VM.ComputerName -ArgumentList $VM -scriptblock {
            param($VM);
            Copy-Item -Path "$($VM.Path)\Virtual Machines" -Destination "$($VM.Path)\Virtual Machines Bak" -recurse
            Get-VM -Id $VM.id | Remove-VM -force
            Copy-Item -Path "$($VM.Path)\Virtual Machines Bak\*" -Destination "$($VM.Path)\Virtual Machines" -recurse
            Remove-Item -Path "$($VM.Path)\Virtual Machines Bak" -recurse
        }
    }

#Flip replication
    Set-SRPartnership -NewSourceComputerName $NewSourceClusterName -SourceRGName $SourceRGName -DestinationComputerName $NewDestinationClusterName -DestinationRGName $DestinationRGName -confirm:$false

#import VMs
    Invoke-Command -ComputerName (get-clusternode -cluster $NewSourceClusterName).Name[0] -ArgumentList $VolumePath -ScriptBlock{
        param($VolumePath);
        get-childitem $VolumePath -Recurse | Where-Object {($_.extension -eq '.vmcx' -and $_.directory -like '*Virtual Machines*') -or ($_.extension -eq '.xml' -and $_.directory -like '*Virtual Machines*')} | ForEach-Object -Process {
            Import-VM -Path $_.FullName
        }
    }

#Add VMs as Highly available and Start
    $VMs=Get-VM -CimSession (Get-ClusterNode -Cluster $NewSourceClusterName).Name | where path -like "$VolumePath*"
    $VMs.Name | ForEach-Object {Add-ClusterVirtualMachineRole -VMName $_ -Cluster $NewSourceClusterName}
    $VMs | Start-VM
 
```

Result: All good again

![](/Scenarios/StorageReplica/S2D_to_S2D_Complex/Screenshots/result-allgoodagain.png)

# Test failover in Windows Server 2019 (insider preview)

```PowerShell
#Create Virtual disk TestFailoverSite1
New-Volume -StoragePoolFriendlyName S2D* -FriendlyName TestFailoverSite1 -FileSystem CSVFS_ReFS -StorageTierFriendlyNames capacity -StorageTierSizes 10GB -CimSession Site1-SR-Clus

#Display SR Groups
Get-SRGroup -CimSession Site1-SR-Clus | select Name -ExpandProperty Replicas | ft Name, DataVolume,ReplicationMode,ReplicationStatus,IsMounted

#Move TestFailoverSite1 and Data2 CSV to Site1-S2D1
Get-ClusterSharedVolume -Cluster site1-sr-clus -Name *Data1* | Move-ClusterSharedVolume -Node Site1-S2D1
Get-ClusterSharedVolume -Cluster site1-sr-clus -Name *TestFailoverSite1* | Move-ClusterSharedVolume -Node Site1-S2D1

#Mount
Mount-SRDestination -ComputerName Site1-S2D1 -Name Data2-Site1 -TemporaryPath c:\ClusterStorage\TestFailoverSite1 -Confirm:0

#Display SR Groups
Get-SRGroup -CimSession Site1-SR-Clus | select Name -ExpandProperty Replicas | ft Name, DataVolume,ReplicationMode,ReplicationStatus,IsMounted

#import VMs
Invoke-Command -ComputerName Site1-S2D1 -ScriptBlock{
    get-childitem c:\ClusterStorage\Data2\ -Recurse | Where-Object {($_.extension -eq '.vmcx' -and $_.directory -like '*Virtual Machines*') -or ($_.extension -eq '.xml' -and $_.directory -like '*Virtual Machines*')} | ForEach-Object -Process {
        Import-VM -Path $_.FullName
    }
}

#Add VMs as Highly available
$VMs=Get-VM -CimSession (Get-ClusterNode -Cluster Site1-SR-Clus).Name | where path -like "c:\ClusterStorage\Data2\*"
$VMs.Name | ForEach-Object {Add-ClusterVirtualMachineRole -VMName $_ -Cluster Site1-SR-Clus}

#disconnect from network and start
$VMs=Get-VM -CimSession (Get-ClusterNode -Cluster Site1-SR-Clus).Name | where path -like "c:\ClusterStorage\Data2\*"
$VMs | Get-VMNetworkAdapter | Disconnect-VMNetworkAdapter
$VMs | Start-VM

##cleanup

#Shut down VMs on Data1 volume
$VMs=Get-VM -Cimsession (Get-ClusterNode -Cluster Site1-SR-Clus).Name | where path -like "c:\ClusterStorage\Data2\*"
$VMs | Stop-VM

#Remove VMs from cluster resources
Foreach ($VM in $VMs){
    Remove-ClusterGroup -Cluster Site1-SR-Clus -Name $VM.name -RemoveResources -Force
}

#remove VMs (no need to backup config)
$VMs | Remove-VM -Force

#Dismount volume
Dismount-SRDestination -ComputerName Site1-S2D1 -Name Data2-Site1 -Confirm:0

```