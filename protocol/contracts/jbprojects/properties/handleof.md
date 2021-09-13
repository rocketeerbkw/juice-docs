# handleOf

Contract: [`JBProjects`](../)

Interface: ****`IJBProjects`

**The unique handle for each project.**

Each project must have a handle.  
****  
Definition:

```javascript
/** 
  @notice 
  The unique handle for each project.

  @dev
  Each project must have a handle.
*/
mapping(uint256 => bytes32) public override handleOf;
```

* `uint256` is the ID of the project to which the handle belongs.
* The resulting view function can be accessed externally by anyone. 
* The resulting function overrides a function definition from the `IJBProjects` interface.
