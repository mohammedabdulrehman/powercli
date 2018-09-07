
param(
$server,
$vmhost)
$report=@()
import-module vmware.vimautomation.core
$cred=Get-Credential
Connect-VIServer -Server $server -Credential $cred
if($vmhost -ne $null){
$hostdetails=get-vmhost -Name $vmhost

}
else
{
$hostdetails=Get-VMHost
}
foreach ($vm in $hostdetails)
{
$number_cores=$vm.ExtensionData.Hardware.CpuInfo.NumCpuCores
$number_sockets=$vm.ExtensionData.Hardware.CpuInfo.NumCpupackages
$number_cores_persocket=$number_cores/$number_sockets
$memory=$vm.MemoryTotalGB
$disks=Get-ScsiLun -VmHost $vm
$num_disk=$disk.Count
foreach($disk in $disks)
{
$totalsize+=$disk.capacitygb

}
$status=$vm.ExtensionData.OverallStatus

$report+=$vm |select Name, @{l="number_cores";e={$number_cores}},@{l="number_sockets";e={$number_sockets}},@{l="Cores_per_sockets";e={$number_cores_persocket}},@{l="Memory(GB)";e={$memory}},
@{l="Total_disk";e={$num_disk}},@{l="Total_capacity";e={$totalsize}},@{l="health_status";e={$status}}  
}
$report|Export-Csv -Path "Hardware_report.csv" -NoTypeInformation -UseCulture
$location=Get-Location
write-host "Hardware_report.csv file is generated at $location"


