
 

Today’s workforce is more distributed than ever before. Organizations are exploring options that enable their employees, partners, and customers to connect to the resources they need from wherever they are. It’s not unusual for organizations to operate across regional and national boundaries, and across time zones. 

## What is Azure Virtual WAN?

An organization might have a large user center at headquarters, multiple branch offices, and multiple remote users. All these sites need to connect to resources throughout the organization. Historically, organizations used a combination of VPNs to provide site-to-site connections for branch offices, point-to-site connections for individual remote users, and connections to Cloud services. Traditional Software Defined Wide Area Network (SD WAN) technologies combine and manage the connections, requiring connectivity appliances such as gateways at each site, and connecting them through internet tunnels. 

Azure Virtual WAN combines all these methods of connectivity to enable the organization to leverage the Microsoft backbone network, which connects Microsoft data centers across Azure regions and a large mesh of edge-nodes around the world. By connecting to the backbone network, organizations can leverage its reliability, capacity, and flexibility to connect their regional Azure VNets, network edge locations like ExpressRoute, and carrier neutral connections.

A Virtual WAN is a security delineation; each instance of a Virtual WAN is self-contained in terms of connectivity and hence also provides security isolation. Organizations will generally only require one instance of a Virtual WAN. Each Virtual WAN is implemented as a hub-and-spoke topology, and can have one or more hubs, which support connectivity between different types of endpoints including connectivity vendors like AT&T, Verizon, and T-Mobile. All hubs are connected in a full mesh topology in a Standard Virtual WAN making it easy for the user to use the Microsoft backbone for any-to-any (any spoke) transitive connectivity.

A secured virtual hub is an Azure Virtual WAN Hub with associated security and routing policies configured by Azure Firewall Manager. Use secured virtual hubs to easily create hub-and-spoke and transitive architectures with native security services for traffic governance and protection. You can use a secured virtual hub to filter traffic between virtual networks (V2V), virtual networks and branch offices (B2V) and traffic to the Internet (B2I/V2I). A secured virtual hub provides automated routing. There's no need to configure your own UDRs (user defined routes) to route traffic through your firewall.

The following diagram shows an organization with a single Virtual WAN hub connecting the spokes. VNets, Site-to-site and point-to-site VPNs, SD WANs, and ExpressRoute connectivity are all supported.

> [!div class="mx-imgBorder"]
>![Azure Virtual WAN with two regional hubs, several VNets, and multiple remote connections including Site-to-Site VPN, Point-to-Site VPN, and ExpressRoute.](../media/azure-wan-regions.png)

## Gateway scale

Gateway scale units allow you pick the aggregate throughput of the VPN gateway being created in the virtual hub to connect sites to. If you pick 1 scale unit = 500 Mbps, it implies that two instances for redundancy will be created, each having a maximum throughput of 500 Mbps. For example, if you had five branches, each doing 10 Mbps at the branch, you will need an aggregate of 50 Mbps at the head end. Planning for aggregate capacity of the Azure VPN gateway should be done after assessing the capacity needed to support the number of branches to the hub.

## Choose a Virtual WAN SKU

There are two types of Virtual WANs: Basic and Standard. The following table shows the available configurations for each type.


| Virtual WAN type | Hub type | Available configurations                                                                                              |
|:-:|:-:|:-:|
| Basic            | Basic    | Site-to-site VPN only                                                                                                 |
| Standard         | Standard | ExpressRoute<br>User VPN (P2S)<br>VPN (site-to-site)<br>Inter-hub and VNet-to-VNet transiting through the virtual hub |


## Hub private address space

The minimum address space is /24 to create a hub. If you use anything in the range from /25 to /32, it will produce an error during creation. You don't need to explicitly plan the subnet address space for the services in the virtual hub. Because Azure Virtual WAN is a managed service, it creates the appropriate subnets in the virtual hub for the different gateways/services (for example, VPN gateways, ExpressRoute gateways, User VPN point-to-site gateways, Firewall, routing, etc.).

## Connect cross-tenant VNets to a Virtual WAN hub

You can use Virtual WAN to connect a VNet to a virtual hub in a different tenant. This architecture is useful if you have client workloads that must be connected to be the same network but are on different tenants. For example, as shown in the following diagram, you can connect a non-Contoso VNet (the Remote Tenant) to a Contoso virtual hub (the Parent Tenant).

> [!div class="mx-imgBorder"]
>![Connect a non-Contoso VNet to a Contoso virtual hub (the Parent Tenant).](../media/cross-tenant-connectivity.png)

Before you can connect a cross-tenant VNet to a Virtual WAN hub, you must have the following configuration already set up:

- A Virtual WAN and virtual hub in the parent subscription.

- A virtual network configured in a subscription in the remote tenant.

- Non-overlapping address spaces in the remote tenant and address spaces within any other VNets already connected to the parent virtual hub.

### Assign permissions

For the parent subscription with the virtual hub to modify and access the virtual networks in the remote tenant, you need to assign Contributor permissions to your parent subscription from the remote tenant subscription. 

- Add the Contributor role assignment to the parent account (the one with the virtual WAN hub). You can use PowerShell or the Azure portal to assign this role.

- Add the remote tenant subscription and the parent tenant subscription to the current session of PowerShell by running the Add-AzAccount command. If you are signed into the parent, you only need to run the command for the remote tenant.

- Verify that the role assignment is successful by logging into Azure PowerShell using the parent credentials and running the Get-AzSubscription command. If the permissions have successfully propagated to the parent and have been added to the session, the output of the command will list the subscription owned by the remote tenant.

## Virtual Hub routing

The routing capabilities in a virtual hub are provided by a router that manages all routing between gateways using Border Gateway Protocol (BGP). A virtual hub can contain multiple gateways such as a Site-to-site VPN gateway, ExpressRoute gateway, Point-to-site gateway, Azure Firewall. This router also provides transit connectivity between virtual networks that connect to a virtual hub and can support up to an aggregate throughput of 50 Gbps. These routing capabilities apply to Standard Virtual WAN customers.

To learn more about how to configure routing, see [How to configure virtual hub routing](https://docs.microsoft.com/en-us/azure/virtual-wan/how-to-virtual-hub-routing).

### Hub route table

A virtual hub route table can contain one or more routes. A route includes its name, a label, a destination type, a list of destination prefixes, and next hop information for a packet to be routed. A Connection typically will have a routing configuration that associated or propagates to a route table.

### Connections

Connections are Resource Manager resources that have a routing configuration. The four types of connections are:

- VPN connection: Connects a VPN site to a virtual hub VPN gateway.

- ExpressRoute connection: Connects an ExpressRoute circuit to a virtual hub ExpressRoute gateway.

- P2S configuration connection: Connects a User VPN (Point-to-site) configuration to a virtual hub User VPN (Point-to-site) gateway.

- Hub virtual network connection: Connects virtual networks to a virtual hub.

You can set up the routing configuration for a virtual network connection during setup. By default, all connections associate and propagate to the Default route table.

### Association

Each connection is associated to one route table. Associating a connection to a route table allows the traffic to be sent to the destinations indicated as routes in the route table. The routing configuration of the connection will show the associated route table. Multiple connections can be associated to the same route table. All VPN, ExpressRoute, and User VPN connections are associated to the same (default) route table.

By default, all connections are associated to a Default route table in a virtual hub. Each virtual hub has its own Default route table, which can be edited to add a static route(s). Routes added statically take precedence over dynamically learned routes for the same prefixes.

> [!div class="mx-imgBorder"]
>![Association - default routing table.](../media/concepts-association.png)

 

### Propagation

Connections dynamically propagate routes to a route table. With a VPN connection, ExpressRoute connection, or P2S configuration connection, routes are propagated from the virtual hub to the on-premises router using BGP. Routes can be propagated to one or multiple route tables.

A None route table is also available for each virtual hub. Propagating to the None route table implies that no routes are required to be propagated from the connection. VPN, ExpressRoute, and User VPN connections propagate routes to the same set of route tables.

> [!div class="mx-imgBorder"]
>![Propagation - default routing table.](../media/concepts-propagation.png)

### Labels

Labels provide a mechanism to logically group route tables. This is especially helpful during propagation of routes from connections to multiple route tables. For example, the Default Route Table has a built-in label called 'Default'. When users propagate connection routes to 'Default' label, it automatically applies to all the Default Route Tables across every hub in the Virtual WAN.

### Configuring static routes in a virtual network connection

Configuring static routes provides a mechanism to steer traffic through a next hop IP, which could be of a Network Virtual Appliance (NVA) provisioned in a Spoke VNet attached to a virtual hub. The static route is composed of a route name, list of destination prefixes, and a next hop IP.

### Route tables for pre-existing routes

Route tables now have features for association and propagation. A pre-existing route table is a route table that does not have these features. If you have pre-existing routes in hub routing and would like to use the new capabilities, consider the following:

- **Standard Virtual WAN Customers with pre-existing routes in virtual hub:** If you have pre-existing routes in Routing section for the hub in Azure portal, you will need to first delete them and then attempt creating new route tables (available in the Route Tables section for the hub in Azure portal).

- **Basic Virtual WAN Customers with pre-existing routes in virtual hub:** If you have pre-existing routes in Routing section for the hub in Azure portal, you will need to first delete them, then upgrade your Basic Virtual WAN to Standard Virtual WAN. 

## Troubleshooting failed resources by using a hub reset

Virtual Hub Reset is available only in the Azure portal. Resetting provides you a way to bring any failed resources such as route tables, hub router, or the virtual hub resource itself back to its rightful provisioning state. Consider resetting the hub prior to contacting Microsoft for support. This operation does not reset any of the gateways in a virtual hub.



## quiz title: Check your knowledge



### Multiple Choice



What is an Azure Virtual WAN?

(x) Azure Virtual WAN is a collection of connectivity resources like VPNs, which enables organizations to use the Microsoft backbone. {{Correct, Azure Virtual WAN is a centrally managed collection of connectivity resources like VPNs, which enables organizations to use the Microsoft backbone in a self-contained, security isolated manner.}}

( ) Azure WAN describes two or more VNets connected through peering. {{Incorrect, Azure WAN connects VNets, on-premises networks and individual computers to the Microsoft backbone.}}

( ) Azure WAN is a collection of on-premises networks connected to each other through VPNs. {{Incorrect, Azure WAN connects VNets, on-premises networks and individual computers to the Microsoft backbone.}} 

### Multiple Choice 



What is the purpose of associating a connection to a route table?

(x) Associating a connection to a route table allows the traffic to be sent to the destinations indicated as routes in the route table. {{Correct, Associating a connection to a route table allows the traffic to be sent to the destination indicated as routes in the route table. Each connection is associated to one route table.}}

( ) Associating a connection to a route table allows the connection to dynamically propagate routes to other route tables. {{Incorrect, Connections dynamically propagate routes to a specified route table.}}

( ) Associating a connection to a route table allows users to be authenticated. {{Incorrect, Users are not authenticated using route tables.}}

