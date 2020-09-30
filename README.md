# Tutorial: Store Azure Media Services Events in Azure Log Analytics

In this tutorial you will learn how to store Azure Media Services events in Azure Log Analytics.
* Create a no code Logic App Flow
* Subscribe to Azure Media Services Event Topics
* Parse Events and store to Azure Log Analytics
* Query Events from Azure Log Analytics

If you don’t have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

## Prerequisites:
* Azure Subscription
* AMS account already created
* Log Analytics Workspace already created

## Azure Media Services Events
Azure Media Services v3 emits events on [Azure Event Grid](https://docs.microsoft.com/en-us/azure/media-services/latest/media-services-event-schemas). You can subscribe to these events in many ways and store the events in various data stores. In this tutorial we will subscribe to these events using a [Log App Flow](https://azure.microsoft.com/en-us/services/logic-apps/). The Logic App will be triggered for each event and store the body of the event in Azure Log Analytics. Once the events are in [Azure Log Analytics](https://docs.microsoft.com/en-us/azure/azure-monitor/learn/quick-create-workspace) we can use other Azure services to monitor, dashboard and alert on these events as desired by your own needs.

For this tutorial we expect you already have an [Azure Media Services](https://docs.microsoft.com/en-us/azure/media-services/latest/create-account-howto) account. During the creation of the Logic App flow we also need to specify an Azure Log Analytics Workspace. 
**It is best you already set one up now using these simple steps: [Quick Create Workspace](https://docs.microsoft.com/en-us/azure/azure-monitor/learn/quick-create-workspace)**

To connect to the Log Analytics Workspace you need the Workspace ID and an Agent Key. In the Azure Portal navigate to your Log Analytics Workspace you created before the start of this tutorial. **To keep the Logic App designer open you can do this in a separate browser tab.** In the Azure Portal, in the Log Analytics workspace you can find the Workspace ID at the top.


![Azure Log Analytics Agents management](src/09.png)

On the left menu locate "Agents Management" and click on it. This will show you the agent keys that have been generated.


![Log Analytics Agent Key](src/10.png)

Copy one of the keys over to your Logic App.


![Create Azure Logic App Connector](src/11.png)

Now click on "Create".

## Walkthrough Steps

1. In the Azure portal, navigate to your Azure Media Services account and select **Events** from the navigation pane on the left. The main panel will display the event handlers available for Azure Media Services events.

1. Select **Logic Apps** to create a Logic App. This opens the Logic App Designer, where you can create the process that will capture events and push them to Log Analytics.

1. Select **+** on the right to authenticate and subscribe to the Event Grid. A window with a list will appear, prompting you to select a tenant.

1. Select your tenant from the list, and then select **Sign in**. Once the authentication is complete, you'll see your user email and a green checkmark.

1. Select **Continue** to subscribe to the Media Services Events. The **When a resource event occurs** window will appear.

1. In the **Resource Type** list, select _Microsoft.Media.MediaServices_.

1. In the **Event Type Item** list, select _Microsoft.Media.LiveEventEncoderConnected_.

1. Below the **When a resource event occurs** window, select **+ New Step**. The **Choose an operation** window will appear.

1. Since we want to push the events to the Azure Log Analytics service, search for _Azure Log Analytics_ and select **Azure Log Analytics Data Collector**. The **Azure Log Analytics Data Collector** window will appear.

1. Enter a connection name, as well as the Workspace ID and Workspace key you copied from your Log Analytics workspace, and then select **Create**. The **Send Data (Preview)** window will appear.

1. Select the **JSON Request body** field. A window will appear with two tabs, **Dynamic Content** and **Expression**.

1. Select the **Dynamic Content** tab and then select **Topic** from the list. Do the same for the **Custom Log Name** tab.

1. Select **Code View** at the top of the page. Near the top of the JSON locate the **inputs** property and change its **body** property from `"@triggerBody()?['topic']"` to `"@{triggerBody()}"`. This will parse the entire message to Log Analytics.

1. In the **inputs** property, change its **Log-Type** property from `"@triggerBody()?['topic']"` to `"@replace(triggerBody()?['eventType'],'.','')"`. This will replace ".", which are not allowed in Log Analytics Log Names.

1. Select **Save As** at the top of the page.

1. In the **Name** field, enter a name for your Logic App, and add it to a resource group.

1. Select **Create**.

1. Check your resource group by searching for _Resource groups_ in the main search bar of the webpage. In your resource group, you should see a Logic App and two API connections, one for the Events and one for Log Analytics.

## Test your logic app

To test your logic app, you'll create a Live Event in Azure Media Services, and then you'll use FFmpeg to push a "live" stream from an MP4 sample file.

1. Create a Live Event in Azure Media Services.

1. Copy the RTMP ingest URL.

1. Create or download an MP4 file to your computer.

1. Open a text editor and paste both the RTMP ingest URL and `ffmpeg -i <sample file> -map 0 -c:v libx264 -c:a copy -f flv <RTMP ingest URL>/mystream`.

1. Replace `<sample file>` with the file path and name of your sample file, and replace `<RTMP ingest URL>` with your RTMP ingest URL. Now your command should look something like this:

```
ffmpeg -i mySampleVideo.mp4 -map 0 -c:v libx264 -c:a copy -f flv rtmp://amsevent-amseventdemo-euwe.channel.media.azure.net:1935/live/4b969cd6ac3e4ad68b539c2a38c6f8f3/mystream
```

1. Open FFmpeg, and then copy and paste your command into the FFmpeg CLI. Press **Enter**.

After a couple seconds you should see the stream in the "Producer view" player. **Refresh the player if needed**.

![Verify proper video ingest in Producer Preview Player](src/18.png)

By now we have a livestream so Azure Media Services is emitting various events that are triggering the Logic App flow. To verify navigate to the Logic App and see if can see any triggers being fired by the events from Media Services.


![Verify successful job execution in Logic App](src/19.png)

In the Logic App Overview page we should see "Run History" at the bottom of the page with jobs that have completed successfully.


![See Job Details during Runtime](src/20.png)

When you click on a Successful Job you can see the details of the job during runtime. In this case you can see that the "MicrosoftMediaLiveEventEncoderConnected" event was captured and we can see the parsed body. This is what is pushed to the Azure Log Analytics Workspace. Let’s go the Log Analytics Workspace to verify. Navigate to Log Analytics Workspace you created earlier.


![See Events in Log Analytics](src/21.png)

In the Log Analytics click on "Logs" in the left menu. This should open de Log Query. There should be a "Custom Logs" with the event name "MicrosoftMediaLiveEventEncoderConnected". **Note: You may need to refresh the page. The first time it can take a couple minutes to create the custom log and the data to populate.**


![Preview Query](src/22.png)

You can expand to see all the fields for this event. When you click on the "eye" icon you can see a preview of the query result.


![Run Query in Query editor](src/23.png)

You can click "See in query editor" to see the raw data of all fields.


![See detailed Event output in Log Analytics](src/24.png)

This is the output from the query where we see all the data of the event "MicrosoftMediaLiveEventEncoderConnected".

## Next steps:
You can create different queries and save them. These can be added to [Azure Dashboard](https://docs.microsoft.com/en-us/azure/azure-monitor/learn/tutorial-logs-dashboards).
