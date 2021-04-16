# Maratona Azure Expert - Migração de ambiente para o Azure

Hands-on Lab

## Maratona Azure Expert
## Exercise #01 - Deploy the On-premises environment (30~60 minutes)

## Requirements

1. You will need Owner or Contributor permissions for an Azure subscription to use in the lab.

2. Your subscription must have sufficient unused quota to deploy the VMs used in this lab. To check your quota:

    - Log in to the [Azure portal](https://portal.azure.com), select **All services** then **Subscriptions**. Select your subscription, then choose **Usage + quotas**.
  
    - From the **Select a provider** drop-down, select **Microsoft.Compute**.
  
    - From the **All service quotas** drop down, select **Standard DSv3 Family vCPUs**, **Standard FSv2 Family vCPUs** and **Total Regional vCPUs**.
  
    - From the **All locations** drop down, select the location where you will deploy the lab.
  
    - From the last drop-down, select **Show all**.
  
    - Check that the selected quotas have sufficient unused capacity:
  
        - Standard DSv3 Family vCPUs: **at least 8 vCPUs**.
  
        - Standard FSv2 Family vCPUs: **at least 6 vCPUs**.

        - Total Regional vCPUs: **at least 14 vCPUs**.

    > **Note:** If you are using an Azure Pass subscription, you may not meet the vCPU quotas above. In this case, you can still complete the lab.

## Deploy the on-premises environment

1. Deploy the template **SmartHotelHost.json** to a new resource group. This template deploys a virtual machine running nested Hyper-V, with 4 nested VMs. This comprises the 'on-premises' environment which you will assess and migrate during this lab.

    You can deploy the template by selecting the 'Deploy to Azure' button below. You will need to create a new resource group **RG-MAE-Onpremises**. You will also need to select a location **East US 2** close to you to deploy the template to. Then choose **Review + create** followed by **Create**. 

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fcloudworkshop.blob.core.windows.net%2Fline-of-business-application-migration%2Fsept-2020%2FSmartHotelHost.json" target="_blank">![Button to deploy the SmartHotelHost template to Azure.](/AllFiles/Images/deploy-to-azure.png)</a>

    > **Note:** The template will take around 6-7 minutes to deploy. Once template deployment is complete, several additional scripts are executed to bootstrap the lab environment. **Allow at least 1 hour from the start of template deployment for the scripts to run.**

### Verify the on-premises environment

1. Navigate to the **SmartHotelHost** VM that was deployed by the template in the previous step.

2. Make a note of the public IP address.

3. Open a browser tab and navigate to **http://\<SmartHotelHostIP-Address\>**. You should see the SmartHotel application, which is running on nested VMs within Hyper-V on the SmartHotelHost. (The application doesn't do much: you can refresh the page to see the list of guests or select 'CheckIn' or 'CheckOut' to toggle their status.)

    ![Browser screenshot showing the SmartHotel application.](/AllFiles/Images/smarthotel.png)

    > **Note:** If the SmartHotel application is not shown, wait 10 minutes and try again. It takes **at least 1 hour** from the start of template deployment. You can also check the CPU, network and disk activity levels for the SmartHotelHost VM in the Azure portal, to see if the provisioning is still active.

You should follow all steps provided *before* performing the Hands-on lab.

## Solution architecture

The SmartHotel application comprises 4 VMs hosted in Hyper-V:

- **Database tier** Hosted on the smarthotelSQL1 VM, which is running Windows Server 2016 and SQL Server 2017.

- **Application tier** Hosted on the smarthotelweb2 VM, which is running Windows Server 2012 R2.

- **Web tier** Hosted on the smarthotelweb1 VM, which is running Windows Server 2012 R2.

- **Web proxy** Hosted on the  UbuntuWAF VM, which is running Nginx on Ubuntu 18.04 LTS.

For simplicity, there is no redundancy in any of the tiers.

![A slide shows the on-premises SmartHotel application architecture.](/AllFiles/Images/overview.png)

## Exercise #02 - Discover and assess the on-premises environment (30 minutes)

1. Open your browser, navigate to **https://portal.azure.com**, and log in with your Azure subscription credentials.

2. Select **All services** in the portal's left navigation, then search for and select **Azure Migrate** to open the Azure Migrate Overview blade.

3. Select **Assess and migrate servers**, then **Create project**.  Select your subscription and create a new resource group named **RG-MAE-SmartHotel**. Enter **SmartHotelMigration** as the Migrate project name, and choose a geography close to you to store the migration assessment data. Then select **Create**.

6. The Azure Migrate deployment will start. Once it has completed, you should see the **Azure Migrate: Server Assessment** and **Azure Migrate: Server Migration** panels for the current migration project.

1.  Under **Azure Migrate: Server Assessment**, select **Discover** to open the **Discover machines** blade. Under **Are your machines virtualized?**, select **Yes, with Hyper-V**.

2.  In **1: Generate Azure Migrate project key**, provide **SmartHotelAppl** as name for the Azure Migrate appliance that you will set up for discovery of Hyper-V VMs. Select **Generate key** to start the creation of the required Azure resources. 

3.  **Wait** for the key to be generated, then copy the **Azure Migrate project key** to your clipboard.

4.  Read through the instructions on how to download, deploy and configure the Azure Migrate appliance. Close the 'Discover machines' blade (do **not** download the .VHD file or .ZIP file, the .VHD has already been downloaded for you).

5. In a separate browser tab, navigate to the Azure portal. In the global search box, enter **SmartHotelHost**, then select the **SmartHotelHost** virtual machine.

6. Select **Connect**, select **RDP**, then download the RDP file and connect to the virtual machine using username **demouser** and password **demo!pass123**.

7. In Server Manager, select **Tools**, then **Hyper-V Manager** (if Server Manager does not open automatically, open it by selecting **Start**, then **Server Manager**). In Hyper-V Manager, select **SMARTHOTELHOST**. You should now see a list of the four VMs that comprise the on-premises SmartHotel application.

8. In Hyper-V Manager, under **Actions**, select **Import Virtual Machine...** to open the **Import Virtual Machine** wizard.

9. At the first step, **Before You Begin**, select **Next**.

10.  At the **Locate Folder** step, select **Browse** and navigate to **F:\\VirtualMachines\\AzureMigrateAppliance** (the folder name may also include a version number), then choose **Select Folder**, then select **Next**.

11. At the **Select Virtual Machine** step, the **AzureMigrateAppliance** VM should already be selected. Select **Next**.

12. At the **Choose Import Type** step, keep the default setting **Register the virtual machine in-place**. Select **Next**.

13. At the **Connect Network** step, you will see an error that the virtual switch previously used by the Azure Migrate appliance could not be found. From the **Connection** drop down, select the **Azure Migrate Switch**, then select **Next**.

    > **Note**:  The Azure Migrate appliance needs access to the Internet to upload data to Azure. It also needs access to the Hyper-V host. However, it does not need direct access to the application VMs running on the Hyper-V host.
    >

14. Review the summary page, then select **Finish** to create the Azure Migrate appliance VM.

15. In Hyper-V Manager, select the **AzureMigrateAppliance** VM, then select **Start** on the left.

1.  In Hyper-V Manager, select the **AzureMigrateAppliance** VM, then select **Connect** on the left.

2.  A new window will open showing the Azure Migrate appliance. Wait for the License terms screen to show, then select **Accept**.

3.  On the **Customize settings** screen, set the Administrator password to **demo!pass123** (twice). Then select **Finish**.

4.  At the **Connect to AzureMigrateAppliance** prompt, set the appliance screen size using the slider, then select **Connect**.

5.  Log in with the Administrator.

6.  **Wait.** After a minute or two, the browser will open showing the Azure Migrate appliance configuration wizard (it can also be launched from the desktop shortcut).

    On opening of the appliance configuration wizard, a pop-up with the license terms will appear. Accept the terms by selecting **I agree**.

7. Under **Set up prerequisites**, the following two steps to verify Internet connectivity and time synchronization should pass automatically.
 
8. **Wait** while the wizard installs the latest Azure Migrate updates. If prompted for credentials, enter user name **Administrator** and password **demo!pass123**. Once the Azure Migrate updates are completed, you may see a pop-up if the management app restart is required, and if so, select **Refresh** to restart the app.  

     Once restarted, the 'Set up prerequisites' steps of the Azure Migrate wizard will re-run automatically. Once the prerequisites are completed, you can proceed to the next panel, **Register with Azure Migrate**.

9.  At the next phase of the wizard, **Register with Azure Migrate**, paste the **Azure Migrate project key** copied from the Azure portal earlier. (If you do not have the key, go to **Server Assessment > Discover > Manage existing appliances**, select the appliance name you provided at the time of key generation and copy the corresponding key.)

10. After you select **Login**, a new window will open asking for a code.  This code is located below the **Azure Migrate project key**.  Copy and paste this code in the login field.  You will then be asked for your Azure portal credentials to complete the login process.

11. Select **Login**. This will open an Azure login prompt in a new browser tab (if it doesn't appear, make sure the pop-up blocker in the browser is disabled). Log in using your Azure credentials. Once you have logged in, return to the Azure Migrate Appliance tab and the appliance registration will start automatically.

    Once the registration has completed, you can proceed to the next panel, **Manage credentials and discovery sources**.

11. In **Step 1: Provide Hyper-V host credentials**, select **Add credentials**.

12. Specify **hostlogin** as the friendly name for credentials, username **demouser**, and password **demo!pass123** for the Hyper-V host/cluster that the appliance will use to discover VMs. Select **Save**.

13. In **Step 2: Provide Hyper-V host/cluster details**, select **Add discovery source** to specify the Hyper-V host/cluster IP address/FQDN and the friendly name for credentials to connect to the host/cluster.

14. Select **Add single item**, select **hostlogin** as the friendly name, and enter **SmartHotelHost** under 'IP Address / FQDN'.

15. Select **Save**. The appliance will validate the connection to the Hyper-V hosts/clusters added and show the **Validation status** in the table against each host/cluster.

    > **Note:** When adding discovery sources:
    > -  For successfully validated hosts/clusters, you can view more details by selecting their IP address/FQDN.
    > -  If validation fails for a host, review the error by selecting the Validation failed in the Status column of the table. Fix the issue and validate again.
 
16. Select **Start discovery** to kick off VM discovery from the successfully validated hosts/clusters.

17. Wait for the Azure Migrate status to show **Discovery has been successfully initiated**. This will take several minutes. After the discovery has been successfully initiated, you can check the discovery status against each host/cluster in the table.

18. Return to the **Azure Migrate** blade in the Azure portal. Select **Servers**, then select **Refresh**. Under **Azure Migrate: Server Assessment** you should see a count of the number of servers discovered so far. If discovery is still in progress, select **Refresh** periodically until 5 discovered servers are shown. This may take several minutes.

    **Wait for the discovery process to complete before proceeding to the next Task**.

1. Continuing, select **Assess** under **Azure Migrate: Server Assessment** to start a new migration assessment.

2. On the Assess servers blade, enter **SmartHotelAssessment** as the assessment name.

3. Under **Assessment properties**, select **View all**.

4. The **Assessment properties** blade allows you to tailor many of the settings used when making a migration assessment report. Take a few moments to explore the wide range of assessment properties. Hover over the information icons to see more details on each setting. Choose any settings you like, then select **Save**. (You have to make a change for the Save button to be enabled; if you don't want to make any changes, just close the blade.)
 
5. Select **Next** to move to the **Select machines to assess** tab. Choose **Create New** and enter the group name **SmartHotel VMs**. Select the **smarthotelweb1**, **smarthotelweb2**, **smarthotelSQL1** and **UbuntuWAF** VMs.

    **Note:** There is no need to include the **AzureMigrateAppliance** VMs in the assessment, since they will not be migrated to Azure. 

6. Select **Next**, followed by **Create assessment**. On the **Azure Migrate - Servers** blade, select **Refresh** periodically until the number of assessments shown is **1**. This may take several minutes.

7. Select **Assessments** under **Azure Migrate: Server Assessment** to see a list of assessments. Then select the actual assessment.

8. Take a moment to study the assessment overview.

9. Select **Edit properties**. Note how you can now modify the assessment properties you chose earlier. Change a selection of settings, and **Save** your changes. After a few moments, the assessment report will update to reflect your changes.

10. Select **Azure readiness** (either the chart or on the left navigation). Note that for the **UbuntuWAF** VM, a specific concern is listed regarding the readiness of the VM for migration.

11. Select **Unknown OS** for **UbuntuWAF**. A new browser tab opens showing Azure Migrate documentation. Note on the page that the issue relates the OS not being specified in the host hypervisor, so you must confirm the OS type and version is supported.

12. Return to the portal browser tab to see details of the issue. Note the recommendation to migrate the VM using **Azure Migrate: Server Migration**.

13. Take a few minutes to explore other aspects of the migration assessment.

## Exercise #03 - Database Server Migration (30~60 minutes)

1. Open the Azure Cloud Shell by navigating to **https://shell.azure.com**. Log in using your Azure subscription credentials if prompted to do so, select a **PowerShell** session, and accept any prompts.

2. Run the following command to register the **Microsoft.DataMigration** resource provider:
   
    ```PowerShell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DataMigration
    ```
    > **Note**: It may take several minutes for the resource provider to register. You can proceed to the next task without waiting for the registration to complete. You will not use the resource provider until task 3.
    >
    > You can check the status by running:

    > ```PowerShell
    > Get-AzResourceProvider -ProviderNamespace Microsoft.DataMigration | Select-Object ProviderNamespace, RegistrationState, ResourceTypes
    > ```

1. Open the Azure portal at https://portal.azure.com and log in using your subscription credentials if it's not still up.

2. Expand the portal's left navigation by selecting **Show portal menu** in the top left then select **+ Create a resource**, then select **Databases**, then select **SQL Database**.

3. The **Create SQL Database** blade opens, showing the **Basics** tab. Complete the form as follows:

    - Subscription: **Select your subscription**.
  
    - Resource group: (create new) **RG-MAE-SmartHotelDB**
  
    - Database name: **smarthoteldb**
  
    - Server: Select **Create new** and fill in the New server blade as follows then select **OK**:
  
        - Server name: **samaesmarthoteldb\[unique number\]**
  
        - Server admin login: **demouser**
  
        - Password: **demo!pass123**
  
        - Location: **IMPORTANT: For most users, select the same region you used when you started your lab - this makes migration faster. If you are using an Azure Pass subscription, choose a different region to stay within the Total Regional vCPU limit.**

    - Use SQL elastic pool: **No**
  
    - Compute + storage: **Standard S0**

    > **Note**: To select the **Standard S0** database tier, select **Configure database**, then **Looking for basic, standard, premium?**, select **Standard** and select **Apply**.

4. Select **Next: Networking >** to move to the **Networking** tab. Confirm that **No access** is selected.

    > **Note**: We will configure private endpoints to access our database later in the lab.

5. Select **Review + Create**, then select **Create** to create the database. Wait for the deployment to complete.

2. In the Azure portal, expand the portal's left navigation and select **+ Create a resource**, search for **Migration**, and then select **Azure Database Migration Service** from the drop-down list.

3. On the **Azure Database Migration Service** blade, select **Create**.

 4. In the **Create Migration Service** blade, on the **Basics** tab, enter the following values:
   
    - Subscription: **Select your Azure subscription**.
  
    - Resource group: **RG-MAE-Migration**
  
    - Service Name: **SmartHotelDBMigration**
  
    - Location: **East US 2**.

    - Service mode: **Azure**
  
    - Pricing tier: **Standard: 1 vCore**

5. Select **Next: Networking** to move to the **Networking** tab, and select the **DMSvnet/DMS** virtual network and subnet in the **SmartHotelHostRG** resource group.
   
6. Select **Review + create**, followed by **Create**.

> **Note**: Creating a new migration service can take around 20 minutes. You can continue to the next task without waiting for the operation to complete.

1. Return to the **Azure Migrate** blade in the Azure portal. Select the **Overview** panel, then select **Assess and migrate databases**.

2. Under **Assessment tools**, select the link to add a tool, then select **Azure Migrate: Database Assessment**, then select **Next**.

3. Under **Migration tool**, select the link to add a tool, then select **Azure Migrate: Database Migration**, then select **Next**.

4. Once the tools are installed in Azure Migrate, the portal should show the **Azure Migrate - Databases** blade. Under **Azure Migrate: Database Assessment** select **+ Assess**.

5. Select **Download** to open the Data Migration Assistant download page. Copy the page URL to the clipboard.

6. Return to your remote desktop session with the **SmartHotelHost** VM. Open **Chrome** from the desktop and paste the Data Migration Assistant download URL into the address bar. Download and install the Data Migration Assistant, but do not launch it yet.

7.  From within **SmartHotelHost**, open **Windows Explorer** and navigate to the **C:\\Program Files\\Microsoft Data Migration Assistant** folder. Open the **Dma.exe.config** file using Notepad. Search for **AzureMigrate** and remove the **\<\!--** and **--\>** around the line setting the **EnableAssessmentUploadToAzureMigrate** key. **Save** the file and close Notepad when done.

8.  From within **SmartHotelHost** launch **Microsoft Data Migration Assistant** using the desktop icon. 

9.  In the Data Migration Assistant, select the **+ New** icon.  Fill in the project details as follows:

    - Project type: **Assessment**
  
    - Project name: **SmartHotelAssessment**
  
    - Assessment type: **Database Engine**
  
    - Source server type: **SQL Server**
  
    - Target server type: **Azure SQL Database**
     
10. Select **Create** to create the project.

11. On the **Options** tab select **Next**.

12. On the **Select sources** page, in the **Connect to a server** dialog box, provide the connection details to the SQL Server, and then select **Connect**.

    - Server name: **192.168.0.6**
  
    - Authentication type: **SQL Server Authentication**
  
    - Username: **sa**
  
    - Password: **demo!pass123**
  
    - Encrypt connection: **Checked**
  
    - Trust server certificate: **Checked**

13. In the **Add sources** dialog box, select **SmartHotel.Registration**, then select **Add**.

14. Select **Start Assessment** to start the assessment. 
 
15. **Wait** for the assessment to complete, and review the results. The results should show two unsupported features, **Service Broker feature is not supported in Azure SQL Database** and **Azure SQL Database does not support EKM and Azure Key Vault integration**. For this migration, you can ignore these issues.

16. Select **Upload to Azure Migrate** to upload the database assessment to your Azure Migrate project (this button may take a few seconds to become enabled).

17. Select **Azure** from the dropdown on the right then select **Connect**. Enter your subscription credentials when prompted. Select your **Subscription** and **Azure Migrate Project** using the dropdowns, then select **Upload**. Once the upload is complete, select **OK** to dismiss the notification.

18. Minimize the remote desktop window and return to the **Azure Migrate - Databases** blade in the Azure portal. Refreshing the page should now show the assessed database.

1. In the Azure portal, navigate to the **RG-MAE-SmartHotelDB** resource group, and then to the database server.

2. Select **Private endpoint connections** under **Security**, then **+ Private endpoint**.

3. On the **Basics** tab that appears, enter the following configuration then select **Next: Resource**. 

    - Resource group: **RG-MAE-Migration**
  
    - Name: **SmartHotel-DB-for-DMS**
  
    - Region: **Select the same location as the DMSvnet (Should be the region closest to you)**.
  
  4. On the **Resource** tab, entering the following configuration then select **Next: Configuration**. 

    - Connection method: **Connect to an Azure resource in my directory**.
  
    - Subscription: **Select your subscription**.
  
    - Resource type: **Microsoft.Sql/servers**
  
    - Resource: **Your SQL database server name**.
  
    - Target sub-resource: **sqlServer**
  
5. On the **Configuration** tab enter the following configuration then select **Review + create**, then **Create**.

    - Virtual network: **DMSvnet**
  
    - Subnet: **DMS (10.1.0.0/24)**
  
    - Integrate with private DNS zone: **No**
  
8. Check that the Database Migration Service resource completed provisioning.

9. Navigate to the Database Migration Service resource blade in the **RG-MAE-Migrate** resource group and select **+ New Migration Project**.

10. the **New migration project** blade, enter **DBMigrate** as the project name. Leave the source server type as **SQL Server** and target server type as **Azure SQL Database**. Select **Choose type of activity** and select **Create project only**. Select **Save** then select **Create**.

11. The Migration Wizard opens, showing the **Select source** step. Complete the settings as follows, then select **Next: Select databases**.

    - Source SQL Server instance name: **10.0.0.4**
  
    - Authentication type: **SQL Authentication**
  
    - User Name: **sa**
  
    - Password: **demo!pass123**

    - Encryption connection: **Checked**
  
    - Trust server certificate: **Checked**

12. In the **Select databases** step, the **Smarthotel.Registration** database should already be selected. Select **Next: Select target**.

13. Complete the **Select target** step as follows, then select **Next: Summary**:

    - Target server name: **Value from your database, {something}.database.windows.net**.
  
    - Authentication type: **SQL Authentication**
  
    - User Name: **demouser**
  
    - Password: **demo!pass123**
  
    - Encrypt connection: **Checked**

    > **Note**: You can find the target server name in the Azure portal by browsing to your database.

14. At the **Project summary** step, review the settings and select **Save project** to create the migration project.

1. In the Azure portal should show a blade for the DBMigrate DMS project. Select **+ New Activity** and select **Schema only migration** from the drop-down.

2. The Migration Wizard is shown. Most settings are already populated from the existing migration project. At the **Select source** step, re-enter the source database password **demo!pass123**, then select **Next: Select target**.

3. At the **Select target** step, enter the password **demo!pass123** and select **Next: Select database and schema**.

4. At the **Select database and schema** step, check that the **SmartHotel.Registration** database is selected. Under **Target Database** select **smarthoteldb** and under **Schema Source** select **Generate from source**. Select **Next: Summary**.

5. At the **Summary** step, enter **SchemaMigration** as the **Activity name**. Select **Start migration** to start the schema migration process.

6. The schema migration will begin. Select the **Refresh** button and watch the migration progress, until it shows as **Completed**.

1. Return to the Azure portal blade for your **DBMigrate** migration project in DMS. Select **+ New Activity** and select **Offline data migration** from the drop-down.

2. The Migration Wizard is shown. Most settings are already populated from the existing migration project. At the **Select source** step, re-enter the source database password **demo!pass123**, then select **Next: Select target**.

 3. At the **Select target** step, enter the password **demo!pass123** and select **Next: Map to target databases**.

4. At the **Map to target databases** step, check the **SmartHotel.Registration** database. Under **Target Database** select **smarthoteldb**. Select **Next: Configure migration settings**.

5. The **Configure migration settings** step allows you to specify which tables should have their data migrated. Select the **Bookings** table (Make sure the **MigrationHistory** table is not checked) and select **Next: Summary**.

6. At the **Migration summary** step, enter **DataMigration** as the **Activity name**. Select **Start migration**.

7. The data migration will begin. Select the **Refresh** button and watch the migration progress, until it shows as **Completed**.

## Exercise #04 - Application and Web Server Migration (60 minutes)

1. In the Azure portal's left navigation, select **+ Create a resource**, then search for and select **Storage account**, followed by **Create**.

2. In the **Create storage account** blade, on the **Basics** tab, use the following values:

    - Subscription: **Select your Azure subscription**.
  
    - Resource group: (create new) **RG-MAE-SmartHotel**
  
    - Storage account name: **samaesmarthotel\[unique number\]**
  
    - Location: **Brazil South**
  
    - Account kind: **Storage (general purpose v2)**.
  
    - Replication: **Locally-redundant storage (LRS)**

3. Select **Review + create**, then select **Create**.

1. In the Azure portal's left navigation, select **All services**, then select **Networking**, followed by **Virtual network**.

1. Select **VNET-MAE-Hub**, and select **Address space** under **Settings** create a new address space, enter the following values:

    - Address space: **192.168.0.0/16**.
  
1. On the **Address space**, select **Save**.

3. Select **Subnets**, and enter the following configuration..
    
    - First subnet: Select **Add subnet** and enter the following then select **Add**

        - Subnet name: **SmartHotel**
   
        - Address range: **192.168.0.0/25**
  
    - Second subnet: Select **Add subnet** and enter the following then select **Add**. 

        - Subnet name: **SmartHotelDB**
   
        - Address range: **192.168.0.128/25**
   
1. **Wait** for the deployment to complete.

1. Return to the **Azure Migrate** blade in the Azure Portal, and select **Servers** under **Migration goals** on the left. Under **Migration Tools**, select **Discover**.

    **Note:** You may need to add the migration tool yourself by following the link below the **Migration Tools** section, selecting **Azure Migrate: Server Migration**, then selecting **Add tool(s)**. 

2. In the **Discover machines** panel, under **Are your machines virtualized**, select **Yes, with Hyper-V**. Under **Target region**, which can be found in the Azure portal and check the confirmation checkbox. Select **Create resources** to begin the deployment of the Azure Site Recovery resource used by Azure Migrate: Server Migration for Hyper-V migrations.

    Once deployment is complete, the 'Discover machines' panel should be updated with additional instructions.
  
3. Copy the **Download** link for the Hyper-V replication provider software installer to your clipboard.

4. Open the **SmartHotelHost** remote desktop window, launch **Chrome** from the desktop shortcut, and paste the link into a new browser tab to download the Azure Site Recovery provider installer.

5. Return to the **Discover machines** page in your browser (outside the SmartHotelHost remote desktop session). Select the blue **Download** button and download the registration key file.

6. Open the file location in Windows Explorer, and copy the file to your clipboard. Return to the **SmartHotelHost** remote desktop session and paste the file to the desktop.

7. Still within the **SmartHotelHost** remote desktop session, open the **AzureSiteRecoveryProvider.exe** installer you downloaded a moment ago. On the **Microsoft Update** tab, select **Off** and select **Next**. Accept the default installation location and select **Install**.

8. When the installation has completed select **Register**. Browse to the location of the key file you downloaded. When the key is loaded select **Next**.

9.  Select **Connect directly to Azure Site Recovery without a proxy server** and select **Next**. The registration of the Hyper-V host with Azure Site Recovery will begin.

10. Wait for registration to complete (this may take several minutes). Then select **Finish**.

11. Minimize the SmartHotelHost remote desktop session and return to the Azure Migrate browser window. **Refresh** your browser, then re-open the **Discover machines** panel by selecting **Discover** under **Azure Migrate: Server Migration** and selecting **Yes, with Hyper-V** for **Are your machines virtualized?**.

12. Select **Finalize registration**, which should now be enabled.

13. Azure Migrate will now complete the registration with the Hyper-V host. **Wait** for the registration to complete. This may take several minutes.

14. Once the registration is complete, close the **Discover machines** panel.

16. The **Azure Migrate: Server Migration** panel should now show 5 discovered servers.

1. Under **Azure Migrate: Server Migration**, select **Replicate**. This opens the **Replicate** wizard.

2. In the **Source settings** tab, under **Are your machines virtualized?**, select **Yes, with Hyper-V** from the drop-down. Then select **Next**.

3. In the **Virtual machines** tab, under **Import migration settings from an assessment**, select **Yes, apply migration settings from an Azure Migrate assessment**. Select the **SmartHotel VMs** VM group and the **SmartHotelAssessment** migration assessment.

4. The **Virtual machines** tab should now show the virtual machines included in the assessment. Select the **UbuntuWAF**, **smarthotelweb1** and **smarthotelweb2** virtual machines, then select **Next**.

5. In the **Target settings** tab, select your subscription and the existing **RG-SmartHotel** resource group. Under **Replication storage account** select the **sasmarthotelmigrate...** storage account and under **Virtual Network** select **VNET-MAE-Hub**. Under **Subnet** select **SmartHotel**. Select **Next**.

    > **Note:** For simplicity, in this lab you will not configure the migrated VMs for high availability, since each application tier is implemented using a single VM.

6. In the **Compute** tab, select the **Standard_B2s** VM size for each virtual machine. Select the **Windows** operating system for the **smarthotelweb** virtual machines and the **Linux** operating system for the **UbuntuWAF** virtual machine. Select **Next**. 

    > **Note**: If you are using an Azure Pass subscription, your subscription may not have a quota allocated for B2s virtual machines. In this case, use other virtual machines instead.

7. In the **Disks** tab, review the settings but do not make any changes. Select **Next**, then select **Replicate** to start the server replication.

8. In the **Azure Migrate - Servers** blade, under **Azure Migrate: Server Migration**, select the **Overview** button.

9. Confirm that the 3 machines are replicating.

10. Select **Replicating Machines** under **Manage** on the left.  Select **Refresh** occasionally and wait until all three machines have a **Protected** status, which shows the initial replication is complete. This will take several minutes.

1. Still using the **Azure Migrate: Server Migration - Replicating machines** blade, select the **smarthotelweb1** virtual machine. This opens a detailed migration and replication blade for this machine. Take a moment to study this information.

2. Select **Compute and Network** under **General** on the left, then select **Edit**.

3. Confirm that the VM is configured to use the **B2s** VM size (or other if using an Azure Pass subscription) and that **Use managed disks** is set to **Yes**.

4. Under **Network Interfaces**, select **InternalNATSwitch** to open the network interface settings.

5. Change the **Private IP address** to **192.168.0.4**.

6. Select **OK** to close the network interface settings blade, then **Save** the **smarthotelweb1** settings.

7. Repeat these steps to configure the private IP address for the other VMs.
 
    - For **smarthotelweb2** use private IP address **192.168.0.5**
 
    - For **UbuntuWAF** use private IP address **192.168.0.8**

1. Return to the **Azure Migrate: Server Migration** overview blade. Under **Step 3: Migrate**, select **Migrate**.

2. On the **Migrate** blade, select the 4 virtual machines then select **Migrate** to start the migration process.

    > **Note**: You can optionally choose whether the on-premises virtual machines should be automatically shut down before migration to minimize data loss. Either setting will work for this lab.

3. The migration process will start.

4. To monitor progress, select **Jobs** under **Manage** on the left and review the status of the three **Planned failover** jobs.

5. **Wait** until all three **Planned failover** jobs show a **Status** of **Successful**. You should not need to refresh your browser. This could take up to 15 minutes.

6. Navigate to the **RG-MAE-SmartHotel** resource group and check that the VM, network interface, and disk resources have been created for each of the virtual machines being migrated.

1. The application tier machine **smarthotelweb2** is configured to connect to the application database running on the **smarthotelsql** machine.

> **Note**: You do not need to update any configuration files on **smarthotelweb1** or the **UbuntuWAF** VMs, since the migration has preserved the private IP addresses of all virtual machines they connect with.

1. Navigate to the **UbuntuWAF** VM blade, select **Networking** under **Settings** on the left, then select the network interface.

2. Select **IP configuration** under **Settings** on the left, then select the IP configuration listed.

3. Set the **Public IP address** to **Associate**, and create a new public IP address named **PI-UbuntuWAF**. Choose a **Basic** tier IP address with **Dynamic** assignment. **Save** your changes.

4. Return to the **UbuntuWAF** VM overview blade and copy the **Public IP address** value.

5. Open a new browser tab and paste the IP address into the address bar. Verify that the SmartHotel360 application is now available in Azure.

1. End of Hands-on.

1. Continue in the **Treinamento Azure Expert**.