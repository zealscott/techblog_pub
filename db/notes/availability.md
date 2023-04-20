<p align="center">
  <b>High availability - fault tolerant</b>
</p>


# Useful link

- [Raft algorthem](https://raft.github.io/)
- [visualization of Raft](http://thesecretlivesofdata.com/raft/)
- [original paper](https://raft.github.io/raft.pdf)
- [zh_cn version](https://github.com/maemual/raft-zh_cn)

# work

- using **Redundancy and Replication** to reach fault tolerant

## Raft

- key point
  - leader 
    - control the whole process, interact with clients
  - candidate
    - if cannot receive heartbeat from leader, wait for a **random timeout** to be cadidate
  - follower

### Leader election
- **two timeout**
  - election timeout
    - is the amount of time a follower waits until becoming a candidate.
    - then it assumes there is no viable leader and begins an election to choose a new leader
    - randomized to be between 150ms and 300ms.
    - After the election timeout the follower becomes a candidate and starts a new election term
    - sends out Request Vote messages to other nodes.
  - heartbeat timeout
    - leader sends heartbeat messages to all of the other servers to establish its authority and prevent new elections.

- **Leader**
  - once get majority votes, leader begins sending out *Append Entries* messages to its followers.
    - send in intervals specified by the heartbeat timeout
  - if node cannot receive heartbeat, then become a candidate and re-election
  - Requiring a majority of votes guarantees that only one leader can be elected per term.

- **split vote**
  - if two or more candidates receive same votes
  - The nodes will wait for a new election and try again.


### log replicate

- This is done by using the same *Append Entries* message that was used for heartbeats.
- committed
  - The leader decides when it is safe to apply a log entry to the state machines; such an entry is called ***committed***. 
  - Raft guarantees that committed entries are durable and will eventually be executed by all of the available state machines.
- once new entry committed
  - leader executes command in its state machine ,return result to client
  - leader notifies followers of committed entries in subsequent *AppendEntries* RPCs
  - followers execute committed commands in their state machines