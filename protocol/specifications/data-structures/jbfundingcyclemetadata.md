# JBFundingCycleMetadata

```solidity
struct JBFundingCycleMetadata {
  // The reserved rate of the funding cycle. This number is a percentage calculated out of 10000.
  uint256 reservedRate;
  // The redemption rate of the funding cycle. This number is a percentage calculated out of 10000.
  uint256 redemptionRate;
  // The redemption rate to use during an active ballot of the funding cycle. This number is a percentage calculated out of 10000.
  uint256 ballotRedemptionRate;
  // If the pay functionality should be paused during the funding cycle.
  bool pausePay;
  // If the withdraw functionality should be paused during the funding cycle.
  bool pauseWithdrawals;
  // If the redeem functionality should be paused during the funding cycle.
  bool pauseRedeem;
  // If the mint functionality should be paused during the funding cycle.
  bool pauseMint;
  // If the burn functionality should be paused during the funding cycle.
  bool pauseBurn;
  // If migrating terminals should be allowed during this funding cycle.
  bool allowTerminalMigration;
  // If migrating controllers should be allowed during this funding cycle.
  bool allowControllerMigration;
  // If fees should be held during this funding cycle.
  bool holdFees;
  // If the data source should be used for pay transactions during this funding cycle.
  bool useDataSourceForPay;
  // If the data source should be used for redeem transactions during this funding cycle.
  bool useDataSourceForRedeem;
  // The data source to use during this funding cycle.
  IJBFundingCycleDataSource dataSource;
}
```