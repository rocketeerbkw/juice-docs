# transferAddressFor

Contract: [`JBProjects`](../)

Interface: [`IJBProjects`](../../../interfaces/ijbprojects.md)

**The address that can reallocate a handle that have been transferred to it.**

# Definition

```solidity
/** 
  @notice 
  The address that can reallocate a handle that have been transferred to it.

  _handle The handle to look for the transfer address for.
*/
mapping(bytes32 => address) public override transferAddressFor;
```

* `_handle` is the handle to look for the transfer address for.
* The resulting view function can be accessed externally by anyone.
* The resulting function overrides a function definition from the `IJBProjects` interface.
