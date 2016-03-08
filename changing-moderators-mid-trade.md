### *A Protocol for a Vendor and Buyer to Change Moderators Mid-Trade*

#### *Motivation:*
A Vendor and Buyer in dispute and with funds in escrow may find themselves unhappy with the arbitration of their current Moderator. Perhaps the Moderator becomes unresponsive, or refuses to settle the dispute for moral/legal/ethical/personal reasons. Or maybe the Moderator is demanding certain information or a higher fee than the other parties are comfortable with. Whatever the reason, the Vendor and Buyer may agree that they want their dispute handled by a new Moderator. This can be done quite simply, and without the current Moderator's permission as follows.

#### *Execution:*
Refer to the 2-of-3 multisig address in which the funds are currently escrowed as _EscrowAddress_.  

1. Vendor and Buyer agree upon a new moderator to whom we will refer as _NewModerator_.

2. Vendor and Buyer create a new 2-of-3 multisig address, _NewEscrowAddress_, using bitcoin addresses from Merchant, Buyer, and NewModerator.

3. Vendor and Buyer sign a transaction that moves all funds out of _EscrowAddress_ and into _NewEscrowAddress_.

The Merchant and Buyer now have no need for the old Moderator, and can work with the NewModerator to settle their dispute.

#### *Alternative Execution (1):*

This method allows the Vendor and Buyer to create a 2-of-2 multisig address initially, and then follow the same steps as above to select a Moderator if they need arbitration; although by doing so they put themselves at risk of having funds essentiall 'frozen' if the other party becomes unresponsive.

#### *Alternative Execution (2):*

Any two of the parties (say, Buyer and Moderator) can create and sign an nTimeLocked transaction, T, from _EscrowAddress_ to _NewEscrowAddress_ **before** the escrow address is funded by the Buyer. Transaction T can be given to both the Vendor and the Buyer for safekeeping. Then the Buyer can fund _EscrowAddress_. 

In the event that the Moderator and one other party both become unresponsive, the remaining party need not lose access to the escrowed coins. The remaining party can wait until the date T becomes valid and then broadcast T; thereby sending the funds to _NewEscrowAddress_ and allowing her to work with _NewModerator_ to release the funds as needed.
