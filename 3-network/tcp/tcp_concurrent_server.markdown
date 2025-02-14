---
title: TCP Part 3 - Handle concurrent connections
parent: Computer Network
nav_order: 4
---
# Making our TCP server concurrent
The current TCP server implementation only allows one connection at a time. When there are multiple connections, each connection is processed sequentially.

To simulate multiple clients, we can use the process APIs to create multiple client processes, and establish one connection per process (WIP).