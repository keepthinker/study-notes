# Network interface controller
## Introduction
A computer hardware component that connects a computer to a computer network.

Modern network interface controllers offer advanced features such as **interrupt and DMA interfaces** to the host processors, support for **multiple receive and transmit queues**, partitioning into **multiple logical interface**s, and **on-controller network traffic processing** such as the TCP offload engine.

The NIC is both a **physical layer** and **data link layer** device, as it provides physical access to a networking medium and, for IEEE 802 and similar networks, provides a low-level addressing system through the use of MAC addresses that are uniquely assigned to network interfaces.

## Indicate the availability of packets to transfer
The NIC may use one or more of the following techniques to indicate the availability of packets to transfer:

Polling is where the CPU examines the status of the peripheral under program control.
Interrupt-driven I/O is where the peripheral alerts the CPU that it is ready to transfer data.

## Transfer packet data
NICs may use one or more of the following techniques to transfer packet data:

Programmed input/output is where the CPU moves the data to or from the NIC to memory.
Direct memory access (DMA) is where some other device other than the CPU assumes control of the system bus to move data to or from the NIC to memory. This removes load from the CPU but requires more logic on the card. In addition, a packet buffer on the NIC may not be required and latency can be reduced.

## Multiple transmit and receive queues
**Multiqueue NICs** provide **multiple transmit and receive queues**, allowing packets received by the NIC to be assigned to one of its receive queues. Each receive queue is assigned to a separate interrupt; by routing each of those interrupts to different CPUs/cores, processing of the interrupt requests triggered by the network traffic received by a single NIC can be distributed among multiple cores, bringing additional performance improvements in interrupt handling.