# RAFT-note
Some notes and thoughts about RAFT learning.

The Raft, which depends on **strong leader**, mainly focus on the quorum read/write to keep consistency in a distributed system.

---

## Basic RAFT
1. Leader Election 😕
   
   * How could a server know when to become follower ❓
     
     The key is the message's `Term` (, not the `Term` of log entries inside it).
     Once there's a higher message term, it means some abnormal state (server's downtime, network partition, etc) happened, so there will be a server coming up with a higher `Term` than other servers.
     **What should we do then?** Just let all servers who can still comunicate (and also who received that higher `Term` message) becomes *None*'s followers, which makes the cluster has no leader at present, and try to start a new term of election.
     
   * How could a server know who to follow ❓
     
     It's a bit different from the previous question.
     And this time the key is the log entry's `Term` and `Index` inside the vote and response message 👀.
     As long as we guarantee that server always votes and followes the owner of the more up-to-date log entry, we can guarantee that the cluster keep the most data's consistency.

     **NOTE:** about `up-to-date`, can see the extended paper's 5.4.1 section and the code of `isUpToDate`, which is really simple.
   
2. Log Replication 📚

   * The cluster will lose messages.
     
     Yeah, Raft cannot guarantee that every message is stable and correct.
     There will be conflicts, and the old conflicted data will be swept, and the new data will be appended.
     (At least in TinyKV and etcd, they're directly appended to the unstable storage)
  
   * Basic Raft is not `Linearizability`. It only guarantees the eventual consistency.

       Sadly, as there always exists time interval for comunication.
   
4. Safety

   * This section may be more sound like Bucket Effect, I guess? 🤔

     The lowest log entry term and index in the *sorted index array* means that other servers (who has higher matched index (, `Progress.Match` in code)) must have this lower index entry. And so we can commit the median of that sorted Match array to make log entry is really stored in cluster (quorum).

## Engineering

* TDD is really useful! 😄 The complete tests (unit tests, integration tests, etc) keep the correctness of program during refactoring and revamping (, which I belive limits the chaos evolution direction).
* Design layer by layer. But this needs more experience to conclude, I can hardly tell this idea clearly now. 😢
* KISS principle. I like it. 😃

## Links
* [my own tinyKV impl](https://github.com/AuruTus/tinyKV)
* [extended Raft paper](https://pdos.csail.mit.edu/6.824/papers/raft-extended.pdf)

  
