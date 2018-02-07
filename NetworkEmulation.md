Network Emulation
=================


As part of Unity's Networking feature set, you can choose to emulate slower internet connection speeds to test out your game experience for users in low-bandwidth areas.

To enable Network emulation, go to __Edit-&gt;Network Emulation__, and choose your desired connection speed emulation.


![Enabling Network Emulation](../uploads/Main/NetworkEmulationMenu.jpg) 

Technical Details
-----------------


Network emulation delays the sending of packets in networking traffic for the Network and NetworkView classes. The ping is artificially inflated for all options, the inflation value increasing as emulated connection speed gets slower. On the __Dial-Up__ setting, packet dropping and variance is also introduced to simulate the worst possible connection ever. Emulation will persist whether you are serving the role of Server or Client.

Network emulation only affects the Network and NetworkView classes, and will not alter or emulate specialized networking code written using .NET sockets.
