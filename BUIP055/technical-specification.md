# BUIP 055 Technical Specification (* time-based)

(*) BUIP055 currently specifies a block height fork. This
specification is currently based around a time-based fork due to
miner requests. It should be possible to change easily to a
height-based fork - the sense of the requirement's would largely
stay the same.

## Requirements

### REQ-1 (fork by default)

The client (with BUIP055 implementation) shall default to activating
a hard fork with new consensus rules as specified by the remaining
requirements.

RATIONALE:: It is better to make the BUIP055 active by default in a
special HF release version. Users have to download a version capable
of HF anyway, it is more convenient for them if the default does not
require them to make additional configuration.

NOTE 1: It will be possible to disable the fork behavior (see
REQ-DISABLE)

### REQ-2 (configurable fork time)

The client shall allow a "fork time" to be configured by the user,
with a default value of 1501545600 (epoch time corresponding to Tue
1 Aug 2017 00:00:00 UTC)

RATIONALE: Make it configurable to adapt easily to UASF activation
time changes.

NOTE 1: Configuring a "fork time" value of zero (0) shall disable
any BUIP055 hard fork special rules (see REQ-DISABLE)

### REQ-3 (fork block must be > 1MB)

The client shall enforce a block size larger than 1,000,000 bytes
for the first block with timestamp >= activation time

RATIONALE: This both enforces the hard fork from the original 1MB
chain, and also prevents a re-organization of the forked chain to
the original chain.

### REQ-4-1 (set EB to minimum of 8MB at fork)

If BUIP055 is not disabled (see REQ-DISABLE) and the block timestamp
is equal to or greater than the fork time, the client shall set EB
to the maximum of 8,000,000 (bytes) and the user's configured EB.

RATIONALE: To immediately allow up to 8MB blocks on the forked
chain, without considering them excessive. Effectively this will
raise the allowed block size on the fork network to a minimum of 8MB
regardless of user's EB configuration.

NOTE: default EB value of 16,000,000 should ensure most clients will
follow the chain without problems. This lifting would give time to
others to raise their EB.

### REQ-4-2 (set MG to minimum of 8MB at fork)

If BUIP055 is not disabled (see REQ-DISABLE), the client shall, for
blocks generated with MTP >= fork time, set the mining generated
(MG) size to the maximum of 8,000,000 (bytes) and the user's
configured MG.

RATIONALE: To immediately generation of up to 8MB blocks on the
forked chain. Effectively this will raise the allowed block size for
mining on the fork to a minimum of 8MB regardless of user's MG
configuration.

### REQ-5 (Re-org Protection)

Once the fork has activated (median time past of active chain tip
exceeds fork time), the client shall not allow a re-organization
which would remove the fork block.

RATIONALE: To prevent the fork chain from being continually
re-organized by an attacker.

### REQ-6-1 (disallow special OP_RETURN-marked transactions)

Once the fork has activated, transactions containing an OP_RETURN output
with a specific magic data value shall be considered invalid.

RATIONALE: To give users on the legacy chain (or other fork chains)
an opt-in way to exclude their transactions from processing on the BUIP055
fork chain.

### REQ-6-2 (opt-in signature shift via nHashType)

Once the fork has activated, transactions shall not be deemed  invalid if
adding a certain magic value to the nHashType before the hash is calculated
results in a successful signature verification.

RATIONALE: To give users on the BUIP055 chain an opt-in way to encumber
replay of their transactions to the legacy chain (and other forks which may
consider such transactions invalid).

NOTE 1: It is possible for other hard forks to defeat this protection by
implementing a compatible signature check that accepts transactions
signed in this special way. However, this does require a counter hard fork.

NOTE 2: The client shall still accept transactions whose signatures
verify according to pre-fork rules, subject to the additional OP_RETURN
constraint introduced by REQ-6-1.

### REQ-DISABLE (disable fork by setting fork time to 0)

If the fork time is configured to 0, the client shall not enforce
the new consensus rules of BUIP055, including the activation of the
fork, the size constraint at a certain time, and the easing of EB/AD
restrictions for any blocks. RATIONALE: To make it possible to use
such a release as a compatible client with legacy chain / i.e. to
decide to not follow the HF on one's node / make a decision at late
stage without needing to change client.


## Test Plan

What follows are rough descriptions of test cases.

Definitions applying below:

"Large block" means a block satisfying 1,000,000 bytes < block
size <= EB, where EB is as adjusted by REQ-4-1 and a regular block
is a block up to 1,000,000 bytes in size.

"Core rules" means all blocks <= 1,000,000 bytes (Base block size).

NOTE: ** wording for excessive in 3, 4, 5, and invalid in 6, 7,
needs to be considered carefully.  I recommend BUIP 55 forcing EB
1MB up to activation, and to be max(8MB, user EB) at activation, for
simplicity.  Similarly for MG.  If user doesn't like that they can
just disable it.  (Consider: If MG is bumped then 4MB is sufficient
for fork block and a while after.

### TEST-1

If BUIP 55 is disabled a large block is considered to break
core rules, as is presently the case.

### TEST-2

If BUIP 55 is disabled, a regular block  is accepted at or
after the "fork time" without being considered invalid.

### TEST-3

If enabled, a large block is considered excessive if all blocks
have time < activation time.

### TEST-4

If enabled, a large block is not considered excessive if its
timestamp >= activation time <--- even after fork, it must be
subject to EB unless it is the fork block, imo. We need something
more precise than >= activation time for the fork block.  <-- I
think we should not accept a fork block >8MB to avoid the risk of an
attack block constructed much larger.

### TEST-5

If enabled, a large block is not considered excessive if its
timestamp < activation time provided a prior block has timestamp >=
activation time.

### TEST-6

If enabled, a regular block that is the first >= activation time
is considered invalid (satisfy REQ-3).

### TEST-7

If enabled, a regular block that is not the first >= activation
time is considered valid.

### TEST-8

A small block more-work chain does not get re-orged to from a big
block chain after activation has kicked in.

### TEST-9

Test that enabling after being disabled renders a small chain going
past activation invalid that was previously valid owing to it being
disabled.  And vice versa (enabled -> disabled).

### TEST-10

If enabled, if a large <8mb block is produced, ensure that the
degenerate case of sigops heavy instructions does not unduly affect
validation times above and beyond the standard expected if BUIP055
is not enabled.

### TEST-11

Test that linear scaling of 20,000 sigops / MB works for block
sizes > 1MB (rounding block size up to nearest MB) (ref. BUIP040).

### TEST-12

(similar to (TEST-9), but test interoperability of datadir with other
clients.)

Test what happens when the unmodified BU / Core / other clients are
used on a datadir where the BUIP 055 client has been run. Should
test again data from disabled (Core rules data, should be fine) ,
and enabled (big block data stored - may need to rebuild DB? or
provide tool to truncate the data back to pre-fork block?)


## Design

TBD
