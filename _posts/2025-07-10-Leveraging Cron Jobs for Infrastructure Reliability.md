---
title: "Leveraging Cron Jobs for Infrastructure Reliability"
date: 2025-06-15 08:42:00 -500
categories: [StreamYard, Crons, Infra]
tags: [production, backend, my journey, optimization]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/OrphanedRooms/orphaned-rooms.png"
---

As I reminisced about my time so far at Bending Spoons, I realized I never got around to writing an article about my first real task in StreamYard, back when I was working in the Platform team (infrastructure and backend). To this day it remains one of my favorite, and of the most interesting opportunities to learned I have been presented.


# Heartbeat and Orphaned Rooms Management

## Overview

In StreamYard's infra, we have something we call the "heartbeat" service - it is essentially a cron job that runs approximately every 20 minutes and monitors the health of pods. It then handles "orphaned rooms".

The entity in which we use to store infra data for a studio or broadcast is called a "room". For example, this will have the `hostIp`, which is the IP address of the node in which the broadcast will be hosted. We call it an "orphaned room" when a room either:
A. Has a `hostIp` which doesn't correspond to any node, i.e. the node doesn't exist (orphaned room of type `noNode`), or
B. Has a `hostIp` that does correspond to the correct node, but the node doesn't have that `roomId` listed in its rooms (orphaned room of type `nodeNotAware`).

And you may be asking yourself how a room can even become orphaned. There are a few main reasons:

1. Network Connectivity Issues: 
Like connection timeouts, since the health check has a 2000ms timeout, or connection refused errors.

2. Pod Health Degradation:
Mainly when a pod's health score reaches 0, which happens after multiple failed health checks, or if the pod becomes unresponsive to HTTP health check requests.

3. Infrastructure Problems:
It could happen that the pod IP address becomes unreachable, which would not be a problem on our side but rather our server provider. Internally, our API endpoint could also become inaccessible due to errors. Or we could have node failures in the Kubernetes cluster.

4. Resource Exhaustion:
Since pods aren't always stable or reliable, it can happen that they become overloaded and stops responding to health checks, crash due to memory or CPU limits, or get evicted by Kubernetes due to resource exhaustion.

5. Cluster Operations:
This is mainly something that can happen when we release new code, realize it has an issue, and have to drain the nodes that had already accessed the new code. But also during cluster downsizing operations, maintenance or updates.


When such an event happens, this is very disruptive for the user, as either they will be unable to continue the boradcast, or the recording can be lost. I was tasked with finding a way to deal with identifying and reassigning such rooms, and I found the heartbeat to be the perfect place for this, as a cron job was in place and it already dealt with node data and health, and it detects and manages pods that become unresponsive.

## Core Components

The heartbeat service consists of several key components:

1. **Patient Monitoring System**
2. **Health Check Mechanism**
3. **Pod Management**
4. **Cluster-aware Decision Making**

## Implementation Details

### 1. Health Check Cycle

The service runs health checks on a regular interval (approximately every 20 minutes with some randomization) for each pod. The check interval is calculated as:

```typescript
const next = () => setTimeout(
    checkHealth,
    heartbeatPeriod * 0.9 + Math.random() * heartbeatPeriod * 0.2
);
```

This randomization helps prevent thundering herd problems by distributing health checks across time.

### 2. Health Tracking

Each pod (called a "patient") maintains a health score that starts at `totalHealth` and decrements when health checks fail. The health check makes an HTTP request to the pod's health endpoint:

```typescript
healthCheck: (podIp: string) => request({
    method: 'GET',
    uri: `http://${podIp}:${INTERNAL_API_PORT}/healthcheck`,
    maxAttempts: 1,
    timeout: 2000
})
```

### 3. Orphaned Room Detection

In pseudo code, we:

- fetch the `roomId` of the Room entity,
- find the corresponding Node entity. 
- If no such node, return `noNode` as orphaned room type.
- Else, check that the `hostIp` from Room is the `rooms` array in Node.
- If not, return `nodeNotAware` as the orphaned room type.
- Otherwise, return.

### 4. Orphaned Room Reassignment

Once we have a list with the orphaned rooms, we run:

```typescript
const checkOrphanedRooms = async () => {
    const orphanedRooms = await findOrphanedRooms({ nodeType });

    // Safety threshold check
    if (orphanedRooms.length > orphanedRoomsThreshold) {
        logger.error(
            'Too many orphaned rooms detected. Skipping reassignment for safety',
            {
                orphanedRoomsCount: orphanedRooms.length,
                threshold: orphanedRoomsThreshold,
                nodeType,
            }
        );
        return;
    }

    // Reassign each orphaned room
    await Promise.allSettled(
        orphanedRooms.map(async room => {
            await reassignOrphanedRoom(room.roomId);
            logger.info('Successfully reassigned orphaned room', {
                room,
                nodeType,
                orphanType: room.orphanType,
            });
        })
    );
};
```

where:
```typescript
reassignOrphanedRoom: (roomId: string): Promise<void> =>
    request({
        method: 'POST',
        uri: `${VIDEO_API_ENDPOINT}/rooms/${roomId}/roomreassignment`,
        timeout: 12000,
        qs: { force: true },  // Forces reassignment even if room has existing assignment
    })
```
but with validation of whether the room was just reassigned elsewhere to avoid race condiitons.

We use the threshold check to avoid reassigning too many orphaned rooms at the same time, as this can cause errors. But most importantly, we don't expect to have more than 5 orphaned rooms per hour, even if we are having issues. So if we do have a number of orphaned rooms above the threshold, probably it is beacause we are having a "mass death" of pods, so reassigning many rooms may actually make matters worse.


## Monitoring and Metrics

I then added the necessary metrics and created a Grafana board to be able to track how many orphaned rooms we have, and see when they have been reassigned. Also, I created critical alerts for the above case when we surpass the orphaned room threshold.

```typescript
// First, set all counters to 0 if no orphaned rooms
if (orphanedRooms.length === 0) {
    ['noNode', 'nodeNotAware'].forEach(orphanType => {
        ORPHANED_ROOMS.set({ nodeType, orphanType }, 0);
    });
}

// If there are orphaned rooms, count them by type
const counts = orphanedRooms.reduce((acc, room) => {
    acc[room.orphanType] = (acc[room.orphanType] || 0) + 1;
    return acc;
}, {} as Record<OrphanType, number>);

// Set the metrics for each type
['noNode', 'nodeNotAware'].forEach(orphanType => {
    ORPHANED_ROOMS.set({ nodeType, orphanType }, counts[orphanType] ?? 0);
});
```

## Conclusion

The heartbeat service's orphaned rooms management is a robust system that provided reliable pod health monitoring, and I enhanced it to also identify and reassign orphaned rooms. This has helped make StreamYard's infrastructure more reliable and to handle errors more gracefully.

