---
id: 490
title: Controlling lights connected to a Raspberry Pi from a Windows Phone app
date: 2015-02-22T13:59:22+00:00
author: Omkar
excerpt: This might work just right
layout: post
guid: http://omkarslab.com/?p=490
permalink: /2015/02/22/controlling-lights-connected-to-a-raspberry-pi-from-a-windows-phone-app/
sfw_pwd:
  - rOVhqWUM2221
image: /wp-content/uploads/2014/03/WP_00012820140324230407-960x360.jpg\
categories:
  - Cloud
  - Experiments
  - General
  - Internet Of Things
tags:
  - Microsoft Azure
  - RaspberryPi
  - Windows Phone
---
<p style="text-align: justify;">
  Last year I spoke at Pune User Group&#8217;s DevCon 2014, an annual event for Developers in Pune that focuses on Microsoft technologies. This year we tried to have some tracks on devices and Internet of Things, and this blog covers a DIY for one of the demos I presented.
</p>

<p style="text-align: justify;">
  You get a glimpse of the demo here &#8211; <a href="http://omkarslab.com/2015/01/20/control-your-lights-with-cortana-watch-this-space-for-a-diy/" target="_blank">Cortana Turn on the lights</a>
</p>

<p style="text-align: justify;">
  To cut this DIY short I will be assuming you have a fair idea of
</p>

<ul style="text-align: justify;">
  <li>
    Some ultra basic linux stuff (because, Raspberry Pi)
  </li>
  <li>
    Basic Windows Phone development
  </li>
  <li>
    Running a Node.js &#8220;hello world&#8221;
  </li>
  <li>
    Heard of Microsoft&#8217;s Azure platform
  </li>
  <li>
    Some experience with REST API calls and OAuth
  </li>
</ul>

<p style="text-align: justify;">
  Even if you had none of these, you could still follow this DIY and use the links above to peek into the details of each pre-requisite as they show up.
</p>

<h4 style="text-align: justify;">
  Tools that you&#8217;ll need
</h4>

<ul style="text-align: justify;">
  <li>
    A Raspberry Pi board running Raspbian (Wheezy) setup to connect to the internet &#8211; I&#8217;ll be using RPi Model B, but the older or newer versions should work just as fine. We are not doing anything fancy with the hardware.
  </li>
  <li>
    An LED.
  </li>
  <li>
    Pair of wires with header connectors
  </li>
  <li>
    A 470 Ohm resistor
  </li>
  <li>
    A Windows Phone &#8211; the SDK emulator will do, but not as much fun
  </li>
  <li>
    A Microsoft Azure account. If you don&#8217;t have one, you can signup for a free trial here with $200 monthly credit. For this DIY you wont need more than a few cents.
  </li>
</ul>

<p style="text-align: justify;">
  Let&#8217;s get started&#8230;
</p>

<h4 style="text-align: justify;">
  Step I &#8211; Setting up Raspberry Pi
</h4>

<p style="text-align: justify;">
  Raspberry Pi running Raspbian is capable of running Node.js. If you haven&#8217;t setup your Pi for Node.js yet or running a Node.js version older than 0.10.2, here&#8217;s a guide to <a href="http://joshondesign.com/2013/10/23/noderpi" target="_blank">install Node.js v0.10.2 on your Raspberry Pi in 5 minutes</a>.
</p>

<p style="text-align: justify;">
  Once you have Node.js up and running, verify if you have the right version entering the following on your terminal.
</p>

<p style="text-align: justify;">
  <code> node -v</code>
</p>

<p style="text-align: justify;">
  OR
</p>

<p style="text-align: justify;">
  <code>nodejs -v</code>
</p>

<p style="text-align: justify;">
  If your version is 0.10.2 or later, you are good to go.
</p>

<p style="text-align: justify;">
  Now we need to figure out a way to interface the LED with your Pi. The Raspberry Pi has 8 safe to use GPIO (General Purpose Input Output) pins. There are more pins that could be used as GPIO, but they do a lot more than that and I prefer to keep them out of my designs. You can find their layout below.
</p>

<p style="text-align: justify;">
  <img class="aligncenter" src="http://elinux.org/images/2/2a/GPIOs.png" alt="" width="254" height="581" />
</p>

<p style="text-align: justify;">
  Also, here is an<a href="http://pi.gadgetoid.com/pinout/gpio" target="_blank"> interactive GPIO layout of the Raspberry Pi</a>. Remember the Pi models until Model B had only 26 pins, the other pins are part of the extended GPIO of Model B+.
</p>

<p style="text-align: justify;">
  To check if your LED is working right, first start your Pi and connect the cathode of the LED to the GND pin i.e Pin 6 and the anode to Pin 1 (3.3v supply). Your LED should glow. Remove the anode and now connect to one of the GPIO pins. I&#8217;ll be using Pin 11 in my code, so you might as well tag along.
</p>

<p style="text-align: justify;">
  One last mile before we get started with Pi. The <code>pi-gpio</code> module for Node.js needs to be installed, so that we can control the GPIO from Node.js. Do install the module, use the following command in the directory you wish to use for your project.
</p>

<p style="text-align: justify;">
  <code>npm install pi-gpio&lt;code></code></code>
</p>

<p style="text-align: justify;">
  Once this goes successful, your Pi is ready for action.
</p>

<h4 style="text-align: justify;">
  Step II &#8211; Controlling LED via Node.js on Raspberry Pi
</h4>

<p style="text-align: justify;">
  By now you should have the directory you&#8217;ll be working in, as you have already installed the pi-gpio module. Create a new JS file with a name of your choice. I&#8217;ll name mine &#8220;led.js&#8221;.
</p>

<p style="text-align: justify;">
  This file will host our Node.js process that talks to the internet and accepts commands to control the LED.
</p>

<p style="text-align: justify;">
  Let us begin by controlling the LED state via Node.js. If you have already been through the pi-gpio documentation on npm or github, you already have a hang of it. But to keep things documented here I&#8217;ll cover it.
</p>

<pre lang="javascript">var gpio = require("pi-gpio");
var pin = 11; // use your pin number here

gpio.open(pin, "output", function(err) {  // configure pin as output
   gpio.write(pin, 1, function() {  // set pin to HIGH state
      gpio.close(pin);  // close pin access
   });
});
</pre>

<p style="text-align: justify;">
  Run this code
</p>

<p style="text-align: justify;">
  <code>node led.js</code>
</p>

<p style="text-align: justify;">
  The LED should turn on if your code runs successfully. With this in place, we are done with the hardware part of this DIY. All we need to do next is, create a channel of communication between this Node.js script and a Windows Phone.
</p>

<h4 style="text-align: justify;">
  Step III &#8211; Setting up Microsoft Azure Service Bus
</h4>

<p style="text-align: justify;">
  The easiest way to send a signal from a Windows Phone to the Node.js script running on your Pi is by creating a Node.js web server and accepting HTTP requests from the Windows Phone. But, this would need the Raspberry Pi to have a static IP. Even then, we wont be able to support use cases where the Pi is behind a router without doing some port forwarding. Our motive is to create a channel that can be easily accessible irrespective of the network hops between the Windows Phone and the Pi. Enter,<a href="http://azure.microsoft.com/en-us/documentation/services/service-bus/" target="_blank"> Azure&#8217;s Service Bus</a>. The Service Bus will act as a mediator for messages being sent between the Windows Phone and the Pi. The Service Bus offers a variety of messaging topologies, we&#8217;ll be using Topics for this particular use case.
</p>

<p style="text-align: justify;">
  Each device will be continuously listening to a channel subscription (<a href="http://azure.microsoft.com/en-us/documentation/articles/service-bus-dotnet-how-to-use-topics-subscriptions/" target="_blank">Service Bus Topic Subscription</a>) assigned to it. Any message transmitted to its topic on the Service Bus will then be available for the device subscription to read and then mark as read when the appropriate action has been taken on the message. Multiple devices can listen to a topic, but a message once read on a topic subscription by a device will not be available for other devices on the same subscription. You can picture a single topic subscription as a single queue. This is particularly useful when you are planning to have multiple devices who perform the same actions using a single topic subscription, and each checks for new messages based on its availability. Example: Consider a fleet of <a href="http://www.irobot.com/For-the-Home/Vacuum-Cleaning/Roomba" target="_blank">Roombas</a> assigned to mop the floor of a large shopping mall, while some Roombas might be already mopping a particular section, the idle ones can keep polling for new messages. The one that gets the next message is dispatched for cleaning, while others keep polling for new messages.
</p>

<p style="text-align: justify;">
  You can use the <a href="http://azure.microsoft.com/en-us/documentation/articles/service-bus-dotnet-how-to-use-topics-subscriptions/" target="_blank">Service Bus Topic Subscription documentation</a> to understand it better.
</p>

<p style="text-align: justify;">
  Another benefit of Service Bus is that it is a polyglot web service. You can communicate with Service Bus using SBMP (Service Bus Messaging Protocol), REST API or even AMQP, a lightweight Machine-to-Machine protocol that allows devices to use a single communication link to the web service to send and receive multiple messages.
</p>

<p style="text-align: justify;">
  Before we move to the Pi, create a service namespace on your azure account, and obtain the connection string for the namespace. Follow the documentation link above to create one.
</p>

<h4 style="text-align: justify;">
  Step IV &#8211; Connecting Raspberry Pi to Service Bus
</h4>

<p style="text-align: justify;">
  The <a href="https://github.com/Azure/azure-sdk-for-node" target="_blank">Node.js Azure SDK </a>has a Service Bus library which can be consumed from our Node.js on Pi. We will create a Topic named &#8216;lights&#8217; and create a subscription for the Pi to listen to. Before we start with that, lets download the azure npm package with the usual ritual in your project directory.
</p>

<pre>npm install azure</pre>

<p style="text-align: justify;">
   Let us start by creating a new JS file. I&#8217;ll name this &#8216;control.js&#8217;. This is where our final code will be. The following lines will check if there is a topic named &#8216;lights&#8217; on the service bus, if not it will create one. Don&#8217;t forget to place your namespace connection strings in the code below.
</p>

<pre lang="javascript">var azure = require('azure');
var gpio = require("pi-gpio");

var AZURE_SERVICEBUS_NAMESPACE = "name-space";
var AZURE_SERVICEBUS_ACCESS_KEY = "access-key";

var DEVICE_NAME = "rpi";
var topic = "lights";
var serviceBusService = azure.createServiceBusService(AZURE_SERVICEBUS_NAMESPACE,AZURE_SERVICEBUS_ACCESS_KEY);
var topicOptions = {
    MaxSizeInMegabytes: '5120',
    DefaultMessageTimeToLive: 'PT1M'
};

serviceBusService.createTopicIfNotExists(topic, topicOptions, function(error){
    if(!error){
        console.log("Topic created/exists.");
        
        serviceBusService.createSubscription(topic,DEVICE_NAME,function(error){
            if(!error){
                console.log("Subscription created...");
            }
            else {
                console.log(error);
            }
        }); 
    }
});

</pre>

The code above, also creates a subscription named &#8216;rpi&#8217;. Now this might work the first time, second run onwards the subscription name might throw an error on the console while we can safely ignore. The topic options are used to set maximum message size and TTL.

#### Step V &#8211; Connect a Windows Phone app to Service Bus

We will be using the Service Bus REST API for Windows Phone to send messages to the Service Bus. So you can easily map the same APIs onto any other mobile device of your choice. Start by creating a Windows Phone &#8216;Blank App&#8217; project in Visual Studio. On the MainPage.xaml paste the following XAML inside your parent grid.

<pre lang="xml">&lt;StackPanel VerticalAlignment="Center" HorizontalAlignment="Center"&gt;
            &lt;TextBlock Name="Status" Foreground="#333" FontSize="24" Margin="20"&gt;Fetching LED Status...&lt;/TextBlock&gt;
            &lt;Button Height="120" Width="120" Name="LightON" Content="ON" HorizontalAlignment="Center" Foreground="#333" BorderBrush="#333" Click="LightON_Click"&gt;&lt;/Button&gt;
            &lt;Button Height="120" Width="120" Name="LightOFF" Content="OFF" HorizontalAlignment="Center" Foreground="#333" BorderBrush="#333" Click="LightOFF_Click"&gt;&lt;/Button&gt;
            &lt;Button Height="120" Width="120" Visibility="Collapsed" Name="LightParty" Content="Party" HorizontalAlignment="Center" Foreground="#333" BorderBrush="#333" Click="LightParty_Click"&gt;&lt;/Button&gt;
        &lt;/StackPanel&gt;
        
        
</pre>

<p style="text-align: justify;">
  This will create two buttons on your UI. ON and OFF. Now your MainPage.xaml.cs should look something like this.
</p>

&nbsp;

<pre lang="csharp">public MainPage()
{
    this.InitializeComponent();

    this.NavigationCacheMode = NavigationCacheMode.Required;
    Loaded += MainPage_Loaded;
}

private void LightON_Click(object sender, RoutedEventArgs e)
{
     sb.SendMessage("lights","{\"status\": true}");
}

private void LightOFF_Click(object sender, RoutedEventArgs e)
{
     sb.SendMessage("lights", "{\"status\": false}");
}
</pre>

You will need to define the ServiceBus class and consume the REST API in there. Create a new class file, ServiceBus.cs. Now let us implement two methods in there. GetToken and SendMessage. GetToken will retreive an authentication token from Service Bus using the management credentials. This token will be used by the SendMessage method for every message being sent. Note, we will not be receiving any messages from Service Bus in Windows Phone. We will only send messages from the Phone and receive it on the Raspberry Pi.

&nbsp;

GetToken is called in the constructor itself, so that it need not be explicitly called unless the token has expired which usually expires after 30 minutes. With this in place, the Windows Phone app is ready!

&nbsp;