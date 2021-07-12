> [!NOTE] 
> To complete this exercise, you will need a Microsoft Azure subscription. If you don't already have one, you can sign up for a free trial at https://azure.com/free.

In this exercise, you will create an internal load balancer for the fictional Contoso Ltd organization. Then you will create a Log Analytics workspace, and use Azure Monitor Insights to view information about your internal load balancer. You will view the Functional Dependency View, then view detailed metrics for the load balancer resource, and view resource health information for the load balancer. Finally, you will configure the load balancer's diagnostic settings to send metrics to the Log Analytics workspace you created. 

The diagram below illustrates the environment you will be deploying in this exercise.

![Diagram illustrating the load balancer architecture that will be created in the exercise - includes load balancer, VNet, subnet, Bastionsubnet, and VMs](../media/exercise-internal-standard-load-balancer-environment-diagram.png)

 In this exercise, you will:

+ Task 1: Create the virtual network
+ Task 2: Create the load balancer
+ Task 3: Create a backend pool
+ Task 4: Create a health probe
+ Task 5: Create a load balancer rule
+ Task 6: Create backend servers
+ Task 7: Add VMs to the backend pool
+ Task 8: Install IIS on the VMs
+ Task 9: Test the load balancer
+ Task 10: Create a Log Analytics Workspace
+ Task 11: Use Functional Dependency View
+ Task 12: View detailed metrics
+ Task 13: View resource health
+ Task 14: Configure diagnostic settings
+ Task 15: Clean up resources



## Task 1: Create the virtual network

In this section, you will create a virtual network and a subnet.

1. Log in to the Azure portal.

2. On the Azure portal home page, click **Create a resource**, then **Networking**, then select **Virtual Network** (if this resource type is not listed on the page, use the search box at the top of the page to search for it and select it).

3. Click **Create**.

   ![Create virtual network](../media/create-virtual-network-1.png)

4. On the **Basics** tab, use the information in the table below to create the virtual network.

   | **Setting**    | **Value**                                           |
   | -------------- | --------------------------------------------------- |
   | Subscription   | Select your subscription                            |
   | Resource group | Select **Create new**<br /><br />Name: **IntLB-RG** |
   | Name           | **IntLB-VNet**                                      |
   | Region         | **(US) West US**                                    |

5. Click **Next : IP Addresses**.

6. On the **IP Addresses** tab, in the **IPv4 address space** box, type **10.1.0.0/16**.

7. Under **Subnet name**, select the word **default**.

8. In the **Edit subnet** pane, provide a subnet name of **myBackendSubnet**, and a subnet address range of **10.1.0.0/24**.

9. Click **Save**.

10. Click **Next : Security**.

11. Under **BastionHost** select **Enable**, then enter the information from the table below.

    | **Setting**                       | **Value**                                              |
    | --------------------------------- | ------------------------------------------------------ |
    | Bastion name                      | **myBastionHost**                                      |
    | AzureBastionSubnet address space  | **10.1.1.0/24**                                        |
    | Public IP address                 | Select **Create new**<br /><br />Name: **myBastionIP** |

12. Click **Review + create**.

13. Click **Create**.

## Task 2: Create the load balancer

In this section, you will create an internal Standard SKU load balancer. The reason we are creating a Standard SKU load balancer here in the exercise, instead of a Basic SKU load balance, is for later exercises that require a Standard SKU version of the load balancer.

1. On the Azure portal home page, click **Create a resource**.

2. In the search box at the top of the page, type **Load Balancer**, then press **Enter** (**Note:** do not select one from the list).

3. Scroll down to the bottom of the page and select **Load Balancer** (the one that says 'Microsoft' and 'Azure Service' under the name).

4. Click **Create**.

   ![Create Load Balancer](../media/create-load-balancer-4.png)

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


## Task 3: Create a backend pool

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

   ![Show backend pool created in load balancer](../media/create-backendpool.png)

   

## Task 4: Create a health probe

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

   ![Show health probe created in load balancer](../media/create-healthprobe.png)



## Task 5: Create a load balancer rule

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

   ![Show load balancing rule created in load balancer](../media/create-loadbalancerrule.png)

## Task 6: Create backend servers

In this section, you will create three VMs, that will be in the same availability set, for the backend pool of the load balancer, add the VMs to the backend pool, and then install IIS on the three VMs to test the load balancer.

### Create VMs

In this section, you will create three VMs (myVM1, myVM2, myVM3). When creating the first VM you will also create a new Availability Set, and then as you create the next two VMs, they will be added to the Availability Set. You will then add these VMs to the backend pool of the load balancer that you created previously.

1. On the Azure portal home page, click **Create a resource**, then **Compute**, then select **Virtual machine** (if this resource type is not listed on the page, use the search box at the top of the page to search for it and select it).

2. On the **Create a virtual machine** page, on the **Basics** tab, use the information in the table below to create the first VM.

   | **Setting**          | **Value**                                                    |
   | -------------------- | ------------------------------------------------------------ |
   | Subscription         | Select your subscription                                     |
   | Resource group       | **IntLB-RG**                                                 |
   | Virtual machine name | **myVM1**                                                    |
   | Region               | **(US) West US**                                             |
   | Availability options | **Availability Set**                                         |
   | Availability set     | Select **Create new**<br /><br />Name: **myAvailabilitySet** |
   | Image                | **Windows Server 2019 Datacenter - Gen 1**                   |
   | Size                 | **Standard_DS1_v2 - 1 vcpu, 3.5 GiB memory**                 |
   | Username             | **TestUser**                                                 |
   | Password             | **TestPa$$w0rd!**                                            |
   | Confirm password     | **TestPa$$w0rd!**                                            |

3. Click **Next : Disks**, then click **Next : Networking**. 

4. On the **Networking** tab, use the information in the table below to configure networking settings.

   | **Setting**                                                  | **Value**                                        |
   | ------------------------------------------------------------ | ------------------------------------------------ |
   | Virtual network                                              | **IntLB-VNet**                                   |
   | Subnet                                                       | **myBackendSubnet**                              |
   | Public IP                                                    | Change to **None**                               |
   | NIC network security group                                   | **Advanced**                                     |
   | Configure network security group                             | Select **Create new**<br /><br />Name: **myNSG** |
   | Place this virtual machine behind an existing load balancing solution? | **Off** (unchecked)                              |

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


## Task 7: Add VMs to the backend pool

1. On the Azure portal home page, click **All resources**, then click on **myIntLoadBalancer** from the resources list.

2. Under **Settings**, select **Backend pools**., and then select **myBackendPool**.

3. In the **Associated to** box, select **Virtual machines**.

4. Under **Virtual machines**, click **Add**.

5. Select the checkboxes for all 3 VMs (**myVM1**, **myVM2**, and **myVM3**), then click **Add**.

6. On the **myBackendPool** page, click **Save**.

   ![Show VMs added to backend pool in load balancer](../media/add-vms-backendpool.png)

 

## Task 8: Install IIS on the VMs

1. On the Azure portal home page, click **All resources**, then click on **myVM1** from the resources list.
2. On the **Overview** page, select **Connect**, then **Bastion**.
3. Click **Use Bastion**.
4. In the **Username** box, type **TestUser** and in the **Password** box, type **TestPa$$w0rd!**, then click **Connect**.
5. The **myVM1** window will open in another browser tab.
6. If a **Networks** pane appears, click **Yes**.
7. Click the **Windows Start icon** in the bottom left corner of the window, then click the **Windows PowerShell** tile.
8. To install IIS, run the following command in PowerShell: Install-WindowsFeature -name Web-Server -IncludeManagementTools
9. To remove the existing default web home page, run the following command in PowerShell: Remove-Item C:\inetpub\wwwroot\iisstart.htm
10. To add a new default web home page and add content to it, run the following command in PowerShell: Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)
11. Close the Bastion session to **myVM1** by closing the browser tab.
12. Repeat steps 1-11 above twice more to install IIS and the updated default home page on the **myVM2** and **myVM3** virtual machines.

 

## Task 9: Test the load balancer

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
   | Size                 | **Standard_DS1_v2 - 1 vcpu, 3.5 GiB memory** |
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
    ![Browser window showing Hello World response from VM1](../media/load-balancer-web-test-1.png)

13. If you click the refresh button in the browser a few times, you will see that the response comes randomly from the different VMs in the backend pool of the internal load balancer.

    ![Browser window showing Hello World response from VM3](../media/load-balancer-web-test-2.png)

## Task 10: Create a Log Analytics Workspace

1. On the Azure portal home page, click **All services**, then in the search box at the top of the page type **Log Analytics**, and select **Log Analytics workspaces** from the filtered list.

   ![Accessing Log Analytics workspaces from the Azure portal home page](../media/log-analytics-workspace-1.png)

2. Click **Create**. 

3. On the **Create Log Analytics workspace** page, on the **Basics** tab, use the information in the table below to create the workspace.

   | **Setting**    | **Value**                |
   | -------------- | ------------------------ |
   | Subscription   | Select your subscription |
   | Resource group | **IntLB-RG**             |
   | Name           | **myLAworkspace**        |
   | Region         | **West US**              |

4. Click **Review + Create**, then click **Create**.

   ![Log Analytics workspaces list](../media/log-analytics-workspace-2.png)



## Task 11: Use Functional Dependency View

1. On the Azure portal home page, click **All resources**, then in the resources list, select **myIntLoadBalancer**.

   ![All resources list in the Azure portal](../media/network-insights-functional-dependency-view-1.png)

2. Under **Monitoring**, select **Insights**.

3. In the top right corner of the page, click the **X** to close the **Metrics** pane for now. You will open it again shortly.

4. This page view is known as Functional Dependency View, and in this view, you get a useful interactive diagram, which illustrates the topology of the selected network resource - in this case a load balancer. For Standard Load Balancers, your backend pool resources are color-coded with Health Probe status indicating the current availability of your backend pool to serve traffic.

5. Use the **Zoom In (+)** and **Zoom Out (-)** buttons in the bottom right corner of the page, to zoom in and out of the topology diagram (alternatively you can use your mouse wheel if you have one). You can also drag the topology diagram around the page to move it.

6. Hover over the **LoadBalancerFrontEnd** component in the diagram, then hover over the **myBackendPool** component. 

7. Notice that you can use the links in these pop-up windows to view information about these load balancer components and open their respective Azure portal blades.

8. Hover over the **myVM3** virtual machine component. Note that you can open the resource blade for the virtual machine, and you can open the **VM Insights** page, or you can run the **Connection troubleshoot** tool from Network Watcher - all from this part of the topology diagram.
   ![Azure Monitor Network Insights functional dependency view](../media/network-insights-functional-dependency-view-2.png)

9. To download a .SVG file copy of the topology diagram, click **Download topology**, and save the file in your **Downloads** folder. 

10. In the top right corner, click **View metrics** to reopen the metrics pane on the right-hand side of the screen.
    ![Azure Monitor Network Insights functional dependency view - View metrics button highlighted](../media/network-insights-functional-dependency-view-3.png)

11. The Metrics pane provides a quick view of some key metrics for this load balancer resource, in the form of bar and line charts.

    ![Azure Monitor Network Insights - Basic metrics view](../media/network-insights-basicmetrics-view.png)

 

## Task 12: View detailed metrics

1. To view more comprehensive metrics for this network resource, click **View detailed metrics**.
   ![Azure Monitor Network Insights - View detailed metrics button highlighted](../media/network-insights-detailedmetrics-1.png)

2. This opens a large full **Metrics** page in the Azure Network Insights platform. The first tab you land on is the **Overview** tab, which shows the availability status of the load balancer and overall Data Throughput and Frontend and Backend Availability for each of the Frontend IPs attached to your Load Balancer. These metrics indicate whether the Frontend IP is responsive and the compute instances in your Backend Pool are individually responsive to inbound connections.
   ![Azure Monitor Network Insights - Detailed metrics view - Overview tab](../media/network-insights-detailedmetrics-2.png)

3. Click the **Frontend &amp; Backend Availability** tab and scroll down the page to see the Health Probe Status charts. If you see **values that are lower than 100** for these items, it indicates an outage of some kind on those resources.
   ![Azure Monitor Network Insights - Detailed metrics view - Health probe status charts highlighted](../media/network-insights-detailedmetrics-5.png)

4. Click the **Data Throughput** tab and scroll down the page to see the other data throughput charts.

5. Hover over some of the data points in the charts, and you will see that the values change to show the exact value at that point in time.
   ![Azure Monitor Network Insights - Detailed metrics view - Data Throughput tab](../media/network-insights-detailedmetrics-3.png)

6. Click the **Flow Distribution** tab and scroll down the page to see the charts under the **VM Flow Creation and Network Traffic** section. 

   ![Azure Monitor Network Insights - Detailed metrics view - VM Flow Creation and Network Traffic charts](../media/network-insights-detailedmetrics-4.png)

 

## Task 13: View resource health

1. To view the health of your Load Balancer resources, on the Azure portal home page, click **All services**, then select **Monitor**.

2. On the **Monitor&gt;Overview** page, in the left-hand menu click **Service Health**.

3. On the **Service Health&gt;Service issues** page, in the left-hand menu click **Resource Health**.

4. On the **Service Health&gt;Resource health** page, in the **Resource type** drop-down list, scroll down the list and select **Load balancer**.

   ![Access Service Health>Resource Health for load balancer resource](../media/resource-health-1.png)

5. Then select the name of your load balancer from the list.

6. The **Resource health** page will identify any major availability issues with your load balancer resource. If there are any events under the **Health History** section, you can expand the health event to see more detail about the event. You can even save the detail about the event as a PDF file for later review and for reporting.

   ![Service Health>Resource health view](../media/resource-health-2.png)

 

## Task 14: Configure diagnostic settings

1. On the Azure portal home page, click **Resource groups**, then select the **IntLB-RG** resource group from the list.

2. On the **IntLB-RG** page, click the name of the **myIntLoadBalancer** load balancer resource in the list of resources.

3. Under **Monitoring**, select **Diagnostic settings**, then click **Add diagnostic setting**.

   ![Diagnostic settings>Add diagnostic setting button highlighted](../media/diagnostic-settings-1.png)

4. On the **Diagnostic setting** page, in the name box, type **myLBDiagnostics**.

5. Select the **AllMetrics** checkbox, then select the **Send to Log Analytics workspace** checkbox.

6. Select your subscription from the list, then select **myLAworkspace (westus)** from the workspace drop-down list.

7. Click **Save**.

   ![Diagnostic setting page for load balancer](../media/diagnostic-settings-2.png)

 

 

## Task 15: Clean up resources

   >**Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

1. In the Azure portal, open the **PowerShell** session within the **Cloud Shell** pane.

1. Delete all resource groups you created throughout the labs of this module by running the following command:

   ```powershell
   Remove-AzResourceGroup -Name 'NAME OF THE RG' -Force -AsJob
   ```

    >**Note**: The command executes asynchronously (as determined by the -AsJob parameter), so while you will be able to run another PowerShell command immediately afterwards within the same PowerShell session, it will take a few minutes before the resource groups are actually removed.
