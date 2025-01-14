# \_standbyOf

Contract:[`JBFundingCycleStore`](../)​

{% tabs %}
{% tab title="Step by step" %}
**The project's stored funding cycle that hasn't yet started, if one exists.**

_A value of 0 is returned if no funding cycle was found._

# Definition

```solidity
function _standbyOf(uint256 _projectId) private view returns (uint256 configuration) { ... }
```

* Arguments:
  * `_projectId` is the ID of a project to look through for a standby cycle.
* The view function is private to this contract.
* The function does not alter state on the blockchain.
* The function returns the configuration of the standby funding cycle if one exists, or 0 if one doesn't exist.

# Body

1.  Get a reference to the latest funding cycle for the project.

    ```solidity
    // Get a reference to the project's latest funding cycle.
    configuration = latestConfigurationOf[_projectId];
    ```

    _Internal references:_

    * [`latestConfigurationOf`](../properties/latestconfigurationof.md)
2.  If there isn't a funding cycle for the project, there isn't a standby cycle either.

    ```solidity
    // If there isn't one, there also isn't a standby funding cycle.
    if (configuration == 0) return 0;
    ```
3.  Get the struct for the latest funding cycle.

    ```solidity
    // Get the necessary properties for the latest funding cycle.
    JBFundingCycle memory _fundingCycle = _getStructFor(_projectId, configuration);
    ```

    _Internal references:_

    * [`_getStructFor`](\_getstructfor.md)
4.  If the cycle has started, return 0 since there is not a stored funding cycle in standby.

    ```solidity
    // There is no upcoming funding cycle if the latest funding cycle has already started.
    if (block.timestamp >= _fundingCycle.start) return 0;
    ```
{% endtab %}

{% tab title="Code" %}
```solidity
/**
  @notice 
  The project's stored funding cycle that hasn't yet started, if one exists.

  @dev
  A value of 0 is returned if no funding cycle was found.
  
  @param _projectId The ID of a project to look through for a standby cycle.

  @return configuration The configuration of the standby funding cycle.
*/
function _standbyOf(uint256 _projectId) private view returns (uint256 configuration) {
  // Get a reference to the project's latest funding cycle.
  configuration = latestConfigurationOf[_projectId];

  // If there isn't one, theres also no standby funding cycle.
  if (configuration == 0) return 0;

  // Get the necessary properties for the latest funding cycle.
  JBFundingCycle memory _fundingCycle = _getStructFor(_projectId, configuration);

  // There is no upcoming funding cycle if the latest funding cycle has already started.
  if (block.timestamp >= _fundingCycle.start) return 0;
}
```
{% endtab %}

{% tab title="Bug bounty" %}
| Category          | Description                                                                                                                            | Reward |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| **Optimization**  | Help make this operation more efficient.                                                                                               | 0.5ETH |
| **Low severity**  | Identify a vulnerability in this operation that could lead to an inconvenience for a user of the protocol or for a protocol developer. | 1ETH   |
| **High severity** | Identify a vulnerability in this operation that could lead to data corruption or loss of funds.                                        | 5+ETH  |
{% endtab %}
{% endtabs %}
