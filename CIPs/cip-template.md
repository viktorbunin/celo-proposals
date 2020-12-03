---
cip: <to be assigned>
title: cip-draft_grace
authors: Viktor Bunin, (@viktorbunin), Tushar Shah (@twoshark), David Michael (@BisonDee)
discussions-to: TBD
status: Draft
type: Standard
category: Ring 1
created: 2020-11-23
license: Apache 2.0
---

## Simple Summary
Extend the time before the validator group score is impacted from 1 minute to 10 minutes of downtime for a single downtime event so operators can safely perform maintenance and upgrade validator infrastructure without harsh penalties.

## Abstract

The Celo protocol currently penalizes validators that do not sign for 12 consecutive blocks, or about 1 minute. This duration is too short for upgrades, maintenance, or even for simple restarts. 

This CIP proposes adding an epochal benevolent period (since “grace period” is already taken) to the smart contract to effectively extend this window without changing the underlying downtime window. Unlike a change to the downtime window, which would precipitate a fork, adding this benevolent period can be done without affecting the validators on the network.

We propose adding a benevolent period of 108 blocks to the current grace period of 12 blocks, creating a grace window of 120 blocks total (10 minutes of downtime)

## Motivation
Celo’s current grace period for downtime of 1 minute is too short to safely perform maintenance or upgrade infrastructure without significant reductions in uptime scores and reward penalties. Alternative zero-downtime methods are extremely expensive, prone to error, and unnecessary.

This grace period was set to encourage infrastructure providers to have high uptime with the goal of making the network more secure. However, setting it this low has the opposite effect and makes the network less secure and more brittle. 

Our proposal to slightly extend this window to 10 minutes will strengthen Celo while still incentivizing the most stringent uptime requirements among layer 1 protocols.

## Specification
Not available, but can be provided if needed.

## Rationale
Celo requires validators to undergo frequent maintenance and upgrades to stay on the bleeding edge, release new features, and quickly squash bugs. However, unlike other networks, Celo has an inordinately high standard for uptime which makes it difficult and expensive to support the network. This disincentivizes infrastructure providers from keeping up with Celo or actively working on making their systems better. Validators currently have three options by which they can perform maintenance, improve their infrastructure, and upgrade their nodes:
Key rotations (initially recommended by the Celo team)
Hot-swap method (recently released by cLabs)
Infrastructure restarts (small downtime, currently big consequences)

This proposal will review the three alternatives, explain why simple infrastructure restarts are required, and chart the path forward on making them tenable. 

### Key Rotations
Key rotations were the first method recommended as a way to perform upgrades without downtime. While it gets the job done, this method is the least effective way for validators to perform maintenance or upgrade their nodes.

Key rotations are a powerful tool to strengthen security against specific types of attacks (e.g., brute force attacks). However, it is important to note that they are a security mechanism. Key rotations are not an appropriate mechanism to conduct routine maintenance and upgrades.
Certain networks have lifespans on their keys (e.g., Algorand) so that infrastructure providers can determine their own comfort levels for frequency of rotation (e.g., 1-12 months). This is a perfectly sensible solution that adds security, but does not exponentially increase the operational burden for routine operations.
Key rotations are an expensive process for operators who must coordinate asset holders, custodians, and node operations. This increases the operational load beyond the node operator to unnecessarily impact other entities that do not need to be as involved on any other network. These entities are not being compensated by the protocol for this work.
Key rotations are prone to errors as we discovered and documented in our incident report. Celo is already a complex network and as it gains additional capabilities (e.g., making Attestation services mandatory for validators or completing the method by which full nodes are incentivized) it will expand the space for sharp edges to develop.

### Hot-Swap Method
We gave feedback on the hot-swap method when it was first being developed in August. While it is preferable to key rotations for some, it is not workable for professional validators. 
It introduces state to the validator cluster and requires coordination and monitoring of the participating nodes
It adds an unreasonable amount of operational complexity as an answer to our desire for operational simplification. Instead of managing 2 nodes, you are now managing at least 4; 2 validators and we can only assume 2 or more proxies when you are upgrading
It does not address the need for restarting the entire cluster as infrastructure providers like Bison Trails upgrade and improve their own platforms
It does not address one of the core challenges of this situation. The validator and proxy cannot establish a connection to one another in under 12 blocks which means that at a client level today there is simply no path to restarting a validator without penalty in any situation. 

While we cannot speak to other infrastructure providers, we think it would be helpful to share some additional context on the hot-swap method as it applies to our infrastructure.

The Bison Trails infrastructure balances the security, uptime, disaster recovery and stability of our participation clusters. Updates to participation clusters are managed atomically and cluster state is retained externally. This allows us to rotate only the resources that need updating to effectively optimize for uptime while protecting against equivocation. As the hot-swap method is an uptime optimization, integrating and testing its operation in the scope of our infrastructure would require investment to ensure that it functions predictably as we would need to monitor the availability and failure modes of additional resources and their states per cluster. 

### Infrastructure Restarts
Being able to quickly restart infrastructure is a mandatory requirement to reasonably maintain blockchain infrastructure. As we mentioned earlier, it is currently not possible, under any circumstances, to restart the proxy/validator pair without a reward penalty and group score ding. There are many problems with this.
It is impossible to restart Celo infrastructure without being punished. The proxy and the validator cannot connect to each other within the 12 block limit. This behavior is well known, documented, and has been discussed with cLabs previously.
The penalty curve is slightly more punitive at the start of the downtime than it is as downtime continues (where negligence or malfeasance is more likely). This seems to be just the nature of the curve.
Robust, secure infrastructure requires slightly more downtime than a simple daemon restart. Operators with robust and secure infrastructure use containerization and orchestration mechanisms like Kubernetes. This layer can marginally extend downtime during restarts or upgrades to ensure fresh instances are running and to protect nodes from double signing. It also increases reliability and ability to recover because of all the default recovery systems and other tooling. As an example, upgrading a Celo node may require pulling a new container image if it does not exist on the platform. Without special accommodations, sometimes the validator will come up before the proxy and will cycle until the proxy is available. Each of these scenarios adds a couple seconds to the process. In particularly extreme cases we have seen clusters take as long as 7 minutes to get back to participation. This downtime though is in no way negligent, quite the opposite in fact. This downtime ensures that the Celo network is as resilient as it can be to unforeseen events and upgrades. This should not be penalized. 
Nearly all double signs on other protocols have been the result of operators over-optimizing for liveness. The complexity and risk of moving from 99.9% uptime to 99.99% uptime in a decentralized protocol is significant and offers almost no benefit. Although Celo does not punish double signing by less than 67% of stake, double signing is still extremely bad and has immediate and long-term consequences for infrastructure providers, hurting their reputation and decreasing their business on all protocols they support.

Further Context on Uptime vs Group Score
Validator group uptime scores are an incredibly important metric by which Celo token holders evaluate infrastructure providers for delegation (even though it largely does not impact them directly). This score is actually one of the only things that infrastructure providers have active control over on the Celo network since they cannot differentiate themselves using fee rates and other aspects (like brand/size) are outside of their immediate control. 

Many professional infrastructure providers have internal and external SLAs for infrastructure performance, with uptime being a key metric. This is typically measured via the proportion of time the infrastructure is performant out of the total time being measured. So a downtime for 1 minute per day would result in an uptime score of 99.93% while 10 minutes of downtime results in 99.3% uptime. This is of course smoothed out over weeks and months so 10 minutes of downtime actually comes out to a 99.9% uptime score over that week or 99.98% over that month.

This is in sharp contrast to how Celo treats uptime as part of the group score. cLabs created this helpful model which shows that even downtime as short as 10 minutes drops the group score from the expected 99.3% to 93.9%. It takes roughly 3 weeks for the score to reflect the more accurate uptime score. This means that although the group score is being used to represent validator performance, it is extremely punitive and not representative of actual performance.

As a final example, in the model above, 3 hours of downtime results in a loss of rewards proportionally equivalent to being down for a little less than a week (!!!) in a single year. This is extremely punitive, especially in the context of how young the Celo client and software are.

## Conclusion
“Show me the incentives and I will show you the outcome” - Charlie “I love crypto” Munger

Although infrastructure restarts are frequently necessary for routine maintenance and upgrades, the brief downtime this method necessitates results in extremely punitive impacts on rewards and uptime scores. The subsequent ding on group scores, a vanity metric, is to the detriment of protocol health more broadly. Not allowing reasonable amounts of downtime has many negative impacts:
Infrastructure providers avoiding making regular upgrades to their infrastructure and systems
Infrastructure providers delaying upgrading to new node software
When upgrading, attempting to upgrade “all at once” which increases the risk of error
Significantly higher operational burdens for infrastructure providers (Celo is an outlier among protocols)
Incentivizes infrastructure providers to run less secure setups
Incentivizes the use of methods like key rotations and hot-swaps, which are less secure and more prone to error (including double signing)

Setting the grace window to 120 blocks (10 minutes) will make Celo more resilient and secure while significantly reducing operational burdens on operators.

## Backwards Compatibility
Not applicable

## Test Cases
Not available, but can be provided if needed.

## Implementation	
 /**
   * @notice Calculates the validator score for an epoch from the uptime value for the epoch.
   * @param uptime The Fixidity representation of the validator's uptime, between 0 and 1.
   * @dev epoch_score = uptime ** exponent
   * @return Fixidity representation of the epoch score between 0 and 1.
   */
  function calculateEpochScore(uint256 uptime) public view returns (uint256) {
    require(uptime <= FixidityLib.fixed1().unwrap(), "Uptime cannot be larger than one");
    uint256 numerator;
    uint256 denominator;
    uptime = uptime + 6250000000000000000000 > FixidityLib.fixed1().unwrap() ? FixidityLib.fixed1().unwrap() : uptime + 6250000000000000000000;
    uint256 updated = uptime + grace_period.unwrap();
    (numerator, denominator) = fractionMulExp(
      FixidityLib.fixed1().unwrap(),
      FixidityLib.fixed1().unwrap(),
      uptime,
      FixidityLib.fixed1().unwrap(),
      validatorScoreParameters.exponent,
      18
    );
    return FixidityLib.newFixedFraction(numerator, denominator).unwrap();
  }

the important line above is 

uptime = uptime + 6250000000000000000000 > FixidityLib.fixed1().unwrap() ? FixidityLib.fixed1().unwrap() : uptime + 6250000000000000000000;

and that function would go here.

1000000000000000000000000 represents a perfect score so we did a simple cross product
17280 is the number of blocks per epoch
108 is our desired grace period
{grace constant} / 1000000000000000000000000 = 108 / 17280

## Security Considerations
Not available, but can be provided if needed.

## License
This work is licensed under the Apache License, Version 2.0.
