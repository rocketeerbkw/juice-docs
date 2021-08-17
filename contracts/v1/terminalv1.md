---
description: 'Interfaces the contracts and the user, working as a top-level contract.'
---

# TerminalV1

## Constructor

**Params:**

*   projects A Projects contract which mints ERC-721's that represent project ownership and transfers**.** 
*   \_fundingCycles A funding cycle configuration store.
*   A contract that manages Ticket printing and redeeming.
*   \_modStore A storage for a project's mods.
*   \_prices A price feed contract to use.
*   \_terminalDirectory A directory of a project's current Juicebox terminal to receive payments in.

\*/

```text

constructor(
    IProjects _projects,
    IFundingCycles _fundingCycles,
    ITicketBooth _ticketBooth,
    IOperatorStore _operatorStore,
    IModStore _modStore,
    IPrices _prices,
    ITerminalDirectory _terminalDirectory,
    address payable _governance
) Operatable(_operatorStore) {
    require(
        _projects != IProjects(address(0)) &&
            _fundingCycles != IFundingCycles(address(0)) &&
            _ticketBooth != ITicketBooth(address(0)) &&
            _modStore != IModStore(address(0)) &&
            _prices != IPrices(address(0)) &&
            _terminalDirectory != ITerminalDirectory(address(0)) &&
            _governance != address(address(0)),
        "TerminalV1: ZERO_ADDRESS"
    );
    projects = _projects;
    fundingCycles = _fundingCycles;
    ticketBooth = _ticketBooth;
    modStore = _modStore;
    prices = _prices;
    terminalDirectory = _terminalDirectory;
    governance = _governance;
}
```

## Read

### currentOverflowOf

Gets the current overflowed amount for a specified project.   

**Params:**

*  \_projectId The ID of the project to get overflow for.  

**@return** overflow The current overflow of funds for the project.

```lua

function currentOverflowOf(uint256 _projectId)
    external
    view
    override
    returns (uint256 overflow)
{
    // Get a reference to the project's current funding cycle.
    FundingCycle memory _fundingCycle = fundingCycles.currentOf(_projectId);

    // There's no overflow if there's no funding cycle.
    if (_fundingCycle.id == 0) return 0;

    return _overflowFrom(_fundingCycle);
}
```



### reservedTicketBalanceOf

Gets the amount of reserved tickets that a project has.

  **Params:**

* \_projectId The ID of the project to get overflow for.
* \_reservedRate The reserved rate to use to make the calculation.  

**@return** amount overflow The current overflow of funds for the project.

```lua

function reservedTicketBalanceOf(uint256 _projectId, uint256 _reservedRate)
    external
    view
    override
    returns (uint256)
{
    return
        _reservedTicketAmountFrom(
            _processedTicketTrackerOf[_projectId],
            _reservedRate,
            ticketBooth.totalSupplyOf(_projectId)
        );
}
```

### claimableOverflowOf

The amount of tokens that can be claimed by the given address.

The \_account must have at least \_count tickets for the specified project.   
If there is a funding cycle reconfiguration ballot open for the project, the project's current bonding curve        is bypassed.

**Params:**

*  \_account The address to get an amount for.  
* \_projectId The ID of the project to get a claimable amount for.  
* \_count The number of Tickets that would be redeemed to get the resulting amount.

  **@return** amount The amount of tokens that can be claimed

```lua
function claimableOverflowOf(
    address _account,
    uint256 _projectId,
    uint256 _count
) public view override returns (uint256) {
    // The holder must have the specified number of the project's tickets.
    require(
        ticketBooth.balanceOf(_account, _projectId) >= _count,
        "TerminalV1::claimableOverflow: INSUFFICIENT_TICKETS"
    );

    // Get a reference to the current funding cycle for the project.
    FundingCycle memory _fundingCycle = fundingCycles.currentOf(_projectId);

    // There's no overflow if there's no funding cycle.
    if (_fundingCycle.id == 0) return 0;

    // Get the amount of current overflow.
    uint256 _currentOverflow = _overflowFrom(_fundingCycle);

    // If there is no overflow, nothing is claimable.
    if (_currentOverflow == 0) return 0;

    // Get the total number of tickets in circulation.
    uint256 _totalSupply = ticketBooth.totalSupplyOf(_projectId);

    // Get the number of reserved tickets the project has.
    // The reserved rate is in bits 8-15 of the metadata.
    uint256 _reservedTicketAmount = _reservedTicketAmountFrom(
        _processedTicketTrackerOf[_projectId],
        uint256(uint8(_fundingCycle.metadata >> 8)),
        _totalSupply
    );

    // If there are reserved tickets, add them to the total supply.
    if (_reservedTicketAmount > 0)
        _totalSupply = _totalSupply + _reservedTicketAmount;

    // If the amount being redeemed is the the total supply, return the rest of the overflow.
    if (_count == _totalSupply) return _currentOverflow;

    // Get a reference to the linear proportion.
    uint256 _base = PRBMath.mulDiv(_currentOverflow, _count, _totalSupply);

    // Use the reconfiguration bonding curve if the queued cycle is pending approval according to the previous funding cycle's ballot.
    uint256 _bondingCurveRate = fundingCycles.currentBallotStateOf(
        _projectId
    ) == BallotState.Active // The reconfiguration bonding curve rate is stored in bytes 24-31 of the metadata property.
        ? uint256(uint8(_fundingCycle.metadata >> 24)) // The bonding curve rate is stored in bytes 16-23 of the data property after.
        : uint256(uint8(_fundingCycle.metadata >> 16));

    // The bonding curve formula.
    // https://www.desmos.com/calculator/sp9ru6zbpk
    // where x is _count, o is _currentOverflow, s is _totalSupply, and r is _bondingCurveRate.

    // These conditions are all part of the same curve. Edge conditions are separated because fewer operation are necessary.
    if (_bondingCurveRate == 200) return _base;
    if (_bondingCurveRate == 0)
        return PRBMath.mulDiv(_base, _count, _totalSupply);
    return
        PRBMath.mulDiv(
            _base,
            _bondingCurveRate +
                PRBMath.mulDiv(
                    _count,
                    200 - _bondingCurveRate,
                    _totalSupply
                ),
            200
        );
}
```

### canPrintPreminedTickets

Whether or not a project can still print premined tickets.

**Params:**

*  \_projectId The ID of the project to get the status of.

**@return** Boolean flag.

```lua

function canPrintPreminedTickets(uint256 _projectId)
    public
    view
    override
    returns (bool)
{
    return
        // The total supply of tickets must equal the preconfigured ticket count.
        ticketBooth.totalSupplyOf(_projectId) ==
        _preconfigureTicketCountOf[_projectId] &&
        // The above condition is still possible after post-configured tickets have been printed due to ticket redeeming.
        // The only case when processedTicketTracker is 0 is before redeeming and printing reserved tickets.
        _processedTicketTrackerOf[_projectId] >= 0 &&
        uint256(_processedTicketTrackerOf[_projectId]) ==
        _preconfigureTicketCountOf[_projectId];
}
```

## Write



### Deploy

Deploys a project. This will mint an ERC-721 into the \_owner's account, configure a first funding cycle, and set up any mods.

Each operation within this transaction can be done in sequence separately  
Anyone can deploy a project on an owner's behalf.

**Params:**

* \_owner The address that will own the project.
* \_handle The project's unique handle.
* \_uri A link to information about the project and this funding cycle.
* \_properties The funding cycle configuration.
  * \_properties.target The amount that the project wants to receive in this funding cycle. Sent as a wad.
  * \_properties.currency The currency of the \`target\`. Send 0 for ETH or 1 for USD.
  * \_properties.duration The duration of the funding stage for which the \`target\` amount is needed. Measured in days. Send 0 for a boundless cycle reconfigurable at any time.
  * \_properties.cycleLimit The number of cycles that this configuration should last for before going back to the last permanent. This has no effect for a project's first funding cycle.
  *  \_properties.discountRate A number from 0-200 indicating how valuable a contribution to this funding stage is compared to the project's previous funding stage.
    * If it's 200, each funding stage will have equal weight.
    * If the number is 180, a contribution to the next funding stage will only give you 90% of tickets given to a contribution of the same amount during the current funding stage.
    * If the number is 0, an non-recurring funding stage will get made.
  * \_configuration.ballot The new ballot that will be used to approve subsequent reconfigurations.
* \_metadata A struct specifying the TerminalV1 specific params \_bondingCurveRate, and \_reservedRate.
  * \_reservedRate A number from 0-200 indicating the percentage of each contribution's tickets that will be reserved for the project owner.
  * \_bondingCurveRate The rate from 0-200 at which a project's Tickets can be redeemed for surplus.
    * The bonding curve formula can be found [here](https://www.desmos.com/calculator/sp9ru6zbpk).

       where x is \_count, o is \_currentOverflow, s is \_totalSupply, and r is \_bondingCurveRate.
  * \_reconfigurationBondingCurveRate The bonding curve rate to apply when there is an active ballot.
* \_payoutMods Any payout mods to set.
* \_ticketMods Any ticket mods to set.

```lua
    function deploy(
        address _owner,
        bytes32 _handle,
        string calldata _uri,
        FundingCycleProperties calldata _properties,
        FundingCycleMetadata calldata _metadata,
        PayoutMod[] memory _payoutMods,
        TicketMod[] memory _ticketMods
    ) external;
```



### Configure

Configures the properties of the current funding cycle if the project hasn't distributed tickets yet, or sets the properties of the proposed funding cycle that will take effect once the current one expires if it is approved by the current funding cycle's ballot.  

Only a project's owner or a designated operator can configure its funding cycles.

**Params:**

* \_projectId The ID of the project being reconfigured. 
* \_properties The funding cycle configuration.
* \_properties.target The amount that the project wants to receive in this funding stage. Sent as a wad.
  * \_properties.currency The currency of the \`target\`. Send 0 for ETH or 1 for USD.
  * \_properties.duration The duration of the funding stage for which the \`target\` amount is needed. Measured in days. Send 0 for a boundless cycle reconfigurable at any time.
  * \_properties.cycleLimit The number of cycles that this configuration should last for before going back to the last permanent. This has no effect for a project's first funding cycle.
  * \_properties.discountRate A number from 0-200 indicating how valuable a contribution to this funding stage is compared to the project's previous funding stage.
    * If it's 200, each funding stage will have equal weight.
    * If the number is 180, a contribution to the next funding stage will only give you 90% of tickets given to a contribution of the same amount during the current funding stage.
    * If the number is 0, an non-all transactions except the constructor are write functions recurring funding stage will get made.
  * \_properties.ballot The new ballot that will be used to approve subsequent reconfigurations.
*  \_metadata is a struct specifying the TerminalV1 specific params \_bondingCurveRate, and \_reservedRate.

  * * \_reservedRate A number from 0-200 indicating the percentage of each contribution's tickets that will be reserved for the project owner.
    * \_bondingCurveRate The rate from 0-200 at which a project's Tickets can be redeemed for surplus. The bonding curve formula can be found [here](https://www.desmos.com/calculator/sp9ru6zbpk), where x is \_count, o is \_currentOverflow, s is \_totalSupply, and r is \_bondingCurveRate.
    * \_reconfigurationBondingCurveRate The bonding curve rate to apply when there is an active ballot.

 **@return** The ID of the funding cycle that was successfully configured.

```lua
    function configure(
        uint256 _projectId,
        FundingCycleProperties calldata _properties,
        FundingCycleMetadata calldata _metadata,
        PayoutMod[] memory _payoutMods,
        TicketMod[] memory _ticketMods
    ) external returns (uint256);
```

### 

### printPreminedTickets

Allows a project to print tickets for a specified beneficiary before payments have been received.  @dev 

This can only be done if the project hasn't yet received a payment after configuring a funding cycle.  
Only a project's owner or a designated operator can print premined tickets.

*  \_projectId The ID of the project to premine tickets for.
*  \_amount The amount to base the ticket premine off of.
*  \_currency The currency of the amount to base the ticket premine off of. 
*  \_beneficiary The address to send the printed tickets to.
*  \_memo A memo to leave with the printing.
*  \_preferUnstakedTickets If there is a preference to unstake the printed tickets.

```lua
function printPreminedTickets(
        uint256 _projectId,
        uint256 _amount,
        uint256 _currency,
        address _beneficiary,
        string memory _memo,
        bool _preferUnstakedTickets
) external;
```



### pay

Contribute ETH to a project.

Print's the project's tickets proportional to the amount of the contribution.  
The msg.value is the amount of the contribution in wei.

**Params:**

*  \_projectId The ID of the project being contribute to.
*  \_beneficiary The address to print Tickets for.  \_memo A memo that will be included in the published event.
*  \_preferUnstakedTickets Whether ERC20's should be unstaked automatically if they have been issued.

**@return** The ID of the funding cycle that the payment was made durin

```text
function pay(
    uint256 _projectId,
    address _beneficiary,
    string calldata _memo,
    bool _preferUnstakedTickets
) external payable returns (uint256 fundingCycleId);
```

### **tap**

Tap into funds that have been contributed to a project's current funding cycle.

Anyone can tap funds on a project's behalf.

**Params:**

*  \_projectId The ID of the project to which the funding cycle being tapped belongs. 
* \_amount The amount being tapped, in the funding cycle's currency.
*  \_currency The expected currency being tapped. 
* \_minReturnedWei The minimum number of wei that the amount should be valued at.

**@return** The ID of the funding cycle that was tapped.

```text
function tap(
    uint256 _projectId,
    uint256 _amount,
    uint256 _currency,
    uint256 _minReturnedWei
) external returns (uint256);
```



### 

### redeem

Addresses can redeem their Tickets to claim the project's overflowed ETH. 

Only a ticket's holder or a designated operator can redeem it.

**Params:**

*  \_account The account to redeem tickets for.
*  \_projectId The ID of the project to which the Tickets being redeemed belong. 
* \_count The number of Tickets to redeem.
*  \_minReturnedWei The minimum amount of Wei expected in return.
*  \_beneficiary The address to send the ETH to.
*  \_preferUnstaked If the preference is to redeem tickets that have been converted to ERC-20s.

  **@return** amount The amount of ETH that the tickets were redeemed for.

```text
    function redeem(
        address _account,
        uint256 _projectId,
        uint256 _amount,
        uint256 _minReturnedWei,
        address payable _beneficiary,
        bool _preferUnstaked
    ) external returns (uint256 returnAmount);
```

### **setFee**

Allow the admin to change the fee. 

Only funding cycle reconfigurations after the new fee is set will use the new fee. All future funding cycles based on configurations made in the past will use the fee that was set at the time of the configuration.  
Only governance can set a new fee.

**Params:**

* \_fee The new fee percent. Out of 200.

```text
function setFee(uint256 _fee) external;
```



### onlyGov

Allows governance to transfer its privileges to another contract.

Only the current governance can appoint a new governance.

**Params:**

* \_pendingGovernance The governance to transition power to. 
  * This address will have to accept the responsibility in a subsequent transaction.

```text
function appointGovernance(address payable _pendingGovernance) external;
```

### acceptGovernance

Allows contract to accept its appointment as the new governance.  
Only the pending governance can accept.

```text
function acceptGovernance() external;
```

### migrate

Allows a project owner to migrate its funds and operations to a new contract.

Only a project's owner or a designated operator can migrate it.

**Params:**

*  \_projectId The ID of the project being migrated. 
* \_to The contract that will gain the project's funds.

```text
event Migrate(
    uint256 indexed projectId,
    ITerminal indexed to,
    uint256 _amount,
    address caller
);
```



### addToBalance

   Receives and allocates funds belonging to the specified project.

**Params:**

*  \_projectId The ID of the project to which the funds received belong.

```text
event AddToBalance(
    uint256 indexed projectId,
    uint256 value,
    address caller
);
```



### allowMigration

Adds to the contract addresses that projects can migrate their Tickets to.

Only governance can add a contract to the migration allow list.

**Params:**

*  \_contract The contract to allow.

```text
function allowMigration(ITerminal _contract) external;
```



### printReservedTickets

Prints all reserved tickets for a project.

**Params:**

*  \_projectId The ID of the project to which the reserved tickets belong.

      **@return** amount The amount of tickets that are being printed

```text
function printReservedTickets(uint256 _projectId)
    external
    returns (uint256 reservedTicketsToPrint);
```





## Events


