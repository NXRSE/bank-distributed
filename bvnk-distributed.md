# Bvnk - Distributed Banking

This is an implementation for a global, distributed bank that still functions within the current banking ecosystem.

----

## Overview
The current internet, including financial institutions, has become too focused and prone to failure. In order to widen service reach, 
provide goals in line with the original ethos of the internet, and provide superior service platforms can be made to be distributed.

In this paper, we analyse an idea for a global, distributed banking system __bvnk__ which is resilient, secure and equitable. We try to take a line 
that is decentralized while realizing the necessity for some centralized functionality. In order to operate within the current 
financial system, centralization of some services is required.

## Central but decentralized
__bvnk__ will be required to speak to territories central banks. This will involve effectively being licensed as a bank in that 
territory. How can we manage a bank but still keep it free and open?

- __bvnk__ would be a non-profit
- GPL licensing
- People who work for __bvnk__ (staff, developers, hosting companies) will be paid from this pool
- Money will be used as collateral for regulations

__bvnk__ needs to be open to participation from anyone. Anyone who has an account with the bank would have one vote. We can give people closer (developers, managers) some extra or 
weighted votes but explicitly limited. We could then all vote on what the sitting money is being used for, corporate governance and more. People could also issue loans to one another using this system.

## Structure
__bvnk__ will be distributed by nature, separated into nodes, clients and broadcasters. Each service will function in the system 
as as whole, providing specific functionality and adding to network resiliency. There can be N instances of each service.

### Nodes
__bvnk__ will operate nodes. All nodes will contain source code and a full copy of the distributed db. Each node will process transactions, taking requests from clients and giving them responses. 
Answers from nodes will be derived from consensus, improving security and mitigating bad actors. Anyone can run a node.

The distributed database will be a RethinkDB cluster. Nodes will report in when they are online, and do period health checks to make sure they are active. Each node should at all times have a full list of other nodes.
Applications who need endpoints (mobile, web, etc) will need to hold a list of nodes too, perhaps this can be restricted to the closest X nodes in a system. Using latency and other checks the best one can be chosen.

### Threshold Cryptography
Resources:

- [Threshold cryptosystem](https://en.wikipedia.org/wiki/Threshold_cryptosystem). 
- [Secret sharing CA](http://www.cs.cornell.edu/Courses/cs513/2000SP/SecretSharingCA.html)
- [More reading on Threshold Cryptography](http://groups.csail.mit.edu/cis/cis-threshold.html)
- [Distributed key generation](https://en.wikipedia.org/wiki/Distributed_key_generation).

Description of flow.

- Each node has only a partial private key
- Transaction is sent to one node and then sent to other nodes
- This is done until the threshold has been reached
- The account data is then decrypted, compared with the request which is bundled in the data, and verified or declined
- The entire above process is done X number of times, reaching required consensus over the network

### Example

- User registers account using cell phone
- pub/pvt keys are generated which are linked to the account
- pub/pvt keys sign and encrypt all transactions
- user makes payment:
    - user sends payment request to node
    - node fetches user's account data
    - node generates request UUID
    - node sends data to user (fully encrypted) with additional UUID encryption
    - user partially decrypts data using pvt key and UUID
    - user appends request to data
    - user sends data back to node, re-encrypting using UUID
    - node partially decrypts using UUID
    - node sends transaction to other nodes in order to decrypt remaining data
        - No one node has the bvnk pvt key
    - once pvt key constructed, account data is decrypted
    - account data compared to request data
    - nodes approve/deny transaction
    - above is repeated until N nodes have approved transaction
    - once transaction approved, request is now constructed for the central bank
    - transaction is constructed for the central bank
    - transaction is sent to user (encrypted bvnk -> user)
    - user decrypts, verifies transaction, signs transcation with pvt key
    - user encrypts data and sends to node (encrypted user -> bvnk)
    - bvnk decrypts request, sends request to central bank with user's signing
        - Details for bvnk request to central bank being authorized by the user to be ironed out

This would entail the software being trusted by the central bank, as well as the software being allowed to make transactions against the central bank.

### Verifiability

The issue comes in when there are now distributed nodes running by anyone who wishes to run them, how do you keep the system secure? It's a decentralized design with necessary centralized issues.

Transactions can be verified by sending the transaction encrypted with the bank's private key. The system needs too make sure that the node sending the transaction is trusted and sending trusted transactions.

- Perhaps the transaction can be sent along with all the keys of the transaction and verified at the edge: sender, recipient, bank. An attacker would then need to have all three keys to forge an attack. They can have one of them - the bank's - but not the other two.
- The bank's key being used in the transaction would act as a proof of consensus on the network. For the transaction to be signed by the bank's key it would have to have been approved by N nodes.

## How will this system make money?

The system will primarily make money through transaction fees. Each node will also be paid a nominal amount for processing and staying active.

This money will be used for funding the system:

- development
- infrastructure
- support
- help desks
- legal
- regulations

The main bank holder will be an account separate. This is where all fees will be paid into, as well as final say on account opening, verification, bank loans, etc.

## Considerations

All deployments will have a partial of the bank private key. Using threshold cryptography we can help ensure that no one bad actor has or can recreate the private key. Ideally we should treat all communications secured by the bank and the device as insecure. This is only when communication is encrypted with the bank's private key - any requests going **into** the bank. To mitigate this attack, all requests should be double verified (req -> resp -> ack -> ack -> transaction).

- Could a bad actor bank node send false requests?
- Could a bad actor save the re-constructed private key at time of decryption?
- User account data must be protected using multi sig. Something like:
    - User opens account
    - Account data is encrypted with bank key and user key (multi sig)

### Keys and accounts

User's **must** have backups of keys. This can be done by backing them up using an easy to remember password with additional security measures. The point is that their key must be recoverable, easily for the account owner but difficult for a bad actor. This could perhaps be sent to a safe hold for the "bvnk.org" organization.

### Double spending

Using threshold cryptography together with reaching a consensus we can require transactions to be verified by a large number of nodes (50%, 70%?). Each node would report back into the 
originating node although this might be susceptable to a bad actor attack where the node just says everything is ok. Each node could have their own pub/pvt keys. This will not work if there 
is bad connectivity, if the network is down, if a lot of the nodes are down but not updated, etc.

### Broadcasters

Separate software that runs and sends transactions to all nodes. Only users use these. Excluding nodes from broadcasters might stop potential attacks.

### Data at rest
Broadly, data at rest will be encrypted using a combination of factors:

- User's key
- Bank's key
- Shared secrets
- Unique user information (password or similar)

The implementation of this is still to be worked out.

### User data
This includes user account details, balances, etc.

A user's data must not be able to be changed unless both the bank and the user agree to it. As such the data must be encrypted by both parties.

## Conclusion
This is a high level run through for a distribured banking system. 

If you would like to contribute to this document, please open [a pull request](https://github.com/bvnk/bank-distributed).
