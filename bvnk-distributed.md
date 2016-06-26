# Bvnk - Distributed

This is an implementation for a global, distributed bank that still functions within the current banking ecosystem.

## Nodes
Bvnk will operate nodes. All nodes will contain source code and full copy of the distributed db.

The distributed database will be a RethinkDB cluster. Nodes will report in when they are online, and do period health checks to make sure they are active. Each node should at all times have a full list of other nodes.

Applications who need endpoints (mobile, web, etc) will need to hold a list of nodes too, perhaps this can be restricted to the closest X nodes in a system. Using latency and other checks the best one can be chosen.   

## Flow

- User registers account using cell phone
- pgp Pub/pvt keys are generated which are linked to the account
- pgp keys sign and encrypt all transactions
- user makes payment:
    - encrypts transaction with recipient key and bank key (maybe sign with bank key would be a better way to put it) 
    - sends to nearest node
    - bank node verifies keys 
    - finds recipient account
    - sends transaction to recipient
    - recipient downloads transaction
    - recipient verifies transaction
    - recipient sends verification using sender's key and bank key
    - bank node verifies transaction

This is where it now gets a little tricky. In order to process the transaction, and assuming we are hooking into the already existing financial infrastructure, we need to take the money from the sender's bank and add it to the recipient's bank.

As these accounts are in the distributed database, the only place through which money is transferred is the respective central banks.

- Bank nodes have both approved transaction
- Transactions are put forward for the central banks, one for sender and recipient each (in their respective territories)
- Transactions **must** be verified to be from the sender (signed by their key? encrypted fully?)
- Either in real time or in batches the transactions are processed
- Regulations are handled at the edge

This would entail the software being trusted by the central bank, as well as the software being allowed to make transactions against the central bank.

### Verifiability

The issue comes in when there are now distributed nodes running by anyone who wishes to run them, how do you keep the system secure? It's a decentralized design with necessary centralized issues.

Transactions can be verified by sending the transaction encrypted with the bank's private key. Some more detail on this in the Considerations section.

There must be another way to do this cryptographically. To make sure that the node sending the transaction is trusted and sending trusted transactions.

- Perhaps the transaction can be sent along with all the keys of the transaction and verified at the edge: sender, recipient, bank. An attacker would then need to have all three keys to forge an attack. They can have one of them - the bank's - but not the other two.
- In the above scenario would the bank's key even be needed? Would it be needed for the initial payment flow, and not this flow?

## How will this system make money?

In short, transaction fees. Each node will also be paid a nominal amount for processing and staying active, but money will be made on transaction fees.

This money will be used for funding the system:

- infrastructure
- support
- help desks
- legal
- regulations

## Central but decentralized

Each territory will have to be cleared for operation on the bank. How can we manage a bank but still keep it free and open?

- GPL licensing
- The bank would be a non-profit
- People who work for the bank (staff, developers, hosting companies) will be paid from this pool
- Money will be used as collateral for regulations

The bank needs to be open to participation from anyone. Anyone who has an account with the bank would have one vote. We can give people closer (developers, managers) some extra or weighted votes but explicitly limited. We could then all vote on what the sitting money is being used for, corporate governance type stuff, and more. People could also issue loans to one another using this system.

## Considerations

All deployments will have the bank private key so effectively all communications secured by the bank and the device are insecure. This is only when communication is encrypted with the bank's private key - any requests going **into** the bank. To mitigate this attack, all requests should be double verified (req -> resp -> ack -> ack -> transaction).

The main bank holder will be an account separate. This is where all fees will be paid into, as well as final say on account opening, verification, bank loans, etc.

- Could a bad actor bank node send false requests?

User account data must be protected using multi sig. Something like:

- User opens account
- Account data is encrypted with bank key and user key (multi sig)

Then:

- User requests to send money
- Bank gets the data
- Bank generates UUID OTP
- Bank sends partially encrypted data to user with OTP encrypted 
- User decrypts it partially with OTP and their pvt key
- User sends back to bank encrypted with OTP (at this stage it is only encrypted with bank's key)
- Bank decrypts data, verifies account balance etc
  - This could be a problem. If there is a bad node it can just *say* the transaction is valid. 
  - For e.g. I try to transfer 1,000 from my account, I do the above check, the bank sees the balance of 100 but *chooses to ignore it*, approves the transaction and then fraud is committed. 
  - Worse, this new, false balance will then be propogated throughout the system.
  - I am sure Bitcoin has fixed this problem.
  - This might be solved by sending a snippet of a transaction to many banks, them checking the balance and confirming (more in section Double Spending)

The above flow is done for all transaction requests. Then the transaction is sent to the central bank. Maybe each central bank has it's own set of keys:

- Transaction gets verified as above
- Bank node repsonds with the appropriate central bank key (the central bank belonging to their account, probably geographically based)
- User then encrypts transaction with central bank key, bank key, their key
- Maybe the bank verifies the transaction (looking up the amount etc, mentioned below) creates the transaction using the central bank's key and has the user sign the transaction. This way the transaction is encrypted for the bank, with the user's approval on each.

### Keys and accounts

User's **must** have backups of keys. This can be done by backing them up using an easy to remember password with additional security measures. The point is that their key must be recoverable, easily for the account owner but difficult for a bad actor. This could perhaps be sent to a safe hold for the "bvnk.org" organization.

### Double spending

In theory, a user could get a list of all bank nodes, write a script that goes through them, and send transactions very quickly. This will result in the transactions being processed by each bank node before the bank node has had a chance to update from the initial request. This is where the blockchain confirmations can come in. We could also somehow couple this with a merkle tree to make sure that the oldest transaction is the oldest. This will be a problem when we are dealing with thousands of transactions per second. 

Perhaps transactions could be verified by a large number of nodes (50%, 70%?) ensuring that the above cannot happen. Each node would report back into the originating node although this might be susceptable to a bad actor attack where the node just says everything is ok. Each node could have their own pgp keys. This will not work if there is bad connectivity, if the network is down, if a lot of the nodes are down but not updated, etc. This idea seems best so far.

Transaction flow with considerations for double spending and balance verifiability.

- User requests to send transaction with a value
- Node responds with a challenge. This challenge includes the user's data, encrypted from bank to user (plain text -> [ bank -> user ]).
- Node sends challenge with user's pubkey to all other nodes
- User responds to challenge, confirming legitimacy. Here the user decrypts the data so that it is now only encrypted with the bank's key/secret and a challenge ([ plain text -> bank ]+Challenge -> user)
- User sends response to broadcaster (which sends to all nodes on behalf of user)
- Nodes decrypt user account data fully
- Nodes verify the transaction
- When a critical point is reached (50%? 70%? More?) the transaction is approved (this is similar to confirmations in the blockchain)
- Once verified, account balance is updated, encrypted, sent to user who signs it, and transaction is settled
  - This needs a lot more thought.

### Broadcasters

Separate software that runs and sends transactions to all nodes. Only users use these. Excluding nodes from broadcasters might stop potential attacks.

### Data at rest
Data at rest is encrypted as follows.


#### User data
This includes user account details, balances, etc.

A user's data must not be able to be changed unless both the bank and the user agree to it. As such the data must be encrypted by both parties.

## Reworked using Threshold Cryptography

- [Threshold cryptosystem](https://en.wikipedia.org/wiki/Threshold_cryptosystem) looks like a good fit. 
- [Secret sharing CA](http://www.cs.cornell.edu/Courses/cs513/2000SP/SecretSharingCA.html)
- [More reading on Threshold Cryptography](http://groups.csail.mit.edu/cis/cis-threshold.html)
- [Distributed key generation](https://en.wikipedia.org/wiki/Distributed_key_generation) might come in handy.

Description of flow.

- Each node has only a partial private key
- Transaction is sent to one node and then sent to other nodes
- This is done until the threshold has been reached
- The account data is then decrypted, compared with the request which is bundled in the data, and verified or declined
- The entire above process is done X number of times, reaching required consensus over the network