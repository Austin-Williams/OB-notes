### (This is an old approach. Ignore this document. Keeping it here only because it has some useful ideas in it).

### One Possible Approach to Storing OB Reviews in the Bitcoin Blockchain

The approach of storing hashes of reviews in the bitcoin blockchain and having Sellers (and possibly notaries) held accountable for hosting the reviews has at least two difficulties. 

1. How do we find all the review hashes for reviews of a given seller in the bitcoin blockchain? 
2. How do we verify that the reviews are legitimate (came from the buyer listed in a completed OB Ricardian contract)?

What follows is one possible approach. This approach of storing reviews is intended to 

1. make it easy to find (hashes of) legitimate OB reviews in the bitcoin blockchain and 
2. make it infeasible to plant (hashes of) false OB reviews directly into the bitcoin blockchain. 

A malicious seller may still create a bunch of notary/buyer sybils, create false trades, and leave reviews for himself that way. That's a separate problem.

This approach requires each seller to have a dedicated bitcoin address, labeled _Seller_Review_Address_, that will be used for the review process. This address should be attached to the seller's pseudonymous identity permanently (perhaps, but not necessarily, generated deterministically from his GUID/OBprivKey).

### BROAD STROKES:

The idea is that all buyer reviews of a seller will be OP_RETURNed in a transaction that contains _Seller_Review_Address_ as one of its outputs.
This way any node can find a seller on OB, look up that seller's _Seller_Review_Address_ on OB, and search the blockchain for all transactions that contain _Seller_Review_Address_ as an output. Let's call this set of transactions _SellerReviewDump_. 

_SellerReviewDump_ will contain (hashes of) all of that sellers' reveiews. Note that _SellerReviewDump_ could also contain a lot of other things (examples: hashes of *fake* reviews OP_RETURNed in transactions with _Seller_Review_Address_ as one of it's outputs, meaningless transactions with no OP_RETURNed data, etc etc). _SellerReviewDump_ will definitely contain *all* the transactions containing valid review hashes for the seller... we need only a way to determine which of these transactions was generated from an OB trade, and which ones were not.

The phrase "generated from an OB trade" will be understood to mean that there exists a completed Ricardian contract (signed by buyer, seller, and notary) that corresponds to that transaction.

### FINER DETAILS:

Buyers, sellers, and notaries would use the following protocol.

* (1) Buyer, Seller, and Notary create a 2-of-3 multisig address, _Trade_Address_.
* (2) Seller sends a small payment (dust) from _Seller_Review_Address_ to _Trade_Address_. (explanation for why later)
* (3) Buyer sends payment for goods to _Trade_Address_.
* (4) Physical goods are sent/received. At this point in protocol the Ricardian contract should be 'complete' -- in the sense that it's been signed with all required signatures and contains the complete transaction history. No further alterations/signatures should be added to the contract beyond this point. We'll call this completed contract the TRADERECEIPT.
* (5) Buyer writes review, let's call it REVIEW, and passes review to Seller.
* (6) Seller checks review for appropriateness (doesn't contain spammy links, isn't 10TB, etc etc).
* (7) If the review is appropriate, Seller creates and signs a transaction that consists of the single input _Trade_Address_, and three outputs:
	- (Output 0) a small payment (dust) to _Seller_Review_Address_
	- (Output 1) the rest of the coins (less miner's fee) to sellers payout address as specified in the Ricardian contract
	- (Output 2) OP_RETURN the value sha256(TRADERECEIPT) XOR sha256(REVIEW)
* (8) Seller passes this signed transaction to Buyer. Buyer verifies the outputs are correct and then signs and broadcasts the transaction to the bitcoin network.
* (9) Seller hosts both TRADERECEIPT and REVIEW indefinitely.

Now when a user discovers Seller's node, she looks up Seller's _Seller_Review_Address_, and scans the blockchain for all transactions with _Seller_Review_Address_ as output. She calls this set of transactions _SellerReviewDump_. She removes from _SellerReviewDump_ all transactions that do not contain any OP_RETURNed data, and removes all transactions with more than one input.

The remaining transactions in _SellerReviewDump_ have only one input. For each of the remaining transactions, she checks the parent transaction of the single input to see whether it has _Seller_Review_Address_ as an input (genuine OB transactions would because of Step (2) above), and throws out the transaction if it doesn't. 

[You may think of Step (2) as the Seller saying "I agree to be held accountable for whatever review ends up coming out of this _Trade_Address_ I'm paying into." This prevents malicious players from attacking the Seller by creating bitcoin transactions with _Seller_Review_Address_ as an output, along with an OP_RETURN of uniform random bits, for which Seller could never provide the sha256 preimage]

What remains is a list of transactions which we'll call SellersBurden. It contains all the transactions with OP_RETURNed hashes for which the Seller must provide preimages. We know for sure that all reviews that resulted from *genuine* OB trades (that is, trades which correspond to completed OB Ricardian contracts) will be represented by hashes in SellersBurden. But there may also be other transactions that did not come from genuine OB trades (like if Seller tried to sneak in fake reviews), so we need to verify as follows.

For every OP_RETURNed hash in SellersBurden, Seller must provide both the TRADERECEIPT and the REVIEW. The challenging node computes sha256(TRADERECEIPT) XOR sha256(REVIEW) to make sure the hash matches (if not, penalize Seller with a lower trust/reputation). The challenger gets to see the TRADERECEIPT, and can thus verify everything (that the transaction was the result of a genuine OB trade, that the btc payout addresses/amounts match those in the contract, who the Notary was, that all signatures are correct, etc etc).

### FINAL THOUGHTS:

This approach requires every OB trade involving a review to have it's contract become public -- so it's imperative that PID/sensitive info be blinded by default for all contracts. 

If so desired, notaries can be also be held accountable for storing trade receipts and reviews by tying to every notary a _Notary_Review_Address_ and requiring the same behavior from notaries that we do from sellers w.r.t. Step (2) (and of course adding a 4th output in Step (7) that sends dust to _Notary_Review_Address_).

If we want to rate sellers and notaries and buyers and products, we can do so all at once. Just OP_RETURN:
sha256(TRADERECEIPT) XOR sha256(BuyerReview) XOR sha256(SellerReview) XOR sha256(NotaryReveiw) XOR sha256(ProductReview)
and have the seller (and any other burden-bearing party) host the plaintext reviews.

This is a rough first-approach. We could improve an approach like this or take a completely different tack. Feel free to add/remove/copy/fork/ignore/critique as you please.
