# Wake-On-LAN [![Travis Build Status](https://travis-ci.org/nikeee/wake-on-lan.svg?branch=master)](https://travis-ci.org/nikeee/wake-on-lan) [![Windows Build Status](https://ci.appveyor.com/api/projects/status/2t6mkn484amt0j4o?svg=true)](https://ci.appveyor.com/project/nikeee/wake-on-lan) [![NuGet version](https://img.shields.io/nuget/v/WakeOnLan.svg)](https://nuget.org/packages/WakeOnLan)

A simple library for sending magic packets and performing IP address operations. The default namespace is `System.Net`. There is an online documentation available [here](https://nikeee.github.io/wake-on-lan).

## Samples
### Sending a Magic Packet
This sample uses `00:11:22:33:44:55` as MAC address.

```C#
using System.Net;
// ...

// Using the IPAddess extension
IPAddress.Broadcast.SendWol(0x00, 0x11, 0x22, 0x33, 0x44, 0x55);

// via core MagicPacket class
var endPoint = new IPEndPoint(IPAddress.Broadcast, 7); // You don't have to use Broadcast.
                                                       // Every IP/port-combination is possible.
MagicPacket.Send(endPoint, 0x00, 0x11, 0x22, 0x33, 0x44, 0x55);

// via IPEndPoint extension
endPoint.SendWol(0x00, 0x11, 0x22, 0x33, 0x44, 0x55);

// ...
using System.Net.NetworkInformation;
PhysicalAddress.Parse("00-11-22-33-44-55").SendWol();
```


### Getting Subnet Information
You can also retrieve information about a subnet.
```C#
using System.Net;
using System.Net.Topology;
// ...

var someIp = new IPAddress(new byte[] { 192, 168, 1, 23 }); // Some IP address in the subnet
var mask = new NetMask(255, 255, 255, 0); // the network mask of the subnet

// CIDR-notation number of the network mask
int cidr = mask.Cidr;

var networkPrefix = someIp & mask; // bitwise operation to get the network address (192.168.1.0)
networkPrefix = someIp.GetNetworkPrefix(mask); // using the extension method for IPAddress

// retrieve broadcast address of the subnet (192.168.1.255)
var broadcastAddress = someIp.GetBroadcastAddress(mask);

IEnumerable<IPAddress> siblings = someIp.GetSiblings(mask, SiblingOptions.ExcludeUnusable);
// Enumerate through all IP addresses in the subnet, except network prefix and broadcast (RFC 950, 2^n-2)
foreach (IPAddress someIpInNetwork in siblings)
{
    Console.WriteLine(someIpInNetwork.ToString());
}

// Get number of possible siblings without someIp, broadcast and network prefix
int siblingCount = mask.GetSiblingCount(SiblingOptions.ExcludeAll);
```

### ARP Requests
To retrieve the MAC address of a host, there is a functionality for ARP-request built-in. It uses the Windows API method [SendArp](http://msdn.microsoft.com/en-us/library/windows/desktop/aa366358(v=vs.85).aspx).
```C#
ArpRequestResult res = ArpRequest.Send(someIp);
if(res.Exception != null)
{
    Console.WriteLine("ARP error occurred: " + res.Exception.Message);
}
else
{
    Console.WriteLine($"Host MAC address: {res.Address}");
}
```
Note that there isn't always an MAC address available although there is a host. The reason for this could be the host is offline and/or the physical address is not cached somewhere.
Also, this function uses a p/invoke. This might cause problems when used on platforms other than Windows.

### Async/Await
This library also supports the Task-based Asynchronous Pattern (TAP). Every method that can send a magic packet synchronously is available as a TAP method returning a `Task`.
```C#
await IPAddress.Broadcast.SendWolAsync(0x00, 0x11, 0x22, 0x33, 0x44, 0x55);
```

### Further Samples
The [System.Net.NetworkInformation.PhysicalAddress](http://msdn.microsoft.com/en-us/library/system.net.networkinformation.physicaladdress(v=vs.110).aspx) class is also supported as it represents a MAC address.
```C#
var mac = new PhysicalAddress(new byte[] {0x00, 0x11, 0x22, 0x33, 0x44, 0x55});
mac.SendWol(); // via extension method
```

### Install
Install the [NuGet package](https://nuget.org/packages/WakeOnLan) of this library:
```Shell
# NuGet CLI
Install-Package WakeOnLan
# dotnet CLI
dotnet add package WakeOnLAN
```

### Compatibility
Version `2+` will be available for .NET Standard only. Version 1 supports the following platforms:
- .NET 2.0 (does not include extension methods and async features)
- .NET 3.5 Client Profile (does not include async features)
- .NET 4.0 Client Profile (does not include async features)
- .NET 4.5
- .NET 4.5.1

To install a version `<2`, have a look at all the available versions of [the NuGet package](https://www.nuget.org/packages/WakeOnLan) and install a specific version. For example:
```Shell
# NuGet CLI
Install-Package WakeOnLAN -Version 1.6.0
# dotnet CLI
dotnet add package --version 1.6.0 WakeOnLAN
```
