---
layout: post
title: "Solving an Enterprise Integration problem with Azure Serverless"
date: 2020-10-03
image:  /images/posts/enterprise-serverless-banner.png
excerpt: "There are proven benefits of abstracting out servers for certain solutions where the focus lies more on the business logic and lesser on the logistics of it. Serverless offerings are modelled for simplicity. This can quickly lead to limitations especially while building enterprise focused solutions surrounded by niche requirements or compliances. Azure seems to have clearly inherited Microsoft's proficiency in catering to Enterprise needs. Azure's serverless offerings have managed to extend an array of features that help build compliant solutions with fair degree of ease. To evaluate just that, here we look at an Enterprise Integration problem and design a solution using Azure's serverless offerings and highlight some of their capabilities."
tags: [serverless, azure, enterprise, integration]
comments: true
---

There are proven benefits of abstracting out servers for certain solutions where the focus lies more on the business logic and lesser on the logistics of it.<sup>[[1]](#reference-1)[[2]](#reference-2)[[3]](#reference-3)</sup> Serverless offerings are modelled for simplicity, but this can quickly lead to limitations especially while building enterprise focused solutions surrounded by niche requirements or compliances. Azure seems to have clearly inherited Microsoft's proficiency in catering to Enterprise needs. Azure's serverless offerings have managed to extend an array of features that help build compliant solutions with fair degree of ease. To evaluate just that, here we look at an Enterprise Integration problem and design a solution using Azure's serverless offerings and highlight some of their capabilities.

> # Problem statement
> North Wind Corp. runs an ERP solution where Request for Proposals (RFP) are uploaded for Vendors to access. North Wind wants to send out notifications to suppliers when they intend to raise RFPs. Vendors like Proseware Inc. would like to stay up to date on RFP events and trigger their internal business processes.

For the sake of simplicity, let's break this problem statement down to three phases and address each of them with one of Azure's serverless offerings.

# <a name="separation-of-concerns" href="#separation-of-concerns">Separation of Concerns</a>
This is a cross-enterprise integration scenario. It is therefore important to understand that the solution needs to have clear separation of concerns between the two parties, North Wind and Proseware.

- North Wind's sole responsibility should be to issue the RFP notifications to all vendors along with relevant metadata.
- Proseware and other vendors should be able to manage and route their own subscriptions.
 
## Solution
North Wind is required to set up a notification infrastructure that its vendors can subscribe to. It is crucial for this infrastructure to support access for vendors to manage their own subscriptions.

RFP Notification being an event, Azure’s Event Grid starts to seem like a good candidate. Event Grid is designed to be used in event-based architectures where applications can emit events. Other applications and services can then subscribe to events of their interest. The general Event Hub offering supports deep integration with Azure allowing your Azure resources to readily start emitting event on the grid. The support for filters is also a huge plus especially for applications that would like to focus on a very specific range of events. If you are interested in a detailed overview, I would encourage you to read up if their [Azure documentation](https://docs.microsoft.com/en-us/azure/event-grid/overview).

*If you want to improve your understanding of the key differences between Event Hubs, Event Grid and in some cases Service Bus head here [https://docs.microsoft.com/en-us/azure/event-grid/compare-messaging-services](https://docs.microsoft.com/en-us/azure/event-grid/compare-messaging-services).<sup>[[4]](#reference-4)</sup>*

We can approach a design where we create a topic dedicated for RFP notifications, but RBAC on the topic would not control vendors from accessing each other’s topic subscriptions. We can create a custom topic for each vendor allowing them to control all subscriptions on that topic. As each topic will have its own event publishing endpoint, North Wind’s internal ERP system will have to emit notifications on each topic endpoint. There should be a better approach.

All of the above constrains can be gracefully bypassed with **Event Domains**. Extending the same Event Grid infrastructure, domains allow applications to [emit events across multiple topics over the same endpoint](https://docs.microsoft.com/en-us/azure/event-grid/event-domains#publishing-to-an-event-domain) and at the same time [allow RBAC to be applied on each topic created under the domain](https://docs.microsoft.com/en-us/azure/event-grid/event-domains#access-management)<sup>[[5]](#reference-5)</sup>. We can therefore have a dedicated topic for each vendor under the Event Domain and assign the `EventGrid EventSubscription Contributor` role to Integration teams at vendors like Proseware. *Note that this role is not visible in the Azure Portal at the time of writing. This role can be assigned via PowerShell.*

![]({{site.baseurl}}/images/posts/contoso-construction-example.png)
*Source: Example of topic based access from Azure Documentation - [Event Domains](https://docs.microsoft.com/en-us/azure/event-grid/event-domains)*

There is still one last caveat here. The event size on Event Grid is limited to 1 MB (messages are metered at 64 KB). There can be RFPs larger than 1 MB and contain multimedia which is not suitable for the JSON schema supported by Event Grid. Even from a design stand point, the Event object should contain only enough metadata for the vendor to be able to locate the actual RFP and consume it into their processes. The location for the actual RFP is trivial to the problem statement as it could very well be in the ERP itself with authorized vendors allowed to pull information. For the sake of brevity, we opt for a Blob storage. We utilize the data property in the Event Grid schema to populate RFP metadata.

{% highlight json %}
{
    "topic": "proseware",
    "subject": "rfp/new",
    "id": "11568938",
    "eventType": "new",
    "eventTime": "2020-10-02T13:41:00.9584103Z",
    "data":{
        "id": "IN934NZ12",
        "xml": "https://northwind.blob.core.windows.net/trader-rfps/rfp.xml"
    },
    "dataVersion": "",
    "metadataVersion": "1"
}
{% endhighlight %}

The RFP XML document hosted on the North Wind blob storage is structured as follows.

{% highlight xml %}
<root>
    <title>Intent to create the largest Lego Death Star</title>
    <id>IN934NZ11</id>
    <type>M582</type>
    <orders>
        <order>
            <name>Darth Vader</name>
            <quantity>1</quantity>
        </order>
        <order>
            <name>Storm troopers</name>
            <quantity>60</quantity>
        </order>
        <order>
            <name>Emperor's Royal Guards</name>
            <quantity>20</quantity>
        </order>
        <order>
            <name>Square blocks</name>
            <quantity>80000000</quantity>
        </order>
        <order>
            <name>Blasters</name>
            <quantity>176</quantity>
        </order>
    </orders>
    <note>Vendor to take care of shipping and other logistics.</note>
</root>
{% endhighlight %}

The solution at this point looks something like this. We have everything in place to:
1. Emit events to a single endpoint.
2. Provide isolated control for each vendor's Active Directory backed guest users to setup event subscriptions on their own topics.

![]({{site.baseurl}}/images/posts/Azure Serverless Enterprise Integration-1.png)
*Separation of Concerns between North Wind and Proseware.*

# <a name="integration-point" href="#integration-point">Integration Point</a>
Proseware, like most businesses operates its own enterprise solutions protected within a private network. The RFP notification should be received by a gateway service that can securely transfer and host the corresponding RFP document within the Proseware network to be picked up by one or more internal business processes. *We assume that Proseware's private network is extended into Azure using either a Site-to-site VPN or ExpressRoute.*

## Solution
Here we play the role of the Integration team at Proseware. We would like to future-proof the solution by creating a single internal integration point where one or more business applications running within the corporate network can pick the RFP for further processing. Simplifying it by storing the RFP in a Blob storage that can be accessed through the Proseware VNET.

We can setup a subscription on the Event Grid Topic. For event destinations, Event Grid supports Webhooks or Azure resources like Functions, Storage Queues, Event Hubs, Hybrid Connections or Service Bus. As this is a cross-enterprise integration scenario (Proseware and North Wind have separate Azure Subscriptions) we won't be able to use the deep integrated Azure event destinations supported by Event Grid. That narrows down our choice to Webhooks.

The webhook subscription needs to point to a web service capable of doing the following.
1. Receive the webhook notification on a HTTP endpoint.
2. Parse the JSON based Event Grid schema to extract the RFP location on North Wind's storage account.
3. Download the RFP from the North Wind storage account and place it on a Proseware storage account that can be accessed only via a VNET.

It is important to understand that Event Grid requires a notification confirmation within 30 seconds. In our case, the HTTP endpoint should be able to respond quickly. The design needs to ensure that RFP downloads in step 3 should not block the HTTP response as the downloads especially in case of batch events might take longer. Also note that this case does not require step 3 to make an authenticated call to fetch the RFP document from North Wind, we would like to use an approach that could be extended to support a custom authentication mechanism.

**Azure Functions** app with a HTTP trigger works well for us here. The ability to implement just the business logic in languages like C#, Python, PowerShell and JavaScript allows us to extend the functionality if needed. We are going to require access to the Proseware VNET, for which the Function app can be hosted on a Premium plan or a dedicated App Service Plan or App Service Environment which allow VNET integration. Here we opt for the App Service Plan as any spare compute can be shared by other functions apps or app services. If you prefer to benefit from serverless scaling, Premium Plan would be the way to go.<sup>[[6]](#reference-6)</sup>

With Functions are now able to implement the service with minimal code ([complete solution](https://github.com/hexeight/azure-labs/tree/main/enterprise-serverless/integration-point)).We can leverage the [Durable Functions](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview) extension to implement asynchronous activities in the functions app. The webhook can be pointed a HTTP trigger that initializes an orchestrator instance. The orchestrator can then use the fan-out/fan-in pattern to download each RFP in the list of events individually without blocking the webhook response.

{% highlight c# %}
[FunctionName("RFPNotificationOrchestrator")]
public static async Task<dynamic> RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    List<EventGridEvent> events = context.GetInput<List<EventGridEvent>>();
    var tasks = new List<Task<bool>>();

    foreach (EventGridEvent e in events) 
    {
        tasks.Add(context.CallActivityAsync<bool>("DownloadRFP", e));
    }
    
    await Task.WhenAll(tasks);
    return from t in tasks select new { TaskId = t.Id, Status = t.Result };
}

[FunctionName("DownloadRFP")]
public static async Task<bool> DownloadRFP([ActivityTrigger] EventGridEvent e,
[Blob("northwind-rfp-collection", Connection = "RFPStorageConnection")] CloudBlobContainer blobContainer,
ILogger log)
{
    log.LogInformation("Processing event {0}.",e.Id);
    // Check if event is for Webhook registration
    if (e.EventType == "Microsoft.EventGrid.SubscriptionValidationEvent") {
        EventGridValidation validationObject = (e.Data as JObject).ToObject<EventGridValidation>();
        log.LogInformation("Validation URL: {0}", validationObject.validationUrl);
        return true;
    }

    RFPDescriptor descriptor = (e.Data as JObject).ToObject<RFPDescriptor>();
    log.LogInformation("Downloading RFP document ID {0} from North Wind", descriptor.id);
    var response = await _httpClient.GetAsync(descriptor.xml);
    var responseString = await response.Content.ReadAsStringAsync();

    // Store RFP to Blob Storage
    string blobName = String.Format("{0}.xml", descriptor.id);
    log.LogInformation("Uploading RFP document ID {0} to internal storage", descriptor.id);
    CloudBlockBlob blob = blobContainer.GetBlockBlobReference(blobName);
    await blob.UploadTextAsync(responseString);

    return true;
}
{% endhighlight %}

After implementing the integration point, Proseware is able to grant internal business processes access to the Blob storage to pick up fresh RFPs coming in from North Wind.

![]({{site.baseurl}}/images/posts/Azure Serverless Enterprise Integration-2.png)
*Proseware's Integration Point to North Wind's RFPs*

# <a name="evolving-processes" href="#evolving-processes">Evolving processes</a>
Businesses units like Sales, within Proseware would like to own and control their processes to reduce operational overheads. The solution needs to provide enough flexibility for the business process to evolve quickly. To begin with, the sales team would like to receive an email notification when a RFP is uploaded to the Proseware RFP Store (Blob storage). The email should include the specifics mentioned in the XML document in human readable format.

# Solution
Considering that businesses would like to own and control the process here, it is ideal to pursue a low-code or no-code solution that businesses can easily maintain. The flexibility offered by Power Platform for such cases is incredible. But we are dealing with a scenario where RFPs are stored in a Blob storage controlled by Proseware with a private network.

Logic Apps on Azure could help, but they do not normally offer VNET integration that would allow us to access the Proseware Blob storage. However, a Logic App can be hosted in an Integrated Service Environment (ISE) - an offering from Azure specifically tailored for Logic Apps which offer the same level of resource isolation that App Service Environments (ASE) offer for App Services. This comes along with the ability to run specific actions with the ISE and leverage the provided VNET integration<sup>[[7]](#reference-7)</sup>.

![]({{site.baseurl}}/images/posts/logic-app-ise.png)
*Actions marked as CORE and ISE are executed in the ISE.*

Further, we need to parse the XML file and extract the RFP specifications and place it in an email. This is where the Enterprise Integration Pack (EIP) comes in handy. It is important to note that EIP offers [much more](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-overview) than just XML parsing. With EIP, an integration account can be used to store mapping instructions in form of Liquid templates, XSLT or HIDX. For XML to text, we will use the following Liquid template. You can learn more about Liquid [here](https://shopify.github.io/liquid/).

{% raw %}
```
Hello team,
<br/><br/><br/>
North Wind has issued a notification requesting proposals against the following order.
<br/><br/>
-----------------------------------------------------------------------<br/>
Title: {{content.title}}<br/>
ID: {{content.id}}<br/>
Type: {{content.type}}<br/>
<br/><br/>
Order details:<br/>
{% for order in content.orders %}<br/>
{{order.name}} x {{order.quantity}}<br/>
{% endfor %}<br/><br/><br/>

{{content.note}}<br/><br/><br/>
-----------------------------------------------------------------------<br/>
Please follow up at the earliest.<br/><br/><br/><br/>



Kind regards,<br/>
Proseware Integration Team<br/>
```
{% endraw %}

If we use this template against the sample North Wind RFP XML, the end result looks like this.

```
Hello team,

North Wind has issued a notification requesting proposals against the following order.

-----------------------------------------------------------------------
Title: Intent to create the largest Lego Death Star
ID: IN934NZ11
Type: M582

Order details:

Darth Vader x 1

Storm troopers x 60

Emperor's Royal Guards x 20

Square blocks x 80000000

Blasters x 176

Vendor to take care of shipping and other logistics.

-----------------------------------------------------------------------
Please follow up at the earliest.

Kind regards,
Proseware Integration Team
```

We implement the following steps in the Logic App, starting with a blob file trigger followed by mapping the XML into text using the above template. The text is then sent over email to the sales team.

![]({{site.baseurl}}/images/posts/logic-app-flow.png)
*Logic App*

With the full solution in order, we achieve the following:
1. North Wind's ERP is able to publish events for vendors to consume.
2. Vendors like Proseware are able to create and manage their own subscriptions.
3. Proseware is able to pick RFP documents and make them accessible to internal processes.
4. The Sales team at Proseware is able to control and setup their own process like receiving emails with RFP details from North Wind.

![]({{site.baseurl}}/images/posts/Azure Serverless Enterprise Integration-3.png)
*Evolving Process*

## Disclaimer
The solution above is prepared with the intention to focus on the key serverless offerings from Azure. The reader is requested to revisit design decisions when building solutions to address their own requirements.

## References
1. <a name="reference-1"></a>[Marketing software company gives customers greater flexibility using serverless functions](https://customers.microsoft.com/en-in/story/stylelabs-media-telco-azure)
2. <a name="reference-2"></a>[E-discovery company uses serverless computing to speed internal development of its SaaS solution](https://customers.microsoft.com/en-in/story/relativity-partner-professional-services-azure)
3. <a name="reference-3"></a>[The Urlist — An application study in Serverless and Azure](https://burkeholland.github.io/posts/the-urlist/)
4. <a name="reference-4"></a>[Choose between Azure messaging services - Event Grid, Event Hubs, and Service Bus](https://docs.microsoft.com/en-us/azure/event-grid/compare-messaging-services)
5. <a name="reference-5"></a>[Azure Documentation - Event Grid Domains](https://docs.microsoft.com/en-us/azure/event-grid/event-domains)
6. <a name="reference-6"></a>[Azure Functions - Scale](https://docs.microsoft.com/en-us/azure/azure-functions/functions-scale)
7. <a name="reference-7"></a>[Connect to Azure virtual networks from Azure Logic Apps by using an integration service environment (ISE)](https://docs.microsoft.com/en-us/azure/logic-apps/connect-virtual-network-vnet-isolated-environment)