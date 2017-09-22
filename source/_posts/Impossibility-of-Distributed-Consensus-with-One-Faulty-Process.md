---
title: Impossibility of Distributed Consensus with One Faulty Process
date: 2017-09-22 13:17:11
categories: Readings
tags:
    - Distributed Systems
    - PODS
    - Classic Papers
    - Reading Notes
---

Recently I started reading the most classical papers in the field of distributed systems.
The first paper I read was *Impossibility of Distributed Consensus with One Faulty Process*.
This paper, published in April 1985 by Fischer, Lynch and Patterson, contributes the community
with the famous *FLP impossibility result*, which placed an upper bound on distributed consensus
in an asynchronous environment.

The proof itself was not easy to follow, but there are already explanations from others all
around, so I would focus on my own reflects on the paper, not on the proof itself.
For a detailed explanation of the proof, I heartily recommend
(this blog post)[http://the-paper-trail.org/blog/a-brief-tour-of-flp-impossibility/].

<!-- more -->

# On the System Model

The paper points out that there isn't a single consensus protocol with which a distributed
system is guaranteed to reach agreement even with only one faulty process.

To do so, the paper first specifies a model for the system. In the model, the system is asynchronous,
meaning that the messages can be arbitrarily delayed, and the clocks of different processes can drift apart^1^.
Also, the only type of failure that would happen in the system is process crashes and the faulty
processes will just stop (i.e. won't generate random responses or behave incorrectly) in such cases.
Messages are guaranteed to be delivered correctly and exactly once (though the messages may be delayed).

Now, a careful reader might have wondered whether the model is practical: on one hand, being totally
asynchronous, the model models the weakest distributed setting (correct me if I'm wrong); on the other hand,
however, the fail-stop failures are the easiest^2^ to solve among all the failure types; the assumptions made on
the message system seem strange too. As the model would determine the applicability of the conclusion, what
does *FLP impossibility result* say indeed? In fact, the strong assumptions made on the failure model
and the message system help generalize the result. By our intuition, with more severe and harder to solve failures,
it is surely harder for the system to reach consensus, right? Thus, the result actually
says that there exist no totally correct consensus protocols for all totally asynchronous systems.


# On the 0-1 Consensus Problem

The proving process would not have been so succinct if it had been done without the simplification of the
0-1 consensus problem. In the paper, the 0-1 consensus problem is defined to more accurately specify the
system setting, and, without loss of generality, simplifies the problem:
the input and output of each process are now binary. The simplification is so clever that many subsequent
works have adopted this simplification in their own proofs.

Together with the definition of the consensus problem, the paper also defines a set of terminologies that
would be used in the proof. These terminologies help describe the proof, too.


# On the FLP Impossibility Result

The result, however, only shows the existence of situations where an agreement can not be reached in
such settings; it does not tell us how often such situations will occur in practical systems.
Thus to some extent the result just reminds you the possibility when your system runs into a non-deciding state.

Nevertheless, the result proved very influential in the theoretical distributed system community.
It spawns a lot of works on many topics, like failure detectors, impossibility of consensus in other settings, etc.
Therefore, in 2001, it won the Dijkstra Prize for being one of the most influential papers in the field of
distributed systems^3^.


# References

1. (What is Synchronous and Asynchronous in Distributed Systems)[https://www.quora.com/What-is-synchronous-and-asynchrounous-in-distributed-systems]
2. (Wikipedia page on Byzantine failures)[https://en.wikipedia.org/wiki/Byzantine_fault_tolerance]
3. (Dijkstra Prize in Distributed Computing)[https://www.podc.org/dijkstra/]
