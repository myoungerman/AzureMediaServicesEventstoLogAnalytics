# Tutorial: Store Azure Media Services Events in Azure Log Analytics

In this tutorial you will learn how to store Azure Media Services events in Azure Log Analytics. Azure Media Services v3 emits events on [Azure Event Grid](https://docs.microsoft.com/en-us/azure/media-services/latest/media-services-event-schemas). You can subscribe to these events and store the events in various data stores. In this tutorial you will subscribe to these events using a [Log App Flow](https://azure.microsoft.com/en-us/services/logic-apps/). The Logic App will be triggered for each event and store the body of the event in Azure Log Analytics. Once the events are in [Azure Log Analytics](https://docs.microsoft.com/en-us/azure/azure-monitor/learn/quick-create-workspace), you can use other Azure services to monitor, dashboard, and alert on these events.

## Prerequisites
* [Azure Subscription](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Azure Media Services account
* [Azure Log Analytics Workspace](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace)

## Record Log Analytics workspace ID and agent key

To connect the Logic App you'll create to your Log Analytics Workspace, you need the Workspace ID and Agent Key.

1. In the Azure Portal, navigate to the Log Analytics Workspace you created before the start of this tutorial.

1. Select **Agents Management** from the left panel. Copy and paste the **Workspace ID** and the **Primary key** to a text editor.

## Create and setup your Logic App

1. In the Azure portal, go to your Azure Media Services account and select **Events** from the panel on the left. The main panel will show the event handlers available for Azure Media Services events.

1. Select **Logic Apps** to open the Logic App Designer, where you can create the process that will capture events and push them to Log Analytics.

1. Select **+** on the right to authenticate and subscribe to the Event Grid. A window with a list will appear, prompting you to select a tenant.

1. Select your tenant from the list, and then select **Sign in**. Once the authentication is complete, you'll see your user email and a green checkmark.

1. Select **Continue** to subscribe to the Media Services Events. The **When a resource event occurs** window will appear.

1. In the **Resource Type** list, select _Microsoft.Media.MediaServices_.

1. In the **Event Type Item** list, select _Microsoft.Media.LiveEventEncoderConnected_, then select **Add new item** to create a new blank item. Create an item for each event that begins with _Microsoft.Media.LiveEvent_ in the list.

1. Below the **When a resource event occurs** window, select **+ New Step**. The **Choose an operation** window will appear.

1. Since we want to push the events to the Azure Log Analytics service, search for _Azure Log Analytics_ and select **Azure Log Analytics Data Collector**. The **Azure Log Analytics Data Collector** window will appear.

1. Enter a connection name, as well as the Workspace ID and Workspace key you copied from your Log Analytics workspace, and then select **Create**. The **Send Data (Preview)** window will appear.

1. Select the **JSON Request body** field. A window will appear with two tabs, **Dynamic Content** and **Expression**.

1. Select the **Dynamic Content** tab and then select **Topic** from the list. Do the same for the **Custom Log Name** tab.

1. Select **Code View** at the top of the page. Near the top of the JSON locate the **inputs** property and change its **body** property from `"@triggerBody()?['topic']"` to `"@{triggerBody()}"`. This will parse the entire message to Log Analytics.

1. In the **inputs** property, change its **Log-Type** property from `"@triggerBody()?['topic']"` to `"@replace(triggerBody()?['eventType'],'.','')"`. This will replace ".", which are not allowed in Log Analytics Log Names.

1. Select **Save As** at the top of the page.

1. In the **Name** field, enter a name for your Logic App, and add it to a resource group.

1. Select **Create**, and then check your resource group by searching for _Resource groups_ in the main search bar of the webpage. In your resource group, you should see a Logic App and two API connections, one for the Events and one for Log Analytics.

## Test your logic app

To test your logic app, you'll create a Live Event in Azure Media Services, and then you'll use FFmpeg to push a "live" stream from an MP4 sample file.

1. Create a live event in Azure Media Services, and copy the live event's RTMP ingest URL.

1. Create or download an MP4 file to your computer.

1. Open a text editor and paste both the RTMP ingest URL and `ffmpeg -i <sample file> -map 0 -c:v libx264 -c:a copy -f flv <RTMP ingest URL>/mystream`.

1. Replace `<sample file>` with the file path and name of your sample file, and replace `<RTMP ingest URL>` with your RTMP ingest URL. Now your command should look similar to this:

  ```
  ffmpeg -i mySampleVideo.mp4 -map 0 -c:v libx264 -c:a copy -f flv rtmp://amsevent-amseventdemo-euwe.channel.media.azure.net:1935/live/4b969cd6ac3e4ad68b539c2a38c6f8f3/mystream
  ```

1. Open **Command Prompt**, and then copy and paste your command. Press **Enter**. After a couple seconds you should see the stream in the **Producer view** livestream player. If you don't see the stream, refresh the player.

![Verify proper video ingest in Producer Preview Player](src/18.png)

1. Go to the **Run History** section of **Logic App Overview** to ensure the Logic App flow is being triggered by the live event. You should see a list of jobs that have completed successfully. When you click on a successful job you can see the details of the job during runtime. In this case the "MicrosoftMediaLiveEventEncoderConnected" event was captured and we can see the parsed body.

1. Navigate to the Log Analytics Workspace you created earlier.

1. In  **Log Analytics** select **Logs** in the left menu. This should open the Log Query containing a **Custom Logs** section with the event name **MicrosoftMediaLiveEventEncoderConnected**. 

  > [!NOTE]
  > You may need to refresh the page. The first time it can take a few minutes to create the custom log and populate the data.**

  You can also:
  * Select the event to see all of its fields.
  * Select the "eye" icon to see a preview of the query result.
  * Select **See in query editor** to see the raw data of all fields.

## Next steps:
You can create different queries and save them. These can be added to [Azure Dashboard](https://docs.microsoft.com/en-us/azure/azure-monitor/learn/tutorial-logs-dashboards).
