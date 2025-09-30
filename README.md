<p align="center">
  <img src="https://github.com/jorge-nbb/images/raw/main/PowerShell_5.0_icon.png" alt="powershell logo" width="228"/>
</p>

<h1 align="center">Active Directory Bulk User Creation</h1>

---

## 🎯 Project Purpose

- Automate **Active Directory** user creation using **PowerShell**
- Automatically create an **Organizational Unit (OU)** 

---

## 🛠️ Tools & Services Used

- **Windows Server 2022** (Domain Controller)  
- **Active Directory Domain Services (AD DS)**  
- **PowerShell 5.1+**  

---

## ⚙️ Configuration Steps

### STEP 1: Open PowerShell ISE

<table>
  <tr>
    <td style="vertical-align: top; width: 60%;">

1. Log into your **Domain Controller**  
2. Press **Start** > Search for **Windows PowerShell ISE**  
3. Right-click > **Run as administrator**  
4. Click **Yes** on the UAC prompt
   
</td>
<td style="vertical-align: top; text-align: right; width: 40%;">
  <img src="https://github.com/jorge-nbb/images-bulk-user/blob/main/p1.png" alt="p1" width="350"/>
</td>
  </tr>
</table>

---

### STEP 2: Create the PowerShell Script

<table>
  <tr>
    <td style="vertical-align: top; width: 60%;">

1. In PowerShell ISE, click **File > New**  
2. Copy and paste the script below:

```powershell
# ================================
# Auto-create OU and Random AD Users
# ================================

Import-Module ActiveDirectory

# Generate unique OU name with timestamp
$timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
$ouName = "Users_Auto_$timestamp"
$ouDN = "OU=$ouName,DC=adminlab,DC=local"

# Create OU
try {
    New-ADOrganizationalUnit -Name $ouName -Path "DC=adminlab,DC=local" -ProtectedFromAccidentalDeletion $false -ErrorAction Stop
    Write-Host "✅ Created OU: $ouDN"
}
catch {
    Write-Warning ("⚠ Failed to create OU ${ouName}: " + $_.Exception.Message)
    exit
}

# Convert test password once (SecureString required by New-ADUser)
$SecurePassword = ConvertTo-SecureString "password.123" -AsPlainText -Force

# Prompt user for number of users to create
do {
    $inputCount = Read-Host "Enter number of users to create (e.g., 50, 100, 1000)"
} while (-not ($inputCount -match '^\d+$'))
$UserCount = [int]$inputCount

# Sample names to pick from
$firstNames = @("Alice","Bob","Charlie","David","Emma","Frank","Grace","Henry","Isla","Jack","Laura","Mike","Nina","Oscar","Paula","Quinn","Ryan","Sophia","Tom","Uma","Victor","Wendy","Xavier","Yara","Zane")
$lastNames  = @("Smith","Johnson","Williams","Brown","Jones","Miller","Davis","Garcia","Rodriguez","Wilson","Martinez","Anderson","Taylor","Thomas","Hernandez","Moore","Martin","Jackson","Thompson","White","Lopez","Lee","Gonzalez","Harris","Clark","Lewis","Robinson","Walker","Perez","Hall")

Write-Host "Creating $UserCount users in $ouDN with password 'password.123'..."

# Create random users
for ($i=1; $i -le $UserCount; $i++) {
    $first = Get-Random $firstNames
    $last  = Get-Random $lastNames
    $sam   = ($first.Substring(0,1) + $last + (Get-Random -Minimum 1000 -Maximum 9999)).ToLower()

    try {
        New-ADUser -SamAccountName $sam `
                   -UserPrincipalName "$sam@adminlab.local" `
                   -Name "$first $last" `
                   -GivenName $first `
                   -Surname $last `
                   -Enabled $true `
                   -AccountPassword $SecurePassword `
                   -Path $ouDN `
                   -ChangePasswordAtLogon $false `
                   -PasswordNeverExpires $true

        Write-Host "✅ Created ${sam}"
    }
    catch {
        Write-Warning ("⚠ Failed to create ${sam}: " + $_.Exception.Message)
    }
}

```

3. Save as: `Create-BulkUsers.ps1` on desktop

</table>

---

### STEP 3: Run the Script

<table>
  <tr>
    <td style="vertical-align: top; width: 60%;">

1. Open PowerShell as administrator
2. Allow script execution (if restricted):
   - Temporarily (only for this session):
     ```
     Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
     ```
   - Permanently (optional, allows local scripts anytime):
     ```
     Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
     ```
3. Navigate to Desktop:
```
cd $env:USERPROFILE\Desktop
```
4. Run the script:
```
.\Create-BulkUsers.ps1
```
5. Specify amount of users to create and press **Enter**. The script will create a new OU in Active Directory Users and Computers

</td>
<td style="vertical-align: top; text-align: right; width: 40%;">
  <img src="https://github.com/jorge-nbb/images-bulk-user/blob/main/p2.png" alt="p2" width="999"/>
</td>
  </tr>
</table>

---

### STEP 4: Verify in Active Directory

<table>
  <tr>
    <td style="vertical-align: top; width: 60%;">

1. Open **Active Directory Users and Computers (ADUC)**
2. Navigate to your domain → the newly created OU (`Users_Auto_YYYYMMDD_HHMMSS`)
3. Confirm all the generated users appear inside the OU
4. Each account will have password `password.123`

</td>
<td style="vertical-align: top; text-align: right; width: 40%;">
  <img src="https://github.com/jorge-nbb/images-bulk-user/blob/main/p3.png" alt="p3" width="350"/>
</td>
  </tr>
</table>

---

## 📝 Results

- Users created directly in AD
- OU automatically generated with timestamped name
- Fully customizable for labs and training environments

---
