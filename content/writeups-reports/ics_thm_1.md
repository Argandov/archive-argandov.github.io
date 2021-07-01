---
title: "Tryhackme writeup: Attacking ICS #1"
date: 2020-05-09T21:06:05-06:00
draft: false
authorbox: false
toc: true
thumbnail: /writeups-reports/files/ics_thumbnail_1.PNG
---

This room covers an introduction to ICS / SCADA controls, and some manipulation that can be done remotely. Starting with a virtuaplant appliance, and using python scripts to send information via Modbus TCP to retrieve register states. <!--more--> We'll also take a closer look at the protocol as it flows across the network, and some additional activities for dynamic communication with the remote server.

## Initial information about this room and the SCADA system

Heading with a browser directly with the IP of the ICS server, we can observe the SCADA system functioning with the help of VNC.

![Virtuaplant normal function](/writeups-reports/files/ics-virtualplant-initial.png)

We can also restart the process with ESC key in case something happens (It will).

The simulation is representing a bottle-filling plant, with the following elements:

- The containers (bottles),
- The nozzle which will fill the bottles with liquid,
- Not visually represented as a figure, but the presence of a carrier or "roller" is also taking part in the process,
- A sensor (green), acting as a cue for the system when the bottle is in place so it can stop the roller and open the nozzle,
- A sensor (red), acting as a liquid level cue, which, when triggered, will send a signal to the system so the the nozzle valve can close and the "roller" starts again, arriving to the next stage, repeating the process again.[^2]

So the phases in this process are as follows:

- Start the system
- Nozzle closed, roller activated (transporting the bottles)
- Roller stopped, nozzle open (Filling the bottles)

Let's not forget that at the start of the process, there is an actuator "cueing" the system to start working with the designed logic. Probably achieved with a killswitch, or some kind of manual & automatic means of control.

## Interacting with SCADA using python

For the next phases we will need the python library "Pymodbus" (https://pymodbus.readthedocs.io/en/latest/index.html), which will allow us to make changes to the registers in the remote SCADA system.

We are provided with a series of scripts to manipulate the system:

![directory - python scripts](/writeups-reports/files/ics-files.png)

A little familiarization with the pymodbus library, and we soon discover all this scripts do some simple functions. Mainly:

- write_register
- read_holding_registers
- ModbusTcpClient
- registers

We'll start using discovery.py, which works by reading the registers (read_holding_registers), and the script is designed to hold an argument which will be the IP address of the remote system. After running it, we continually receive the "states" of the registers, with 1 second pause after the loop starts again:

![discovery.py](/writeups-reports/files/ics-discovery.png)

Since we are reading the system registers in real time, the value changes become apparent as the actuators & sensors interact on the system.

- The first 2 rows are the initial state of the system (nozzle closed, roller active), or stage 1
- The next 3 rows are the stage 2 of the process: Nozzle open, roller inactive (Bottle filling)
- The last 3 rows are the stage 1 again, or stage 1'.

Dissecting our python script:

- ```client = ModbusClient(ip, port=502)```
	Will initiate the connection to our remote system (ip "grabs" the ```argv[1]``` which will be the IP address), to port 502, which is the default for Modbus protocol (This can be changed)
- ```rr = client.read_holding_registers(1, 16)```
	As the name of the function implies, we will be reading the holding registers. The first argument taken (1), is the start of our reading, and the second (16) instructs the function as to how many registers we want to be recorded, starting from our first instruction (The first register). That's the reason we have 16 registers in the stdout of our script. (If we inverse the args, for example, to "16, 1", we would only record the 16th register).

## Taking a closer look at the Modbus TCP protocol with wireshark

![wireshark modbus general](/writeups-reports/files/ics-pcap.png)

In our script, we wait 1 second with the function time.sleep(1), before sending again a new request for read_holding_registers, so the timestamps in our pcap help making sense of this initial investigation.[^1]

The initial TCP packets (SYN-SYN/ACK-ACK) are now apparent before initiating the Modbus TCP communication. Here we see clearly our query, followed by another ACK TCP response, before the Modbus response for our query and the next ACK response. The following packets are just the same communication our python loop created.

Double-click on both the query and the response, we get to the following screen in **wireshark**:

![wireshark - modbus](/writeups-reports/files/ics-modbus-pcap.png)

In the query Modbus packet (left), we see clearly the Function code as 3 (0011), indicating our queue to "reading holding registers". On the right, we see the response, with our registers.

![modbus query/response wireshark](/writeups-reports/files/ics-modbus-pcap.png)

*Note: this was a second test, and the 16th byte was 2 instead of our initial 1. This can serve as a reminder that we're not dealing with digital or binary information, but with 16bit values.*[^3]

Making sense of the registers, and what values they indicate:

Following some logic in the system's flow, we agree on the following information (I ordered them as they appear in our discovery):

- The 16th byte or registry is the state of the system itself (Being constantly "on" or > 0)
- The 3rd registry is the state of the roller,
- The 1st registry is the state of the nozzle,
- The 2nd registry is the level sensor,
- The 4th registry is the nozzle actuator.

Now we're more familiar with how the system works, and can glance at the flow's logic.

Let's start actually modifying the registers

From the exercise information:

	Modbus register types:
	    Discrete Input (Status Input): 1bit, RO
	    Coil (Discrete Output): 1bit, R/W
	    Input Register: 16bit, RO
	    Holding Register: 16bit R/W

The holding registers are **both readable and writable**

Dissecting the next python script, we see the functions and the way they work are pretty intuitive:

From **set_registry.py**:

```write_register(registry, value)```

The way this short script was designed, makes the function even more clear. We can now being sending ```write_register``` information to the remote system, with our desired "state" for each register.

If we, for example, send a 0 to the 16th byte (The state of the system itself), we would assume it will stop it completely. And that's exactly what it happens. This would be accomplished by modifying the arguments to ```write_register``` to (16, 0).

For further manipulation of the system, we already know what the other registers do, and we can control the flow of the whole system, one byte at a time. What we can do right away is modifying the nozzle values as well as the roller.

After some testing, the sensors' state didn't last for too long, so I assume they are maybe mechanical rollers, so they are "stateful". This means they react to physical information, such as pressure, velocity, etc. But keep their state until changed again. This is important, because as soon as we stop sending a change in their value, they take their original state almost immediately. This would make it tricky for manipulation, and some fixing of the scripts would be needed to "restrain" or keep our actuators from "grabbing" their real state.

For abusing sensor registries, it takes more logic in the flow of the script. However, any manipulation can be done at this point, and we have full control of the flow of the system, even breaking the logic it was intended to work in.

![virtuaplant modified state](/writeups-reports/files/ics-virtualplant-modified.png)

Whoops.

That's it for this introduction to ICS / SCADA systems. We now have more information about how this process works, and begin wondering how can we secure this systems.


[^1]: [You can download the .pcap file *here](/writeups-reports/files/discovery.pcap). It only contains the first queries for the register reading. *(All the original info from the lab machine was modified in the pcap)*
[^2]: How sensors and actuators actually produce the signals we are reading: https://electrical-engineering-portal.com/sensors-actuators-work-rtus-scada
[^3]: More about the Modbus Protocol: https://www.dpstele.com/modbus/index.php
