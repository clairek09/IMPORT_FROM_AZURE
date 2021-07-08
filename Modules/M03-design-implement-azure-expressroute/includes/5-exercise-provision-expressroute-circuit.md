# Exercise - Provision an ExpressRoute circuit

In this exercise, you will create an ExpressRoute circuit using the Azure portal and the Azure Resource Manager deployment model. 

![Diagram ExpressRoute circuit layout for exercise](../media/environment-diagram.png)

To watch a demonstration of how to create an ExpressRoute circuit, see [Azure ExpressRoute - How to create an ExpressRoute circuit | Azure | Channel 9 (msdn.com)](https://channel9.msdn.com/Blogs/Azure/Azure-ExpressRoute-How-to-create-an-ExpressRoute-circuit?term=ExpressRoute&lang-en=true&pageSize=15&skip=15).

## **Create and provision an ExpressRoute circuit**

 

1. From a browser, navigate to the [Azure portal](https://portal.azure.com/) and sign in with your Azure account.

   > [!Important] 
   >
   > Your ExpressRoute circuit is billed from the moment a service key is issued. Ensure that you perform this operation when the connectivity provider is ready to provision the circuit.

2. On the Azure portal menu, select **+ Create a resource**. Select **Networking**, and then select **ExpressRoute**, as shown in the following image. If ExpressRoute does not appear in the list, use **Search the marketplace** to search for it:

   ![Azure portal - create ExpressRoute circuit menu](../media/create-expressroute-circuit-menu.png)

3. On the **Create ExpressRoute** page, provide the **Resource Group**, **Region**, and **Name** for the circuit. Then select **Next: Configuration &gt;**.

   ![Azure portal - Create ExpressRoute basic tab](../media/expressroute-create-basic.png)

4. When you are filling in the values on this page, make sure that you specify the correct SKU tier (Local, Standard, or Premium) and data metering billing model (Unlimited or Metered).

![Azure portal - Create ExpressRoute configuration tab](../media/expressroute-create-configuration.png)

 

- Port type determines if you are connecting to a service provider or directly into Microsoft's global network at a peering location.
- Create new or import from classic determines if a new circuit is being created or if you are migrating a classic circuit to Azure Resource Manager.
- Provider is the internet service provider who you will be requesting your service from.
- Peering Location is the physical location where you are peering with Microsoft.

> [!Important]
>
> The Peering Location indicates the [physical location](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-locations) where you are peering with Microsoft. This is not linked to "Location" property, which refers to the geography where the Azure Network Resource Provider is located. While they are not related, it is a good practice to choose a Network Resource Provider geographically close to the Peering Location of the circuit.

- **SKU** determines whether an ExpressRoute local, ExpressRoute standard, or an ExpressRoute premium add-on is enabled. You can specify **Local** to get the local SKU, **Standard** to get the standard SKU or **Premium** for the premium add-on. You can change the SKU to enable the premium add-on.

> [!Important]
>
> You cannot change the SKU from Standard/Premium to Local.

- **Billing model** determines the billing type. You can specify **Metered** for a metered data plan and **Unlimited** for an unlimited data plan. You can change the billing type from **Metered** to **Unlimited**.

> [!Important]
>
> You cannot change the type from Unlimited to Metered.

- **Allow classic operation** will allow classic virtual networks to be link to the circuit.

## **Retrieve your Service key**

 

1. You can view all the circuits that you created by selecting **All services &gt; Networking &gt; ExpressRoute circuits**.

   ![Azure portal - Create ExpressRoute resource menu](../media/expressroute-circuit-menu.png)

2. All ExpressRoute circuits created in the subscription will appear here. 

   ![Azure portal - show existing Expressroute circuits](../media/expressroute-circuit-list.png)

3. The circuit page displays the properties of the circuit. The service key appears in the service key field. Your service provider will need the Service Key to complete the provisioning process. The service key is specific to your circuit. **You must send the service key to your connectivity provider for provisioning.**

   ![Azure portal - ExpressRoute Circuit properties showing service key](../media/expressroute-circuit-overview.png)

4. On this page, **Provider status** gives you the current state of provisioning on the service-provider side. **Circuit status** provides you the state on the Microsoft side. 

5. When you create a new ExpressRoute circuit, the circuit is in the following state:

   - Provider status: Not provisioned
   - Circuit status: Enabled



   - The circuit changes to the following state when the connectivity provider is currently enabling it for you:
     - Provider status: Provisioning
     - Circuit status: Enabled
   - To use the ExpressRoute circuit, it must be in the following state:
     - Provider status: Provisioned
     - Circuit status: Enabled
   - You should periodically check the provisioning status and the state of the circuit key.

6. You can view the properties of the circuit that you are interested in by selecting it. Check the **Provider status** and ensure that it has moved to **Provisioned** before you continue.

![Azure portal - ExpressRoute circuit properties showing status is now provisioned](../media/provisioned.png)

 

## Deprovisioning an ExpressRoute circuit

If the ExpressRoute circuit service provider provisioning state is **Provisioning** or **Provisioned,** you must work with your service provider to deprovision the circuit on their side. Microsoft can continue to reserve resources and bill you until the service provider completes deprovisioning the circuit and notifies us.

> [!Note]
>
> You must unlink all virtual networks from the ExpressRoute circuit before deprovisioning. If this operation fails, check whether any virtual networks are linked to the circuit.
>
> If the service provider has deprovisioned the circuit (the service provider provisioning state is set to Not provisioned), you can delete the circuit. This stops billing for the circuit.

## Clean up resources

You can delete your ExpressRoute circuit by selecting the **Delete** icon. Ensure the provider status is Not provisioned before proceeding.

![Azure portal - delete an ExpressRoute circuit](../media/expressroute-circuit-delete.png)

1. Watch this demonstration of how to create and provision an ExpressRoute circuit: [Azure ExpressRoute - How to create an ExpressRoute circuit | Azure | Channel 9 (msdn.com)](https://channel9.msdn.com/Blogs/Azure/Azure-ExpressRoute-How-to-create-an-ExpressRoute-circuit?term=ExpressRoute&lang-en=true&pageSize=15&skip=15). 

Congratulations! You have created an ExpressRoute circuit and located the Service key, which you would need to complete the provisioning of the circuit.

 

## quiz title: Check your knowledge

## Multiple choice

Which of the following steps is part of provisioning an ExpressRoute circuit?

(x) Send the ExpressRoute circuit service key to the service provider.{{Correct. To complete the provisioning of the ExpressRoute circuit, the service provider must enable the circuit. This requires the service key.}} 

( )Get the ExpressRoute circuit key from the service provider.{{Incorrect. The service provider does not provide the ExpressRoute circuit service key.}}

( ) Add the ExpressRoute circuit service key to the ExpressRoute Gateway key property.{{Incorrect. The circuit service key does not have to be configured in the ExpressRoute Gateway key property.}}

## Multiple choice

You need access to regions in or near the same metropolitan area. Which SKU should you select?

(x)Local SKU.{{Correct. Local SKU gives you access only to one or two Azure regions in or near the same metropolitan area.}}

( )Premium.{{Incorrect. Premium SKU will give you more functionality than you require in this case.}}

( )Standard.{{Incorrect. Standard SKU will give you more functionality than you require in this case.}}

## Multiple choice

When does billing begin for an ExpressRoute circuit?

(x)Billing begins the moment a service key is used.{{Correct. Billing begins the moment a service key is used.}}

( )When data starts flowing between circuits.{{Incorrect. Data flowing between circuits does not trigger the start of billing.}}

( )When the SKU is selected.{{Incorrect. The SKU being selected does not trigger the start of billing.}}

