

In this exercise, you will create an internal load balancer for the fictional Contoso Ltd organization. 

The steps to create an internal load balancer, are very similar to those you have already learnt about in this module, to create a public load balancer. The key difference is that with a public load balancer the front end is accessed via a public IP address, and you test connectivity from a host which is located outside your virtual network; whereas, with an internal load balancer, the front end is a private IP address inside your virtual network, and you test connectivity from a host inside the same network.

The diagram below illustrates the environment you will be deploying in this exercise.

![Picture 15](../media/exercise-internal-standard-load-balancer-environment-diagram.png)

 

## Create the virtual network

In this section, you will create a virtual network and a subnet.

1. Log in to the Azure portal.

2. On the Azure portal home page, click **Create a resource**, then **Networking**, then select **Virtual Network** (if this resource type is not listed on the page, use the search box at the top of the page to search for it and select it).

3. Click **Create**.

![Picture 2](../media/create-virtual-network-1.png)

4. On the **Basics** tab, use the information in the table below to create the virtual network.

   | **Setting**    | **Value**                                  |
   | -------------- | ------------------------------------------ |
   | Subscription   | Select your subscription                   |
   | Resource group | Select **Create  new**  Name: **IntLB-RG** |
   | Name           | **IntLB-VNet**                             |
   | Region         | **(US) West US**                           |


5. Click **Next : IP Addresses**.

6. On the **IP Addresses** tab, in the **IPv4 address space** box, type **10.1.0.0/16**.

7. Under **Subnet name**, select the word **default**.

8. In the **Edit subnet** pane, provide a subnet name of **myBackendSubnet**, and a subnet address range of **10.1.0.0/24**.

9. Click **Save**.

10. Click **Next : Security**.

11. Under **BastionHost** select **Enable**, then enter the information from the table below.

    | **Setting**                       | **Value**                                     |
    | --------------------------------- | --------------------------------------------- |
    | Bastion name                      | **myBastionHost**                             |
    | AzureBastionSubnet address space  | **10.1.1.0/24**                               |
    | Public IP address                 | Select **Create  new**  Name: **myBastionIP** |


12. Click **Review + create**.

13. Click **Create**.

## Create the load balancer

In this section, you will create an internal Standard SKU load balancer. The reason we are creating a Standard SKU load balancer here in the exercise, instead of a Basic SKU load balance, is for later exercises that require a Standard SKU version of the load balancer.

1. On the Azure portal home page, click **Create a resource**.

2. In the search box at the top of the page, type **Load Balancer**, then press **Enter** (**Note:** do not select one from the list).

3. Scroll down to the bottom of the page and select **Load Balancer** (the one that says 'Microsoft' and 'Azure Service' under the name).

4. Click **Create**.
   ![Picture 3](../media/create-load-balancer-4.png)

5. On the **Basics** tab, use the information in the table below to create the load balancer.

   | **Setting**           | **Value**                |
   | --------------------- | ------------------------ |
   | Subscription          | Select your subscription |
   | Resource group        | **IntLB-RG**             |
   | Name                  | **myIntLoadBalancer**    |
   | Region                | **(US) West US**         |
   | Type                  | **Internal**             |
   | SKU                   | **Standard**             |
   | Virtual network       | **IntLB-VNet**           |
   | Subnet                | **myBackendSubnet**      |
   | IP address assignment | **Dynamic**              |


6. Click **Review + create**.

7. Click **Create**.

## Create load balancer resources

In this section, you will configure load balancer settings for a backend address pool, then create a health probe and a load balancer rule.

### Create a backend pool

The backend address pool contains the IP addresses of the virtual NICs connected to the load balancer.

1. On the Azure portal home page, click **All resources**, then click on **myIntLoadBalancer** from the resources list.

2. Under **Settings**, select **Backend pools**, and then click **Add**.

3. On the **Add backend pool** page, enter the information from the table below.

   | **Setting**     | **Value**            |
   | --------------- | -------------------- |
   | Name            | **myBackendPool**    |
   | Virtual network | **IntLB-VNet**       |
   | Associated to   | **Virtual machines** |


4. Click **Add**.
   ![Picture 4](../media/create-backendpool.png)

 

### Create a health probe

The load balancer monitors the status of your app with a health probe. The health probe adds or removes VMs from the load balancer based on their response to health checks. Here you will create a health probe to monitor the health of the VMs.

1. From the **Backend pools** page of your load balancer, under **Settings**, click **Health probes**, then click **Add**.

2. On the **Add health probe** page, enter the information from the table below.

   | **Setting**         | **Value**         |
   | ------------------- | ----------------- |
   | Name                | **myHealthProbe** |
   | Protocol            | **HTTP**          |
   | Port                | **80**            |
   | Path                | **/**             |
   | Interval            | **15**            |
   | Unhealthy threshold | **2**             |


3. Click **Add**.
   ![Picture 5](../media/create-healthprobe.png)

 

### Create a load balancer rule

A load balancer rule is used to define how traffic is distributed to the VMs. You define the frontend IP configuration for the incoming traffic and the backend IP pool to receive the traffic. The source and destination port are defined in the rule. Here you will create a load balancer rule.

1. From the **Backend pools** page of your load balancer, under **Settings**, click **Load balancing rules**, then click **Add**.

2. On the **Add load balancing rule** page, enter the information from the table below.

   | **Setting**            | **Value**                |
   | ---------------------- | ------------------------ |
   | Name                   | **myHTTPRule**           |
   | IP Version             | **IPv4**                 |
   | Frontend IP address    | **LoadBalancerFrontEnd** |
   | Protocol               | **TCP**                  |
   | Port                   | **80**                   |
   | Backend port           | **80**                   |
   | Backend pool           | **myBackendPool**        |
   | Health probe           | **myHealthProbe**        |
   | Session persistence    | **None**                 |
   | Idle timeout (minutes) | **15**                   |
   | Floating IP            | **Disabled**             |


3. Click **Add**.
   ![Picture 6](../media/create-loadbalancerrule.png)

 

## Create backend servers

In this section, you will create three VMs, that will be in the same availability set, for the backend pool of the load balancer, add the VMs to the backend pool, and then install IIS on the three VMs to test the load balancer.

### Create VMs

In this section, you will create three VMs (myVM1, myVM2, myVM3). When creating the first VM you will also create a new Availability Set, and then as you create the next two VMs, they will be added to the Availability Set. You will then add these VMs to the backend pool of the load balancer that you created previously.

1. On the Azure portal home page, click **Create a resource**, then **Compute**, then select **Virtual machine** (if this resource type is not listed on the page, use the search box at the top of the page to search for it and select it).

2. On the **Create a virtual machine** page, on the **Basics** tab, use the information in the table below to create the first VM.

   | **Setting**          | **Value**                                            |
   | -------------------- | ---------------------------------------------------- |
   | Subscription         | Select your subscription                             |
   | Resource group       | **IntLB-RG**                                         |
   | Virtual machine name | **myVM1**                                            |
   | Region               | **(US) West US**                                     |
   | Availability options | **Availability Set**                                 |
   | Availability set     | Select **Create  new**  Name:  **myAvailabilitySet** |
   | Image                | **Windows Server 2019 Datacenter - Gen 1**           |
   | Size                 | **Standard_DS1_v2 - 1 vCPU, 3.5 GiB  memory**        |
   | Username             | **TestUser**                                         |
   | Password             | **TestPa$$w0rd!**                                    |
   | Confirm password     | **TestPa$$w0rd!**                                    |


3. Click **Next : Disks**, then click **Next : Networking**. 

4. On the **Networking** tab, use the information in the table below to configure networking settings.

   | **Setting**                                                  | **Value**                               |
   | ------------------------------------------------------------ | --------------------------------------- |
   | Virtual network                                              | **IntLB-VNet**                          |
   | Subnet                                                       | **myBackendSubnet**                     |
   | Public IP                                                    | Change to **None**                      |
   | NIC network security group                                   | **Advanced**                            |
   | Configure network security group                             | Select **Create  new**  Name: **myNSG** |
   | Place this virtual machine behind an  existing load balancing solution? | **Off** (unchecked)                     |


5. Click **Review + create**.

6. Click **Create**.

7. When the deployment of the first VM is complete, click **Create another VM**.

8. On the **Create a virtual machine** page, repeat steps 2-6 above to create a second VM. Use the same settings as before except for the information in the table below. 

   | **Setting**                      | **Value**                                 |
   | -------------------------------- | ----------------------------------------- |
   | Virtual machine name             | **myVM2**                                 |
   | Availability set                 | Select the existing **myAvailabilitySet** |
   | Configure network security group | Select the existing **myNSG**             |


9. When the deployment of the second VM is complete, click **Create another VM**.

10. On the **Create a virtual machine** page, repeat steps 2-6 above to create a third VM. Use the same settings as before except for the information in the table below. 

    | **Setting**                      | **Value**                                 |
    | -------------------------------- | ----------------------------------------- |
    | Virtual machine name             | **myVM3**                                 |
    | Availability set                 | Select the existing **myAvailabilitySet** |
    | Configure network security group | Select the existing **myNSG**             |


### Add VMs to the backend pool

1. On the Azure portal home page, click **All resources**, then click on **myIntLoadBalancer** from the resources list.

2. Under **Settings**, select **Backend pools**., and then select **myBackendPool**.

3. In the **Associated to** box, select **Virtual machines**.

4. Under **Virtual machines**, click **Add**.

5. Select the checkboxes for all 3 VMs (**myVM1**, **myVM2**, and **myVM3**), then click **Add**.

6. On the **myBackendPool** page, click **Save**.
   ![Picture 7](../media/add-vms-backendpool.png)

 

### Install IIS on the VMs

1. On the Azure portal home page, click **All resources**, then click on **myVM1** from the resources list.

2. On the **Overview** page, select **Connect**, then **Bastion**.

3. Click **Use Bastion**.

4. In the **Username** box, type **TestUser** and in the **Password** box, type **TestPa$$w0rd!**, then click **Connect**.

5. The **myVM1** window will open in another browser tab.

6. If a **Networks** pane appears, click **Yes**.

7. Click the **Windows Start icon** in the bottom left corner of the window, then click the **Windows PowerShell** tile.

8. To install IIS, run the following command in PowerShell: ```Install-WindowsFeature -name Web-Server -IncludeManagementTools```

9. To remove the existing default web home page, run the following command in PowerShell: ```Remove-Item C:\inetpub\wwwroot\iisstart.htm```

10. To add a new default web home page and add content to it, run the following command in PowerShell: ```Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)```

11. Close the Bastion session to **myVM1** by closing the browser tab.

12. Repeat steps 1-11 above twice more to install IIS and the updated default home page on the **myVM2** and **myVM3** virtual machines.

 

## Test the load balancer

In this section, you will create a test VM, and then test the load balancer.

### Create test VM

1. On the Azure portal home page, click **Create a resource**, then **Compute**, then select **Virtual machine** (if this resource type is not listed on the page, use the search box at the top of the page to search for it and select it).

2. On the **Create a virtual machine** page, on the **Basics** tab, use the information in the table below to create the first VM.

   | **Setting**          | **Value**                                    |
   | -------------------- | -------------------------------------------- |
   | Subscription         | Select your subscription                     |
   | Resource group       | **IntLB-RG**                                 |
   | Virtual machine name | **myTestVM**                                 |
   | Region               | **(US) West US**                             |
   | Availability options | **No infrastructure redundancy required**    |
   | Image                | **Windows Server 2019 Datacenter - Gen 1**   |
   | Size                 | **Standard_DS1_v2 - 1 vCPU, 3.5 GiB memory** |
   | Username             | **TestUser**                                 |
   | Password             | **TestPa$$w0rd!**                            |
   | Confirm password     | **TestPa$$w0rd!**                            |


3. Click **Next : Disks**, then click **Next : Networking**. 

4. On the **Networking** tab, use the information in the table below to configure networking settings.

   | **Setting**                                                  | **Value**                     |
   | ------------------------------------------------------------ | ----------------------------- |
   | Virtual network                                              | **IntLB-VNet**                |
   | Subnet                                                       | **myBackendSubnet**           |
   | Public IP                                                    | Change to **None**            |
   | NIC network security group                                   | **Advanced**                  |
   | Configure network security group                             | Select the existing **myNSG** |
   | Place this virtual machine behind an existing load balancing solution? | **Off** (unchecked)           |


5. Click **Review + create**.

6. Click **Create**.

7. Wait for this last VM to be deployed before moving forward with the next task.

### Connect to the test VM to test the load balancer

1. On the Azure portal home page, click **All resources**, then click on **myIntLoadBalancer** from the resources list.

2. On the **Overview** page, make a note of the **Private IP address**, or copy it to the clipboard.

3. Click **Home**, then on the Azure portal home page, click **All resources**, then click on the **myTestVM** virtual machine that you just created.

4. On the **Overview** page, select **Connect**, then **Bastion**.

5. Click **Use Bastion**.

6. In the **Username** box, type **TestUser** and in the **Password** box, type **TestPa$$w0rd!**, then click **Connect**.

7. The **myTestVM** window will open in another browser tab.

8. If a **Networks** pane appears, click **Yes**.

9. Click the **Internet Explorer** icon in the task bar to open the web browser.

10. Click **OK** on the **Set up Internet Explorer 11** dialog box.

11. Enter (or paste) the **Private IP address** (e.g. 10.1.0.4) from the previous step into the address bar of the browser and press Enter.

12. The default web home page of the IIS Web server is displayed in the browser window. One of the three virtual machines in the backend pool will respond.
    ![Picture 8](../media/load-balancer-web-test-1.png)

13. If you click the refresh button in the browser a few times, you will see that the response comes randomly from the different VMs in the backend pool of the internal load balancer.
    ![Picture 9](../media/load-balancer-web-test-2.png)

### Clean up the Azure exercise environment

After you complete this exercise, and if you no longer need the resources, delete the resource group, load balancer, and all their related resources. The easiest way to do this is to delete the resource group, which deletes all its resources too.

1. On the Azure portal home page, click **Resource groups**.
   ![Picture 10](../media/delete-resource-group-1.png)

2. In the list of resource groups, click on the name of the **IntLB-RG** resource group.
   ![Picture 11](../media/delete-resource-group-2.png)

3. On the **IntLB-RG** resource group page, in the menu, click **Delete resource group**.

   ![Picture 12](../media/delete-resource-group-3.png)

4. In the warning pane that opens, type the name of the resource group into the text box, and then click **Delete**. (The delete button will only become available once you have successfully typed in the full name of the resource group.)

   ![Picture 13](../media/delete-resource-group-4.png)