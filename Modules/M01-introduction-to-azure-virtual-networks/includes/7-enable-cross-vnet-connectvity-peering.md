# Enable cross-VNet connectivity with peering

Organizations with large scale operations will often need to create connections between different parts of their virtual network infrastructure. Virtual network peering enables you to seamlessly connect separate VNets with optimal network performance, whether they are in the same Azure region (VNet peering) or in different regions (Global VNet peering). Network traffic between peered virtual networks is private. The virtual networks appear as one for connectivity purposes. The traffic between virtual machines in peered virtual networks uses the Microsoft backbone infrastructure, and no public Internet, gateways, or encryption is required in the communication between the virtual networks.

Virtual network peering enables you to seamlessly connect two Azure virtual networks. Once peered, the virtual networks appear as one, for connectivity purposes. There are two types of VNet peering.

- **Regional VNet peering** connects Azure virtual networks in the same region.

- **Global VNet peering** connects Azure virtual networks in different regions. When creating a global peering, the peered virtual networks can exist in any Azure public cloud region or China cloud regions, but not in Government cloud regions. You can only peer virtual networks in the same region in Azure Government cloud regions.

![Illustration showing VNet1 in Region 1, and VNet2 and VNet3 in Region 2. VNet2 and VNet3 are connected with regional VNet peering. VNet1 and VNet2 are connected with a global VNet peering](../media/global-vnet-peering.png)

 

The benefits of using virtual network peering, whether local or global, include:

- A low-latency, high-bandwidth connection between resources in different virtual networks.

- The ability to apply network security groups in either virtual network to block access to other virtual networks or subnets. 

- The ability to transfer data between virtual networks across Azure subscriptions, Azure Active Directory tenants, deployment models, and Azure regions.

- The ability to peer virtual networks created through the Azure Resource Manager.

- The ability to peer a virtual network created through Resource Manager to one created through the classic deployment model. 

- No downtime to resources in either virtual network is required when creating the peering, or after the peering is created.

The following diagram shows a scenario where resources on the Contoso VNet and resources on the Fabrikam VNet need to communicate. The Contoso subscription in the US West region, is connected to the Fabrikam subscription in the US West region.

![Scenario requiring cross-VNet connectivity from Contoso subscription (../media/az-700-azure-networking-solutions-module-1-34.png) in US West region to Fabrikam (10.10.26.0/24) subscription in US East region.](../media/vnet-peering.png)

The routing tables show the routes known to the resources in each subscription. The following routing table shows the routes known to Contoso, with the final entry being the Global VNet peering entry to the Fabrikam 10.10.26.0/24 subnet.

![Contoso routing table showing Global VNet peering entry 10.10.26.0/24.](../media/contosovm-routes-peering-annotated.png)

The following routing table shows the routes known to Fabrikam. Again, the final entry is the Global VNet peering entry, this time to the Contoso 10.1.26.0/25 subnet.

![Fabrikam routing table showing Global VNet peering entry 10.17.26.0/24.](../media/fabrikamvm-routes-peering-annotated.png)

## Gateway Transit and Connectivity

When virtual networks are peered, you configure a VPN gateway in the peered virtual network as a transit point. In this case, a peered virtual network uses the remote gateway to gain access to other resources. A virtual network can have only one gateway. Gateway transit is supported for both VNet Peering and Global VNet Peering.

When you Allow Gateway Transit the virtual network can communicate to resources outside the peering. For example, the subnet gateway could:

- Use a site-to-site VPN to connect to an on-premises network.

- Use a VNet-to-VNet connection to another virtual network.

- Use a point-to-site VPN to connect to a client.

In these scenarios, gateway transit allows peered virtual networks to share the gateway and get access to resources. This means you do not need to deploy a VPN gateway in the peer virtual network.

> [!NOTE]
>
> Network security groups can be applied in either virtual network to block access to other virtual networks or subnets. When configuring virtual network peering, you can either open or close the network security group rules between the virtual networks.

 

## Use service chaining to direct traffic to a gateway

Suppose you want to direct traffic from the Contoso VNet to a specific network virtual appliance (NVA). Create user-defined routes to direct traffic from the Contoso VNet to the NVA in the Fabrikam VNet. This technique is known as service chaining. 

To enable service chaining, add user-defined routes pointing to virtual machines in the peered virtual network as the next hop IP address. User-defined routes can also point to virtual network gateways.

Azure virtual networks can be deployed in a hub-and-spoke topology, with the hub VNet acting as a central point of connectivity to all the spoke VNets. The hub virtual network hosts infrastructure components such as an NVA, virtual machines and a VPN gateway. All the spoke virtual networks peer with the hub virtual network. Traffic flows through network virtual appliances or VPN gateways in the hub virtual network. The benefits of using a hub and spoke configuration include cost savings, overcoming subscription limits, and workload isolation.

The following diagram shows a scenario in which hub VNet hosts a VPN gateway that manages traffic to the on-premises network, enabling controlled communication between the on-premises network and the peered Azure VNets.

![Hub-and-spoke configuration - Contoso and Fabrikam peer to Hub VNet.
Hub VNet contains NVA, VMs, and a VPN Gateway connected to on-premises network.](../media/service-chaining.png)

## Check your knowledge

Choose the best response for each of the questions below. When you're done, select **Check your answers**.

## quiz title:

## Multiple Choice

You want the resources in one VNet to communicate with resources in a subnet in a different VNet. Which Azure network feature can you use?
( ) VNet peering. {{that is correct, Virtual network peering enables you to seamlessly connect separate VNets with optimal network performance, whether they are in the same Azure region (VNet peering) or in different regions (Global VNet peering).}}
( ) Internal DNS. {{that is incorrect, Internal DNS is a service provided by Azure.}}
( ) Azure Availability Zones. {{that is incorrect, Azure Availability Zones are a high availbility feature. Each zone is made up of one or more datacenters equipped with independent power, cooling, and networking.}} 
( ) Resources on Azure VNets can communicate with each other by default, you don't need to perform any further configuration.{{that is incorrect, Resources on separate VNets cannot communicate by default.}} 

## Multiple Choice 

When you configure global peering, what changes will you see in the peered VNets?
( ) A peering entry is added to the routing table in each VNet. {{that is correct, VNetGlobalPeering entries are added to the routing tables in each VNet to direct traffic to the peered VNet.}}
( ) A peering entry is added to the routing table in the source VNet only. {{that is incorrect, The VNets communicate as peers, so resources in each VNet must be able to communicate with each other. Adding a single peering entry only allows traffic to be routed one way.}}
( ) All traffic on the Vnet must be routed through a Gateway. {{that is incorrect, Resources on each VNet can communicate seamlessly with one another.}}
( ) No changes are made in the peered VNets. {{that is incorrect, VNetGlobalPeering entries are added to the routing tables in each VNet to direct traffic to the peered VNet.}} 