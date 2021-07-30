## Introduction

The performance of large scale distributed systems depends critically on readability and scalability of membership protocol

Membership protocol provide a list of non-faulty process in the group. The membership protocol insures that list of members is update when one member leave or enter the group.

Performance metrics of membership protocol:

- How fast the membership changes are identified
- Rate of false positive(when process is market as unavailable but it is available) detection

Different ideas on implementation of membership protocol:

- Sending all hearbeats to a central server leads to hot-spot creation
- Sending hearbeats to all members leads to quadratically growth of message in the network → leads to load of network

SWIM details:

- imposes constant message load per group member
- detects a process failure in a constant time
- provides a deterministics bound on the local time that a non-faulty process takes to detect failure of another process
- propagates membership updates, information about infection-style
- provides mechanism to reduce the rate of false positives by suspecting a process before declaring it as failed within the group

## SWIM basic protocol

1. Select a random member Mj
2. Send ping message to Mj and wait T.

    2.1 .If ack then we have info about Mj

    2.2 If not ack in time period T, select other random member and ask info about Mj