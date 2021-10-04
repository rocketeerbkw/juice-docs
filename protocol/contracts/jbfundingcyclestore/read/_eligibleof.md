# \_eligibleOf

Contract:[`JBFundingCycleStore`](../)​

{% tabs %}
{% tab title="Step by step" %}
**The project's stored funding cycle that has started and hasn't yet expired.**

_A value of 0 is returned if no funding cycle was found._

Definition:

```javascript
function _eligibleOf(uint256 _projectId) private view returns (uint256 fundingCycleId) { ... } 
```

* `_projectId` is the ID of a project to look through for an eligible cycle.
* The view function is private to this contract.
* The function does not alter state on the blockchain.
* The function returns an ID of an eligible funding cycle if one exists, or 0 if one doesn't exist.

1. Get a reference to the latest funding cycle for the project.  


   _Internal references:_

   * [`latestIdOf`](../properties/latestidof.md)

   ```javascript
   // Get a reference to the project's latest funding cycle.
   fundingCycleId = latestIdOf[_projectId];
   ```

2. If there isn't a funding cycle for the project, there isn't an eligible cycle either.

   ```javascript
   // If there isn't one, theres also no eligible funding cycle.
   if (fundingCycleId == 0) return 0;
   ```

3. Get the struct for the latest funding cycle.  


   _Internal references:_

   * [`_getStructFor`](_getstructfor.md)

   ```javascript
   // Get the necessary properties for the latest funding cycle.
   JBFundingCycle memory _fundingCycle = _getStructFor(fundingCycleId);
   ```

4. If the latest is expired, return an empty funding cycle since there can't be a stored eligible cycle.  


   _Internal references:_

   * [`_SECONDS_IN_DAY`](../properties/_seconds_in_day.md)

   ```javascript
   // If the latest is expired, return an empty funding cycle.
   // A duration of 0 can not be expired.
   if (
     _fundingCycle.duration > 0 &&
     block.timestamp >= _fundingCycle.start + (_fundingCycle.duration * _SECONDS_IN_DAY)
   ) return 0;
   ```

5. Get a reference to the funding cycle that the current cycle is based on.  


   _Internal references:_

   * [`_getStructFor`](_getstructfor.md)

   ```javascript
   // The base cant be expired.
   JBFundingCycle memory _baseFundingCycle = _getStructFor(_fundingCycle.basedOn);
   ```

6. If the base is expired, return an empty funding cycle since there can't be a stored eligible cycle.  


   _Internal references:_

   * [`_SECONDS_IN_DAY`](../properties/_seconds_in_day.md)

   ```javascript
   // If the current time is past the end of the base, return 0.
   // A duration of 0 is always eligible.
   if (
     _baseFundingCycle.duration > 0 &&
     block.timestamp >= _baseFundingCycle.start + (_baseFundingCycle.duration * _SECONDS_IN_DAY)
   ) return 0;
   ```

7. Return the ID that the latest funding cycle is based on.

   ```javascript
   // Return the funding cycle immediately before the latest.
   fundingCycleId = _fundingCycle.basedOn;
   ```
{% endtab %}

{% tab title="Only code" %}
```javascript
/**
  @notice 
  The project's stored funding cycle that has started and hasn't yet expired.
  
  @dev
  A value of 0 is returned if no funding cycle was found.
  
  @param _projectId The ID of the project to look through.

  @return fundingCycleId The ID of the active funding cycle.
*/
function _eligibleOf(uint256 _projectId) private view returns (uint256 fundingCycleId) {
  // Get a reference to the project's latest funding cycle.
  fundingCycleId = latestIdOf[_projectId];

  // If there isn't one, theres also no eligible funding cycle.
  if (fundingCycleId == 0) return 0;

  // Get the necessary properties for the latest funding cycle.
  JBFundingCycle memory _fundingCycle = _getStructFor(fundingCycleId);

  // If the latest is expired, return an undefined funding cycle.
  // A duration of 0 can not be expired.
  if (
    _fundingCycle.duration > 0 &&
    block.timestamp >= _fundingCycle.start + (_fundingCycle.duration * _SECONDS_IN_DAY)
  ) return 0;

  // The base cant be expired.
  JBFundingCycle memory _baseFundingCycle = _getStructFor(_fundingCycle.basedOn);

  // If the current time is past the end of the base, return 0.
  // A duration of 0 is always eligible.
  if (
    _baseFundingCycle.duration > 0 &&
    block.timestamp >= _baseFundingCycle.start + (_baseFundingCycle.duration * _SECONDS_IN_DAY)
  ) return 0;

  // Return the funding cycle immediately before the latest.
  fundingCycleId = _fundingCycle.basedOn;
}
```
{% endtab %}

{% tab title="Bug bounty" %}
| Category | Description | Reward |
| :--- | :--- | :--- |
| **Optimization** | Help make this operation more efficient. | 0.5ETH |
| **Low severity** | Identify a vulnerability in this operation that could lead to an inconvenience for a user of the protocol or for a protocol developer. | 1ETH |
| **High severity** | Identify a vulnerability in this operation that could lead to data corruption or loss of funds. | 5+ETH |
{% endtab %}
{% endtabs %}
