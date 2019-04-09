### What is the Filecoin Mining Process
Filecoin 挖矿过程

An active participant in the filecoin consensus process is a storage miner and expected consensus block proposer. They are responsible for storing data for the filecoin network and also for driving the filecoin consensus process. Miners should constantly be performing Proofs of SpaceTime, and also checking if they have a winning `ticket` to propose a block for each round. Rounds are currently set to take around 30 seconds, in order to account for network propagation around the world. The details of both processes are defined here.

Filecoin 共识过程中的积极参与者即是存储挖掘者，又是 Experted Consensus 的区块提议者。它们负责为 Filecoin 网络存储数据，并驱动 Filecoin 共识过程。矿工们应该不断地对时空进行证明，并检查他们是否有一张中奖的“彩票”，为每一轮提出一个区块。目前轮数设置为30秒左右，以便满足世界的网络传播。这里定义了这两个过程的详细信息。

Any block proposer must be a storage miner, but storage miners can avoid performing the block proposer tasks, however in this way, they will be losing out on block rewards and transaction fees.

所有的区块提议者都必须是存储矿工，但存储矿工能够不担任区块提议任务，但这样的话，他们也会损失区块的奖励。

（这里有个问题，就是对于VRF算法，集中化矿工能否获得比相同算力的分散矿工的更多机会的记账权？）

## The Miner Actor
挖矿执行者

After successfully calling `CreateMiner`, a miner actor will be created on-chain, and registered in the storage market. This miner, like all other Filecoin State Machine actors, has a fixed set of methods that can be used to interact with or control it.

在成功创建矿工后，一个矿工执行者将会被在链上创建出来，并注册在存储市场中。这个矿工，像其他 Filecoin State Machine 执行者一样，有一组固定的方法，可以用来与它交互或控制它。

For details on the methods on the miner actor, see its entry in the [actors spec](actors.md#storage-miner-actor).

挖矿执行者的方法的细节，参考 [actors spec](actors.md#storage-miner-actor)。

### Owner Worker distinction
Owner Worker 区别

The miner actor has two distinct 'controller' addresses. One is the worker, which is the address which will be responsible for doing all of the work, submitting proofs, committing new sectors, and all other day to day activities. The owner address is the address that created the miner, paid the collateral, and has block rewards paid out to it. The reason for the distinction is to allow different parties to fulfil the different roles. One example would be for the owner to be a multisig wallet, or a cold storage key, and the worker key to be a 'hot wallet' key.

挖矿执行者有两个不同的‘控制者’地址。一个是 Worker，这个地址将被用来表示它的全部工作，提交证明，确认新的扇区，以及全部日常活动。Owner 地址是用于创建矿工，支付抵押品，以及获得区块奖励。这样区分的原因是允许不同的参与方负责不通的角色。例如，Owner 可以是一个多签钱包，或者一个冷存储密钥，而 Worker 密钥可以是一个“热钱包”密钥。

### Storage Mining Cycle
存储挖矿流程

Storage miners must continually produce Proofs of SpaceTime over their storage to convince the network that they are actually storing the sectors that they have committed to. Each PoSt covers a miner's entire storage.

存储矿商必须不断提供其存储的时空证明，以使网络相信，它们实际上是在存储它们所承诺的扇区。每个时空证明都包含了一个矿工的全部存储。

#### Step 0: Registration
第 0 步：注册

To initially become a miner, a miner first register a new miner actor on-chain. This is done through the storage market actor's [`CreateMiner`](actors.md#createstorageminer) method. The call will then create a new miner actor instance and return its address.

为了最初成为一个矿工，矿工首先在chain上注册一个新的矿工角色。这是通过存储市场参与者的[`CreateMiner`](actors.md#createstorageminer)方法完成的。然后，调用将创建一个新的miner actor实例并返回它的地址。

The next step is to place one or more storage market asks on the market. This is done through the storage markets [`AddAsk`](actors.md#addask) method. A miner may create a single ask for their entire storage, or partition their storage up in some way with multiple asks (at potentially different prices).

下一步是将一个或多个存储市场需求放到市场上。这是通过存储市场[`AddAsk`](actors.md#addask)方法完成的。一个矿商可能创建一个对其整个存储的请求，或者以某种方式用多个请求(价格可能不同)分割存储。

After that, they need to make deals with clients and begin filling up sectors with data. For more information on making deals, see the section on [deal](storage-market.md#deal).

之后，他们需要与客户进行交易，并开始用数据填充行业。有关交易的更多信息，请参见[deal](storage-market.md#deal)一节。

When they have a full sector, they should seal it. This is done by invoking [`PoRep.Seal`](proofs.md#seal) on the sector.

当他们有一个完整的扇区，他们应该密封它。这是通过调用[`PoRep.Seal`](proofs.md#seal)完成。

#### Step 1: Commit
第 1 步：确认

When the miner has completed their first seal, they should post it on-chain using [CommitSector](actors.md#commitsector). If the miner had zero committed sectors prior to this call, this begins their proving period.

当矿工完成第一个密封后，他们应该用[CommitSector](actors.md#commitsector)方法将它发布到链上。

The proving period is a fixed amount of time in which the miner must submit a Proof of Space Time to the network.

证明周期是矿工需要不断向网络提交时空证明的一个固定的时间周期。

During this period, the miner may also commit to new sectors, but they will not be included in proofs of space time until the next proving period starts.

在这个周期中，矿工也可以确认新的扇区，但需要等到下一个证明周期开始后才能被包含在时空证明中。

TODO: sectors need to be globally unique. This can be done either by having the seal proof prove the sector is unique to this miner in some way, or by having a giant global map on-chain is checked against on each submission. As the system moves towards sector aggregation, the latter option will become unworkable, so more thought needs to go into how that proof statement could work.

TODO: 扇区需要具有全球独特性。要做到这一点，要么有密封证明，以某种方式证明该扇区是属于唯一的矿工的，或有一个巨大的全局的链上映射来检查每一个提交。随着系统朝着部门聚合的方向发展，后一种选择将变得不可行的，因此需要更多的思考来研究证明语句如何工作。

#### Step 2: Proving Storage (PoSt creation)
第 2 步: 证明存储（PoSt创建）

At the beginning of their proving period, miners collect the proving set (the set of all live sealed sectors on the chain at this point), and then call `ProveStorage`. This process will take the entire proving period to complete.

在证明周期的开始，矿工收集证明集（在这个时间点所有在链上完成密封的扇区），然后调用`ProveStorage`。这个过程将会使用整个验证周期来完成。

```go
func ProveStorage(sectors []commR, startTime BlockHeight) (PoSTProof, []Fault) {
	var proofs []Proofs
	var seeds []Seed
	var faults []Fault
	for t := 0; t < ProvingPeriod; t += ReseedPeriod {
		seeds = append(seeds, GetSeedFromBlock(startTime+t))
		proof, fault := GenPost(sectors, seeds[t], vdfParams)
		proofs = append(proofs, proof)
		faults = append(faults, fault)
	}
	return GenPostSnark(sectors, seeds, proofs), faults
}
```

Note: See ['Proof of Space Time'](proofs.md#proof-of-space-time) for more details.

The proving set remains consistent during the proving period. Any sectors added in the meantime will be included in the next proving set, at the beginning of the next proving period.

在证明期间，证明集保持一致。在下一个验证周期开始时，在此期间添加的任何扇区都将包含在下一个验证集中。

#### Step 3: PoSt Submission
第 3 步: PoSt 提交

When the miner has completed their PoSt, they must submit it to the network by calling [SubmitPoSt](actors.md#submitpost). There are two different times that this *could* be done.

当矿工完成PoSt，他们必须通过调用[SubmitPoSt](actors.md#submitpost)方法来向网络提交PoSt。有两种不同的 *可能* 完成的时间。

1. **Standard Submission**: A standard submission is one that makes it on-chain before the end of the proving period. The length of time it takes to compute the PoSts is set such that there is a grace period between then and the actual end of the proving period, so that the effects of network congestion on typical miner actions is minimized.

1. **标准提交**: 一个标准的提交是在证明周期结束前在链上提交。计算post所需的时间长度设置为从那时到验证期的实际结束之间有一个宽限期，这样就可以将网络拥塞对典型矿工操作的影响降到最低。

2. **Penalized Submission**: A penalized submission is one that makes it on-chain after the end of the proving period, but before the generation attack threshold. These submissions count as valid PoSt submissions, but the miner must pay a penalty for their late submission. (See '[Faults](faults.md)' for more information)
   - Note: In this case, the next PoSt should still be started at the beginning of the proving period, even if the current one is not yet complete. Miners must submit one PoSt per proving period.

2. **应处罚提交**: 惩罚提交是指在验证期结束后，但在生成攻击阈值之前使其处于链上的提交。这些意见书被视为有效的意见书，但矿工必须为他们的逾期提交支付罚款。（参阅[Faults](faults.md)' 了解更多信息)
   - 提示：这种情况，下一个PoSt应在证明周期之初开始，即使当前的证明提交没有完成。矿工必须在每个证明周期内提交PoSt。

Along with the PoSt submission, miners may also submit a set of sectors that they wish to remove from their proving set. This is done by selecting the sectors in the 'done' bitfield passed to `SubmitPoSt`.

随着PoSt的提交，矿工也会提交他们希望从证明集中删去的一部分扇区。这是通过选择“完成”位字段中传递给“SubmitPoSt”的扇区来完成的。

### Stop Mining
停止挖矿

In order to stop mining, a miner must complete all of its storage contracts, and remove them from their proving set during a PoSt submission. A miner may then call [`DePledge()`](actors.md#depledge) to retrieve their collateral. `DePledge` must be called twice, once to start the cooldown, and once again after the cooldown to reclaim the funds. The cooldown period is to allow clients whose files have been dropped by a miner to slash them before they get their money back and get away with it.

为了停止挖矿，矿工必须完成所有的存储契约，并在提交PoSt时将它们从证明集中删除。然后，矿工可以调用[`DePledge()`](actors.md#depledge)来取回他们的抵押品。“取消质押”必须调用两次，一次启动冷却，一次冷却后再次收回资金。“冷却期”是让那些文件被矿商删除的客户在拿回自己的钱并安然无恙之前删除文件。

### Faults
故障

Faults are described in the [faults document](faults.md).

故障在 [faults document](faults.md) 中描述。

### On Being Slashed (WIP, needs discussion)
关于被削减（持续完善中，需要讨论）

If a miner is slashed for failing to submit their PoSt on time, they currently lose all their pledge collateral. They do not necessarily lose their storage collateral. Storage collateral is lost when a miner's clients slash them for no longer having the data. Missing a PoSt does not necessarily imply that a miner no longer has the data. There should be an additional timeout here where the miner can submit a PoSt, along with 'refilling' their pledge collateral. If a miner does this, they can continue mining, their mining power will be reinstated, and clients can be assured that their data is still there.

如果一家矿商因为未能按时提交帖子而被裁掉，他们目前将失去所有的质押抵押品。它们并不一定会失去它们的存储抵押品。当矿商的客户因为不再拥有数据而削减存储抵押品时，存储抵押品就会丢失。缺少一篇文章并不一定意味着矿工不再拥有数据。这里应该有一个额外的暂停时间，矿商可以提交一篇帖子，同时“补充”他们的质押抵押品。如果矿工这样做，他们可以继续挖掘，他们的挖掘能力将恢复，客户可以放心，他们的数据仍然在那里。

TODO: disambiguate the two collaterals across the entire spec

TODO: 在整个规范中消除两个抵押品的歧义。

Review Discussion Note: Taking all of a miners collateral for going over the deadline for PoSt submission is really really painful, and is likely to dissuade people from even mining filecoin in the first place (If my internet going out could cause me to lose a very large amount of money, that leads to some pretty hard decisions around profitability). One potential strategy could be to only penalize miners for the amount of sectors they could have generated in that timeframe.

回顾讨论提示: 接管所有的矿工抵押品后提交的最后期限是真的痛苦,甚至可能会阻止人们挖掘filecoin首先(如果我的互联网出去可能会导致我失去了大量的钱,导致一些非常艰难的决定盈利能力)。一种潜在的策略可能是，只根据矿业公司在这段时间内能够创造的行业数量对它们进行惩罚。

## Mining Blocks
挖出区块

Having registered as a miner, it's time to start making and checking tickets. At this point, the miner should already be running chain validation, which includes keeping track of the latest [TipSets](./expected-consensus.md#tipsets) seen on the network.

注册成为矿工后，就可以开始制作和检查 ticket。此时，miner应该已经在运行链验证，其中包括跟踪网络上最新的 [TipSets](./expected-consensus.md#tipsets)。

For additional details around how consensus works in Filecoin, see the [expected consensus spec](./expected-consensus.md). For the purposes of this section, there is a consensus protocol (Expected Consensus) that guarantees a fair process for determining what blocks have been generated in a round, whether a miner should mine a block themselves, and some rules pertaining to how "Tickets" should be validated during block validation.

有关共识如何在Filecoin中工作的更多细节，请参见 [expected consensus spec](./expected-consensus.md)。对于本节的目的，有一个共识协议(Expected consensus)，它保证了一个公平的过程，用于确定在一轮中生成了哪些块，采矿者是否应该自己挖掘一个块，以及关于在块验证期间应该如何验证“Ticket”的一些规则。

### Receiving Blocks
接收区块

When receiving blocks from the network (via [block propagation](data-propagation.md)), a miner must do the following:
当从网络上接收区块（通过[block propagation](data-propagation.md)），矿工必须做如下工作：

1. Check their validity (see [below](#block-validation)).
1. 校验他们的正确性(see [below](#block-validation))。

2. Assemble a TipSet with all valid blocks with common parents and the same number of tickets in their `Tickets` array.
2. 用所有有效的块和相同的父块组装一个 TipSet，并在它们的 `Tickets` 数组中使用相同数量的 tickets。

A miner may sometimes receive blocks belonging to different TipSets (i.e. whose parents are not the same). In that case, they must choose which TipSet to mine on.

矿工有时可能收到属于不同提示集的块(即其父提示集不相同)。在这种情况下，他们必须选择使用哪个TipSet。

Chain selection is a crucial component of how the Filecoin blockchain works. Every chain has an associated weight accounting for the number of blocks mined on it and so the power (storage) they track. It is always preferable to mine atop a heavier TipSet rather than a lighter one. While a miner may be foregoing block rewards earned in the past, this lighter chain is likely to be abandoned by other miners forfeiting any block reward earned as miners converge on a final chain. For more on this, see [chain selection](./expected-consensus.md#chain-selection) in the Expected Consensus spec.

链选择是Filecoin区块链工作原理的一个重要组成部分。每条链都有一个相关的权重，用于计算其上开采的块数，因此它们跟踪的电力(存储)。它总是比我的在一个较重的顶部，而不是一个较轻的。虽然矿工可能会放弃过去获得的区块奖励，但这条较轻的链条可能会被其他矿工放弃，因为当矿工聚集在最后一条链条上时，他们将失去获得的区块奖励。有关这方面的更多信息，请参见预期共识规范中的[chain selection](./expected-consensus.md#chain-selection)。

### Block Validation
区块验证

The block structure and serialization is detailed in [the datastructures spec - block](data-structures.md#block). Check there for details on fields and types.

区块结构和序列化的详细说明在[the datastructures spec - block](data-structures.md#block)。查看详细的字段和类型。

In order to validate a block coming in from the network at round `N` was well mined a miner must do the following:

为了验证来自网络上的第N轮区块被挖出，必须做下列工作：

```go
func VerifyBlock(blk Block) {
	// 1. Verify Signature
	pubk := GetPublicKey(blk.Miner)
	if !ValidateSignature(blk.Signature, pubk, blk) {
		Fatal("invalid block signature")
	}

	// 2. Verify ParentWeight
	if blk.ParentWeight != ComputeWeight(blk.Parents) {
		Fatal("invalid parent weight")
	}

	// 3. Verify Tickets
	if !VerifyTickets(blk) {
		Fatal("tickets were invalid")
	}

	// 4. Verify ElectionProof
	randomnessLookbackTipset := RandomnessLookback(blk)
	lookbackTicket := minTicket(randomnessLookbackTipset)
	challenge := sha256.Sum(lookbackTicket)

	if !ValidateSignature(blk.ElectionProof, pubk, challenge) {
		Fatal("election proof was not a valid signature of the last ticket")
	}

	powerLookbackTipset := PowerLookback(blk)
	minerPower := GetPower(powerLookbackTipset.state, blk.Miner)
	totalPower := GetTotalPower(powerLookbackTipset.state)
	if !IsProofAWinner(blk.ElectionProof, minerPower, totalPower) {
		Fatal("election proof was not a winner")
	}

	// 5. Verify StateRoot
	state := GetParentState(blk.Parents)
	for i, msg := range blk.Messages {
		receipt := ApplyMessage(state, msg)
		if receipt != blk.MessageReceipts[i] {
			Fatal("message receipt mismatch")
		}
	}
	if state.Cid() != blk.StateRoot {
		Fatal("state roots mismatch")
	}
}
```

Ticket validation is detailed as follows:

Ticket 验证详细方法被如下：

```go
func RandomnessLookback(blk Block) TipSet {
	return chain.GetAncestorTipset(blk, K)
}

func PowerLookback(blk Block) TipSet {
	return chain.GetAncestorTipset(blk, L)
}

func IsProofAWinner(p ElectionProof, minersPower, totalPower Integer) bool {
	return Integer.FromBytes(sha256.Sum(p))*totalPower < minersPower*2^32
}

func VerifyTickets(b Block) error {
	// Start with the `Tickets` array
	// get the smallest ticket from the blocks parent tipset
	parTicket := GetSmallestTicket(b.Parents)

	// Verify each ticket in the chain of tickets. There will be one ticket
	// plus one ticket for each failed election attempt.
	for _, ticket := range b.Tickets {
		challenge := sha256.Sum(parTicket.Signature)

		// Check VDF
		if !VerifyVDF(ticket.VDFProof, ticket.VDFResult, challenge) {
			return "VDF was not run properly"
		}

		// Check VRF
		pubk := getPublicKeyForMiner(b.Miner)
		if !VerifySignature(ticket.Signature, pubk, ticket.VDFResult) {
			return "Ticket was not a valid signature over the parent ticket"
		}
		// in case mining this block generated multiple tickets
		parTicket = ticket
	}

	return nil
}
```


If all of this lines up, the block is valid. The miner repeats this for all blocks in a TipSet, and for all TipSets formed from incoming blocks.

如果所有这些都对齐，那么块就是有效的。采集器将对TipSet中的所有块以及从传入块形成的所有TipSet重复此操作。

Once they've ensured all blocks in the heaviest TipSet received were properly mined, they can mine on top of it. If they weren't, the miner may need to ensure the next heaviest `Tipset` was properly mined. This might mean the same `Tipset` with invalid blocks removed, or an altogether different one.


一旦他们确保收到的最重的情报集中的所有区块都被正确地开采了，他们就可以在上面开采了。如果他们没有，矿工可能需要确保下一个最重的“Tipset”被正确开采。这可能意味着删除无效块的相同“Tipset”，或者是完全不同的“Tipset”。

If no valid blocks are received, a miner may mine atop the same `TipSet` running leader election again using the next ticket in the ticket chain, and also generating a [new ticket](./expected-consensus.md#losing-tickets) in the process (see the [expected consensus spec](./expected-consensus.md) for more).

如果没有收到有效的块，矿工可以使用选票链中的下一个选票在相同的“提示集”上再次挖掘，并在此过程中生成[new ticket](./expected-consensus.md#losing-tickets)(更多信息请参见[expected consensus spec](./expected-consensus.md)。

### Ticket Generation
Ticket 产生

For details of ticket generation, see the [expected consensus spec](./expected-consensus.md#ticket-generation).
了解更多，参阅 [expected consensus spec](./expected-consensus.md#ticket-generation)。

Ticket generation is the twin process of leader election (i.e. generating `ElectionProof`s). Every ticket scratch (i.e. round of leader election) has the miner generate a new ticket to include in the `Tickets` array of their block.

产生选票是领导人选举的双重过程(即产生“选举证明”)。每个ticket scratch(即一轮领导人选举)都会让矿工生成一个新的ticket，并将其包含在其块的 `Tickets` 数组中。

New tickets are generated using the smallest ticket from the parent TipSet at height `H` and added to the `Tickets` array. Generating a new ticket will take some amount of time (as imposed by the VDF in Expected Consensus).

使用父TipSet中高度为 `H` 的最小票券生成新票券，并将其添加到 `Tickets` 数组中。生成一个新的票证将需要一些时间(正如 VDF 在预期的共识中所规定的那样)。

Because of this, on expectation, as it is produced, the miner will hear about other blocks being mined on the network. By the time they have generated their new ticket, they can check whether they themselves are eligible to mine a new block.

正因为如此，当它被生成时，矿工将监听网络上正在开采的其他块。当他们生成新的ticket时，他们可以检查自己是否有资格挖出一个新的区块。

If the lookback ticket yields a valid `ElectionProof`, the miner publishes their block (see [block generation](#block-creation)) including the new ticket and earns a block reward. They then assemble a new TipSet using any valid blocks they heard about while generating the ticket  (likely of height `H+1`) and mine atop the smallest ticket in that new TipSet.

如果回溯票证生成一个有效的 `ElectionProof`，矿工将发布他们的块(参见[block generation](#block-creation))，包括新ticket，并获得块奖励。然后，他们使用他们在生成ticket(高度可能为`H+1`)时所监听到的任何有效块组装一个新的票证集，并在新票证集中最小的票证上使用我的票证。

### Scratching a losing ticket
抓一张丢掉的ticket

If a miner fails to generate a valid `ElectionProof` using their lookback ticket, they may not yet publish a new block at height `H+1`. The miner will likely hear about other blocks being mined on the network at height `H+1` and can thus assemble a new TipSet to mine off of with these blocks (as above).

如果矿工未能使用其回退票生成有效的“选举证明”，则他们可能尚未在高度为“H+1”处发布新块。矿工者可能会监听到在网络上以“H+1”高度开采其他区块，因此可以用这些区块装配一个新的采矿TipSet(如上所述)。

If the miner hears of no new blocks, they must instead draw a new ticket to scratch in order to try leader election again (as other miners will). In order to do so, they must generate a new ticket once more.

如果矿工没有听说有新的开采区块，他们就必须重新抽签，以便再次尝试选举领导人(其他矿工也会这样做)。为了做到这一点，他们必须再次生成一个新的票据。

Now, rather than generating this new ticket from the smallest ticket from the parent TipSet (as above), the miner will instead use their ticket from the last round, now in the `Tickets` array.

现在，不是从父TipSet的最小Ticket生成这个新Ticket(如上所述)，而是使用上一轮的ticket，即现在在`Tickets`数组中。

This process is repeated until either a winning ticket is found (and block published) or a new valid TipSet comes in from the network. If a new TipSet comes in from the network, and it is heavier chain than the miner's own, they should abandon their process to mine atop this new block. Due to the way chain selection works in filecoin, a chain with more blocks will be preferred (see the [Expected Consensus spec](./expected-consensus.md#chain-selection) for more details).

此过程将重复，直到找到中奖彩票(并发布块)或新的有效提示集从网络中传入为止。如果一个新的TipSet来自网络，并且它比矿工自己的更重的链，他们应该放弃他们的进程，在这个新的块上采矿。由于在filecoin中链选择的工作方式，将首选具有更多块的链(请参阅[Expected Consensus spec](./expected-consensus.md#chain-selection)以获得更多细节)。

The `Tickets` array in the block to be published grows with each round (and a new ticket generated).

要发布的块中的 `Tickets` 数组随着每一轮的生成而增长(并生成一个新的ticket)。

### Block Creation
区块创建

Scratching a winning ticket, and armed with a valid `ElectionProof`, a miner can now publish a new block!

抓着一张中奖彩票，拿着有效的“选举证明”，矿工现在可以发布一个新的区块了!

To create a block, the eligible miner must compute a few fields:

为了创建一个区块，合格的矿工必须进行如下计算：

- `Tickets` - An array containing a new ticket, and, if applicable, any intermediary tickets generated to prove appropriate delay for any failed election attempts. See [ticket generation](./expected-consensus.md#ticket-generation).
- `ElectionProof` - A signature over the final ticket from the `Tickets` array proving. See [ticket generation](./expected-consensus.md#ticket-generation).
- `ParentWeight` - As described in [Chain Weighting](./expected-consensus.md#chain-weighting).
- `ParentState` - Note that it will not end up in the newly generated block, but is necessary to compute to generate other fields. To compute this:
  - Take the `ParentState` of one of the blocks in the chosen parent set (invariant: this is the same value for all blocks in a given parent set).
  - For each block in the parent set, ordered by their tickets:
    - Apply each message in the block to the parent state, in order. If a message was already applied in a previous block, skip it.
    - Transaction fees are given to the miner of the block that the first occurance of the message is included in. If there are two blocks in the parent set, and they both contain the exact same set of messages, the second one will receive no fees.
    - It is valid for messages in two different blocks of the parent set to conflict, that is, A conflicting message from the combined set of messages will always error.  Regardless of conflicts all messages are applied to the state.
    - TODO: define message conflicts in the state-machine doc, and link to it from here
- `MsgRoot` - To compute this:
  - Select a set of messages from the mempool to include in the block.
  - Insert them into a Merkle Tree and take its root.
- `StateRoot` - Apply each chosen message to the `ParentState` to get this.
- `ReceiptsRoot` - To compute this:
  - Apply the set of messages selected above to the parent state, collecting invocation receipts as this happens.
  - Insert them into a Merkle Tree and take its root.
- `BlockSig` - A signature with the miner's private key (must also match the ticket signature) over the entire block. This is to ensure that nobody tampers with the block after it propagates to the network, since unlike normal PoW blockchains, a winning ticket is found independently of block generation.

- `Tickets` - 包含新ticket的数组，以及(如果适用)为证明对任何失败的选举尝试有适当延迟而生成的任何中间ticket。参考 [ticket generation](./expected-consensus.md#ticket-generation)。
- `ElectionProof` - 一个在从 `tickets` 数组证明中选出的最终ticket的签名。参考 [ticket generation](./expected-consensus.md#ticket-generation)。
- `ParentWeight` - 如 [Chain Weighting](./expected-consensus.md#chain-weighting) 的描述。
- `ParentState` - 注意它不会结束在新生成的块，但必须计算生成其他字段。计算:
	- 取选定父集合中一个块的`ParentState`(不变式:对于给定父集合中的所有块，这个值都是相同的)。
	- 对于父集合中的每个块，按它们的票排序:
		- 按顺序将块中的每个消息应用到父状态。如果消息已经应用于前一个块，则跳过它。
		- 交易费用是给予矿商的块，其中第一次出现的消息是包括在。如果父集合中有两个块，并且它们都包含完全相同的消息集，则第二个块将不收取任何费用。
		- 对于父集中两个不同块中的消息冲突是有效的，也就是说，来自组合消息集的冲突消息总是错误的。不管冲突如何，所有消息都应用于状态。
		- TODO: 在状态机文档中定义消息冲突，并从这里链接到它
- `MsgRoot` - 计算这个:
	- 从mempool中选择要包含在块中的一组消息。
	- 把它们插到一棵梅克尔树上，然后生根。
- `StateRoot` - 将每个选择的消息应用到`ParentState`以得到这个。
- `ReceiptsRoot` - 计算这个:
	- 将上面选择的消息集应用于父状态，在此过程中收集调用收据。
	- 把它们插到一棵梅克尔树上，然后生根。
- `BlockSig` - 一个签名与矿工的私钥(也必须匹配的门票签名)在整个区块。这是为了确保在块传播到网络之后没有人篡改它，因为与普通PoW块链不同，中奖彩票是独立于块生成找到的。

An eligible miner can start by filling out `Parents`, `Tickets` and `ElectionProof` with values from the ticket checking process.

一名合格的矿工可以从填写 `Parents`, `Tickets` 和 `ElectionProof` 开始，填上验票过程中的值。

Next, they compute the aggregate state of their selected parent blocks, the `ParentState`. This is done by taking the aggregate parent state of the blocks' parent TipSet, sorting the parent blocks by their tickets, and applying each message in each block to that state. Any message whose nonce is already used (duplicate message) in an earlier block should be skipped (application of this message should fail anyway). Note that re-applied messages may result in different receipts than they produced in their original blocks, an open question is how to represent the receipt trie of this tipsets 'virtual block'. For more details on message execution and state transitions, see the [Filecoin state machine](state-machine.md) document.

接下来，计算所选父块的聚合状态，即“ParentState”。这是通过获取块的父TipSet的聚合父状态，根据它们的票证对父块进行排序，并将每个块中的每个消息应用于该状态来实现的。在前面的块中已经使用了nonce的任何消息(重复消息)都应该跳过(无论如何，此消息的应用程序都应该失败)。请注意，重新应用的消息可能会产生不同于它们在原始块中生成的收据，一个开放的问题是如何表示这个tips集“虚拟块”的收据trie。有关消息执行和状态转换的详细信息，请参阅[Filecoin state machine](state-machine.md)文档。

Once the miner has the aggregate `ParentState`, they must apply the mining reward. This is done by adding the correct amount to the miner owner's account balance in the state tree. See [block reward](#block-rewards) for details.

一旦矿工有了聚合的“ParentState”，他们必须申请采矿奖励。这是通过在状态树中将正确的金额添加到矿商所有者的帐户余额来实现的。详情请参见[block reward](#block-rewards)。

Now, a set of messages is selected to put into the block. For each message, the miner subtracts `msg.GasPrice * msg.GasLimit` from the sender's account balance, returning a fatal processing error if the sender does not have enough funds (this message should not be included in the chain).

现在，将选择一组消息放入块中。对于每条消息，矿机都会减去 `msg.GasPrice * msg.GasLimit`。如果发送方没有足够的资金，则返回一个致命的处理错误(此消息不应包含在链中)。

They then apply the messages state transition, and generate a receipt for it containing the total gas actually used by the execution, the executions exit code, and the return value (see [receipt](data-structures.md#message-receipt) for more details). Then, they refund the sender in the amount of `(msg.GasLimit - GasUsed) * msg.GasPrice`. In the event of a message processing error, the remaining gas is refunded to the user, and all other state changes are reverted. (Note: this is a divergence from the way things are done in Ethereum)

然后，它们应用消息状态转换，并为其生成一个包含执行实际使用的总气体、执行退出代码和返回值的收据(有关详细信息，请参见[receipt](data-structures.md#message-receipt))。然后，他们退还发件人的金额 `(msg.GasLimit - GasUsed) * msg.GasPrice`。在消息处理出错的情况下，剩余的气体将退还给用户，所有其他状态更改都将恢复。(注意:这与在以太坊中做事的方式不同)

Each message should be applied on the resultant state of the previous message execution, unless that message execution failed, in which case all state changes caused by that message are thrown out. The final state tree after this process will be the block's `StateRoot`.

每个消息都应该应用于前一个消息执行的结果状态，除非该消息执行失败，否则将抛出由该消息引起的所有状态更改。这个过程之后的最终状态树将是块的 `StateRoot`。

The miner merklizes the set of messages selected, and put the root in `MsgRoot`. They gather the receipts from each execution into a set, merklize them, and put that root in `ReceiptsRoot`. Finally, they set the `StateRoot` field with the resultant state.

矿工merklizes的一组选定的消息，并把根放在 `MsgRoot`。他们把每次执行的收据收集成一组，然后把它们合并起来，再把那个根放进 `ReceiptsRoot` 中。最后，他们用结果状态设置了 `StateRoot` 字段。

Note that the `ParentState` field from the expected consensus document is left out, this is to help minimize the size of the block header. The parent state for any given parent set should be computed by the client and cached locally.

注意，预期共识文档中的 `ParentState` 字段被省略了，这是为了帮助最小化块头的大小。任何给定父集的父状态都应该由客户机计算并在本地缓存。

Now the block is complete, all that's left is to sign it. The miner serializes the block now (without the signature field), takes the sha256 hash of it, and signs that hash. They place the resultant signature in the `BlockSig` field.

现在代码块完成了，剩下的就是签名。矿工现在序列化块(没有signature字段)，获取它的sha256哈希值，并对该哈希值进行签名。他们将生成的签名放在 `BlockSig` 字段中。

#### Block Broadcast
区块广播

An eligible miner broadcasts the completed block to the network (via [block propagation](data-propagation.md)), and assuming everything was done correctly, the network will accept it and other miners will mine on top of it, earning the miner a block reward!

一个合格的采矿者将完成的块广播到网络(通过[block propagation](data-propagation.md)），并且假设所有操作都正确，网络将接受它，其他采矿者将在它上面开采，从而为采矿者赢得块奖励!

### Block Rewards
区块奖励

Over the entire lifetime of the protocol, 1,400,000,000 FIL (`TotalIssuance`) will be given out to miners. The rate at which the funds are given out is set to halve every six years, smoothly (not a fixed jump like in Bitcoin). These funds are initially held by the network account actor, and are transferred to miners in blocks that they mine. The reward amount remains fixed for a period of 1 week (given our 30 second block time, this  is 20,160 blocks, the `AdjustmentPeriod`) and is then adjusted. Over time, the reward will eventually become zero as the fractional amount given out at each step shrinks the network account's balance to 0.

在《议定书》的整个生命周期内，将向矿工发放14亿 FIL (`TotalIssuance`)。基金的发放速度将平滑地每6年减半(不像比特币那样出现固定的阶跃)。这些资金最初由网络帐户参与者持有，并以他们开采的区块的形式转移给矿工。奖励金额保持固定，为期一周(考虑到我们的30秒块时间，这是20,160块，“调整期”)，然后进行调整。随着时间的推移，奖励最终将变为零，因为在每一步给出的分数金额将网络帐户的余额缩减到0。

The equation for the current block reward is of the form:

当前区块奖励的公式为:

```
Reward = IV * (Decay ^ (BlockHeight / 20160))
```

`IV` is the initial value, and is computed by taking:

`IV` 是初始值，由以下公式计算得到:

```
IV = TotalIssuance * (1 - Decay)
```

`Decay` is computed by:

```
Decay = e^(ln(0.5) / (HalvingPeriodBlocks / AdjustmentPeriod))
```

```
// Given one block every 30 seconds, this is how many blocks are in six years
HalvingPeriodBlocks = 6 * 365 * 24 * 60 * 2
```

Note: Due to jitter in EC, and the gregorian calendar, there may be some error in the issuance schedule over time. This is expected to be small enough that it's not worth correcting for. Additionally, since the payout mechanism is transferring from the network account to the miner, there is no risk of minting *too much* FIL.

注: 由于EC和公历的抖动，随着时间的推移，发行日程可能会出现一些错误。这个值应该足够小，不值得纠正。此外，由于支付机制正从网络账户转移到矿商，因此不存在产生 *过多* 收益的风险。

### Open Questions
开放的问题

- How should receipts for tipsets 'virtual blocks' be referenced? It is common for applications to provide the merkleproof of a receipt to prove that a transaction was successfully executed.

- Tipsets'虚拟区块'的收据应该如何被引用？ 应用程序通常提供收据的 merkleproof 来证明交易成功执行。

### Future Work
未来的工作

There are many ideas for improving upon the storage miner, here are ideas that may be potentially implemented in the future.

对于改进storage miner有很多想法，下面是将来可能实现的一些想法。

- **Sector Resealing**: Miners should be able to 're-seal' sectors, to allow them to take a set of sectors with mostly expired pieces, and combine the not-yet-expired pieces into a single (or multiple) sectors.

- **扇区密封**: 矿工应该有能力“再密封”扇区，来允许他们来获取一组包含失效的分片的扇区，将尚未失效的分片组成到一个（或多个）扇区中。

- **Sector Transfer**: Miners should be able to re-delegate the responsibility of storing data to another miner. This is tricky for many reasons, and will not be implemented in the initial release of Filecoin, but could provide interesting capabilities down the road.

- **扇区转移**: 矿工应该能够再次委托数据存储责任给另一个矿工。这个棘手，由于许多原因，并不会在filecoin初始的版本中实现，但是可以作为后续的一个的有趣的功能。
