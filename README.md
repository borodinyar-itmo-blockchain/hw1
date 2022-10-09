# hw1

## Task 1

https://github.com/gnosis/MultiSigWallet/blob/master/contracts/MultiSigWallet.sol - сделать, чтобы с баланса multisig-контракта за одну транзакцию не могло бы уйти больше, чем 66 ETH

```diff
diff --git a/MultiSigWallet.sol b/MultiSigWallet.sol       
index 5a3717d..0861f0f 100644
--- a/MultiSigWallet.sol
+++ b/MultiSigWallet.sol
@@ -22,6 +22,7 @@ contract MultiSigWallet {
      *  Constants
      */
     uint constant public MAX_OWNER_COUNT = 50;
+    uint constant public MAX_TRANSACTION_ETHER = 66 ether;

     /*
      *  Storage
@@ -91,6 +92,11 @@ contract MultiSigWallet {
         _;
     }

+    modifier validValue(uint value) {
+        require(value <= MAX_TRANSACTION_ETHER);
+        _;
+    }
+
     /// @dev Fallback function allows to deposit ether.
     function()
         payable
@@ -289,6 +295,7 @@ contract MultiSigWallet {
     function addTransaction(address destination, uint value, bytes data)
         internal
         notNull(destination)
+        validValue(value)
         returns (uint transactionId)
     {
         transactionId = transactionCount;
```

## Task 2

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/f2112be4d8e2b8798f789b948f2a7625b2350fe7/contracts/token/ERC20/ERC20.sol - сделать, чтобы токен не мог бы быть transferred по субботам