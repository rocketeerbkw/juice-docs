# \_isConfigurationApproved

Contract:[`JBFundingCycleStore`](../)​

{% tabs %}
{% tab title="Step by step" %}
**Checks to see if the funding cycle of the provided configuration is approved according to the correct ballot.**

# Definition

```solidity
function _isConfigurationApproved(uint256 _projectId, uint256 _configuration)
  private
  view
  returns (bool) { ... } 
```

* Arguments:
  * `_projectId` is the ID of the project to which the funding cycle belongs.
  * `_configuration` is the configuration of the funding cycle to get an approval flag for.
* The view function is private to this contract.
* The function does not alter state on the blockchain.
* The function returns the approval flag.

# Body

1.  Find the [`JBFundingCycle`](../../../data-structures/jbfundingcycle.md) struct for the provided ID.

    ```solidity
    JBFundingCycle memory _fundingCycle = _getStructFor(_projectId, _configuration);
    ```

    _Internal references:_

    * [`_getStructFor`](\_getstructfor.md)
2.  Return the approval flag from the [`_isApproved`](\_isapproved.md) function.

    ```solidity
    return _isApproved(_projectId, _fundingCycle);
    ```

    _Internal references:_

    * [`_isApproved`](\_isapproved.md)
{% endtab %}

{% tab title="Code" %}
```solidity
/** 
  @notice 
  Checks to see if the funding cycle of the provided configuration is approved according to the correct ballot.

  @param _projectId The ID of the project to which the funding cycle belongs.
  @param _configuration The configuration of the funding cycle to get an approval flag for.

  @return The approval flag.
*/
function _isConfigurationApproved(uint256 _projectId, uint256 _configuration)
  private
  view
  returns (bool)
{
  JBFundingCycle memory _fundingCycle = _getStructFor(_projectId, _configuration);
  return _isApproved(_projectId, _fundingCycle);
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
