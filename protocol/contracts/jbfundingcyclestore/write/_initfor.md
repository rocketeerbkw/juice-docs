# \_initFor

Contract:[`JBFundingCycleStore`](../)​

{% tabs %}
{% tab title="Step by step" %}
**Initializes a funding cycle with the appropriate properties.**  
  
Definition:

```javascript
function _initFor(
  uint256 _projectId,
  JBFundingCycle memory _baseFundingCycle,
  uint256 _mustStartOnOrAfter,
  uint256 _weight
) private returns (uint256 newFundingCycleId) { ... }
```

* `_projectId` is the ID of the project to which the funding cycle being initialized belongs.
* `_baseFundingCycle` is the funding cycle to base the initialized one on.
* `_mustStartOnOrAfter` is the time before which the initialized funding cycle can't start.
* The function is private to this contract.
* The function returns the ID of the initialized funding cycle.

1. If no base funding cycle was provided, create a first funding cycle for the project. Otherwise, create a new funding cycle by calling `_updateFundingCycleBasedOn`, which will derive properties for the funding cycle that follows the specified base cycle, and store them to an ID.

   ```javascript
   // If there is no base, initialize a first cycle.
   if (_baseFundingCycle.id == 0) {
     // The first number is 1.
     uint256 _number = 1;

     // Get the formatted ID.
     newFundingCycleId = _idFor(_projectId, _number);

     // Set fresh intrinsic properties.
     _packAndStoreIntrinsicPropertiesOf(
       _projectId,
       _number,
       _weight,
       _baseFundingCycle.id,
       block.timestamp
     );
  
   } else {
     // Update the intrinsic properties of the funding cycle being initialized.
     newFundingCycleId = _updateFundingCycleBasedOn(_baseFundingCycle, _mustStartOnOrAfter, _weight);
   }
   ```

2. Store the initialized ID as the `latestIdOf` the project.

   ```javascript
   // Set the project's latest funding cycle ID to the new count.
   latestIdOf[_projectId] = newFundingCycleId;
   ```

3. Get a reference to the [`JBFundingCycle`](../../../data-structures/jbfundingcycle.md) structure for the newly initialized funding cycle.

   ```javascript
   // Get a reference to the funding cycle with updated intrinsic properties.
   JBFundingCycle memory _fundingCycle = _getStructFor(newFundingCycleId);
   ```

4. Emit an `Init` event with the all relevant parameters.   


   _Event references:_

   * [`Init`](../events/init.md) 

   ```javascript
   emit Init(
     newFundingCycleId,
     _fundingCycle.projectId,
     _fundingCycle.number,
     _fundingCycle.basedOn,
     _fundingCycle.weight,
     _fundingCycle.start
   );
   ```
{% endtab %}

{% tab title="Only code" %}
```javascript
/**
  @notice 
  Initializes a funding cycle with the appropriate properties.

  @param _projectId The ID of the project to which the funding cycle being initialized belongs.
  @param _baseFundingCycle The funding cycle to base the initialized one on.
  @param _mustStartOnOrAfter The time before which the initialized funding cycle can't start.

  @return newFundingCycleId The ID of the initialized funding cycle.
*/
function _initFor(
  uint256 _projectId,
  JBFundingCycle memory _baseFundingCycle,
  uint256 _mustStartOnOrAfter,
  uint256 _weight
) private returns (uint256 newFundingCycleId) {
  // If there is no base, initialize a first cycle.
  if (_baseFundingCycle.id == 0) {
    // The first number is 1.
    uint256 _number = 1;

    // Get the formatted ID.
    newFundingCycleId = _idFor(_projectId, _number);

    // Set fresh intrinsic properties.
    _packAndStoreIntrinsicPropertiesOf(
      _projectId,
      _number,
      _weight,
      _baseFundingCycle.id,
      block.timestamp
    );
    
  } else {
    // Update the intrinsic properties of the funding cycle being initialized.
    newFundingCycleId = _updateFundingCycleBasedOn(_baseFundingCycle, _mustStartOnOrAfter, _weight);
  }

  // Set the project's latest funding cycle ID to the new count.
  latestIdOf[_projectId] = newFundingCycleId;

  // Get a reference to the funding cycle with updated intrinsic properties.
  JBFundingCycle memory _fundingCycle = _getStructFor(newFundingCycleId);

  emit Init(
    newFundingCycleId,
    _fundingCycle.projectId,
    _fundingCycle.number,
    _fundingCycle.basedOn,
    _fundingCycle.weight,
    _fundingCycle.start
  );
}
```
{% endtab %}

{% tab title="Events" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">Name</th>
      <th style="text-align:left">Data</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b><code>Init</code></b>
      </td>
      <td style="text-align:left">
        <ul>
          <li><code>uint256 indexed fundingCycleId</code> 
          </li>
          <li><code>uint256 indexed projectId</code> 
          </li>
          <li><code>uint256 indexed number</code> 
          </li>
          <li><code>uint256 basedOn</code> 
          </li>
          <li><code>uint256 weight</code> 
          </li>
          <li><code>uint256 start</code>
          </li>
        </ul>
        <p><a href="../events/init.md">more</a>
        </p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Bug bounty" %}
| Category | Description | Reward |
| :--- | :--- | :--- |
| **Optimization** | Help make this operation more efficient. | 0.5ETH |
| **Low severity** | Identify a vulnerability in this operation that could lead to an inconvenience for a user of the protocol or for a protocol developer. | 1ETH |
| **High severity** | Identify a vulnerability in this operation that could lead to data corruption or loss of funds. | 5+ETH |
{% endtab %}
{% endtabs %}


