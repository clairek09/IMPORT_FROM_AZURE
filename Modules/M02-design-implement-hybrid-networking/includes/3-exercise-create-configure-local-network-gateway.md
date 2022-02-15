> [!NOTE] 
> To complete this exercise, you will need a Microsoft Azure subscription. If you don't already have one, you can sign up for a free trial at https://azure.com/free.

In this exercise you will configure a virtual network gateway to connect the Contoso Core Services VNet and Manufacturing VNet.

In this exercise, you will:

 -  Task 1: Create CoreServicesVnet and ManufacturingVnet
 -  Task 2: Create CoreServicesTestVM
 -  Task 3: Create ManufacturingTestVM
 -  Task 4: Connect to the Test VMs using RDP
 -  Task 5: Test the connection between the VMs
 -  Task 6: Create CoreServicesVnet Gateway
 -  Task 7: Create ManufacturingVnet Gateway
 -  Task 8: CoreServicesVnet to ManufacturingVnet
 -  Task 9: Connect ManufacturingVnet to CoreServicesVnet
 -  Task 10: Verify that the connections connect
 -  Task 11: Test the connection between the VMs

## Task 1: Create CoreServicesVnet and ManufacturingVnet

1.  In the Azure portal, open the **PowerShell** session within the **Cloud Shell** pane.
2.  In the toolbar of the Cloud Shell pane, click the Upload/Download files icon, in the drop-down menu, click Upload and upload the following files azuredeploy.json and azuredeploy.parameters.json into the Cloud Shell home directory. [Azure Resource Manager Templates for creating VNets](https://github.com/MicrosoftLearning/AZ-700-Designing-and-Implementing-Microsoft-Azure-Networking-Solutions/tree/master/Allfiles/Exercises/M02)
3.  Deploy the following Azure Resource Manager templates to create the virtual network and subnets needed for this exercise:
    
    ```powershell
    $RGName = "ContosoResourceGroup"
    ```
    
    ```powershell
    #create resource group if it doesnt exist
    ```
    
    ```powershell
    New-AzResourceGroup -Name $RGName -Location "eastus"
    ```
    
    ```powershell
    
    New-AzResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile azuredeploy.json -TemplateParameterFile azuredeploy.parameters.json
    
    ```

## Task 2: Create CoreServicesTestVM

1. In the Azure portal, open the **PowerShell** session within the **Cloud Shell** pane.

2. In the toolbar of the Cloud Shell pane, click the Upload/Download files icon, in the drop-down menu, click Upload and upload the following files **CoreServicesVMazuredeploy.json** and **CoreServicesVMazuredeploy.parameters.json** into the Cloud Shell home directory from the source folder **F:\Allfiles\Exercises\M02**.

3. Deploy the following ARM templates to create the VMs needed for this exercise:

   ```powershell
   $RGName = "ContosoResourceGroup"
   
   New-AzResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile CoreServicesVMazuredeploy.json -TemplateParameterFile CoreServicesVMazuredeploy.parameters.json
   ```
  
4. When the deployment is complete, go to the Azure portal home page, and then select **Virtual Machines**.

5. Verify that the virtual machine has been created.

## Task 3: Create ManufacturingTestVM

1. In the Azure portal, open the **PowerShell** session within the **Cloud Shell** pane.

2. In the toolbar of the Cloud Shell pane, click the Upload/Download files icon, in the drop-down menu, click Upload and upload the following files **ManufacturingVMazuredeploy.json** and **ManufacturingVMazuredeploy.parameters.json** into the Cloud Shell home directory from the source folder **F:\Allfiles\Exercises\M02**.

3. Deploy the following ARM templates to create the VMs needed for this exercise:

   ```powershell
   $RGName = "ContosoResourceGroup"
   
   New-AzResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile ManufacturingVMazuredeploy.json -TemplateParameterFile ManufacturingVMazuredeploy.parameters.json
   ```
  
4. When the deployment is complete, go to the Azure portal home page, and then select **Virtual Machines**.

5. Verify that the virtual machine has been created.

## Task 4: Connect to the Test VMs using RDP

1.  On the Azure portal home page, select **Virtual Machines**.
2.  Select **ManufacturingTestVM**.
3.  In **ManufacturingTestVM**, select **Connect > RDP**.
4.  In **ManufacturingTestVM \| Connect**, select **Download RDP file**.
5.  Save the RDP file to your desktop.
6.  Connect to ManufacturingTestVM using the RDP file, and the username and password you specified when you created the VM.
7.  On the Azure portal home page, select **Virtual Machines**.
8.  Select **CoreServicesTestVM**.
9.  In **CoreServicesTestVM**, select **Connect > RDP**.
10. In **CoreServicesTestVM \| Connect**, select **Download RDP file**.
11. Save the RDP file to your desktop.
12. Connect to CoreServicesTestVM using the RDP file, and the username and password you specified when you created the VM.
13. On both VMs, in **Choose privacy settings for your device**, select **Accept**.
14. On both VMs, in **Networks**, select **Yes**.
15. On CoreServicesTestVM, open PowerShell, and run the following command: ipconfig
16. Note the IPv4 address.

## Task 5: Test the connection between the VMs

1.  On the **ManufacturingTestVM**, open PowerShell.
2.  Use the following command to verify that there is no connection to CoreServicesTestVM on CoreServicesVnet. Be sure to use the IPv4 address for CoreServicesTestVM.
    
    ```Powershell
    Test-NetConnection 10.20.20.4 -port 3389
    
    ```
3.  The test connection should fail, and you will see a result similar to the following:

:::image type="content" source="../media/test-netconnection-fail-41df8b2b.png" alt-text="Test-NetConnection failed.":::


## Task 6: Create CoreServicesVnet Gateway

1.  In **Search resources, services, and docs (G+/)**, enter **Virtual network gateway**, and then select **Virtual network gateways** from the results.

:::image type="content" source="../media/virtual-network-gateway-search-0baf93c0.png" alt-text="Search for virtual network gateway on Azure portal.":::


2.  In Virtual network gateways, select **+ Create**.
3.  Use the information in the following table to create the virtual network gateway:
    
    |     **Tab**     |    **Section**    |                 **Option**                  |          **Value**           |
    |:---------------:|:-----------------:|:-------------------------------------------:|:----------------------------:|
    |     Basics      |  Project Details  |                Subscription                 |     No changes required      |
    |                 |                   |                ResourceGroup                |     ContosoResourceGroup     |
    |                 | Instance Details  |                    Name                     |   CoreServicesVnetGateway    |
    |                 |                   |                   Region                    |           East US            |
    |                 |                   |                Gateway type                 |             VPN              |
    |                 |                   |                  VPN type                   |         Route-based          |
    |                 |                   |                     SKU                     |            VpnGw1            |
    |                 |                   |                 Generation                  |         Generation1          |
    |                 |                   |               Virtual network               |       CoreServicesVnet       |
    |                 |                   |                   Subnet                    | GatewaySubnet (10.20.0.0/27) |
    |                 | Public IP address |              Public IP address              |          Create new          |
    |                 |                   |           Public IP address name            |  CoreServicesVnetGateway-ip  |
    |                 |                   |            Public IP address SKU            |            Basic             |
    |                 |                   |          Enable active-active mode          |           Disabled           |
    |                 |                   |                Configure BGP                |           Disabled           |
    | Review + create |                   | Review your settings and select **Create**. |                              |

> [!NOTE]
> It can take up to 45 minutes to create a virtual network gateway.

## Task 7: Create ManufacturingVnet Gateway

1.  In **Search resources, services, and docs (G+/)**, enter **Virtual network gateway**, and then select **Virtual network gateways** from the results.
2.  In Virtual network gateways, select **+ Create**.
3.  Use the information in the following table to create the virtual network gateway:
    
    |     **Tab**     |    **Section**    |                 **Option**                  |          **Value**           |
    |:---------------:|:-----------------:|:-------------------------------------------:|:----------------------------:|
    |     Basics      |  Project Details  |                Subscription                 |     No changes required      |
    |                 |                   |                ResourceGroup                |     ContosoResourceGroup     |
    |                 | Instance Details  |                    Name                     |   ManufacturingVnetGateway   |
    |                 |                   |                   Region                    |         West Europe          |
    |                 |                   |                Gateway type                 |             VPN              |
    |                 |                   |                  VPN type                   |         Route-based          |
    |                 |                   |                     SKU                     |            VpnGw1            |
    |                 |                   |                 Generation                  |         Generation1          |
    |                 |                   |               Virtual network               |      ManufacturingVnet       |
    |                 |                   |                   Subnet                    | GatewaySubnet (10.30.0.0/27) |
    |                 | Public IP address |              Public IP address              |          Create new          |
    |                 |                   |           Public IP address name            | ManufacturingVnetGateway-ip  |
    |                 |                   |            Public IP address SKU            |            Basic             |
    |                 |                   |          Enable active-active mode          |           Disabled           |
    |                 |                   |                Configure BGP                |           Disabled           |
    | Review + create |                   | Review your settings and select **Create**. |                              |

> [!NOTE]
> It can take up to 45 minutes to create a virtual network gateway.

## Task 8: Connect CoreServicesVnet to ManufacturingVnet

1.  In **Search resources, services, and docs (G+/)**, enter **Virtual network gateway**, and then select **Virtual network gateways** from the results.
2.  In Virtual network gateways, select **CoreServicesVnetGateway**.
3.  In CoreServicesGateway, select **Connections**, and then select **+ Add**.

> [!NOTE]
> You will not be able to complete this configuration until the virtual network gateways are fully deployed.

4.  Use the information in the following table to create the connection:
    
    |           **Option**           |             **Value**             |
    |:------------------------------:|:---------------------------------:|
    |              Name              | CoreServicesGW-to-ManufacturingGW |
    |        Connection type         |           VNet-to-VNet            |
    | First virtual network gateway  |      CoreServicesVnetGateway      |
    | Second virtual network gateway |     ManufacturingVnetGateway      |
    |        Shared key (PSK)        |              abc123               |
    |  Use Azure Private IP Address  |           Not selected            |
    |           Enable BGP           |           Not selected            |
    |          IKE Protocol          |               IKEv2               |
    |          Subscription          |        No changes required        |
    |         Resource group         |        No changes required        |
    |            Location            |              East US              |
5.  To create the connection, select **Create**.

## Task 9: Connect ManufacturingVnet to CoreServicesVnet

1.  In **Search resources, services, and docs (G+/)**, enter **Virtual network gateway**, and then select **Virtual network gateways** from the results.
2.  In Virtual network gateways, select **ManufacturingVnetGateway**.
3.  In CoreServicesGateway, select **Connections**, and then select **+ Add**.
4.  Use the information in the following table to create the connection:
    
    |           **Option**           |             **Value**             |
    |:------------------------------:|:---------------------------------:|
    |              Name              | ManufacturingGW-to-CoreServicesGW |
    |        Connection type         |           VNet-to-VNet            |
    | First virtual network gateway  |     ManufacturingVnetGateway      |
    | Second virtual network gateway |      CoreServicesVnetGateway      |
    |        Shared key (PSK)        |              abc123               |
    |  Use Azure Private IP Address  |           Not selected            |
    |           Enable BGP           |           Not selected            |
    |          IKE Protocol          |               IKEv2               |
    |          Subscription          |        No changes required        |
    |         Resource group         |        No changes required        |
    |            Location            |           West Europe             |
    |                                |                                   |
5.  To create the connection, select **Create**.

## Task 10: Verify that the connections connect

1.  In **Search resources, services, and docs (G+/)**, enter **connections**, and then select **connections** from the results.
2.  Wait until the status of both connections is **Connected**. You may need to refresh your screen.

:::image type="content" source="../media/connections-status-connected-a6a99c0a.png" alt-text="VPN Gateway connections successfully created.":::


## Task 11: Test the connection between the VMs

1.  On the **ManufacturingTestVM**, open PowerShell.
2.  Use the following command to verify that there is now a connection to CoreServicesTestVM on CoreServicesVnet. Be sure to use the IPv4 address for CoreServicesTestVM.
    
    ```Powershell
    Test-NetConnection 10.20.20.4 -port 3389
    
    ```
3.  The test connection should succeed, and you will see a result similar to the following:

:::image type="content" source="../media/test-connection-succeeded-ad24c05a.png" alt-text="Test-NetConnection succeeded.":::


Congratulations! You have configured a VNet-to-VNet connection by using a virtual network gateway.
