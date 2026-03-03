#Windows System Information Script (Powershell)

This project is a system information tool that collects and displays detailed information about Windows machine. The project is based on Powershell and demonstrates practical usage of system-level scripting, object filtering and structured output formatting.

The script gathers:

- Operating system details;
- PowerShell version;
- Network configuration;
- CPU and RAM information;
- GPU and driver details;
- Disk information;
- Local user accounts;
- Running processes;

```powershell

$algusAeg = Get-Date # Määrab skripti algusaja
Valjasta 0 "ALGUS" ("Aeg: " + $algusAeg.ToString("dddd MM/dd/yyyy HH:mm K")) # Kuvab

$masinaNimi = $env:COMPUTERNAME #Võtab masina nime
Valjasta 1 "Masina nimi" $masinaNimi # Valjastab

$psVersioon = $PSVersionTable.PSVersion.ToString() # Võtab Powershelli versiooni ja muudab stringiks
Valjasta 1 "PowerShelli versioon" $psVersioon # # Kuvab

# Loeb ja kuvab Windowsi operatsioonisüsteemi versiooni 
$windowsVersioon = (Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion').ProductName
Valjasta 1 "Windowsi versioon" $windowsVersioon

# Võrgu konfiguratsioon
$adapterid = Get-NetAdapter | Where-Object { $_.Status -eq 'Up' }
foreach ($adapter in $adapterid) { # Loeb iga andmed (IP-aadress, gateway jne.)
    $adapteriNimi = $adapter.Name
    $macAadress = $adapter.MacAddress
    $ipConfig = Get-NetIPConfiguration -InterfaceIndex $adapter.InterfaceIndex
    $dhcpLubatud = $ipConfig.DhcpEnabled
    $ipv4Aadress = $ipConfig.IPv4Address.IPAddress
    $subnetMask = $ipConfig.IPv4Address.PrefixLength
    $gateway = $ipConfig.IPv4DefaultGateway.NextHop

    Valjasta 2 "Võrgu adapter" $adapteriNimi
    Valjasta 3 "MAC-aadress" $macAadress
    Valjasta 3 "DHCP lubatud" $dhcpLubatud
    Valjasta 3 "IP-aadress" $ipv4Aadress
    Valjasta 3 "Võrgumask" $subnetMask
    Valjasta 3 "Gateway" $gateway
}

# Protsessor ja RAM
$systemInfo = Get-CimInstance -ClassName Win32_ComputerSystem
$protsessor = (Get-CimInstance -ClassName Win32_Processor).Name
$ramKogus = [math]::round($systemInfo.TotalPhysicalMemory / 1GB, 2) # Kogu RAM Gigabaitides

Valjasta 1 "Protsessor" $protsessor
Valjasta 1 "RAM kokku (GB)" $ramKogus

# Graafikakaardi info
$videoKontrollerid = Get-CimInstance -ClassName Win32_VideoController
foreach ($kontroller in $videoKontrollerid) {
    $gpuNimi = $kontroller.Name
    $draiveriVersioon = $kontroller.DriverVersion 
    $draiveriKuupäev = $kontroller.DriverDate
    $ekraaniLahutus = "$($kontroller.CurrentHorizontalResolution) x $($kontroller.CurrentVerticalResolution)"

    Valjasta 2 "Graafikakaart" $gpuNimi
    Valjasta 3 "Draiveri versioon" $draiveriVersioon
    Valjasta 3 "Draiveri kuupäev" $draiveriKuupäev
    Valjasta 3 "Ekraani lahutus" $ekraaniLahutus
}

# Kuvab kõvaketaste mudelid ja mahutavuse Gigabaitides
$kettad = Get-CimInstance -ClassName Win32_DiskDrive
foreach ($ketas in $kettad) {
    $kettaMudel = $ketas.Model
    $kettaSuurus = [math]::round($ketas.Size / 1GB, 2)

    Valjasta 2 "Ketas" $kettaMudel
    Valjasta 3 "Maht (GB)" $kettaSuurus
}

# C: ketta vaba ruum Gigabaitides
$cKetas = Get-CimInstance -ClassName Win32_LogicalDisk -Filter "DeviceID='C:'"
$cKettaVabaRuum = [math]::round($cKetas.FreeSpace / 1GB, 2)
Valjasta 1 "C: ketas - Vaba ruum (GB)" $cKettaVabaRuum

# PCI seadmete draiverite info (seadme nimi, tootja, draiveri versioon).
$pciSeadmed = Get-CimInstance -ClassName Win32_PnPSignedDriver | Where-Object { $_.DeviceClass -eq "PCI" }
foreach ($seade in $pciSeadmed) {
    $seadmeNimi = $seade.DeviceName
    $tootja = $seade.Manufacturer
    $draiveriVersioon = $seade.DriverVersion

    Valjasta 2 "PCI seade" $seadmeNimi
    Valjasta 3 "Tootja" $tootja
    Valjasta 3 "Draiveri versioon" $draiveriVersioon
}

# Kuvab süsteemis olevate kasutajakontode andmed
$kasutajad = Get-CimInstance -ClassName Win32_UserAccount
foreach ($kasutaja in $kasutajad) {
    $kasutajaNimi = $kasutaja.Name
    $kirjeldus = $kasutaja.Description
    $lokaalneKasutaja = $kasutaja.LocalAccount
    $keelatud = $kasutaja.Disabled

    Valjasta 2 "Kasutaja" $kasutajaNimi
    Valjasta 3 "Kirjeldus" $kirjeldus
    Valjasta 3 "Lokaalne kasutaja" $lokaalneKasutaja
    Valjasta 3 "Keelatud" $keelatud
}

# Sel hetkel käimasolevad protsessid
$protsessideArv = (Get-Process | Measure-Object).Count
Valjasta 1 "Käimasolevate protsesside arv" $protsessideArv

# 10 viimasena käivitatud protsessi
$viimasedProtsessid = Get-Process | Where-Object { $_.StartTime -ne $null } | Sort-Object StartTime -Descending | Select-Object -First 10
foreach ($protsess in $viimasedProtsessid) {
    $protsessiNimi = $protsess.Name
    $protsessiId = $protsess.Id
    $kaivitamiseAeg = $protsess.StartTime

    Valjasta 2 "Protsess" $protsessiNimi
    Valjasta 3 "PID" $protsessiId
    Valjasta 3 "Käivitamise aeg" $kaivitamiseAeg
}

```

[Esimene pilt](infotestarvuti1.png) <br> <br>
[Teine pilt](infotestarvuti2.png)
