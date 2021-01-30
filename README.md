# Tips

## Gas Fees
Normally there's no benefit to using these sub-types because Solidity reserves 256 bits of storage regardless of the uint size. For example, using uint8 instead of uint (uint256) won't save you any gas.

But there's an exception to this: inside structs.

If you have multiple uints inside a struct, using a smaller-sized uint when possible will allow Solidity to pack these variables together to take up less storage. For example:

struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// `mini` will cost less gas than `normal` because of struct packing
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30); 
For this reason, inside a struct you'll want to use the smallest integer sub-types you can get away with.

You'll also want to cluster identical data types together (i.e. put them next to each other in the struct) so that Solidity can minimize the required storage space. For example, a struct with fields uint c; uint32 a; uint32 b; will cost less gas than a struct with fields uint32 a; uint c; uint32 b; because the uint32 fields are clustered together.

## Generate random number
https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract

The best source of randomness we have in Solidity is the keccak256 hash function.

We could do something like the following to generate a random number:

// Generate a random number between 1 and 100:
uint randNonce = 0;
uint random = uint(keccak256(abi.encodePacked(now, msg.sender, randNonce))) % 100;
randNonce++;
uint random2 = uint(keccak256(abi.encodePacked(now, msg.sender, randNonce))) % 100;
What this would do is take the timestamp of now, the msg.sender, and an incrementing nonce (a number that is only ever used once, so we don't run the same hash function with the same input parameters twice).

It would then "pack" the inputs and use keccak to convert them to a random hash. Next, it would convert that hash to a uint, and then use % 100 to take only the last 2 digits. This will give us a totally random number between 0 and 99.

This method is vulnerable to attack by a dishonest node
In Ethereum, when you call a function on a contract, you broadcast it to a node or nodes on the network as a transaction. The nodes on the network then collect a bunch of transactions, try to be the first to solve a computationally-intensive mathematical problem as a "Proof of Work", and then publish that group of transactions along with their Proof of Work (PoW) as a block to the rest of the network.

Once a node has solved the PoW, the other nodes stop trying to solve the PoW, verify that the other node's list of transactions are valid, and then accept the block and move on to trying to solve the next block.

This makes our random number function exploitable.

Let's say we had a coin flip contract — heads you double your money, tails you lose everything. Let's say it used the above random function to determine heads or tails. (random >= 50 is heads, random < 50 is tails).

If I were running a node, I could publish a transaction only to my own node and not share it. I could then run the coin flip function to see if I won — and if I lost, choose not to include that transaction in the next block I'm solving. I could keep doing this indefinitely until I finally won the coin flip and solved the next block, and profit.

So how do we generate random numbers safely in Ethereum?
Because the entire contents of the blockchain are visible to all participants, this is a hard problem, and its solution is beyond the scope of this tutorial. You can read this StackOverflow thread for some ideas. One idea would be to use an oracle to access a random number function from outside of the Ethereum blockchain.

