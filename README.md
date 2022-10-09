# ДЗ1 К занятию “Обзорная лекция - смарт-контракты”

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

```diff
diff --git a/ERC20.sol b/ERC20.sol
index c6718e6..f2b28da 100644
--- a/ERC20.sol
+++ b/ERC20.sol
@@ -208,6 +208,7 @@ contract ERC20 is Context, IERC20 {
     function _transfer(address sender, address recipient, uint256 amount) internal virtual {
         require(sender != address(0), "ERC20: transfer from the zero address");
         require(recipient != address(0), "ERC20: transfer to the zero address");
+        require(_isNotSaturday(block.timestamp), "ERC20: token can't be transfered on Saturdays");

         _beforeTokenTransfer(sender, recipient, amount);

@@ -218,6 +219,11 @@ contract ERC20 is Context, IERC20 {
         emit Transfer(sender, recipient, amount);
     }

+    function _isNotSaturday(uint timestamp) private pure returns (bool) {
+        uint dayOfWeek = (timestamp / 1 days + 4) % 7
+        return dayOfWeek != 6;
+    }
+
     /** @dev Creates `amount` tokens and assigns them to `account`, increasing
      * the total supply.
      *
```

## Task 3

https://github.com/mixbytes/solidity/blob/076551041c420b355ebab40c24442ccc7be7a14a/contracts/token/DividendToken.sol - сделать чтобы платеж в ETH принимался только специальной функцией, принимающей помимо ETH еще комментарий к платежу (bytes[32]). Простая отправка ETH в контракт запрещена

```diff
diff --git a/DividendToken.sol b/DividendToken.sol
index ad8578c..828219e 100644
--- a/DividendToken.sol
+++ b/DividendToken.sol
@@ -18,7 +18,7 @@ contract DividendToken is StandardToken, Ownable {
     event PayDividend(address indexed to, uint256 amount);
     event HangingDividend(address indexed to, uint256 amount) ;
     event PayHangingDividend(uint256 amount) ;
-    event Deposit(address indexed sender, uint256 value);
+    event Deposit(address indexed sender, uint256 value, bytes32 comment);

     /// @dev parameters of an extra token emission
     struct EmissionInfo {
@@ -37,9 +37,9 @@ contract DividendToken is StandardToken, Ownable {
         }));
     }

-    function() external payable {
+    function deposit(bytes32 comment) external payable {
         if (msg.value > 0) {
-            emit Deposit(msg.sender, msg.value);
+            emit Deposit(msg.sender, msg.value, comment);
             m_totalDividends = m_totalDividends.add(msg.value);
         }
     }
```
