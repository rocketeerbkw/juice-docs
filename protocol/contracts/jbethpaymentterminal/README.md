---
description: >-
  Manages all inflows and outflows of ETH into the Juicebox ecosystem. This
  contract also stores all treasury ETH for all projects.
---

# JBETHPaymentTerminal

## Overview

### Code

{% embed url="https://github.com/jbx-protocol/juice-contracts/tree/main/contracts/v2/JBPaymentTerminal.sol" %}

### **Addresses**

Ethereum mainnet: _Not yet deployed_\
Rinkeby testnet: _Not yet deployed_

### **Interfaces**

| Name                     | Description                                                                                                                                      |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`IJBPaymentTerminal`** | General interface for the methods in this contract that send and receive funds according to the Juicebox protocol's rules.                       |
| **`IJBTerminal`**        | Allows projects to migrate to this contract from other IJBTerminals (like TerminalV1), and to facilitate a project's future migration decisions. |

### **Inheritance**

| Contract              | Description                                                                                                                                                            |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`JBOperatable`**    | <p>Includes convenience functionality for checking a message sender's permissions before executing certain transactions.</p><p><a href="../jboperatable/">more</a></p> |
| **`ReentrancyGuard`** | Includes convenience functionality for preventing access to certain functions while certain other functions are being executed.                                        |



## Constructor

```solidity
constructor(
  IJBOperatorStore _operatorStore,
  IJBPrices _prices,
  IJBProjects _projects,
  IJBDirectory _directory,
  IJBFundingCycleStore _fundingCycleStore,
  IJBTokenStore _tokenStore,
  IJBSplitsStore _splitsStore,
  IJBVault _vault
) JBOperatable(_operatorStore) {
  prices = _prices;
  projects = _projects;
  directory = _directory;
  fundingCycleStore = _fundingCycleStore;
  tokenStore = _tokenStore;
  splitsStore = _splitsStore;
  vault = _vault;
}
```

* Arguments:
  * `_operatorStore` is an [`IJBOperatorStore`](../../interfaces/ijboperatorstore.md) contract storing operator assignments.
  * `_prices` is an [`IJBPrices`](../../interfaces/ijbprices.md) contract that exposes price feeds.
  * `_projects` is an [`IJBProjects`](../../interfaces/ijbprojects.md) contract which mints ERC-721's that represent project ownership and transfers.
  * `_directory` is an [`IJBDirectory`](../../interfaces/ijbdirectory.md) contract storing directories of terminals and controllers for each project.
  * `_fundingCycleStore` is an [`IJBFundingCycleStore`](../../interfaces/ijbfundingcyclestore.md) contract storing all funding cycle configurations.
  * `_tokenStore` is an [`IJBTokenStore`](../../interfaces/ijbtokenstore.md) contract that manages token minting and burning.
  * `_splitStore` is an [`IJBSplitStore`](../jbsplitstore/) contract that stores splits for each project.
  * `_vault` is an [`IJBVault`](../jbethvault.md) contract to store funds in.

## Events

| Name                          | Data                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`AddToBalance`**            | <ul><li><code>uint256 indexed projectId</code></li><li><code>uint256 value</code></li><li><code>string memo</code></li><li><code>address caller</code></li></ul><p><a href="events/addtobalance.md">more</a></p>                                                                                                                                                                                                                                              |
| **`TransferBalance`**         | <ul><li><code>uint256 indexed projectId</code></li><li><code>IJBTerminal indexed terminal</code></li><li><code>uint256 amount</code></li><li><code>address caller</code></li></ul><p><a href="events/transferbalance.md">more</a></p>                                                                                                                                                                                                                         |
| **`DistributePayouts`**       | <ul><li><code>uint256 indexed fundingCycleId</code></li><li><code>uint256 indexed projectId</code></li><li><code>address projectOwner</code></li><li><code>uint256 amount</code></li><li><code>uint256 tappedAmount</code></li><li><code>uint256 feeAmount</code></li><li><code>uint256 projectOwnerTransferAmount</code></li><li><code>string memo</code></li><li><code>address caller</code></li></ul><p><a href="events/distributepayouts.md">more</a></p> |
| **`UseAllowance`**            | <ul><li><code>uint256 indexed fundingCycleId</code></li><li><code>uint256 indexed configuration</code></li><li><code>uint256 indexed projectId</code></li><li><code>address beneficiary</code></li><li><code>uint256 amount</code></li><li><code>uint256 feeAmount</code></li><li><code>uint256 transferAmount</code></li><li><code>address caller</code></li></ul><p><a href="events/useallowance.md">more</a></p>                                           |
| **`Pay`**                     | <ul><li><code>uint256 indexed fundingCycleId</code></li><li><code>uint256 indexed projectId</code></li><li><code>address indexed beneficiary</code></li><li><code>FundingCycle fundingCycle</code></li><li><code>uint256 amount</code></li><li><code>uint256 weight</code></li><li><code>uint256 tokenCount</code></li><li><code>string memo</code></li><li><code>address caller</code></li></ul><p><a href="events/pay.md">more</a></p>                      |
| **`Redeem`**                  | <ul><li><code>uint256 indexed fundingCycleId</code></li><li><code>uint256 indexed projectId</code></li><li><code>address indexed holder</code></li><li><code>FundingCycle fundingCycle</code></li><li><code>address beneficiary</code></li><li><code>uint256 tokenCount</code></li><li><code>uint256 claimedAmount</code></li><li><code>string memo</code></li><li><code>address caller</code></li></ul><p><a href="events/redeem.md">more</a></p>            |
| **`DistributeToPayoutSplit`** | <ul><li><code>uint256 indexed fundingCycleId</code></li><li><code>uint256 indexed projectId</code></li><li><code>Split split</code></li><li><code>uint256 amount</code></li><li><code>address caller</code></li></ul><p><a href="events/distributetopayoutsplit.md">more</a></p>                                                                                                                                                                              |
| **`AllowBalanceTransfer`**    | <ul><li><code>IJBTerminal terminal</code></li></ul><p><a href="events/allowmigration.md">more</a></p>                                                                                                                                                                                                                                                                                                                                                         |

## Properties

| Function                         | Definition                                                                                                                                                                                                                          |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`projects`**                   | <p><strong>Traits</strong></p><ul><li><code>immutable</code></li></ul><p><strong>Returns</strong></p><ul><li><code>IJBProjects projects</code></li></ul><p><a href="../jbdirectory/read/projects.md">more</a></p>                   |
| **`splitStore`**                 | <p><strong>Traits</strong></p><ul><li><code>immutable</code></li></ul><p><strong>Returns</strong></p><ul><li><code>IJBSplitsStore splitsStore</code></li></ul><p><a href="read/splitstore.md">more</a></p>                          |
| **`directory`**                  | <p><strong>Traits</strong></p><ul><li><code>immutable</code></li></ul><p><strong>Returns</strong></p><ul><li><code>IJBDirectory directory</code></li></ul><p><a href="read/directory.md">more</a></p>                               |
| **`data`**                       | <p><strong>Traits</strong></p><ul><li><code>immutable</code></li></ul><p><strong>Returns</strong></p><ul><li><code>IJBPaymentTerminalData data</code></li></ul><p><a href="read/data.md">more</a></p>                               |
| **`balanceTransferIsAllowedTo`** | <p><strong>Params</strong></p><ul><li><code>IJBTerminal _terminal</code></li></ul><p><strong>Returns</strong></p><ul><li><code>bool migrationIsAllowed</code></li></ul><p><a href="read/balancetransferisallowedto.md">more</a></p> |
| **`dataAuthority`**              | <p><strong>Returns</strong></p><ul><li><code>address dataAuthority</code></li></ul><p><a href="read/dataauthority.md">more</a></p>                                                                                                  |

## Write

| Function                       | Definition                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`pay`**                      | <p><strong>Traits</strong></p><ul><li><code>payable</code></li></ul><p><strong>Params</strong></p><ul><li><code>uint256 _projectId</code></li><li><code>address _beneficiary</code></li><li><code>uint256 _minReturnedTokens</code></li><li><code>bool _preferUnstakedTokens</code></li><li><code>string _memo</code></li><li><code>bytes _delegateMetadata</code></li></ul><p><strong>Returns</strong></p><ul><li><code>uint256 fundingCycleId</code></li></ul><p><a href="write/pay-1.md">more</a></p>                                                                                      |
| **`distributePayoutsOf`**      | <p><strong>Traits</strong></p><ul><li><code>nonReentrant</code></li></ul><p><strong>Params</strong></p><ul><li><code>uint256 _projectId</code></li><li><code>uint256 _amount</code></li><li><code>uint256 _currency</code></li><li><code>uint256 _minReturnedWei</code></li><li><code>string _memo</code></li></ul><p><strong>Returns</strong></p><ul><li><code>uint256 fundingCycleId</code></li></ul><p><a href="write/distributepayoutsof.md">more</a></p>                                                                                                                                 |
| **`useAllowanceOf`**           | <p><strong>Traits</strong></p><ul><li><code>nonReentrant</code></li><li><code>requirePermission</code></li></ul><p><br><strong>Params</strong></p><ul><li><code>uint256 _projectId</code></li><li><code>uint256 _amount</code></li><li><code>uint256 _currency</code></li><li><code>uint256 _minReturnedWei</code></li><li><code>address payable _beneficiary</code></li></ul><p><strong>Returns</strong></p><ul><li><code>uint256 fundingCycleId</code></li></ul><p><a href="write/useallowanceof.md">more</a></p>                                                                           |
| **`redeemTokensOf`**           | <p><strong>Traits</strong></p><ul><li><code>nonReentrant</code></li><li><code>requirePermission</code></li></ul><p><strong>Params</strong></p><ul><li><code>address _holder</code></li><li><code>uint256 _projectId</code></li><li><code>uint256 _tokenCount</code></li><li><code>uint256 _minReturnedWei</code></li><li><code>address payable _beneficiary</code></li><li><code>string _memo</code></li><li><code>bytes _delegateMetadata</code></li></ul><p><strong>Returns</strong></p><ul><li><code>uint256 claimAmount</code></li></ul><p><a href="write/redeemtokensof.md">more</a></p> |
| **`transferBalanceOf`**        | <p><strong>Traits</strong></p><ul><li><code>nonReentrant</code></li><li><code>requirePermission</code></li></ul><p><strong>Params</strong></p><ul><li><code>uint256 _projectId</code></li><li><code>IJBTerminal _terminal</code></li></ul><p><a href="write/migrate.md">more</a></p>                                                                                                                                                                                                                                                                                                          |
| **`addToBalanceOf`**           | <p><strong>Traits</strong></p><ul><li><code>payable</code></li></ul><p><strong>Params</strong></p><ul><li><code>uint256 _projectId</code></li><li><code>string _memo</code></li></ul><p><a href="write/addtobalanceof.md">more</a></p>                                                                                                                                                                                                                                                                                                                                                        |
| **`allowBalanceTransferTo`**   | <p><strong>Traits</strong></p><ul><li><code>onlyOwner</code></li></ul><p><strong>Params</strong></p><ul><li><code>IJBTerminal _terminal</code></li></ul><p><a href="write/allowmigration.md">more</a></p>                                                                                                                                                                                                                                                                                                                                                                                     |
| **`prepForBalanceTransferOf`** | <p><strong>Traits</strong></p><ul><li><code>nonReentrant</code></li></ul><p><strong>Params</strong></p><ul><li><code>uint256 _projectId</code></li></ul><p><a href="write/prepformigrationof.md">more</a></p>                                                                                                                                                                                                                                                                                                                                                                                 |