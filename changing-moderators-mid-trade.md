### *A Protocol for a Vendor and Buyer to Change Moderators Mid-Trade*

#### *Motivation:*
A Vendor and Buyer in dispute and with funds in escrow may find themselves unhappy with the arbitration of their current Moderator. Perhaps the Moderator becomes unresponsive, or refuses to settle the dispute for moral/legal/ethical/personal reasons. Or maybe the Moderator is demanding certain data fields to be unblinded that the other parties are  not comfortable unblinding. Whatever the reason, the Vendor and Buyer may agree that they want their dispute handled by a new Moderator. This can be done quite simply, and without the current Moderator's permission as follows.

#### *Execution:*
Refer to the 2-of-3 multisig address in which the funds are currently escrowed as _TradeAddress_.  

1. Vendor and Buyer agree upon a new moderator to whom we will refer as _NewModerator_.

2. Vendor and Buyer create a new 2-of-3 multisig address, _NewTradeAddress_, using bitcoin addresses from Merchant, Buyer, and NewModerator.

3. Vendor and Buyer sign a transaction that moves all funds out of _TradeAddress_ and into _NewTradeAddress_.

The Merchant and Buyer now have no need for the old Moderator, and can work with the NewModerator to settle their dispute.

#### *Alternative Execution:*

This method allows the Vendor and Buyer to create a 2-of-2 multisig address initially, and then follow the same steps as above to select a Moderator if they need arbitration.
