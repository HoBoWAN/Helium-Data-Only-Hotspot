# Turn the RAKwireless WisGate Edge Lite 2 into a Helium Data-Only hotspot

Using a Data-Only Hotspot provides you with the ability to contribute to expanding the Helium network providing coverage to end-node devices at a relatively low price point, which is always good.

A Data-Only Helium hotspot is essentially a LoRaWAN gateway that relays data and consequently earns token rewards for its packet transfer services.

It should be mentioned that such a hotspot will not provide you with any Proof of Coverage (PoC) rewards, due to the fact that it cannot be challenged by a validator to send beacons or witness. This is not a full Helium hotspots and it will receive substantially less HNT as earnings.

In this article we will be discussing the transformation of a RAKwireless WisGate Edge Lite 2 (RAK7268) gateway into a Data-Only Helium Hotspot. The gateway in question comes at a very affordable price and is therefore an ideal point of entry for anyone looking to contribute to the expansion of The People's Network.

In order to convert the RAK7268 to a Data-only Hotspot we would need to have the following accomplished:

- Have the RAK7268 properly set up to work in your frequency region (EU868 in this example)
- Have the LoRa Packet Forwarder pointed to the right address and port (in this case the local loopback address)
- Install the Helium miner service so the packet forwarder can send packet data to it and in turn it can send it to the Helium back-end server

Once the above are all accounted for your gateway will be able to forward Data packets coming from an end-node registered with the [Helium Console](https://console.helium.com/welcome). We will have follow up articles on the topic.

## Setting up the gateway

Refer to the official [Quick Start Guide](https://docs.rakwireless.com/Product-Categories/WisGate/RAK7268/Quickstart/#rak7268-quick-start-guide) for detailed instructions on setting up the gateway, so you can connect it to your local network and in turn to the internet.

Before proceeding with the actual gateway setup, it is advisable to first upgrade the gateway with the latest available firmware, which you can find [here](https://downloads.rakwireless.com/LoRa/RAK7268/Firmware/).

Make sure you have it set up to work with the appropriate LoRa frequency band for your region (we are using EU868 for this example).

## Setting up the packet forwarder

There are two logical blocks that run on the Data-only Hotspot

- The LoRa Packet forwarder

  It takes the received data packets and forwards them to an end-point (server). In this case, instead of pointing it to a remote server it will be pointed internally to the local loopback address of the device, we want it to communicate to the miner service we will be installing.

- The Miner service

  This is what separate the Hotspot from a regular gateway, it receives the data-packets from the forwarder and sends them to the Helium servers, where one can see them in the Helium console if desired.

As the Miner service listens for data packets on port 1680 we need to configure this in the gateway settings together with the local loopback address (127.0.0.1)

Log into your gateway via the Web UI and head to the LoRa Network -> Network Settings section

Make sure to select "Packet Forwarder" for the Mode of operation in case you were using another. Click Switch, Save & Apply.

![SwitchPacketForwarder](.\images\SwitchPacketForwarder.png)

When the mode has been switched you need to make sure that you are using the following settings:

```
Server Address: 127.0.0.1
Server Port Up: 1680
Server Port Down: 1680
```

Update them in the respective fields (see example image below). Leave the rest of the fields with their default values. Save & Apply via the button at the end of the page.

![acketForwarderSettings](.\images\PacketForwarderSettings.png)

## Setting up the miner

When this process is completed, you can proceed and download the appropriate version of gateway-rs on your computer. This is the package file we are going to be installing for the Miner service.

You can download the gateway-rs package from the repository in [this](https://github.com/helium/gateway-rs/releases) link using your preferred browser.

The version that works with RAKwireless gateways ends in **24kec.ipk**. (note there are different file versions and types as they support gateways from multiple brands, make sure to get the right one.) At the time of writing the latest available version is [v1.0.0-alpha.26](https://github.com/helium/gateway-rs/releases/tag/v1.0.0-alpha.26).

The next step is to move the package file to the RAK7268. In this example we will be using the Windows PowerShell command line tool to move the gateway-rs file from a PC to the gateway in the case where we are directly connected to the gateway's AP (check out the  [Quick Start Guide](https://docs.rakwireless.com/Product-Categories/WisGate/RAK7268/Quickstart/#rak7268-quick-start-guide) for details on this method of connection). Navigate to the folder where you copied the package from the repot and execute the command

```
[~/Download] scp helium-gateway-v<version>-ramips_24kec.ipk root@192.168.230.1:/tmp
```

This will copy the file to the device with IP address 192.168.230.1 (address of the gateway in our case), in the "tmp" folder (meant for temporary files). The command will have the following output and will require you to enter the login password for the gateway for the transfer to start.

Output:

```
root@192.168.230.1's password:
helium-gateway-v1.0.0-alpha.27-ramips_24kec.ipk                         100% 1850KB 538.5KB/s   00:03
```

The next set of commands will help you establish an SSH connection to the gateway, move the installation file for the gateway-rs service to the /tmp folder, initiate the installation of the new gateway-rs service and finally delete the now obsolete installation package from the /tmp folder.

```
[~/Download] ssh root@192.168.230.1
[root@RAK7268:~#] opkg install /tmp/helium-gateway-v1.0.0-alpha.27-ramips_24kec.ipk 
[root@RAK7268:~#] rm /tmp/helium-gateway-v1.0.0-alpha.27-ramips_24kec.ipk
```

The installation process starts and makes sure that the new service will start automatically after reboot. Once the installation process is finalized, if you are not based in the USA, you will have to change the zone from US915 (default) to your current one by editing the "/etc/helium_gateway/settings.toml" file.

An example command that would allow you to edit the file in question is the following:

```
sudo vi /etc/helium_gateway/settings.toml
```

It is also a very good idea to deactivate the auto-update mechanism while editing the file, because the automatic updates are not officially supported at this point yet.

You must add your corresponding zone on top of the file, if you are based in Europe, the changes you would make to the file in question should look similar to the following:

```
region = "EU868"

[log]
...
[update]
enabled=false
```

Once the changes have been saved you should restart the "helium_gateway" service by using the command below:

```none
[root@RAK7268:~#] /etc/init.d/helium_gateway restart
```

If you would like to confirm that everything has gone smoothly so far, you should run the following command:

```
[root@RAK7268:~#] helium_gateway key info
```

Output:

```
{
  "name": "winning-opaque-bear",
  "key": "13WFBtm45fN7c9RjP7JisiqNsXyJDDRoMfn6JDfxoxW7QEywaVP",
  "onboarding": "13WFBtm45fN7c9RjP7JisiqNsXyJDDRoMfn6JDfxoxW7QEywaVP"
}
```

If you have received a name for the gateway and a public key address, you have a good reason to be happy, because you were successful.

Your gateway has now officially become a Helium Data-Only Hotspot.

In the next article we will be discussing the remaining steps, which will allow you to register the created hotspot on the Helium network. This will require you to have a Helium wallet as you will need to be able to cover the onboarding fee. Rest assured this will be worth it as this will allow you to earn rewards for transferring packet data.
