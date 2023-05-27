# Building a Decentralized Trust Network
This project tries to provide a foundation for creating a decentralized trust network. Our objective is to build a starting point for users, community administrators, and developers to understand and implement trust networks.

This project is open-source and non-commercial, consisting of these components:  
1. A trust network protocol
2. A library implementing this protocol
3. An optional server companion app
4. A user-friendly authenticator web app

## Overview {#overview}
Trust networks are founded on mutual trust and reputation, with individuals depending on the recommendations and referrals of their trusted connections. Decentralized systems, on the other hand, are networks without a central authority. They provide enhanced security, privacy, and anonymity compared to client-server models.

Our implementation is currently in the internal testing phase, and we aim to announce its launch on our website, labdqnt.com, in the upcoming weeks.

The goal of this network is to strike a balance between anonymity, accountability, and the right to be forgotten. Through a decentralized system, we aim to offer a more secure and private way for individuals to connect and share information within their trusted networks.
## Introduction to the Protocol {#introduction-to-the-protocol}
The protocol can run over any type of underlying layer and is designed to be agnostic and flexible about the exact implementations of some rules. This intentional design promotes different trust metrics. In some instances, separate networks may negotiate (outside the protocol) a convergence rule or metric. In other situations, the two networks may simply disconnect.

The trust metric within the protocol stipulates that:
1. Greater distance between nodes implies less trust.
2. More connections with different nodes indicate more trust.
3. Nodes that are trusted are more likely to be trusted by others.
4. Nodes with numerous connections have less individual influence on the trustworthiness of their neighbors.

Similar to Google's PageRank algorithm, our trust metric is recursive and designed to converge to a set of stable trust values, given a certain minimum degree of connections.

For those interested in the mathematical underpinnings of our trust metric's convergence, we direct you to the well-established convergence proof for Google's PageRank algorithm.  While our trust metric shares fundamental similarities with PageRank, it also has unique elements tailored for trust networks. These differences, however, do not impact the convergence property and thus, the referenced PageRank convergence proof remains applicable. 

However, without a central point to compute and store these rankings, we need to introduce new mechanisms such as ID space and routing, borrowed from the DHT protocol. We'll define these terms as we explain their functionality.

Let's begin by exploring how users and community admins can leverage this network, presenting an overview of how the protocol operates at each stage. Afterward, we'll delve deeper into the protocol definition and mathematical formulas. Once we establish that definition, we'll present two use cases to help illustrate the concept. Finally we'll also discuss potential challenges and vulnerabilities this system might face. 
## Utilizing the Network {#utilizing-the-network}
### User Perspective {#user-perspective}
People use the network to demonstrate their trustworthiness within a community. They can enhance their trust by forming first-degree connections with people they know and trust.

While revealing their identity to their trusted nodes, they limit disclosure beyond that circle.

In the course of an interaction with another individual, if a user encounters any disruptive behavior, they can report it immediately within the network. Because identities beyond immediate connections are not fully known, this in-the-moment feedback is critical. It allows users to highlight an issue linked to an interaction without needing to know the other party's identity, ensuring the trust system remains responsive and dynamic. 

They are held accountable when they do not meet a certain trust threshold during the authorization process within a community.

They can exercise the right to be forgotten by simply creating a new profile, thus starting from a clean slate, but with zero trust.
### Community Administrator Perspective {#community-administrator-perspective}
Community administrators use the network to evaluate the trustworthiness of their users. They do this by providing users with a QR code, which the user scans to confirm their identity.

Administrators can also offer their users an interface to anonymously report disruptive behavior or to give negative feedback about other users through the network.

Furthermore, administrators provide a way for users to report disruptive behavior in real time, during the occurrence of an interaction. This mechanism is especially important given the semi-anonymous nature of the network, where a user might not know the identity of the individual they're interacting with. This allows for immediate insights into community dynamics and enables administrators to take timely action when issues emerge.
## Joining the Network {#joining-the-network}
This trust network operates on an invite-only basis. To join the network, a user must receive an invitation, outside of the protocol, from someone they know. Therefore, the identity of each node is known only to their first-degree connections. Following this, the network only synchronizes node information, identifying each node by a self-generated ID.
Users can generate and distribute invitations via the web app.
## From Centralized Graph Ranking to Decentralized Trust Network {#from-centralized-graph-ranking-to-decentralized-trust-network}
### Glossary {#glossary}
Before we delve into the definition of our trust network, let's establish some preliminary concepts:
1. Node: A node simultaneously represents a user or community profile and an instance of the client along with its associated IP address.
2. Node ID: Upon joining the network, the node generates a unique ID that identifies it within the network. The node must store this ID to easily restore its user profile later.
3. ID embedded space: To facilitate operations on nodes, we embed them in an XOR distance space by mapping the node ID to a point in this space.
4. XOR: The use of XOR as the distance function has several desirable properties, including the ability to efficiently locate the node responsible for a given ID using a logarithmic number of hops during the routing process.
5. Routing: XOR-based routing involves calculating the XOR distance between the node's ID and the target's ID, then forwarding the request to the node closest to the target in terms of XOR distance. This process is recursively repeated until the target node is reached.
6. First-degree trusted node: Also referred to as first-degree nodes or trust nodes, these are individuals you know and have either invited or received an invitation from. They form a subset of all the nodes that a given node becomes aware of through the node replication process. Other nodes that are replicated across the network are simply referred to as nodes.
### Constructing our Network {#constructing-our-network}
Now, let's delve into the task at hand. We have a perfectly suitable algorithm to rank nodes in a graph, essentially mirroring the principles of Google's PageRank algorithm. This algorithm is defined by the four simple rules we presented in the introduction to the protocol. Our task now is to transform it into a decentralized trust network.

Given the nature of decentralized systems and the vast amounts of data they can store, a node only has knowledge of a small portion of this data. To calculate the rank of each node, we need to distribute pieces of that work and propagate the resulting data. Then, we need to retrieve this data.

To efficiently distribute the work, we first need to perform some maintenance on the nodes and the paths that our node knows. Let's start by placing the node IDs in an ID space and define XOR as a distance metric in this space. Nodes can then participate in a node exchange process where they share other nodes' IDs along with their calculated trust and all the paths they have to them. Each node can then prioritize understanding its surroundings better.

After we've organized this space and defined the basic node exchange rules for nodes and paths, we need to maintain the paths, which are lists of nodes linking this node to other nodes.

During the node exchange process, the node ends up with some redundant routes. A route is a sequential partition of a path. So, the node discards routes that have been shortcut and replaces these parts with the new shorter route sequences.

Now that the node has a good understanding of its surroundings and a mechanism to maintain their paths, it can be responsible for a fraction of the calculation. That is, they will have a probability of calculating the trust of other nodes proportional to the XOR distance between themselves and those nodes. They will also have the ability to propagate and receive more nodes, their trusts, and paths from their established connections.

We now have a network of first-degree nodes that have calculated the trust and exchanged connections with each other. The only thing missing now is a way for two nodes that don't share a connection to assess the trustworthiness of each other. The use case here involves a well-known node A (for example, some community) attempting to assess the trustworthiness of an unknown node B to authorize their access.

To evaluate the trustworthiness of node B, node A searches for the path with the highest trust linking the two nodes. This search is guided by the XOR distance in the ID space, making successive hops to minimize the XOR distance at each step, as described in the introduction to this section.

Setting aside the obvious lack of mathematical definitions, we now have, at least conceptually, a fully operational trust network with respectable protections for anonymity and enforcement of accountability, as well as a mechanism for starting anew. Now, let's delve into those missing definitions.
## Technical Definitions {#technical-definitions}
### Node Exchange {#node-exchange}
Let's define our set of nodes in the network as \$N = {n_1, n_2, \ldots, n_m}\$, where \$m\$ is the total number of nodes. Each node \$n_i\$ in \$N\$ has a unique identifier \$ID(n_i)\$, and maintains a list of known nodes \$K(n_i)\$ and their associated paths \$P(n_i)\$.
The Node Exchange process between two nodes \$n_i\$ and \$n_j\$ can be represented as:

$$
\begin{align*} 
\text{Exchange}(n_i, n_j) : \{  \\
n_j \leftarrow [ID(n_i), K(n_i), P(n_i)],  \\
n_i \leftarrow [ID(n_j), K(n_j), P(n_j)]  
\} 
\end{align*}
$$

After the exchange, each node updates its known nodes and paths:

$$
\begin{align*} 
\text{Update}(n_i, n_j) : \{ 
K(n_i) \leftarrow K(n_i) \cup K(n_j), \\
P(n_i) \leftarrow P(n_i) \cup P(n_j) \}
\end{align*}
$$

The trust values are updated based on a Trust Metric, which we define next.
### Trust Metric {#trust-metric}
The Trust Metric in this network is driven by four rules and can be defined mathematically as follows:

Let \$d(n_i, n_j)\$ denote the distance between nodes \$n_i\$ and \$n_j\$, \$C(n_i)\$ denote the number of connections of node \$n_i\$, and \$T(n_i)\$ denote the trust value of node \$n_i\$.

$$
\begin{align*} 
T(n_i) = \frac{f(C(n_i))}{g(d(n_i, n_j))} \cdot T(n_j)
\end{align*}
$$

where:
* \$f(C(n_i))\$ is a function that increases with the number of connections of node \$n_i\$.
* \$g(d(n_i, n_j))\$ is a function that increases with the distance between nodes \$n_i\$ and \$n_j\$.
* \$T(n_j)\$ is the trust value of the node \$n_j\$.

The trust values across the network will be computed iteratively until convergence is reached. The exact form of the functions \$f\$ and \$g\$ would be dependent on the specific implementation of the network and the trust dynamics of the community.
## Challenges {#challenges}
1. Scalability: As the number of nodes increases, the computational overhead may grow, leading to potential inefficiencies and latency.
2. Identity Verification: Ensuring that one physical entity doesn't manipulate the system by creating multiple nodes (Sybil attack). In the context of this network, such attacks would be restricted to the fringes where the attacker may get a first-degree connection through impersonation.
3. Trust Convergence: In a dynamic environment with continually changing connections, achieving stable trust metric values might be challenging.
4. Trust Metric Manipulation: Nodes might attempt to game the system to appear more trustworthy, which needs to be guarded against.
5. Network Partition: Unreliable connections or malicious attacks could isolate nodes, affecting the efficiency of the network.
6. Data Privacy: Ensuring user data is encrypted and securely transmitted during network interactions is a must.
7. Adoption and User Experience: Creating a user-friendly experience and ensuring new users understand the network is crucial for its success.
 
## Use Cases {#use-cases}
In this section, we'll explore two applications of the decentralized trust network. First, we'll examine an Anonymous Community Forum, focusing on user authentication through trust pathways. Then, we'll discuss a Real-life Party scenario, emphasizing reporting and handling of disruptive behavior within the network's structure. These cases will illustrate the network's potential for enhancing security and accountability in diverse settings.
### Anonymous Community Forum
Consider a user, Alice, who wishes to join an anonymous community forum. Alice has been invited by her friend Bob, who is already a member of the forum. Bob sends Alice an invitation via the web app and Alice accepts, thus creating her node in the trust network.

Alice's trustworthiness within the community is initially influenced by Bob's trustworthiness, as Bob is her only first-degree connection. As Alice interacts with the forum, she has the opportunity to form additional first-degree connections, thereby increasing her own trust score.

When Alice decides to join the forum, the forum system, acting as another node in the network, needs to authenticate Alice. To do this, the forum system searches for the path of highest trust linking it to Alice.

Given that the forum system likely doesn't have a direct connection with Alice, it utilizes the XOR distance in the ID space to guide the search, making successive hops towards Alice. The forum system might travel through several nodes, including Bob, to reach Alice. The trustworthiness of each node along the path influences the overall trust score between the forum system and Alice.

Upon finding the path of highest trust, the forum system evaluates Alice's trust score. If Alice meets the trust threshold, she is authenticated and granted access to the forum. If not, her access is denied until she improves her trust score.
### Real-life Party {#real-life-party}
Consider a real-life party where attendees are part of the trust network. John receives an invitation to the party from his friend Jane, who is hosting the party.

At the party, John observes a group of attendees engaging in disruptive behavior. He doesn't know their identities but wants to report their behavior to Jane. He does this anonymously through the network, identifying the disruptive nodes by their self-generated IDs.

Jane receives the report and investigates the behavior of the disruptive nodes. Through the trust network, she identifies a common invitee ancestor of the disruptive group: a trusted member named Susan, who had invited the disruptive group to the party. Jane communicates with Susan about the reported behavior.

Susan, in turn, communicates with the nodes she invited, holding them accountable for their behavior. If the disruptive behavior continues, Susan has the power to decrease the trustworthiness of the disruptive nodes, potentially blocking them from future events.

Furthermore, since Susan's trustworthiness was associated with the disruptive nodes, her trust score within the network can also decrease due to their behavior. This cascading effect of decreased trustworthiness serves as a mechanism to discourage misbehavior and maintain the integrity of the network.

These use cases demonstrate the power and flexibility of the decentralized trust network in managing and enforcing trust in both digital and real-world contexts.

## Conclusion

This project outlines a plan for a decentralized trust network that balances anonymity, accountability, and user rights. The protocol detailed allows for secure and private connections, managing trust metrics through mechanisms such as ID space, routing, and node exchange. Despite potential challenges, the proposed benefits of this network involve a dynamic trust system for digital communities. We are looking forward to disclosing the full implementation in the near future.
