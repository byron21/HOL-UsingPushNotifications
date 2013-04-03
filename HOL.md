<a name="title" />
#Using Push Notifications#

---
<a name="Overview" />
## Overview ##

Except for background agents, WP8 applications can't run in the background (you protect performance, battery life, usage of radio) and therefore there is a need to have a mechanism that facilitates a communication channel between backend services and application on the phone. Push Notification is the mechanism through which backend services can send "messages" to the phone to notify the user for any state changes. The Push Notification service is designed to provide a cloud service with a dedicated, resilient, and persistent channel for pushing a notification to a mobile device.

 ![Push Notifications](./Images/Push-Notifications.png?raw=true "Push Notifications")

_Push Notifications_

When a cloud service needs to send a push notification to a device, it sends a notification request to the Push Notification service, which in turn routes the notification to the application, or to the device as a toast or tile notification, depending on the type of notification that is sent. 
The Push Client on the device receives push notifications through a notification channel. When the channel is created, a subscription is also created which allows the cloud service to push notifications to that channel. A channel is represented by a URI which contains all of the information associated with the subscription.
Once an application receives the push notification, it can access the cloud service using the cloud service's protocol to retrieve any needed information.
Since, applications today tends to share multiple screens running on local computer, in the cloud and on the phone. The push notification services, enables seamless integration between all three with a simple and unified API.
This lab covers the push notification and also introduces the usage of http services in Windows Phone apps. During this lab you will create server side logic needed to send messages through Push Notification Service. You will create a simple Windows Phone 8 application which serves as a client to receive such notifications. The client application will receive weather updates. The server side business application (simple WPF application) will send weather alerts to registered client applications through Push Notification Services. Once client Windows Phone 8 application will receive such alert it will display received information.

 
### Objectives ###

At the end of the lab you will:

- Familiarize with the communication capabilities of Windows Phone 8 application

- Familiarize with the push notification concepts and the behaviors they enable on the phone

- Understand how push notification works on the phone and in the cloud

- Use the phone push notification services to create a subscription for Tokens (tiles), Toasts, and raw push notification

- Use web client to register for Push Notifications

- Use the network status to display the current status of the phone network

- Create a Windows Phone application that registers for push notification services (both token and toast)

	- Handle push events (token, toast, and raw)during run time

	- Show token and toast on shell

<a name="technologies" /> 
### Prerequisites ###

The following is required to complete this hands-on lab:

- Microsoft Visual Studio 2012 Express for Windows Phone or Microsoft Visual Studio 2012

- Windows Phone Developer Tools

> **Note:** All of these Tools can be downloaded together in a single package from [http://developer.windowsphone.com](http://developer.windowsphone.com/)

---
<a name="Exercises" />
## Exercises ##

This hands-on lab comprises the following exercises:

1. [Introduction to the Toast and Tile Notifications for Alerts](#Exercise1)


Estimated time to complete this lab: **30 minutes**.

<a name="Exercise1" />
## Exercise 1: Introduction to the Toast and Tile Notifications for Alerts ##

In this section you will learn about two notification types - Toast and Tile notification. This lab covers the backend server that posts data to MPNS, as well as how to register and handle these events on your Windows Phone 8 Application.

Tiles and toast notifications are two mechanisms that enable a cloud service to deliver relevant, actionable feedback to users outside an application's own user interface. Additionally, a cloud service can send a raw notification request. Depending on the type of notification sent, the notification will be routed either to the application or the shell.

**Tile Notifications**

A tile is a visual, dynamic representation of an application or its content within the Quick Launch area of the phone's Start experience. For example, a weather application may choose to display the user's local time and climate conditions in a tile. Because a cloud service can alter its tile's appearance at any time, this mechanism can be used to communicate information to the user on an ongoing basis. Each application that the user can launch on the phone is associated with a single tile, but only the user can control which of these tiles are pinned to the Quick Launch area.

A cloud service can control a tile's background image, counter (or 'badge'), and title properties. These properties are configured using the Windows Phone Developer Tools. Animation and sound properties are controlled by how the platform is configured, not by the application. For example, if the platform is configured to animate and beep upon any tile update, that is what will occur for any tile.

A tile's background image can reference either a local resource, which is part of the application deployment, or a cloud resource. By referencing a resource in the cloud, applications are enabled to dynamically update a tile's background image. This enables scenarios which require processing of the background image before it is displayed. In most scenarios, the application package should include all needed background images for the tile, since this is the best solution for performance and battery life.

**Toast Notifications**

A cloud service can generate a special kind of push notification known as a toast notification, which displays as an overlay onto the user's current screen. For example, a weather application may wish to display a toast notification if a severe weather alert is in effect. If the user decides to click the toast notification, the application can launch and perform other actions.

A cloud service can control a toast notification's title and sub-title. The toast notification will also display the application's icon that is included in the application's deployment package.

**Best Practices**

- Toast notifications should be personally relevant and time critical.

<a name="Ex1Task1" />
#### Task 1 - Modify the WCF Service Endpoint to Allow Remote Host Access ####

1. Open Microsoft Visual Studio 2012 Express for Windows Phone from **Start | All Programs | Microsoft Visual Studio 2012 Express | Microsoft Visual Studio 2012 Express for Windows Phone**.

	> **Visual Studio 2012**: Open Visual Studio 2012 from **Start | All Programs | Microsoft Visual Studio 2012**.

	> **Important note**: In order to run self-hosted WCF services within Visual Phone 2012 Express for Windows Phone or Microsoft Visual Studio 2012 it must be opened in Administrator Mode. For reference about creating and hosting self-hosted WCF services see MSDN article ([http://msdn.microsoft.com/en-us/library/ms731758.aspx](http://msdn.microsoft.com/en-us/library/ms731758.aspx)). In order to open Visual Studio 2012 Express for Windows Phone or Visual Studio 2012 in Administrative Mode locate the Microsoft Visual Studio 2012 Express for Windows Phone shortcut at **Start | All Programs | Microsoft Visual Studio 2012 Express** or Microsoft Visual Studio 2012 shortcut at **Start | All Programs | Microsoft Visual Studio 2012**, right-click on the icon and select "Run as administrator" from the opened context menu. The UAC notification will pop up. Click "Yes" in order to allow running the Visual Studio 2012 Express for Windows Phone or Visual Studio 2012 with elevated permissions.

1. Open the **Begin.sln** starter solution from the **Source\Ex2-TileToastNotifications\Begin** folder of this lab. 

	> The Windows Phone 8 emulator runs as a completely separate host on your network. By default, the WCF service is set to allow connections only from clients on the same PC. You will now change this to allow remote connections.

1. Open app.config in the Weather project. This file defines the endpoint for the WCF service.

	1. Change the following line:

		````XML
		<endpoint address=http://localhost:8000/RegistrationService
````

		Replace **'localhost'** with the real name of your PC.

1. In the PushNotifications project (the WP8 client), open file MainPage.xaml.cs.

	1. Change the following line:

		````C#
		// Hardcode for solution - need to be updated in case the REST  
		// WCF service address change
		string baseUri = "http://localhost:8000/RegistrationService/Register?uri={0}";
````

		Replace '**localhost'** with the real name of your PC.

1. You must now open the port 8000 in your firewall for incoming HTTP access so that the client application running on the Windows Phone emulator can connect to the WCF service.

	1. TEMPORARY FIX: Disable the Firewall for now...

<a name="Ex1Task2" />
#### Task 2 - Implementing Server Side of Sending Tiles & Toasts ####

1. Open the NotificationSenderUtility.cs file located under the NotificationSenderUtility project.

1. Toast and Tile notification are system defined notification in Windows Phone 8 platform. This is different from RAW notifications, where all applications could create their own payload format and parse it accordingly. During next couple steps you will create general functionality to create Tile & Toast messages payload - this payload will work for any Windows Phone 8 application. In addition you will expose public functionality to send such messages and connect WPF Push Notification Client's buttons events to it. Locate the **SendXXXNotification functionality** region. Add the following public functions to the region (they will be used by the WPF application later):

	<!-- mark:1-19 -->
	````C#
	public void SendToastNotification(List<Uri> Uris, string message1, 
					string message2, SendNotificationToMPNSCompleted callback)
	{
		byte[] payload = prepareToastPayload(message1, message2);

		foreach (var uri in Uris)
			SendNotificationByType(uri, payload, NotificationType.Toast, callback);
	}

	public void SendTileNotification(List<Uri> Uris, string TokenID, 
						  string BackgroundImageUri, string SmallBackgroundImageUri, 
						  int Count, string Title, 
						  SendNotificationToMPNSCompleted callback)
	{
		byte[] payload = prepareTilePayload(TokenID, BackgroundImageUri, SmallBackgroundImageUri, Count, Title);

		foreach (var uri in Uris)
			SendNotificationByType(uri, payload, NotificationType.Token, callback);
	}
````

1. Next you'll create a new region with two functions - first to prepare the Toast notification payload, and second for Tile notification payload preparation.

	Tile notification message to update tiles using the FlipTemplate should be in the following format.

	<!-- mark:1-18 -->
	````XML
	Content-Type: text/xml
	X-WindowsPhone-Target: token

	<?xml version="1.0"?>
	<wp:Notification xmlns:wp="WPNotification" Version="2.0">
		 <wp:Tile Id="[TileId]" Template="FlipTile">
			<wp:SmallBackgroundImage Action="Clear">[sm tile img URI]</wp:SmallBackgroundImage>
			<wp:WideBackgroundImage Action="Clear">[front of wd tile img URI]</wp:WideBackgroundImage>
			<wp:WideBackBackgroundImage Action="Clear">[back of wd tile img URI]</wp:WideBackBackgroundImage>
			<wp:WideBackContent Action="Clear">[back of wd tile content text]</wp:WideBackContent>
			<wp:BackgroundImage Action="Clear">[front of med tile img URI]</wp:BackgroundImage>
			<wp:Count Action="Clear">[count]</wp:Count>
			<wp:Title Action="Clear">[title text]</wp:Title>
			<wp:BackBackgroundImage Action="Clear">[back of med tile img URI]</wp:BackBackgroundImage>
			<wp:BackTitle Action="Clear">[back of tile title text]</wp:BackTitle>
			<wp:BackContent Action="Clear">[back of med tile content text]</wp:BackContent>
		 </wp:Tile>
	</wp:Notification>
````

	Toast notification message on the other hand should be in the following format. Notice that _\<Text1\>_ and _\<Text2\>_ are in string format.

	<!-- mark:1-10 -->
	````XML
	Content-Type: text/xml
	X-WindowsPhone-Target: toast
	
	<?xml version="1.0" encoding="utf-8"?>
	<wp:Notification xmlns:wp="WPNotification">
	   <wp:Toast>
	      <wp:Text1><string></wp:Text1>
	      <wp:Text2><string></wp:Text2>
	   </wp:Toast>
	</wp:Notification>
````

1. Create the **Prepare Payload** family of functions according to the following code snippet:

	<!-- mark:3-88 -->
	````C#
	#region Prepare Payloads

	private static byte[] prepareToastPayload(string text1, string text2)
	{
	    MemoryStream stream = new MemoryStream();
	    XmlWriterSettings settings = new XmlWriterSettings() { Indent = true, Encoding = Encoding.UTF8 };
	    XmlWriter writer = XmlWriter.Create(stream, settings);
	    writer.WriteStartDocument();
	    writer.WriteStartElement("wp", "Notification", "WPNotification");
	    writer.WriteStartElement("wp", "Toast", "WPNotification");
	    writer.WriteStartElement("wp", "Text1", "WPNotification");
	    writer.WriteValue(text1);
	    writer.WriteEndElement();
	    writer.WriteStartElement("wp", "Text2", "WPNotification");
	    writer.WriteValue(text2);
	    writer.WriteEndElement();
	    writer.WriteEndElement();
	    writer.WriteEndDocument();
	    writer.Close();
	
	    byte[] payload = stream.ToArray();
	    return payload;
	} 

	private static byte[] prepareTilePayload(string tokenId, string backgroundImageUri, string smallbackgroundImageUri, int count, string title)
	{
		MemoryStream stream = new MemoryStream();
		XmlWriterSettings settings = new XmlWriterSettings() { Indent = true, Encoding = Encoding.UTF8 };
		XmlWriter writer = XmlWriter.Create(stream, settings);
		writer.WriteStartDocument();
		writer.WriteStartElement("wp", "Notification", "WPNotification");
		writer.WriteStartAttribute("Version");
		writer.WriteValue("2.0");
		writer.WriteEndAttribute();
		writer.WriteStartElement("wp", "Tile", "WPNotification");
		writer.WriteStartAttribute("Template");
		writer.WriteValue("FlipTile");
		writer.WriteEndAttribute();
		writer.WriteStartElement("wp", "SmallBackgroundImage", "WPNotification");
		writer.WriteStartAttribute("Action");
		writer.WriteValue("Clear");
		writer.WriteEndAttribute();
		writer.WriteEndElement();
		writer.WriteStartElement("wp", "WideBackgroundImage", "WPNotification");
		writer.WriteStartAttribute("Action");
		writer.WriteValue("Clear");
		writer.WriteEndAttribute();
		writer.WriteEndElement();
		writer.WriteStartElement("wp", "WideBackBackgroundImage", "WPNotification");
		writer.WriteStartAttribute("Action");
		writer.WriteValue("Clear");
		writer.WriteEndAttribute();
		writer.WriteEndElement();
		writer.WriteStartElement("wp", "WideBackContent", "WPNotification");
		writer.WriteStartAttribute("Action");
		writer.WriteValue("Clear");
		writer.WriteEndAttribute();
		writer.WriteEndElement(); 
		writer.WriteStartElement("wp", "BackgroundImage", "WPNotification");
		writer.WriteValue(backgroundImageUri);
		writer.WriteEndElement();
		writer.WriteStartElement("wp", "Count", "WPNotification");
		writer.WriteValue(count.ToString());
		writer.WriteEndElement();
		writer.WriteStartElement("wp", "Title", "WPNotification");
		writer.WriteValue(title);
		writer.WriteEndElement();
		writer.WriteStartElement("wp", "BackBackgroundImage", "WPNotification");
		writer.WriteStartAttribute("Action");
		writer.WriteValue("Clear");
		writer.WriteEndAttribute();
		writer.WriteEndElement();
		writer.WriteStartElement("wp", "BackTitle", "WPNotification");
		writer.WriteStartAttribute("Action");
		writer.WriteValue("Clear");
		writer.WriteEndAttribute();
		writer.WriteEndElement();
		writer.WriteStartElement("wp", "BackContent", "WPNotification");
		writer.WriteStartAttribute("Action");
		writer.WriteValue("Clear");
		writer.WriteEndAttribute();
		writer.WriteEndElement(); 
		writer.WriteEndElement();
		writer.Close();
		byte[] payload = stream.ToArray();
		
		return payload;
	}

	#endregion
````

	> **Note:** The code for the **prepareTilePayload** method above only allows the BackgroundImageUri, SmallBackgroundImageUri, Title and Count tile properties to be set. This is done for simplicity in this lab.

	> The other tile properties are all cleared using the **Action="Clear"** attribute. In your own solutions, you can set any of these properties in your tile update Push Notification messages.


1. Open **MainWindow.xaml.cs** from the **Weather** project.

1. Locate the **sendToast** function. This function should get the Toast message from the Push Notification Client UI and send it to all the subscribers. Add the following code snippet in the function body:

	<!-- mark:3-7 -->
	````C#
	private void sendToast()
	{
	    string msg = txtToastMessage.Text;
	    txtToastMessage.Text = "";
	    List<Uri> subscribers = RegistrationService.GetSubscribers();
	    ThreadPool.QueueUserWorkItem((unused) => notifier.SendToastNotification(subscribers,
	        "WEATHER ALERT", msg, OnMessageSent));
	}
````

1. Locate the **sendTile** function. This function should get the parameters from Push Notification Client UI and send it to all the subscribers. Add the following code snippet in the function body:

	<!-- mark:3-7 -->
	````C#
	private void sendTile()
	{
	    string weatherType = cmbWeather.SelectedValue as string;
	    int temperature = (int)sld.Value;
	    string location = cmbLocation.SelectedValue as string;
	    List<Uri> subscribers = RegistrationService.GetSubscribers();
	    ThreadPool.QueueUserWorkItem((unused) => notifier.SendTileNotification(subscribers, "PushNotificationsToken", "/Images/" + weatherType + ".png", "/Images/Small/" + weatherType + ".png", temperature, location, OnMessageSent));
	}
````

1. Compile and run the applications. Check that messages are dispatched to the Push Notification Service.

 	![WPF Push Notifications Client log](./Images/WPF-Push-Notifications-Client-log.png?raw=true "WPF Push Notifications Client log")
 
	_WPF Push Notifications Client log_

	> **Note:** You first need to define multiple startup projects for this solution in order to run WPF Push Notification Client and Windows Phone 8 Push client together.

	> In order to do this, in Solution Explorer right-click on the solution name and select Properties from the context menu. Select the **Startup Projects** page from **Common Properties** (if not selected automatically), select **Multiple startup projects** and set the **PushNotifications** and **Weather** projects as **Start** from the **Action** drop-down list.

	> ![Multiple Startup Projects](./Images/Multiple-Startup-Projects.png?raw=true)

	> _Selecting Multiple Startup Projects_

<a name="Ex1Task3" />
#### Task 3 - Processing Tile & Toast Notifications on the Phone ####

1. Open **MainPage.xaml.cs** in the **PushNotifications** project.

1. In the next few steps you will subscribe to Tile and Toast notification events and will handle those events. Locate the **Subscriptions** region and add the following code snippet:

	<!-- mark:1-47 -->
	````C#
	private void SubscribeToNotifications()
	{
	    //////////////////////////////////////////
	    // Bind to Toast Notification 
	    //////////////////////////////////////////
	    try
	    {
	        if (httpChannel.IsShellToastBound == true)
	        {
	            Trace("Already bound (registered) to Toast notification");
	        }
	        else
	        {
	            Trace("Registering to Toast Notifications");
	            httpChannel.BindToShellToast();
	        }
	    }
	    catch (Exception ex)
	    {
	        // handle error here
	    }
	
	    //////////////////////////////////////////
	    // Bind to Tile Notification 
	    //////////////////////////////////////////
	    try
	    {
	        if (httpChannel.IsShellTileBound == true)
	        {
	            Trace("Already bound (registered) to Tile Notifications");
	        }
	        else
	        {
	            Trace("Registering to Tile Notifications");
	
	            // you can register the phone application to receive tile images from remote servers [this is optional]
	            Collection<Uri> uris = new Collection<Uri>();
	            uris.Add(new Uri("http://jquery.andreaseberhard.de/pngFix/pngtest.png"));
	
	            httpChannel.BindToShellTile(uris);
	        }
	    }
	    catch (Exception ex)
	    {
	        //handle error here
	    }
	}
````

1. Now locate the **SubscribeToChannelEvents** function and add the following highlighted code snippet to the function body:

	<!-- mark:12-13 -->
	````C#
	private void SubscribeToChannelEvents()
	{
		//Register to UriUpdated event - occurs when channel successfully opens
		httpChannel.ChannelUriUpdated += new EventHandler<NotificationChannelUriEventArgs>(httpChannel_ChannelUriUpdated);
		
		//Subscribed to Raw Notification
		httpChannel.HttpNotificationReceived += new EventHandler<HttpNotificationEventArgs>(httpChannel_HttpNotificationReceived);

		//general error handling for push channel
		httpChannel.ErrorOccurred += new EventHandler<NotificationChannelErrorEventArgs>(httpChannel_ExceptionOccurred);

		//subscrive to toast notification when running app    
		httpChannel.ShellToastNotificationReceived += new EventHandler<NotificationEventArgs>(httpChannel_ShellToastNotificationReceived);
	}
````

1. Locate the **Channel event handlers** region and add the following function to handle the events:

	<!-- mark:1-14 -->
	````C#
	void httpChannel_ShellToastNotificationReceived(object sender, NotificationEventArgs e)
	{ 
	   Trace("===============================================");
	   Trace("Toast/Tile notification arrived:");
	   foreach (var key in e.Collection.Keys)
	   {
	      string msg = e.Collection[key];
	
	      Trace(msg);
	      Dispatcher.BeginInvoke(() => UpdateStatus("Toast/Tile message: " + msg));
	   }
	         
	   Trace("===============================================");
	}
````

	> **Note:** In our simple case the functions just tracing the message payload, but in real-world application you could use it to perform any business logic.

1. Lastly, add the following code snippet to the following number of locations:

	<!-- mark:1 -->
	````C#
	SubscribeToNotifications();
````

	The locations to add this code snippet are the following:

	1. In the **DoConnect** function, in the **try** block, between "_SubscribeToService();_" and "_Dispatcher.BeginInvoke();_"

		<!-- mark:9-10 -->
		````C#
		if (null != httpChannel)
		{
		    Trace("Channel Exists - no need to create a new one");
		    SubscribeToChannelEvents();
		
		    Trace("Register the URI with 3rd party web service");
		    SubscribeToService();
		
		    Trace("Subscribe to the channel to Tile and Toast notifications");
		    SubscribeToNotifications();
		
		    Dispatcher.BeginInvoke(() => UpdateStatus("Channel recovered"));
		}
````

	1. In the **httpChannel_ChannelUriUpdated** function, between "_SubscribeToService();_" and "_Dispatcher.BeginInvoke(...);_"

		<!-- mark:7 -->
		````C#
		void httpChannel_ChannelUriUpdated(object sender, NotificationChannelUriEventArgs e)
		{
		    Trace("Channel opened. Got Uri:\n" + httpChannel.ChannelUri.ToString());
		    Dispatcher.BeginInvoke(() => SaveChannelInfo());
		    Trace("Subscribing to channel events");
		    SubscribeToService();
		    SubscribeToNotifications();
		    Dispatcher.BeginInvoke(() => UpdateStatus("Channel created successfully"));
		}
````

1. Press **F5** to compile and run the applications. On the phone emulator, click the Back button (![the Start Screen.](./Images/the-Start-Screen..png?raw=true "the Start Screen.")) to exit the Push Notification application and go to the Start Screen.

1. In the Start Screen area, swipe your finger quickly to the right to get to the "All Applications" screen.

1. Locate the **PushNotifications** item, press it down and hold until the context menu pops-up. Click **Pin to Start**.

 	![PushNotifications Tile](./Images/Pinning-Tile-To-Start-Screen.png?raw=true "Pinning tile to Start screen")

	_Pinning tile to Start screen_

1. The emulator will automatically go back to the Start Screen where you'll notice the **PushNotifications** tile pinned.

 	![PushNotifications Tile](./Images/PushNotifications-Tile.png?raw=true "PushNotifications Tile")
 
	_PushNotifications Tile_

1. On WPF Push Notification test app, change some notifications parameters and click the **Send Tile** button. Observe the Tile change in emulator. If you click the _Toast message_ you will be taken back to the application.

1. On the emulator, tap and hold on the application tile on the start screen and change it to the small tile size.
Send a few different tile designs through from the WPF test app and notice how the small tile also changes but uses different images to the medium tile size. 

1. On the WPF Push Notification Client enter some message and click the **Send Toast** button. Observe how the Toast message arrives to the phone. Click the _Toast message_ to switch back to the application.

	> **Note:** The complete solution for this exercise is provided at the following location: **Source\Ex1-ToastAndTileNotificationsForAlerts\End**.

---
<a name="Summary" /> 
## Summary ##

During this lab you learned about the notification services for the Windows Phone 8 platform. You learned about notifications types, and how to prepare and send them through Microsoft Push Notification Service. You added functionality to the business client application to prepare and send such messages and Windows Phone 8 client application. The Windows Phone 8 client application subscribes to the notifications and updates the UI according to the information received in the messages.





