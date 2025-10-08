# Hyper-V_Backup
This script will export all running vm and zip them with 7z and remove the exported files
To deploy the `backup.ps1` script as part of an **MCP (Microsoft Cloud Platform)** server setup, you can integrate it into a **Microsoft Azure** or **on-premises Hyper-V environment**. Below is a step-by-step guide to create a server that automates Hyper-V VM backups using this script in a cloud or hybrid setup.

---

### **Option 1: Deploy on Azure (Cloud-Based MCP Server)**
If you want to run this script in **Azure**, you’ll need a **Windows Server VM** with Hyper-V enabled (nested virtualization). Here’s how:

#### **1. Create an Azure VM**
- Use a **Windows Server 2022** VM with **Generation 2** (required for nested virtualization).
- Ensure the VM size supports nested virtualization (e.g., `Dsv3` or `Esv3` series).
- Enable **Hyper-V role** on the VM via Server Manager.

#### **2. Install Dependencies**
- **7-Zip**: Install 7-Zip on the VM (the script uses `C:\Program Files\7-Zip\7z.exe`).
- **PowerShell**: Ensure PowerShell 7+ is installed for compatibility.

#### **3. Configure the Script**
- Modify the script’s variables to match your Azure environment:
  ```powershell
  $base = "C:\backup\backup\"  # Ensure this path exists
  $date = Get-Date -Format "MM-dd-yyyy"
  ```
- Test the script manually to confirm it works in your Azure VM.

#### **4. Automate Backups**
- Schedule the script using **Task Scheduler** or **Azure Automation** to run daily/weekly.
- Example Task Scheduler trigger:
  - Daily at 2:00 AM.

#### **5. Store Backups in Azure Blob Storage**
- Modify the script to upload `.7z` files to **Azure Blob Storage** for offsite backups:
  ```powershell
  # Add Azure PowerShell module and upload logic
  Install-Module -Name Az -Force
  Connect-AzAccount  # Use Managed Identity or Service Principal
  $context = New-AzStorageContext -StorageAccountName "yourstorageaccount" -StorageAccountKey "yourkey"
  Set-AzStorageBlobContent -File $zipPath -Container "backups" -Blob "$vm-$date.7z" -Context $context
  ```

#### **6. Monitor with Azure Monitor**
- Set up alerts for script failures using **Azure Log Analytics** or **Application Insights**.

---

### **Option 2: On-Premises Hyper-V Server**
If you’re setting up an **on-premises Hyper-V server**, follow these steps:

#### **1. Set Up a Hyper-V Host**
- Install **Windows Server 2022** with Hyper-V role enabled.
- Create VMs to back up.

#### **2. Install 7-Zip**
- Download and install 7-Zip from [https://www.7-zip.org/](https://www.7-zip.org/).

#### **3. Deploy the Script**
- Place the `backup.ps1` script in a dedicated folder (e.g., `C:\Scripts\HyperVBackup`).
- Modify the script’s variables:
  ```powershell
  $base = "D:\HyperVBackups\"  # Use a dedicated drive for backups
  ```

#### **4. Schedule the Script**
- Use **Task Scheduler** to run the script daily:
  - Open Task Scheduler > Create Basic Task > Trigger: Daily > Action: Start a Program.
  - Program/script: `powershell.exe`
  - Add arguments: `-File "C:\Scripts\HyperVBackup\backup.ps1"`

#### **5. Optional: Offsite Backups**
- Use **Azure File Sync** or **Rsync** to sync backups to Azure or another remote server.

---

### **Key Considerations**
- **Permissions**: Ensure the script runs with administrative privileges.
- **Storage**: Monitor disk space for the backup location (`C:\backup\backup\`).
- **Notifications**: Add email alerts using `Send-MailMessage` in PowerShell if backups fail.
- **Security**: Restrict access to the backup folder and 7-Zip executable.

---

### **Example: Full Script with Azure Upload**
Here’s an updated version of the script with Azure Blob Storage integration:
```powershell
# Azure-specific variables
$storageAccount = "yourstorageaccount"
$storageKey = "yourstoragekey"
$containerName = "hyperv-backups"

# Existing script logic...
foreach ($vm in $vms) {
    # Export and compress as before...

    # Upload to Azure Blob Storage
    $blobName = "$vm-$date.7z"
    $ctx = New-AzStorageContext -StorageAccountName $storageAccount -StorageAccountKey $storageKey
    Set-AzStorageBlobContent -File $zipPath -Container $containerName -Blob $blobName -Context $ctx
    Write-Host -ForegroundColor Cyan "$vm backup uploaded to Azure"
}
```

---

### **Next Steps**
1. Test the script in a non-production environment.
2. Validate backups by restoring a VM from the `.7z` archive.
3. Monitor logs (`hyperv-export-job.log`) for errors.

Would you like help automating this setup further (e.g., ARM templates for Azure, Docker containers, or GitHub Actions for deployment)?
