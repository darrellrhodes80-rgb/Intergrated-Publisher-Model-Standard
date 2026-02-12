# Integrated Publisher Model (IPM) Protocol
### Technical Logic & Smart Contract Specifications v1.0

**Purpose:** This document outlines the core logic flows for the IPM. Developers should use this pseudo-code as the reference for building the actual smart contracts or backend logic.

---

## MODULE 1: THE "GLASS BOX" (Revenue Transparency)
**Goal:** Automate payments instantly. Eliminate "Net-60" terms.

```javascript
FUNCTION DistributeRevenue(IncomingPayment):
  IF IncomingPayment > 0:
    // Define the Splits
    Calculate ProjectPoolShare = IncomingPayment * 0.10  // 10% for operations/servers
    Calculate ContributorPool  = IncomingPayment * 0.90  // 90% for the talent

    FOR EACH Contributor in ActiveTeam:
      IF Contributor.Status is "VESTED":
        // Direct Payment
        Send (ContributorPool * Contributor.EquityShare) to Contributor.Wallet
      ELSE:
        // Hold in Escrow until they prove themselves
        Send (ContributorPool * Contributor.EquityShare) to Escrow.HoldingAccount

    LogTransactionToBlockchain("Payment ID", "Timestamp", "Split Details")
  ELSE:
    Return Error("No funds received")
MODULE 2: THE "VESTING CLIFF" (Anti-Freeloader)
Goal: Ensure only committed partners get equity.

JavaScript
CONSTANT VESTING_PERIOD = 90 Days  // 3 Months

FUNCTION CheckVestingStatus(Contributor):
  // TRIGGER 1: The Clock (Loyalty)
  Boolean TimeCondition = (CurrentDate - StartDate >= 90 Days)

  // TRIGGER 2: The Work (Performance)
  // Checks if the Contributor has completed a "Major Milestone"
  Boolean WorkCondition = (Contributor.CompletedMilestones >= 1)

  IF (TimeCondition OR WorkCondition):
    Unlock Contributor.Status = "VESTED"
    Release Escrow.HoldingAccount to Contributor.Wallet
    Log "Vesting Unlocked via " + (TimeCondition ? "Time" : "Merit")
  ELSE:
    Keep Status = "PROBATION"
    // They get paid hourly/flat fee, but no permanent rev-share yet.
MODULE 3: THE "ANCHOR" (Immutable Attribution)
Goal: Prevent "scraping off" names from the credits.

JavaScript
FUNCTION RecordContribution(WorkID, ContributorID):
  // Create a permanent link between the work and the worker
  Create DigitalSignature = Hash(WorkID + ContributorID + Timestamp)
  Store Signature on Blockchain

  // This creates an unbreakable link.
  // Even if the project is sold, this signature remains.

  IF Project.Sold to NewOwner:
    Retain Contributor.RoyaltyRights  // New owner inherits the debt to the creator.
MODULE 4: THE "CLEAN ROOM" (Dispute Resolution)
Goal: The "Nuclear Option." If you fire them for cause, you lose their work to avoid liability.

JavaScript
FUNCTION TerminateContributor(ContributorID, Reason):
  IF Reason is "BadActor":
    // The Penalty
    Revoke Contributor.Access
    Set Contributor.Equity = 0%

    // The Cost (The "Clean Room" Requirement)
    // We cannot keep their work if we don't pay them.
    Identify AllWork linked to ContributorID
    Flag Work as "UNLICENSED"

    REQUIRE ProjectOwner to:
      1. Delete "UNLICENSED" Work from Repository
      2. Commission Rewrite from New Contributor
      3. Verify Rewrite does not match Original (Plagiarism Check)

  ELSE (Amicable Departure):
    Freeze Contributor.Equity at current level
    Keep Work in Repository
MODULE 5: THE "BOUNTY" ORACLE (Verification)
Goal: How to prove the work is done without a bad boss blocking it.

JavaScript
FUNCTION VerifyMilestone(WorkID):
  // WE USE A 2-of-3 MULTI-SIG VOTE TO APPROVE WORK
  Voters = [ProjectOwner, LeadDeveloper, IndependentArbiter]
  VotesFor = 0

  FOR EACH Voter in Voters:
    IF Voter.Signoff(WorkID) == TRUE:
      VotesFor += 1

  IF VotesFor >= 2:
    Mark WorkID as "COMPLETED"
    Increment Contributor.CompletedMilestones
    Trigger CheckVestingStatus(Contributor) // Re-check if this unlocks them
