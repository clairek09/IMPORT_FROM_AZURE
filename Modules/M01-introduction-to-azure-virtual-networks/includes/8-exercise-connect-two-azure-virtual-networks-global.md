> [!NOTE]
> To complete this exercise, you will need a Microsoft Azure subscription. If you don't already have one, you can sign up for a free trial at https://azure.com/free.

## Exercise scenario

In this unit, you will configure connectivity between the CoreServicesVnet and the ManufacturingVnet by adding peerings to allow traffic flow.

In this unit, you will:

 -  Task 1: Create a Virtual Machine to test the configuration
 -  Task 2: Connect to the Test VMs using RDP
 -  Task 3: Test the connection between the VMs
 -  Task 4: Create VNet peerings between CoreServicesVnet and ManufacturingVnet
 -  Task 5: Test the connection between the VMs
 -  Task 6: Clean up resources

## Task 1: Create a Virtual Machine to test the configuration

In this section, you will create a test VM on the Manufacturing VNet to test if you can access resources inside another Azure virtual network from your ManufacturingVnet.

### Create ManufacturingVM
1. On the Azure home page, using the global search type **Virtual Machines** and select virtual machines under services.

2. In Virtual Machines, select **+ Create; + Virtual machine**.

3. Use the information in the following table to create your VM.
    
    :::row:::
      :::column:::
        **Tab**
      :::column-end:::
      :::column:::
        **Option**
      :::column-end:::
      :::column:::
        **Value**
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        Basics
      :::column-end:::
      :::column:::
        Resource group
      :::column-end:::
      :::column:::
        ContosoResourceGroup
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Virtual machine name
      :::column-end:::
      :::column:::
        ManufacturingVM
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Region
      :::column-end:::
      :::column:::
        West Europe
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Availability options
      :::column-end:::
      :::column:::
        No infrastructure redundancy required
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Image
      :::column-end:::
      :::column:::
        Windows Server 2022 Datacenter- Gen1
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Azure Spot instance
      :::column-end:::
      :::column:::
        Not selected
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Size
      :::column-end:::
      :::column:::
        Standard\_D2s\_v3 - 2vcpus, 8GiB memory
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Username
      :::column-end:::
      :::column:::
        TestUser
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Password
      :::column-end:::
      :::column:::
        TestPa$$w0rd!
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Public inbound ports
      :::column-end:::
      :::column:::
        Allow selected ports
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Select inbound ports
      :::column-end:::
      :::column:::
        RDP (3389)
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        Disks
      :::column-end:::
      :::column:::
        No changes required
      :::column-end:::
      :::column:::
        
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        Networking
      :::column-end:::
      :::column:::
        Virtual network
      :::column-end:::
      :::column:::
        ManufacturingVnet
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Subnet
      :::column-end:::
      :::column:::
        ManufacturingSystemSubnet (10.30.10.0/24)
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Public IP
      :::column-end:::
      :::column:::
        (new) ManufacturingVM-ip
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        NIC network security group
      :::column-end:::
      :::column:::
        Basic
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Public inbound ports
      :::column-end:::
      :::column:::
        Allow selected ports
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Select inbound ports
      :::column-end:::
      :::column:::
        RDP (3389)
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Load balancing
      :::column-end:::
      :::column:::
        Not selected
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        Management
      :::column-end:::
      :::column:::
        No changes required
      :::column-end:::
      :::column:::
        
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        Advanced
      :::column-end:::
      :::column:::
        No changes required
      :::column-end:::
      :::column:::
        
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        Tags
      :::column-end:::
      :::column:::
        No changes required
      :::column-end:::
      :::column:::
        
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        Review + create
      :::column-end:::
      :::column:::
        Review your settings and select Create
      :::column-end:::
      :::column:::
        
      :::column-end:::
    :::row-end:::
    
4.  When the deployment is complete, select **Go to resource**.

## Task 2: Connect to the Test VMs using RDP

1.  On the Azure portal home page, select **Virtual Machines**.
2.  Select **ManufacturingVM**.
3.  In ManufacturingVM, select **Connect > RDP**.
4.  In ManufacturingVM \| Connect, select **Download RDP file**.
5.  Save the RDP file to your desktop.
6.  Connect to ManufacturingVM using the RDP file, and the username and password you specified when you created the VM.
7.  On the Azure portal home page, select **Virtual Machines**.
8.  Select **TestVM1**.
9.  In TestVM1, select **Connect > RDP**.
10. In TestVM1 \| Connect, select **Download RDP file**.
11. Save the RDP file to your desktop.
12. Connect to TestVM1 using the RDP file, and the username and password you specified when you created the VM.
13. On both VMs, in **Choose privacy settings for your device**, select **Accept**.
14. On both VMs, in **Networks**, select **Yes**.
15. On TestVM1, open a PowerShell prompt, and run the following command: ipconfig
16. Note the IPv4 address.

## Task 3: Test the connection between the VMs

1.  On the ManufacturingVM, open a PowerShell prompt.
2.  Use the following command to verify that there is no connection to TestVM1 on CoreServicesVnet. Be sure to use the IPv4 address for TestVM1.
    
    :::row:::
      :::column:::
        **PowerShell**
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        Test-NetConnection 10.20.20.4 -port 3389
      :::column-end:::
    :::row-end:::
    
3.  The test connection should fail, and you will see a result similar to the following: 

:::image type="content" source="../media/test-netconnection-fail-7f001091.png" alt-text="PowerShell window with Test-NetConnection 10.20.20.4 -port 3389 showing failed":::
    

## Task 4: Create VNet peerings between CoreServicesVnet and ManufacturingVnet

1.  On the Azure home page, select **Virtual Networks**, and then select **CoreServicesVnet**.
2.  In CoreServicesVnet, under **Settings**, select **Peerings**.

    :::image type="content" source="../media/create-peering-coreservicesvnet-53ca7694.png" alt-text="screen shot of core services VNet Peering settings":::
    
3.  On CoreServicesVnet \| Peerings, select **+ Add**.
4.  Use the information in the following table to create the peering.
    
    :::row:::
      :::column:::
        **Section**
      :::column-end:::
      :::column:::
        **Option**
      :::column-end:::
      :::column:::
        **Value**
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        This virtual network
      :::column-end:::
      :::column:::
        
      :::column-end:::
      :::column:::
        
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Peering link name
      :::column-end:::
      :::column:::
        CoreServicesVnet-to-ManufacturingVnet
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Traffic to remote virtual network
      :::column-end:::
      :::column:::
        Allow (default)
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Traffic forwarded from remote virtual network
      :::column-end:::
      :::column:::
        Allow (default)
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Virtual network gateway or Route Server
      :::column-end:::
      :::column:::
        None (default)
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        Remote virtual network
      :::column-end:::
      :::column:::
        
      :::column-end:::
      :::column:::
        
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Peering link name
      :::column-end:::
      :::column:::
        ManufacturingVnet-to-CoreServicesVnet
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Virtual network deployment model
      :::column-end:::
      :::column:::
        Resource Manager
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        I know my resource ID
      :::column-end:::
      :::column:::
        Not selected
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Subscription
      :::column-end:::
      :::column:::
        MOC Subscription-lodxxxxxxxx
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Virtual network
      :::column-end:::
      :::column:::
        ManufacturingVnet
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Traffic to remote virtual network
      :::column-end:::
      :::column:::
        Allow (default)
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Traffic forwarded from remote virtual network
      :::column-end:::
      :::column:::
        Allow (default)
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        Virtual network gateway or Route Server
      :::column-end:::
      :::column:::
        None (default)
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        Review your settings and select Add.
      :::column-end:::
      :::column:::
        
      :::column-end:::
      :::column:::
        
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        
      :::column-end:::
      :::column:::
        
      :::column-end:::
      :::column:::
        
      :::column-end:::
    :::row-end:::
    
5.  In CoreServicesVnet \| Peerings, verify that the **CoreServicesVnet-to-ManufacturingVnet** peering is listed.
6.  Under Virtual networks, select **ManufacturingVnet**, and verify the **ManufacturingVnet-to-CoreServicesVnet** peering is listed.

## Task 5: Test the connection between the VMs

1.  On the ManufacturingVM, open a PowerShell prompt.
2.  Use the following command to verify that there is now a connection to TestVM1 on CoreServicesVnet.
    
    :::row:::
      :::column:::
        **PowerShell**
      :::column-end:::
    :::row-end:::
    :::row:::
      :::column:::
        Test-NetConnection 10.20.20.4 -port 3389
      :::column-end:::
    :::row-end:::
    
3.  The test connection should succeed, and you will see a result similar to the following: 

    :::image type="content" source="../media/test-connection-succeeded-7c2cc7f6.png" alt-text="Screenshot of test connection succeeded":::
    

Congratulations! You have successful configured connectivity between VNets by adding peerings.

## Task 6: Clean up resources

> [!NOTE]
> Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

1.  In the Azure portal, open the **PowerShell** session within the **Cloud Shell** pane.
2.  Delete all resource groups you created throughout the labs of this module by running the following command:
    
    ```powershell
    Remove-AzResourceGroup -Name 'NAME OF THE RG' -Force -AsJob
    
    ```

> [!NOTE] 
> The command executes asynchronously (as determined by the -AsJob parameter), so while you will be able to run another PowerShell command immediately afterwards within the same PowerShell session, it will take a few minutes before the resource groups are actually removed.
