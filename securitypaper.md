**Bitcoin Layer 2 Executive Summary**

**Lightning Network Security Overview**

_by RecklessApotheosis_

> Table of Contents
* [Revisions](#revisions)
* [Contributions](#contributions)
* [Disclosures](#disclosures)
* [Terminology](#terminology)
* [Introductions](#introductions)
  * [Bitcoin](#bitcoin)
  * [Layer 2](#layer-2)
  * [Lightning](#lightning)
* [Security Considerations](#security-considerations) 
  * [Infrastructure](#infrastructure)
  * [Node Operation](#node-operation)
  * [Wallet Operation](#wallet-operation)
  * [Social Engineering](#social-engineering)
  * [Examples](#examples)
* [Conclusion](#conclusion)
* [Additional Reading](#additional-reading)
* [Endnotes](#endnotes)

# Revisions

* 2023-02-12 – Initial Draft
* 2023-02-14 - Updates from Contributors, footnote formatting fixes
* 2023-02-26 - Additional content

# Contributions
Thanks to the Lightning innovators who assisted with peer review of this document:
* @fiatjaf - lntxbot, lnbits, & nostr Developer
* @Claytantor - Rapaygo Founder
* @RecklessApotheosis - author of this document & noderunner
* @EricYHF - High Availability Noderunner
* @DarthCoin - Author of Multiple Lightning Guides
* @GaryHoward - Bitcoin Engineer working on Blockscale

# Terminology
* BTC - short for Bitcoin
* Channel - the term for the 2-of-2 multisignature transaction that creates the smart contract that establishes an open funding channel between two nodes.
* Eltoo - A proposed enforcement layer for LN that allows any later channel state to replace any earlier channel state.
* Cold Wallet - Typically reffered to as a Bitcoin private key or seed phrase which may or may not be attached to software, but whose hosting device is not ever connected to the Internet. More broadly some users will permit usage of the phrase cold wallet to also refer to hardware wallets which require physical access to transmit funds, even though they are intermittently attached to the Internet. Operating a Lightning Node funded from a cold wallet is possible but presents technical challenges.
* Hot Wallet - As above, but a wallet software or device which keeps a decrypted private key in memory, at the ready to transact Bitcoin in realtime. Often specific prviate key signing operations will still require additional verification: Two Factor Authentication, a PIN, a passphrase, etc. Typically a live Lightning Node operates on top of a hot wallet.
* Custodial Wallet - A Bitcoin on-chain wallet where all of the funds are held on public keys for which the owner does not hold the private keys or seedphrase. For Lightning, this often refers to Bitcoin on the Lightning network that are held in a Lightning Wallet on a channel hosted by a node that the owner does not operate nor have access to close the channel.
* Non-Custodial Wallet - A Bitcoin or Lightning Wallet where the seedphrase for the funds on the wallet, and any hot Lightning channels opened, are in full control of the person or entity that owns the funds. There is no 3rd party "custodian" between the funds and their owner.
* HTCL - acronym for Hashed Timelock Contract, the basis of all Lightning Network off-chain transactions.
* Satoshi - A Satoshi (or sat) is the smallest indivisible on-chain unit of measure of Bitcoin. There are 100 million sats per Bitcoin, so a sat is 100-millionth of a Bitcoin.
* SCB - Acronym for Static Channel Backup, the basis of channel recovery
* Taproot - This is privacy and make it more scalable by allowing multiple parties to sign a single transaction, reducing the time required to open channels.
* Watchtower - A relationship between two nodes to protect both nodes from participants attempt to defraud a node by broadcasting an invalid state.

# Disclosures

The author of this summary is neither a programmer nor a professional security researcher. I have been using the Lightning Network for payments since Mach of 2019, and have been operating a mid-sized Lightning Routing Node<sup>[i](https://amboss.space/c/reckless)</sup> since block 686061 in June of 2021.

This is a best effort high level assessment of the potential risk exposure at running a Lightning Node and conducting business on the Lightning Network. Maintaining a Bitcoin Node with a balance of Bitcoin is a potentially high risk venture. Operating a Lightning Node is by definition high risk and requires a varying level of technical sophistication, and a high degree of technical and security know-how to execute well enough to avoid common pitfalls for loss of funds. This document is not to be taken as comprehensive, and the environment changes quickly which will result in the contents being outdated relatively rapidly. Never risk more capital than you are willing to lose on any experimental financial tool, and ensure that your network & backup strategies are sound and in place prior to beginning to explore the Lightning Network.

# Introductions

## Bitcoin

Bitcoin is a decentralized currency secured by proof-of-work providing essentially a protocol for economic movement that is disassociated from any single government’s control. Bitcoin blocks are secured by the electrical grids of miners around the globe, which results in a block emission fixed to average 1 block of transactions every 10 minutes (this is an average over time, so some blocks confirm mere seconds after the previous block, and one took 122 minutes!<sup>[ii](https://u.today/bitcoin-just-recorded-longest-time-between-two-blocks-in-almost-10-years)</sup>). The mining difficulty adjusts roughly every 2 weeks, compensating for global hashrate changes, to maintain that 10 minute average confirmation time. Each block can contain a maximum of 4MB of data, which on average is about 2,280<sup>[iii](https://ycharts.com/indicators/bitcoin_average_transactions_per_block)</sup> transactions per block, or 3.8 transactions per second. Bitcoin has a small intentionally non-turing complete language<sup>[iv](https://wiki.bitcoinsv.io/index.php/Opcodes_used_in_Bitcoin_Script)</sup>. The deliberately limited scope of the Bitcoin language serves as a component in its security model. Changes to Bitcoin are presented as either hard-forks or soft-forks come through extensive validation & conversation via the formal BIP<sup>[v](https://github.com/bitcoin/bips)</sup> Process.

## Layer 2

Layer 2 (L2) refers to any technology built upon an existing blockchain (not Bitcoin specific) designed typically to remediate perceived deficiencies or to bypass specific safeguards in the underlying protocol by building atop it, thus leveraging its base value proposition. The increased risk inherent in these second layer solutions requires greater caution with funds kept on them, definitely never expose more capital on an L2 network than you are prepared to lose.

## Lightning

The Lightning Network is a specific L2 implementation built atop the Bitcoin Protocol to reduce the friction present with sending Bitcoin between parties in on-chain transactions. Specifically, Lightning seeks to address the speed with which a transaction may be sent and receipt confirmed (from highly variable and often over 60 minutes, to instantaneous), as well as the expense of sending said transaction (measured in vBytes per Satoshi) from an average of 8,900 sats per transaction on-chain (roughly $1.98) to under 10 sats per transaction off-chain. Final settlement between two nodes always results in an on-chain transaction, which serves to secure the Lightning Network from fraud. At the very outset, the concept of the Lightning Network has been advertised as being a work-in-progress where activity moves rapidly, often with insufficient validation for new features. Lightning development occurs with a modus operandi of “#Reckless” where the risky nature of these enhancements upon Bitcoin Core is acknowledged at every step.

# Security Considerations

Any technology such as Bitcoin introduces multiple new attack vectors on the storage and transmission of currency, many of which are still being discovered and exploits against existing fixes may arise in the future. Keeping the overall approach as simple as possible and leveraging existing security models as well as is possible is key to keeping Bitcoin secure from a financial markets perspective. Thus, by necessity, building a second layer of complexity atop Bitcoin introduces additional potential attack surfaces. We will attempt to introduce and explore some of those here.

There are two significant attack vectors that are common between Lightning and Bitcoin: Operating secure infrastructure for the safe storage of a wallet and funds, and safely and securely exchanging Satoshis between two wallets. For on-chain Bitcoin numerous software and hardware wallets exist to simplify the task of holding Bitcoin securely, additionally multi-signature wallets and encrypted seed phrases can improve the security of an account with moderate increases to the complexity of doing so. Likewise, an on-chain Bitcoin transaction is secured through a well understood and uncompromisable signing of a transaction, and record of the transaction being recorded immutably on the Bitcoin Blockchain.

## Infrastructure

For Lightning, the infrastructure complexity begins with the Bitcoin Core software running on a host platform where every additional installed piece of software is a potential attack vector. Resting encryption of the private key for the node, and the requirement to keep it decrypted while the system is running makes Lightning Nodes susceptible to network intrusions causing the exfiltration of funds if not properly locked down<sup>[vi](https://www.computer.org/csdl/proceedings-article/csf/2020/09155145/1m1jOxuJKF2)</sup>. Due to the dynamic nature of operating a Lightning Node, the underlying support software changes more rapidly and the software dependencies list grows regularly. These changes also result in system demand characteristics changing more rapidly than with Bitcoin Core. Whereas Bitcoin grows at a rate of roughly 50GB per year, the introduction of Lightning transactions can dramatically increase the storage requirements of the hardware as well as the CPU requirements, as more tasks are taken on by the node operator.

## Node Operation

As an explorative technology, Lightning also has a proposal system like Bitcoin’s BIPS, called BOLTs<sup>[vii](https://github.com/lightning/bolts)</sup> but unlike Bitcoin, lacks the rigorous demands for backward or cross-compatibility, and is considered (as a more experimental platform) acceptable to make big changes more rapidly. There are currently two main node servers used in the hobbyist & mid-sized node operation space: LND by Lightning Labs and CLN by Core Lightning. Additionally, there are more specialized Lightning implementations used by larger service providers and other more custom applications, examples of these would be LDK and Eclair. These Lightning implementations have minor inconsistencies in which BOLTs they implement, and the manner by which their supported BOLTs are implemented, causing interoperability issues at times.

For example, both LND and CLN support opening balanced channels, however, a balanced Lightning Channel cannot currently be opened between an LND-based server and a CLN-based sever. Additionally, in a commercial node there are considerations for routing nodes (designed to provide liquidity to the network) and for custodial nodes (providing wallets for individual users). These considerations involve the installation of additional 3rd party software such as LNDg, balanceofsatoshis, LNBits, electrs plugins, etc. This additional software depends on Macaroons<sup>[viii](https://docs.lightning.engineering/the-lightning-network/lsat/macaroons)</sup> for authorizing varying levels of security access to wallet operations, a process which is proven secure. However, an inexperienced node operator may create a macaroon for a tool with excessive permissions granted. Likewise inexperienced node operators often opt to save the password to decrypt the node’s private key on disk so it unlocks on boot, and often fail to secure the filesystem sufficiently to protect against targeted attacks.

The additional software installed to extend the functionality of the Lightning Node has its own dependencies and is often updated rapidly. Like all modern system administration, many depend on nodejs, npm, or docker hub, and pull in a considerable number of unvetted dependencies. Often, security update notices come through non-traditional avenues such as Twitter, Telegram, or Substack articles. When bugs are discovered, they are often 0-day exploited by the researcher, causing immediate issues with noderunners across the domain. To their credit, software developers stay on top of the technology, and push out updates within hours of issues being reported. Remediation however then becomes incumbent upon the node operator and failure to rapidly address the issue can result in node downtime, the force closure of channels, or the theoretical loss of funds by a sophisticated attack mechanism (the author is not aware of any such successful exploits as of the writing of this document).

Operating a custodial wallet brings additional concerns, well highlighted by the recent issues with the lntxbot Telegram Lightning Bot that offered the ability to send/receive payments from a custodial wallet within the Telegram ecosystem. The bot operated for 5 years and the tool owner encountered about 5 exploits: 2 because of CLN bugs, 1 eclair bug, and 2 from his own development mistakes. When the bot software stack had a bug that caused recurring memory leaks, it would crash and restart frequently, which caused accounting issues and a poor user experience. The tool owner decided to sunset the bot and begain the process of notifying users who then  experienced challenges & delays in recovering their funds (initially due to what amounted to a bank run on the node's hot wallet) while the author (who maintains the tool in his free time) worked on providing a mechanism for recovery. Issues with running complex software offerings on top of the Lightning server itself benefit from a threat mitigation and security assessment prior to making public, and a proactive set of strategies for software, protocol, or social engineering eventualities.

## Wallet Operation

As a Lighting user, the threat landscape is far more straight-forward. The primary threat vector is from a compromised counterparty or loss of security of the custodial application itself. In the event above with lntxbot, although not a malicious act, the wallet users in this case were subject to delayed access to funds due to a bug with the wallet provider, invoking their anger and some even threatening legal action if access to their funds was not immediately restored. The common phrase amongst the community in this case is, “Not your keys, not your coins”. That is to say, if you rely on a custodial service and do not self-host your own Bitcoin and/or Lightning wallets, then you do not truly own those Satoshis.

## Social Engineering

For the custodial application vector, this is the same as with any other application you use for storage of funds. If the device the software is installed on, the credentials to access the software, or the software itself has a security vulnerability either through social engineering or technical, then funds can be stolen. One of the most common threat vectors is a social engineering attack where the attacker convinces the mark to provide credentials or to pay funds in an apparent effort to verify a balance or by way of a scam to receive additional Bitcoin. The efficacy of such attacks is poorly documented, and presumed rare in the Lightning community due to the current high degree of technical savvy necessary to use the network successfully.

## Examples

There have been numerous technical challenges as the Lightning Network has grown since 2018. These have had varying levels of success, but are extremely well documented across multiple sources<sup>[ix](https://github.com/davidshares/Lightning-Network)</sup>. A few good examples that highlight the types of issues the network faces are presented here.

On October 4th, 2022 the author's node, RecklessApotheosis, had been operating for about a year and a half on a Raspberry Pi with a USB attached 1 Terabyte SSD, using the turn-key Umbrel Lightning Node software. The node was behaving poorly, with frequent software crashes of LND, with transactions failing to route. Often a reboot would resolve the issue. This went on for about 2 weeks, and the author was remiss in taking a more assertive handle on the situation, and on October 4th, when the Pi had locked up fully and required a hard reboot by pulling power, when the node came back up, the bolt channels.db database had been corrupted. Despite significant troubleshooting and forensics on the file and attempts at restoring from backups, the node's current channelstate was irrecoverable. The node was rebuilt on x86 hardware with dual m.2 SSDs using RAID1. After the hardware rebuild, the private keys for the node were restored on the new hardware, and recovery from SCB was begun. As this process force-closed nearly 100 channels, it was a very expensive on-chain event for the node, as well as time consuming, and damaging to the noderunner's community goodwill as a competent noderunner. It took roughly 2 months to rebuild most of the channels lost and get the node back into shape as a routing node. The culprit of the failures on the Pi was a failing USB-C power supply, a relatively inexpensive and easy fix that could have averted months of rebuilding & costs associated with the node failure.

On October 9th, 2022 in Bitcoin block 757922<sup>[x](https://mempool.space/tx/7393096d97bfee8660f4100ffd61874d62f9a65de9fb6acf740c4c386990ef73)</sup> a security researcher known on Twitter as Buraq<sup>[xi](https://twitter.com/brqgoo/status/1579216353780957185)</sup> announced that he had successfully crafted a 998-of-999 tapscript multisig transation. This transaction triggered a bug in the popular LND Lightning server that caused it to fail to synchronize its on-chain wallet at startup, so all nodes rebooted after that transaction would fail to start. The issue was reported<sup>[xii](https://github.com/lightningnetwork/lnd/issues/7002)</sup> to the LND team within minutes, and was fixed and pushed within 5 hours, active nodes had already begun updating from the pending pull request<sup>[xiii](https://github.com/lightningnetwork/lnd/pull/7004)</sup>, and although unattended nodes eventually went dark until their noderunners updated them or shut them down, the event itself was well handled and did not result in significant downtime for LND based nodes. As a research project the news it generated<sup>[xiv](https://protos.com/taproot-bug-freezes-bitcoin-inside-lightning-network-for-hours/)</sup> highlighted the need for improved backup channel options for node operators such as through the employment of watchtower services.

On November 1st, 2022 the same security researcher created a transaction that was included in block 761249<sup>[xv](https://mempool.space/tx/73be398c4bdc43709db7398106609eea2a7841aaf3a4fa2000dc18184faa2a7e)</sup> which utilized a common Bitcoin feature to record witness items into the transaction, but in this case, 500,003 entries were encoded. The researcher announced his activity this time on Twitter<sup>[xvi](https://twitter.com/brqgoo/status/1587397646125260802)</sup>. This transaction’s unrealistically large message size exceeded the maximum hard-coded limit in btcd, a Bitcoin client that LND uses (CLN used a different client and thus was unimpacted). Again, LND nodes were unable to sync, a bug was filed<sup>[xvii](https://github.com/lightningnetwork/lnd/issues/7096)</sup>, and a patch was merged just an hour later<sup>[xviii](https://github.com/lightningnetwork/lnd/pull/7098)</sup>. Upon triage no known loss of funds took place through this event. The security researcher published a good overview of the process he employed<sup>[xix](https://burakkeceli.medium.com/channel-addresses-bd85e9ab8fe1)</sup>, and this encouraged further scrutiny of not just LND, but also its core dependencies.

# Conclusion

The Lightning Network represents a ground-breaking and novel approach to disruption of the legacy financial systems. Concepts like SWIFT and SPFS running international settlement layers, with financial instruments such as the Visa & Master Card networks, and moderately innovative payment gateways like PayPal, Zelle, CashApp,Strike, etc, are all constructed within high trust environments with low amounts of transparency. These more rigorous formal institutions are change adverse and resistant to technologies which may disrupt their extremely lucrative revenue streams through remittance processing and bank settlements.

As a result of being uniquely situated at the inflection point in global finance Bitcoin and by extension the Lightning Network combine many of the challenges facing a moderately sized financial institution performing payment processing of around $8.2 trillion in 2022<sup>[xx](https://coinunited.io/news/en/2023-01-04/crypto/btc-blockchain-processed-over-8-trillion-in-transactions-last-year-as-bitcoin-soars)</sup>. This exposes the Lightning network to the same degree of scrutiny by criminal activity as a banking network. Currently the Lightning Network has over 5,500 Bitcoin locked in channels (over $121m at the time of writing)<sup>[xxi](https://www.theblock.co/post/208817/lightning-network-reaches-all-time-high-in-bitcoin-capacity)</sup> and as that quantity grows, so does the value proposition that a successful technical exploit against the network represents. Although there is a considerable “bounty” on a successful hack, executing such a hack remains extremely challenging due to the underlying security the Bitcoin network provides, the deliberate avoidance of unnecessary complexity within Lightning itself, and the Open Source approach to software maintenance and encouragement of white hat adversarial activities to improve the security of the network.

In addition to facing the challenges the legacy financial system brings, Lightning also brings the complexity of building within an emergent ecosystem of cryptocurrency, some of which as detailed above with the complexity in developing this software, in successfully running it with real capital behind it, and in navigating the potential regulatory and legislative morass that is rapidly growing around these new agile payment tools.

Even with these daunting challenges considered, the Lightning Network represents such a significant “warp speed” improvement upon existing financial systems across key value propositions in velocity, user cost, micropayment capacity, and resistance to debasement through inflation. These benefits have led multiple financial institutions such as River Financial<sup>[xxii](https://www.youtube.com/watch?v=Itkcurc0Bms)</sup>, Block (formerly Square)<sup>[xxiii](https://www.youtube.com/watch?v=rSSnyJpFNZU)</sup>, Strike<sup>[xxiv](https://www.youtube.com/watch?v=dD2-T7TX2rk)</sup>, and even MicroStrategy<sup>[xxv](https://www.youtube.com/watch?v=Cgdwme_GhIw)</sup> to make heavy investments in this nascent environment, all of which with great success.

These efforts seek to challenge the status quo and disrupt it at such a rapid pace, that the only avenue the existing institutions have at their disposal is to seek legal action to prevent its adoption & growth. However, just as unsuccessful as hacking Lightning so far has been, legislative action has been equally challenging due to the fact that its underpinning technology, Bitcoin, is not a security per the SEC, and distinct from all other cryptocurrencies in its truly decentralized nature and Lightning avoids the foibles that banking is currently suffering from through its overt reliance on fractional reserve banking and under-collateralized loan instruments. By creating a sound alternative to the legacy financial system, and improving upon it not merely in confidence in the currency but also in the other areas mentioned above, it is uniquely positioned to grow extremely rapidly, consuming market share from slower moving incumbents adapting poorly to the changing financial environment. Although there will be additional security issues, complexity and cost of running a routing node will increase, and the market will not be without its own challenges, it remains in the author’s opinion, a very bright new avenue for financial technological innovation.

# Additional Reading

Some papers relevant to the security complexities facing the Lightning network have been compiled here for reference.

* [https://arxiv.org/abs/2208.01908](https://arxiv.org/abs/2208.01908)
* [https://www.semanticscholar.org/paper/An-Empirical-Analysis-of-Privacy-in-the-Lightning-Kappos-Yousaf/886d22608c6704a000c59e6d52d14bd06b0a3565](https://www.semanticscholar.org/paper/An-Empirical-Analysis-of-Privacy-in-the-Lightning-Kappos-Yousaf/886d22608c6704a000c59e6d52d14bd06b0a3565)
* [https://arxiv.org/abs/2103.08576](https://arxiv.org/abs/2103.08576)
* [https://www.computer.org/csdl/proceedings-article/csf/2020/09155145/1m1jOxuJKF2](https://www.computer.org/csdl/proceedings-article/csf/2020/09155145/1m1jOxuJKF2)
* [https://docs.lightning.engineering/lightning-network-tools/lnd/secure-your-lightning-network-node](https://docs.lightning.engineering/lightning-network-tools/lnd/secure-your-lightning-network-node)
* [https://river.com/learn/what-is-the-lightning-network/](https://river.com/learn/what-is-the-lightning-network/)
* [https://dci.mit.edu/lightning-network](https://dci.mit.edu/lightning-network)
* [https://www.coinhouse.com/insights/news/lightning-network-technical-introduction/](https://www.coinhouse.com/insights/news/lightning-network-technical-introduction/)
* [https://kryptografen.no/lightning-101/lightning-101-why-is-the-lightning-network-secure/](https://kryptografen.no/lightning-101/lightning-101-why-is-the-lightning-network-secure/)
* [https://www.coindesk.com/tech/2020/10/27/4-bitcoin-lightning-network-vulnerabilities-that-havent-been-exploited-yet/](https://www.coindesk.com/tech/2020/10/27/4-bitcoin-lightning-network-vulnerabilities-that-havent-been-exploited-yet/)
* [https://coingeek.com/the-unsecure-lightning-network-as-btc-layer-2-scaling-protocol/](https://coingeek.com/the-unsecure-lightning-network-as-btc-layer-2-scaling-protocol/)

# Endnotes

* [i](https://amboss.space/c/reckless)
* [ii](https://u.today/bitcoin-just-recorded-longest-time-between-two-blocks-in-almost-10-years)
* [iii](https://ycharts.com/indicators/bitcoin_average_transactions_per_block)
* [iv](https://wiki.bitcoinsv.io/index.php/Opcodes_used_in_Bitcoin_Script)
* [v](https://github.com/bitcoin/bips)
* [vi](https://www.computer.org/csdl/proceedings-article/csf/2020/09155145/1m1jOxuJKF2)
* [vii](https://github.com/lightning/bolts)
* [viii](https://docs.lightning.engineering/the-lightning-network/lsat/macaroons)
* [ix](https://github.com/davidshares/Lightning-Network)
* [x](https://mempool.space/tx/7393096d97bfee8660f4100ffd61874d62f9a65de9fb6acf740c4c386990ef73)
* [xi](https://twitter.com/brqgoo/status/1579216353780957185)
* [xii](https://github.com/lightningnetwork/lnd/issues/7002)
* [xiii](https://github.com/lightningnetwork/lnd/pull/7004)
* [xiv](https://protos.com/taproot-bug-freezes-bitcoin-inside-lightning-network-for-hours/)
* [xv](https://mempool.space/tx/73be398c4bdc43709db7398106609eea2a7841aaf3a4fa2000dc18184faa2a7e)
* [xvi](https://twitter.com/brqgoo/status/1587397646125260802)
* [xvii](https://github.com/lightningnetwork/lnd/issues/7096)
* [xviii](https://github.com/lightningnetwork/lnd/pull/7098)
* [xix](https://burakkeceli.medium.com/channel-addresses-bd85e9ab8fe1)
* [xx](https://coinunited.io/news/en/2023-01-04/crypto/btc-blockchain-processed-over-8-trillion-in-transactions-last-year-as-bitcoin-soars)
* [xxi](https://www.theblock.co/post/208817/lightning-network-reaches-all-time-high-in-bitcoin-capacity)
* [xxii](https://www.youtube.com/watch?v=Itkcurc0Bms)
* [xxiii](https://www.youtube.com/watch?v=rSSnyJpFNZU)
* [xxiv](https://www.youtube.com/watch?v=dD2-T7TX2rk)
* [xxv](https://www.youtube.com/watch?v=Cgdwme_GhIw)
