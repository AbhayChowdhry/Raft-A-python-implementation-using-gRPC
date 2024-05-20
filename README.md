# Raft-A-python-implementation-using-gRPC
Implemented a modification with leader-lease 


## Leader Lease

It is possible to rely on a time-based “lease” for Raft leadership that gets propagated through the heartbeat mechanism. And if we have well-behaved clocks, we can obtain linearizable reads without paying a round-trip latency penalty. This is achieved using the concept of Leases.


#### What is a Leader Lease?

Leases can be considered as tokens that are valid for a certain period of time, known as lease duration. A node can serve read and write requests from the client if and only if the node has this valid token/lease.

In Raft, it is possible for more than one leader to exist (which is why even read requests require a quorum to fetch the latest value), but leases are designed in such a way that, at a time, only one lease can exist. Only leader nodes are allowed to acquire a lease; therefore, the lease is known as the leader lease.


#### Behavior of Leader & Follower

The leader acquires and renews the lease using its heartbeat mechanism. When the leader has acquired/renewed its lease, it starts a countdown (lease duration). If the leader is unable to renew its lease within this countdown, it needs to step down from being the leader.

The leader also propagates the end time of the acquired lease in its heartbeat. All the follower nodes keep track of this leader lease timeout and use this information in the next election process.


#### Leader Election

During a leader election, a voter must propagate the old leader’s lease timeout known to that voter to the new candidate it is voting for. Upon receiving a majority of votes, the new leader must wait out the longest old leader’s lease duration before acquiring its lease. The old leader steps down and no longer functions as a leader upon the expiry of its leader lease.


## Edge Cases handled

0. Start the cluster with 5 nodes. Wait for the leader to be elected and a NO-OP entry to be appended in all the logs.


#### Leader Election


1. Have the client perform 3 set requests and then 3 get requests

Verify if the set request has been replicated in each log.

Get requests should be returned immediately without any additional heartbeats by the leader.


#### Log Replication, Basic Functionality


2. Terminate one or two of the follower nodes and perform 3 SET requests and 3 GET requests. Restart the terminated followers.


Verify if the terminated follower rejoins the cluster with updated logs and state.

#### Fault Tolerance, Follower Catch-up


3. Terminate the current leader process. Wait for the new leader to be elected & perform 2 set and 2 get requests. Restart the terminated (old leader) process.


Verify if a new leader is elected & cluster continues operation without data loss in the logs. Also, ensure the old leader node should join the cluster as a follower with an updated state.

#### Fault Tolerance, Leader Failure, Leader Election, Log Replication


4. Terminate 3 (majority) follower nodes, and before the lease times out, send a Get request to the leader. Also, observe if the leader’s lease times out after some time and the leader steps down to become a follower node.


The leader should be able to return the correct value for the GET request if it's within the lease interval. Timing is crucial, and the dump.txt can verify if the request was received timely. The lease should also time-out since the leader can't reach any of the follower nodes.

#### O(1) Read Request, Leader Lease Timeout


5. Terminate all the nodes except two follower nodes, send a Get and Set request to any of the followers.

