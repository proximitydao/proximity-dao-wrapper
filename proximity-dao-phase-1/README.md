# ⚡ Proximity DAO ⚡

## Proximity DAO

### Proximity DAO

#### Hackathon Track

ReFi + DAO

##### Region location

Netherlands 🇳🇱 + Singapore 🇸🇬

##### Team Members

- [@shenshin](https://github.com/shenshin) - solidity + javascript
- [@bguiz](https://github.com/bguiz) - solidity + javascript

#### Project Description

##### Pitch

**Problem**:

Sustainability issues are typically *not* tackled
at or near their source,
and those with the most to lose
(especially to climate change),
typically have the least to say about it.
This results in the underlying issues being tackled
with not only the obvious unequal allocation of resources,
but also in sub-optimal utilisation of those resources.

What if there was a way to allocate and utilise funds
in a much more egalitarian manner?

**Solution**:

This team envisions Proximity DAO,
which is a decentralised application
that allows participants to vote on sustainability issues.
Its voting outcomes are used to decide and allocate
funding for these issues.

What makes this DAO different is that
there is greater weight given to votes
from those **most directly affected**.
To ensure that this truly happens,
the proposed solution makes use of
a novel **bi-variate quadratic voting** mechanism

##### Uses cases for the project

**(1)** Allocation of funds to sustainability projects

Recipients: Individuals or projects

This is the most basic objective of the project.
The DAO receives funds from it members
and those funds are henceforth controlled solely by the DAO.
These funds may only be disbursed when
the DAO's members collectively decide to to do so,
and only in the specific manner that they allow.

When funds are disbursed, the DAO decides on attributes
such as who to disburse it to,
how much to disburse,
and the specific mechanisms and rules that apply to it.

**(2)** Micro-lending

Recipients: Individuals

This is inspired by Grameen bank, from Bangladesh,
which disburses funds to unbanked/ underbanked segments
of the population who are unable to increase their
economic opportunities due to a lack of access to loans.

The DAO may implement micro-lending mechanisms
to those who meet the criteria as one of its allocation mechanisms.

**(3)** Universal basic income

Recipients: Individuals

This is where funds are disbursed on a regular basis
to an entire qualifying population.

The DAO may implement universal basic income
as one of its allocation mechanisms.

**(4)** Payment for permanent GHG removal

Recipients: Projects

This is where funds are disbursed to a project
which captures and stores greenhouse gases.
This is measured not only in terms of
the quantity of carbon dioxide equivalent that is removed,
but also in terms of the duration for which it is stored.

The DAO may implement an escrow based system
to allocate funds on a time-locked tranche basis
to these projects based on performance.

The DAO not only votes of the initial allocation
but also vote on the accuracy of
the claimed greenhouse gas removals and storage,
perhaps even commissioning independent audits.
This would affect the amounts disbursed to the project
and the timeline.

**(5)** Payment for renewable energy production and storage

Recipients: Projects

This is where funds are disbursed to projects that
produce and store renewable energy.
This is measured not only in terms of
the quantity of energy produced,
but also the duration for which the energy can be stored.
For example sub-day duration gets the lowest multiplier,
whereas seasonal duration storage gets the highest multiplier.

The DAO may implement an escrow based system
to allocate funds on a time-locked tranche basis
to these projects based on performance.

The DAO not only votes of the initial allocation
but also vote on the accuracy of
the claimed energy production and storage,
perhaps even commissioning independent audits.
This would affect the amounts disbursed to the project
and the timeline.

##### Identity requirements

TODO

#### Summary

##### What to build - baseline

- Smart Contracts
  - Build a DAO
  - Build a Proposal Target interface for the DAO
  - Build instances for the Proposal Target interface
    which implement 1 or more of use cases
  - Build an Identity NFT that is non-transferable (soul-bound)
  - Implement the bi-variate quadratic voting mechanism
    that is linked to both the DAO and the Identity NFT
- Client
  - Build a DApp client to interact with the DAO smart contracts
  - Build a DApp client ot interact with the Proposal Target smart contracts
- Backend
  - Build a stubbed KYC process used for the Identity NFTs


##### Design decisions

###### Proximity score calculation method

- Minimum value for proximity score: 1
- Maximum value for proximity score: 10000
- Measures how directly affected a particular voter is by the issue being voted upon
- Is calculated based on data stored in the identity NFT owned by the voter
	- The identity NFT stores various traits that are stored on-chain
	- These traits include demographic data including:
		- location, income, nationality, gender, education, employment
		- NOTE that these, in future implementations, should not be stored on-chain directly
		  but instead be stored in a manner that can be demonstrated without revealing (ZKP),
		  however, for the purposes of the POC will be stored as-is,
		  and privacy concerns will be addressed post-POC.
- Is also calculated specific to each vote based on the traits that are specific to that proposal
	- A proposal will also have corresponding information:
		- location, income, nationality, gender, education, employment
	- For each of the information fields present,
	  the proposal will also contain a coefficient indicating its relative importance to the rest (ratio)
- Formula
	- Each trait received a score of 1 to 10 based on how well the identity matches the proposal
	- `proximityScoreRaw = (trait1Coefficient * trait1MatchScore) + ... (traitNCoefficient * traitNMatchScore)`
	- The raw score is then scaled linearly to the possible min-max range based on the number of coefficients used, and their coefficients.
	- `proximityScoreMax = (trait1Coefficient * 10) + ... (traitNCoefficient * 10)
	- `proximityScore = proximityScoreRaw * 10000 / proximityScoreMax`
- Reference values for outcomes:
	- `voterIndentityTraits`
		- `location`: `gazipur, bangladesh`
		- `income`: `1000USD`
		- `nationality`: `bangladeshi`
		- `gender`: `female`
		- `education`: `primary school`
		- `employment`: `unemployed`
	- `proposalTraits`
		- `location`: coefficient `10` value `dhaka, bangladesh`
		- `income`: coefficient `5` value `under 2500USD`
		- `gender`: coefficient `3` value `female`
		- `education`: coefficient `2` value `none`
	- `proximityScoreRaw = (10 * 9) + (5 * 10) + (3 * 10) + (2 * 8) = 90 + 50 + 30 + 16 = 186`
	- `proximityScoreRaw = (10 * 10) + (5 * 10) + (3 * 10) + (2 * 10) = 200`
	- `proximityScore = 186 * 10000 / 200 = 9300`

###### Proximity score for voting

- Anyone can vote for a proposal, no matter their proximity score for that particular proposal.
- However, the weightage of their vote is dependent upon their proximity score for that proposal,
  affecting how much their vote counts, which is the intended outcome,
  as per the bi-variate quadratic voting.

###### Proximity score for receiving assets

- Minimum threshold for proximity score to receive asset
- This is stated within the proposal
- This is only relevant within proposals which choose to funding users directly
	- Note that some proposals may be to fund projects or organisations
- This uses the same proximity score as calculated above,
  however in this case, it is used *after* the bi-variate quadratic voting

###### Bi-variate quadratic voting

Calculations:
- Formula: `voteWeight = (fungibleTokenCount^0.5) * (proximityScore^0.5)`
- where `fungibleTokenCount` is the number of governance (fungible) tokens held by the voter, and
	- if the voter holds more than 10000, it is capped at that maximum value
- where `proximityScore` is the score calculate using the approaches described above,
  based on the identity non fungible token held by the voter.

These are reference values for outcomes:
* 1 token, 1 proximity = 1 vote weight
* 1 token, 100 proximity = 10 vote weight
* 1 token, 10000 proximity = 100 vote weight
* 100 tokens, 1 proximity = 10 votes weight
* 100 tokens, 100 proximity = 100 votes weight
* 100 tokens, 10000 proximity = 1000 votes weight
* 10000 tokens, 1 proximity = 100 votes weight
* 10000 tokens, 100 proximity = 1000 votes weight
* 10000 tokens, 10000 proximity = 10000 votes weight

For a proposal to pass, there must be a minimum quorum threshold met.
This threshold is specified in terms of the sum of vote weight.
It is specified neither in terms of the number of votes cast, nor the number of tokens.

###### Avoiding centralisation on decentralised technology

- Avoiding an "elected core", no vote delegation
  as these are both forms of centralisation,
  even if they are built on top of decentralised technology.
- This is by design, as it is critical to the mission of this DAO
  that votes on proposals are cast by those who are
  most affected by the outcomes of that same proposal.

###### Sustainable development goals

- The [sustainable development goals from the United Nations](https://www.un.org/sustainabledevelopment/sustainable-development-goals/)
  are used as the guiding principles for "what should this achieve".
- As such, proposals submitted to this DAO should describe their impact
  in terms of the positive impact it aims to have on one or more of the SDGs.

TODO further elaboration/ mapping

###### Types of equality

TODO

#### URLs

TODO

List any URLs relevant to demonstrating your prototype

#### Presentation

TODO

List any links to your presentation or any related visuals you want to share.

#### Next Steps

##### What to build - stretch goals

The hackathon submission will aim to implement the fundamental operations.
However, the scope envisioned is much further reaching that that,
and not something that could be attained reasonably within the timeframe.
Not to limit ambition though!
These are stretch goals that we intend to go for,
as soon as the baseline is attained.

**Proposal targets**:

- Staged escrow proposal target implementation
- Recurring payments proposal targets implementation
- Borrow/ lend proposal targets implementation

Each of the above could be implemented as a new SC in a modular manner, and invoked by adding new functions to the upgradable target SC.

**Identity**:

- KYC implementation for Identity NFTs
- Zero-knowledge proofs that allows Identity NFTs
  with the intent to preserve privacy of its owners

The baseline implementation only will contain stubbed implementations
of the above, however are unsuitable for "production" usage
with real world people.

#### License

GPL-3.0
