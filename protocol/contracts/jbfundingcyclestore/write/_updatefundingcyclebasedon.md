# \_updateFundingCycleBasedOn

Contract:[`JBFundingCycleStore`](../)​

{% tabs %}
{% tab title="Step by step" %}
**Updates intrinsic properties for a funding cycle given a base cycle.**  
  
Definition:

```javascript
function _updateFundingCycleBasedOn(
  JBFundingCycle memory _baseFundingCycle,
  uint256 _mustStartOnOrAfter,
  uint256 _weight,
  bool _copy
) private returns (uint256 fundingCycleId) { ... }
```

* `_baseFundingCycle` is the cycle that the one being updated is based on.
* `_mustStartOnOrAfter` is the time before which the initialized funding cycle can't start.
* `_weight` is the weight to store along with a newly updated configurable funding cycle.
* `_copy` is a flag indicating if non-intrinsic properties should be copied from the base funding cycle.
* The function is private to this contract.
* The function returns the ID of the updated funding cycle.
{% endtab %}

{% tab title="Only code" %}
```javascript
/** 
  @notice
  Updates intrinsic properties for a funding cycle given a base cycle.

  @param _baseFundingCycle The cycle that the one being updated is based on.
  @param _mustStartOnOrAfter The time before which the initialized funding cycle can't start.
  @param _weight The weight to store along with a newly updated configurable funding cycle.
  @param _copy A flag indicating if non-intrinsic properties should be copied from the base funding cycle.

  @return fundingCycleId The ID of the funding cycle that was updated.
*/
function _updateFundingCycleBasedOn(
  JBFundingCycle memory _baseFundingCycle,
  uint256 _mustStartOnOrAfter,
  uint256 _weight,
  bool _copy
) private returns (uint256 fundingCycleId) {
  // Derive the correct next start time from the base.
  uint256 _start = _deriveStartFrom(_baseFundingCycle, _mustStartOnOrAfter);

  // A weight of 1 is treated as a weight of 0.
  _weight = _weight > 0
    ? (_weight == 1 ? 0 : _weight)
    : _deriveWeightFrom(_baseFundingCycle, _start);

  // Derive the correct number.
  uint256 _number = _deriveNumberFrom(_baseFundingCycle, _start);

  // Update the intrinsic properties.
  fundingCycleId = _packAndStoreIntrinsicPropertiesOf(
    _baseFundingCycle.projectId,
    _number,
    _weight,
    _baseFundingCycle.id,
    _start
  );

  // Copy if needed.
  if (_copy) {
    // Save the configuration efficiently.
    _packAndStoreConfigurationPropertiesOf(
      fundingCycleId,
      _baseFundingCycle.configured,
      _baseFundingCycle.ballot,
      _baseFundingCycle.duration,
      _baseFundingCycle.currency,
      _baseFundingCycle.fee,
      _baseFundingCycle.discountRate
    );

    _metadataOf[fundingCycleId] = _metadataOf[_baseFundingCycle.id];
    _targetOf[fundingCycleId] = _targetOf[_baseFundingCycle.id];
  }
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
