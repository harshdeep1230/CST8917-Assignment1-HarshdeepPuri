# CST8917-Assignment1-HarshdeepPuri

Part 1: 
Hellerstein et al. (2019) provide a critical review of the first generation serverless architectures, with a particular eye on Function-as-a-Service (FaaS) platforms such as AWS Lambda. The main claim of the paper is that serverless computing is "one step forward, two steps back" for cloud management because it allows developers to seamlessly scale up and down their applications, pay only for the resources that they use, and avoid the hassle of provisioning and managing servers. The authors claim that, by explicitly separating compute from storage and using a stateless, short-lived execution environment, the first generation of FaaS purposefully challenges decades of important development in distributed databases and parallel computing.

The authors list five basic constraints that exist in early FaaS environments to make implementing complex distributed architectures difficult:

Time Limited Functions: Functions have strict execution time limits (historically 5-15 minutes). It makes it hard to execute long running background processes, data stream applications, or complicated data analysis applications without prematurely ending them.

Communication and Network Bottlenecks: FaaS instances are not directly addressable via the network. Functions are unable to directly communicate with or share data with each other, due to the fact that they are not able to establish peer-to-peer connections or keep open network sockets.

The Data Shipping Anti-Pattern: Functions are stateless and isolated, meaning they have to continuously retrieve data from external storage engines (cloud databases, object storage, etc.) for operations and then return the data right back to the external storage engine. The “data shipping” model is essentially flipping the "code shipping" approach that says “ship lightweight query logic into the database”, to “ship ephemeral code to massive data”, and adding huge network latency and I/O overhead.

First Generation of Serverless: Early serverless platforms sandbox workloads in traditional, standard, generic CPU cores. The developers can't use any special hardware accelerators like GPUs, TPUs, or high-performance non-volatile memory, which would otherwise make it easier for FaaS to process modern machine learning or AI training or heavy crypto loads.

The problem with Distributed Stateful Workloads: The standard distributed coordination patterns require consensus, shared state and predictable topologies. The stateful applications or distributed transactional processing is very challenging and inefficient to implement because the FaaS instances are dynamically created and destroyed at random times and do not maintain a state locally.


Hellerstein et al. suggest three general directions to take cloud architecture in the future to overcome these constraints. First, it calls for fluid code placement, giving the dynamic freedom to the cloud infrastructure to either ship code to data or data to code, depending on costs.Last but not least, they want hardware acceleration for infrastructure, which makes serverless platforms possible to dynamically allocate GPUs and specialized network fabrics on the fly. Finally, the paper serves as an alert that serverless is not just about stateless functions, and if it isn't then it's not going to be a viable approach for today's distributed computing.



Part 2: Azure Durable Functions
1. Orchestration Model
Azure Durable Functions builds on the concept of FaaS by adding a stateful coordination layer with three types of function: Client functions, Orchestrator functions, and Activity functions. Client functions are the gateway that respond to external signals (queues or HTTP requests) to initiate and/or control an orchestration instance. As the centralized conductor, Orchestrator functions are defined as normal sequential code, loops and conditional structures that define the workflow logic. Activity functions perform the work, processing data, I/O or calling external APIs. This paradigm tackles the dilemma that Hellerstein states in his criticism of the fragmented FaaS architectures: it resolves the issue of having to interact with disconnected and independent functions and instead provides a deterministic, structured workflow engine with the orchestration state fully managed by the platform.

2. State Management
With traditional FaaS platforms, the functions must be strictly stateless; with Azure Durable Functions, the architecture is built around state with a pattern known as event sourcing. The Durable Task Framework under the hood saves an event history of all actions, choices, and checkpoints performed by an orchestrator as they are completed into an append-only Azure Storage event log. An orchestrator will yield execution and sleep when an await boundary is asynchronous. When awakened, the orchestrator calls the function again from the start, and quickly re-constructs the current state of the function by reading the history of events, not re-executing previously done actions. This elegantly defends against the paper's comments on statelessness, abstracting the persistence of state completely out of the developer's hands, without violating serverless scaling laws.

3. Execution Timeouts
The first generation of serverless platforms tended to chop off long-running processes, which Hellerstein et al. called their strict execution time limits.The orchestrator is written asynchronously and does not keep running on a live server until tasks are completed; it sleeps, records its status and wakes up only when an activity returns something. This means an orchestrator may run for days, weeks and longer without timeouts. Note that each Activity function continues to have its own runtime constraints (e.g., the default, 5 to 10 minutes, timeout on an Azure Consumption plan), so heavy, continuous computation still needs to be segmented into smaller, modular pieces.


4. Communication Between Functions
The paper from 2019 drew attention to the fact that there was an extreme lack of direct addressability, meaning that functions had to communicate through using slow external storage tiers.This is still using an intermediary storage account (Azure Storage Queues, Blobs, or Event Hubs), but the platform handles the serialization, passing, and scheduling of messages completely in the background. This partially remedies the criticism of the paper: although this does not offer true peer-to-peer network addressability between functions, it removes the need for developers to manually construct.

Part 3: 
Restrictions which are not overcome
Although Azure Durable Functions represents significant architectural improvements, it fails to address two of Hellerstein et al.'s key pain points:

The vision of “moving code to data”, presented in the paper, has not been captured by Durable Functions.Durable Functions does not capture the vision of “moving code to data” that is stated in the paper. If an activity function runs with a large file or a large database payload, these data elements still need to be retrieved from an external source, serialized and sent across the network to the ephemeral VM instance executing the compute. With Fan-Out/Fan-In processing, you have to send a lot of data over the network, incurring significant network expenses and serialization delays.

Storage Intermediary I/O Bottleneck is still a fundamental problem: The Storage Intermediary I/O Bottleneck is that the Storage Queue Replay mechanism and the message queues are totally dependent on Azure Storage (Table, Queue, Blob). All actions, state changes and task transfers involve read and write operations to and from the storage. This means that it has a hidden latency floor, since it is not the low-latency peer-to-peer network routing that allows transactions to be processed in sub-millisecond timeframes with high throughput in high-performance distributed computing applications, such as real-time gaming systems or high frequency financial trading platforms.


## References
*  J. M. Hellerstein et al., "Serverless Computing: One Step Forward, Two Steps Back," CIDR 2019. Available: https://www.cidrdb.org/cidr2019/papers/p119-hellerstein-cidr19.pdf
*  Microsoft Corporation, "What are Durable Functions?," Microsoft Learn, 2026. [Insert working URL to Azure Durable Functions Documentation here]

## AI Disclosure Statement
Used Gemini to help research the core tenets of Hellerstein's paper, organize the structured headings, verify how event-sourcing relates to Azure Storage, and proofread the text sections to align strictly with the word count boundaries required by the CST8917 syllabus.
