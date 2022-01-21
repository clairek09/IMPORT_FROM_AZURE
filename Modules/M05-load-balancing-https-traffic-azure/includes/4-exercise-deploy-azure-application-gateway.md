> [!NOTE]
> To complete this exercise, you will need a Microsoft Azure subscription. If you don't already have one, you can sign up for a free trial at https://azure.com/free.

In this exercise, you use the Azure portal to create an application gateway. Then you test it to make sure it works correctly.

The application gateway directs application web traffic to specific resources in a backend pool. You assign listeners to ports, create rules, and add resources to a backend pool. For the sake of simplicity, this article uses a simple setup with a public front-end IP, a basic listener to host a single site on the application gateway, a basic request routing rule, and two virtual machines in the backend pool.

For Azure to communicate between the resources that you create, it needs a virtual network. You can either create a new virtual network or use an existing one. In this example, you'll create a new virtual network while you create the application gateway. Application Gateway instances are created in separate subnets. You create two subnets in this example: one for the application gateway, and another for the backend servers.

In this exercise, you will:

 -  Task 1: Create an application gateway
 -  Task 2: Add backend targets
 -  Task 3: Add backend servers to backend pool
 -  Task 4: Test the application gateway

## Task 1: Create an application gateway

1.  Sign in to the [Azure portal](https://portal.azure.com/) with your Azure account.
2.  On any Azure portal page, in **Search resources, services and docs (G+/)**, enter application gateway, and then select **Application gateways** from the results. \[!div class="mx-imgBorder"\] :::image type="content" source="../media/search-application-gateway-36e26855.png" alt-text="Azure portal search for application gateway":::
    
3.  On the Application gateways page, select **+ Create**.
4.  On the Create application gateway **Basics** tab, enter, or select the following information:
    
    |     **Setting**     |                 **Value**                  |
    |:-------------------:|:------------------------------------------:|
    |    Subscription     |         Select your subscription.          |
    |   Resource group    | Select **Create new** ContosoResourceGroup |
    | Application Gateway |             ContosoAppGateway              |
    |       Region        |             Select **East US**             |
    |   Virtual Network   |           Select **Create new**            |
5.  In Create virtual network, enter, or select the following information:
    
    |    **Setting**    |             **Value**              |
    |:-----------------:|:----------------------------------:|
    |       Name        |            ContosoVNet             |
    | **ADDRESS SPACE** |                                    |
    |   Address range   |            10.0.0.0/16             |
    |    **SUBNETS**    |                                    |
    |    Subnet name    | Change **default** to **AGSubnet** |
    |   Address range   |            10.0.0.0/24             |
    |    Subnet name    |           BackendSubnet            |
    |   Address range   |            10.0.1.0/24             |
6.  Select **OK** to return to the Create application gateway Basics tab.
7.  Accept the default values for the other settings and then select **Next: Frontends**.
8.  On the **Frontends** tab, verify **Frontend IP address type** is set to **Public**.
9.  Select **Add new** for the **Public IP address** and enter AGPublicIPAddress for the public IP address name, and then select **OK**.
10. Select **Next: Backends**.
11. On the **Backends** tab, select **Add a backend pool**.
12. In the **Add a backend pool** window that opens, enter the following values to create an empty backend pool:
    
    |           **Setting**            |  **Value**  |
    |:--------------------------------:|:-----------:|
    |               Name               | BackendPool |
    | Add backend pool without targets |     Yes     |
13. In the **Add a backend pool** window, select **Add** to save the backend pool configuration and return to the **Backends** tab.
14. On the **Backends** tab, select **Next: Configuration**.
15. On the **Configuration** tab, you'll connect the frontend and backend pool you created using a routing rule.
16. In the **Routing rules** column, select **Add a routing rule**.
17. In the **Rule name** box, enter **RoutingRule**.
18. On the **Listener** tab, enter or select the following information:
    
    |  **Setting**  |     **Value**     |
    |:-------------:|:-----------------:|
    | Listener name |     Listener      |
    |  Frontend IP  | Select **Public** |
19. Accept the default values for the other settings on the **Listener** tab.

    :::image type="content" source="../media/routing-rule-listener-tab-f17d3760.png" alt-text="Azure portal add an Application Gateway routing rule":::


20. Select the **Backend targets** tab to configure the rest of the routing rule.
21. On the **Backend targets** tab, enter or select the following information:
    
    |  **Setting**  |   **Value**    |
    |:-------------:|:--------------:|
    |  Target type  |  Backend pool  |
    | HTTP Settings | **Create new** |
22. In **Add a HTTP setting**, enter or select the following information:
    
    |    **Setting**     |  **Value**  |
    |:------------------:|:-----------:|
    | HTTP settings name | HTTPSetting |
    |    Backend port    |     80      |
23. Accept the default values for the other settings in the **Add an HTTP setting** window, then select **Add** to return to **Add a routing rule**.
24. Select **Add** to save the routing rule and return to the **Configuration** tab.
25. Select **Next: Tags** and then **Next: Review + create**.
26. Review the settings on the **Review + create** tab
27. Select **Create** to create the virtual network, the public IP address, and the application gateway.

It may take several minutes for Azure to create the application gateway. Wait until the deployment finishes successfully before moving on to the next section.

## Task 2: Add backend targets

In this example, you'll use virtual machines as the target backend. You'll create two virtual machines as backend servers for the application gateway.

To do this, you'll:

 -  Create two new VMs, BackendVM1 and BackendVM2, to be used as backend servers.
 -  Install IIS on the virtual machines to verify that the application gateway was created successfully.
 -  Add the backend servers to the backend pool.

### Create virtual machines

1.  On any Azure portal page, in **Search resources, services and docs (G+/)**, enter virtual machine, and then select **Virtual machines** from the results.
2.  On the Virtual machines page, select **+ Create** &gt; **+ Virtual machine**.
3.  In **Create a virtual machine**, enter, or select the following information (If the setting is not listed, use the default value):
    
    |     **Setting**      |                    **Value**                     |
    |:--------------------:|:------------------------------------------------:|
    |    **Basics** tab    |                                                  |
    |     Subscription     |            Select your subscription.             |
    |    Resource group    |           Select ContosoResourceGroup            |
    | Virtual machine name |                    BackendVM1                    |
    |        Image         | Select **Windows Server 2022 Datacenter- Gen1**  |
    |       Username       |                     TestUser                     |
    |       Password       |                   TestPa$$w0rd                   |
    | Public inbound ports |                       None                       |
    |    **Networking**    |                                                  |
    |   Virtual network    |                   ContosoVnet                    |
    |        Subnet        |           BackendSubnet (10.0.1.0/24)            |
    |    **Management**    |                                                  |
    |   Boot diagnostics   |                     Disable                      |
4.  Accept the other defaults and then select **Review + create**.
5.  On the **Review + create** tab, review the settings, correct any validation errors, and then select **Create**.

Wait for the virtual machine creation to complete before continuing.

### Install IIS for testing

In this example, you install IIS on the virtual machines to verify Azure created the application gateway successfully.

1.  Open Azure PowerShell.
2.  Select **Cloud Shell** from the top navigation bar of the Azure portal and then select **PowerShell** from the drop-down list. :::image type="content" source="../media/application-gateway-extension.png" alt-text="Azure portal and Azure PowerShell Install IIS on backends":::
    
3.  Run the following command to install IIS on the virtual machine. Change the Location parameter if necessary:
    
    ```Azure
    Set-AzVMExtension `
    
    -ResourceGroupName ContosoResourceGroup `
    
    -ExtensionName IIS `
    
    -VMName BackendVM1 `
    
    -Publisher Microsoft.Compute `
    
    -ExtensionType CustomScriptExtension `
    
    -TypeHandlerVersion 1.4 `
    
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
    
    -Location EastUS
    
    ```
4.  Create a second virtual machine and install IIS by using the Create virtual machines and Install IIS for testing steps that you previously completed. Use BackendVM2 for the virtual machine name and for the **VMName** setting of the **Set-AzVMExtension** cmdlet.

## Task 3: Add backend servers to backend pool

1.  On the Azure portal menu, select **All resources** or search for and select All resources. Then select **ContosoAppGateway**.
2.  Under **Settings**, select **Backend pools**.
3.  Select **BackendPool**.
4.  On the Edit backend pool page, under **Backend targets**, in **Target type**, select **Virtual machine**.
5.  Under **Target**, select **BackendVM1.**
6.  In **Target type**, select **Virtual machine**.
7.  Under **Target**, select **BackendVM2.**

    :::image type="content" source="../media/edit-backend-pool-bf78b276.png" alt-text="Azure portal add target backends to backend pool":::


8.  Select **Save**.

Wait for the deployment to complete before proceeding to the next step.

## Task 4: Test the application gateway

Although IIS isn't required to create the application gateway, you installed it in this exercise to verify if Azure successfully created the application gateway.

### Use IIS to test the application gateway:

1.  Find the public IP address for the application gateway on its **Overview** page.

    :::image type="content" source="../media/app-gateway-public-ip-3b4bbcae.png" alt-text="Azure portal look up Frontend Public IP address":::


2.  Copy the public IP address, and then paste it into the address bar of your browser to browse that IP address.
3.  Check the response. A valid response verifies that the application gateway was successfully created and can successfully connect with the backend.

    :::image type="content" source="../media/browse-backend-79de39d0.png" alt-text="Broswer - display BackendVM1 or BackendVM2 depending which backend server reponds to request.":::


4.  Refresh the browser multiple times and you should see connections to both BackendVM1 and BackendVM2.

Congratulations! You have configured and tested an Azure Application Gateway.
