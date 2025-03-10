# Trust in Canisters

## Background

A key aspect of DeFi and related applications in canisters is the ability to transfer value, e.g. ICP or Bitcoin. Such functionality makes trust in canisters essential. How can one ensure that it is safe to entrust ICPs to a canister?

The answer to this question has two separate dimensions:

1.  confidence that the canister does what it is supposed to do and

2.  confidence that the canister behavior will not unexpectedly change.

## The Canister does what it is Supposed to do

The correct behavior of a canister can be checked in two steps. First, inspect the source code used to generate the Wasm code deployed in a canister to ensure that it implements the expected/claimed functionality, and only this functionality. Second, ensure that the Wasm module the canister runs, has indeed been generated from the claimed source code. Here, reproducibility of the build is crucial: the developer should have constructed the Wasm module so that precisely the same Wasm can be rebuilt from scratch. The user can then compare the hash of the rebuilt Wasm module with the module hash reported by the IC. Developers and users can find guidance on ensuring reproducibility in [Reproducible canisters](/developer-docs/backend/reproducible-builds.md).

## The Behavior of the Canister cannot Unexpectedly Change

Canister smart contracts are deployed and managed by controllers. Among other capabilities, the controllers can change the code for the canisters which they control so canister code is **mutable**, unlike smart contracts on other blockchains and the controllers have complete control over the assets like ICP tokens or Bitcoins held by the canister they manage. This feature brings canisters closer to typical software and makes them suitable for a broad range of applications where software logic can be changed on an as-needed basis.

For critical applications like those used in DeFI, mutability can be dangerous; the controller could change a benign canister into a canister that steals assets. Below we outline some options available to developers on how to verifiably restrict mutability.

:::caution

The canisters, if not voluntarily made immutable, have complete control over the user assets held by them, for example, any ICP Tokens or Bitcoin held by the canister on the user's behalf. The canister, if malicious, can steal all the assets. In other words, as a user, if you interact with a canister that deals with your assets, inspect the canister to know how it handles them. If you determine that the canister is storing the assets in its subaccounts, ensure that the canister is immutable or has decentralized governance.

:::

### Complete Immutability

The simplest option is to make the canister immutable by removing its controller. A user can verify the list of controllers for a canister &lt;canister&gt; using dfx. For example:

    dfx canister --network ic info ryjl3-tyaaa-aaaaa-aaaba-cai

will return the list of controllers for the canister with principal `ryjl3-tyaaa-aaaaa-aaaba-cai` (in this example, the ledger canister).

If the controller list is empty then the canister is immutable.

A user can obtain the list of controllers of another canister via a [`read_state` request](/references/ic-interface-spec.md/#http-read-state) to get the relevant [canister information](/references/ic-interface-spec.md#state-tree-canister-information) which includes the list of controllers. NB: currently a canister cannot obtain this information.

Immutability can also be achieved by setting the controller of a canister to be itself. In this case, however, you need to carefully verify that the canister cannot somehow submit a request to upgrade itself, e.g. by issuing a reinstall request. Here, code inspection and reproducible builds are crucial.

Finally, a somewhat more useful solution is to pass control of the canister to a so-called [“black hole” canister](https://github.com/ninegua/ic-blackhole). This canister is itself immutable (it has only itself as controller) but allows third parties to obtain useful information about the canisters the black hole controls, such as the available cycles balance of a black-holed canister. An instance of a black hole canister is [e3mmv-5qaaa-aaaah-aadma-cai](https://ic.rocks/principal/e3mmv-5qaaa-aaaah-aadma-cai) which is thoroughly documented [here](https://github.com/ninegua/ic-blackhole).

### Governed Mutability

A more complex but powerful approach is to control the canister via a distributed governance mechanism. One can imagine different levels of complexity and control that such a governance mechanism may implement. An example is the (upcoming) [SNS feature](https://medium.com/dfinity/how-the-service-nervous-system-sns-will-bring-tokenized-governance-to-on-chain-dapps-b74fb8364a5c) which allows developers to set the controller of their canister to some governing canister.

Needless to say, the trust requirements are moved to the SNS controlling the canister where all of the considerations regarding code inspection and reproducibility apply.
