---
title: 'Bitcoin Optech Newsletter #163'
permalink: /en/newsletters/2021/08/25/
name: 2021-08-25-newsletter
slug: 2021-08-25-newsletter
type: newsletter
layout: newsletter
lang: en
---
This week's newsletter summarizes a discussion about setting LN channel
base fees to zero and includes our regular sections with popular
questions and answers from the Bitcoin Stack Exchange, how you can prepare for
taproot, new releases and release candidates, and notable changes to popular Bitcoin
infrastructure projects.

## News

- **Zero base fee LN discussion:** in the LN protocol, a spender can
  choose how much to pay each node that helps successfully route the
  payment to its final destination.  Routing nodes in turn can choose to
  reject any payment attempt that doesn't offer them enough fee.  For
  this division of responsibilities to work, routing nodes need to
  communicate to spenders what fees they expect, so [BOLT7][] provides
  routing nodes with the ability to advertise two fee-related
  parameters, `fee_base_msat` (base fee) and
  `fee_proportional_millionths` (proportional fee).

    A [recent paper][pickhardt richter paper] by René Pickhardt and
    Stefan Richter proposes a new pathfinding technique that payers
    will be able to use to minimize their fees and the number of
    payment attempts they'll need to successfully send a payment (among
    other benefits).  But deploying the technique on the network today
    encounters two problems related to the LN base fee and [multipath
    payments][topic multipath payments]:

    - **More splits, more fees:** compare a payment with a single path
      and a payment for the same amount with two equivalent paths: they
      would both pay the same amount of total proportional fee (since the
      overall payment amount is the same) but the two-path payment would
      pay twice as much total base fee (since it uses twice as many
      hops).  For `x` equivalent paths, the base fee would be `x` times
      as high.  This makes using multipath payments more expensive and
      so penalizes techniques that use them, such as the technique
      proposed.

     {% comment %}<!-- The explanation in the paper is unintelligible to
     me, so the following description is deliberately bland in order to
     avoid being wrong.  -->{% endcomment %}

     - **Computational difficulties:** as described in section 2.3 of
       the paper, the proposed pathfinding algorithm can't easily
       compute paths and payment splits when there's a base fee.  It may
       be possible to solve this problem algorithmically in the long
       term, but the easiest solution for implementers of the algorithm
       would be to eliminate the base fee.

    In [podcasts][honigdachs podcast] and on [Twitter][zbf tweet], the authors suggested the problem could
    be addressed without any immediate change to the LN protocol if node
    operators set their base fee to zero.  They further suggested
    operators could begin doing this immediately, even though their work
    is not yet deployable in production.  This led to several
    discussions between LN developers on Twitter, which Anthony Towns
    helped migrate to the Lightning-Dev mailing list with a [post][towns
    post].

    Towns was in favor of users setting the base fee to zero, noting
    both its benefits for multipath splitting and that it should be
    easier for node operators to optimize the single remaining
    fee parameter, the proportional fee.

    Matt Corallo [replied][corallo post] with the concern that the
    creation of [HTLCs][topic htlc] for routing payments places several
    burdens on nodes that are constant regardless of the amount of the
    payment.  The base fee allows a node to require that it be
    compensated for those costs.  But those costs, Towns countered, are
    essentially the same both for successfully routed payments and
    unsuccessfully routed payments---yet LN nodes are only paid in the
    successful case.  If nodes are willing to accept those costs without
    compensation in some cases, why not accept them in all cases?  The
    same, though, could be said of proportional fees, and this led to
    some brief discussion of [upfront fees][topic channel jamming
    attacks] which could allow nodes to be compensated even for
    unsuccessful payments.

    Towns also suggested that it is possible for a node to ensure it
    receives a minimal fee even without a base fee by simply refusing
    to route payments below a certain size.  For example, a node with
    a current base fee of 1 sat can ensure it receives at least that
    much with a 0.1% proportional fee and a minimum amount of 1,000
    sat.  This would stifle micropayments, but LN nodes already handle
    small payments without using HTLCs, which eliminates some of the
    fixed costs and may make purely proportional costs more
    appropriate, though this remained debated.

    Later in the discussion, Olaoluwa Osuntokun [emphasized][osuntokun
    post] a point made earlier that there's no clear current need for
    node operators to change a parameter today for a new pathfinding
    algorithm that nobody is currently ready to use in production.   He and
    Corallo want to see if further research and development can allow
    the algorithm (or a similar one based on different principles) to
    work nearly equally as well even when base fees are non-zero.

    No clear conclusion to the discussion was reached as of this
    writing.

## Selected Q&A from Bitcoin Stack Exchange

*[Bitcoin Stack Exchange][bitcoin.se] is one of the first places Optech
contributors look for answers to their questions---or when we have a
few spare moments to help curious or confused users.  In
this monthly feature, we highlight some of the top-voted questions and
answers posted since our last update.*

{% comment %}<!-- https://bitcoin.stackexchange.com/search?tab=votes&q=created%3a1m..%20is%3aanswer -->{% endcomment %}
{% assign bse = "https://bitcoin.stackexchange.com/a/" %}

FIXME:bitschmidty

## Preparing for taproot #10: PTLCs

*A weekly [series][series preparing for taproot] about how developers
and service providers can prepare for the upcoming activation of taproot
at block height {{site.trb}}.*

{% include specials/taproot/en/09-ptlcs.md %}

## Releases and release candidates

*New releases and release candidates for popular Bitcoin infrastructure
projects.  Please consider upgrading to new releases or helping to test
release candidates.*

- [Rust-Lightning 0.0.100][] is a new release that supports sending and
  receiving [keysend payments][topic spontaneous payments] and makes it
  easier to track successfully routed payments and record the amount of
  fee income the node earned from them.

- [Bitcoin Core 22.0rc2][bitcoin core 22.0] is a release candidate
  for the next major version of this full node implementation and its
  associated wallet and other software. Major changes in this new
  version include support for [I2P][topic anonymity networks] connections,
  removal of support for [version 2 Tor][topic anonymity networks] connections,
  and enhanced support for hardware wallets.

- [Bitcoin Core 0.21.2rc1][bitcoin core 0.21.2] is a release candidate
  for a maintenance version of Bitcoin Core.  It contains several bug
  fixes and small improvements.

## Notable code and documentation changes

*Notable changes this week in [Bitcoin Core][bitcoin core repo],
[C-Lightning][c-lightning repo], [Eclair][eclair repo], [LND][lnd repo],
[Rust-Lightning][rust-lightning repo], [libsecp256k1][libsecp256k1
repo], [Hardware Wallet Interface (HWI)][hwi repo],
[Rust Bitcoin][rust bitcoin repo], [BTCPay Server][btcpay server repo],
[Bitcoin Improvement Proposals (BIPs)][bips repo], and [Lightning
BOLTs][bolts repo].*

- [Bitcoin Core #22541][] Add a new RPC command: restorewallet FIXME:Xekyo

- [LND #5442][] allows adding inputs to a [PSBT][topic psbt] without
  adding any new outputs, which is useful when creating a [CPFP fee
  bump][topic cpfp].

- [Rust-Lightning #1011][] adds support for not-yet-merged [BOLTs
  #847][], which allows two channel peers to negotiate what fee should
  be paid in a mutual close transaction.  In the current protocol, only
  a single fee is sent, and the other party must either accept or reject
  that precise fee.

- [BOLTs #887][] Make payment secret mandatory and update Bolt 11 test vectors FIXME:dongcarl

{% include references.md %}
{% include linkers/issues.md issues="22541,5442,1011,847,887" %}
[bitcoin core 22.0]: https://bitcoincore.org/bin/bitcoin-core-22.0/
[bitcoin core 0.21.2]: https://bitcoincore.org/bin/bitcoin-core-0.21.2/
[rust-lightning 0.0.100]: https://github.com/rust-bitcoin/rust-lightning/releases/tag/v0.0.100
[pickhardt richter paper]: https://arxiv.org/abs/2107.05322
[towns post]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-August/003174.html
[corallo post]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-August/003179.html
[osuntokun post]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-August/003187.html
[honigdachs podcast]: https://coinspondent.de/2021/07/11/honigdachs-62-pickhardt-payments/
[zbf tweet]: https://twitter.com/renepickhardt/status/1414895869078523910