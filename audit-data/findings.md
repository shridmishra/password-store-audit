### [H-1] storing the password on-chain makes it visible to anyone and no longer private.

**Description:** All data stored on-chain is visible to anyone, and can be read directly from the blockchain.
The `PasswordStore::s_password` variable is intended to be a private variable and can be accessed through a `PasswordStore::getPassword` function, which is intended to be called by the owner of the contract.

We show one such method of reading data off chain below.

**Impact:** Anyone can read the private password, severely breaking the functionality of the protocol.

**Proof of Concept:**

1. Create a local chain

```bash
make anvil
```

2. Deploy the contract

```
make deploy
```

3.

```
cast storage 0x5FbDB2315678afecb367f032d93F642f64180aa3 1 --rpc-url http://127.0.0.1:8545
```

4.

```
 cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```

**Recommended Mitigation:** Due this you, overall contract should be rethought, encrypt the password off-chain.

### [H-2] `PasswordStore::setPassword` has no access control meaning a non-owner could change it.

**Description:** The `PasswordStore::setPassword` is set to be `external` , however the purpose of the smart contact is that `This function only allows owner to set new password.`

```javascript

  function setPassword(string memory newPassword) external {
   @>  // @audit - There are no access controls
        s_password = newPassword;
        emit SetNetPassword();
    }


```

**Impact:** Anyone can change the password of contract, severely breaking the contract intended functionality.

**Proof of Concept:** Add this to `PasswordStore.t.sol` test file.

<details>
<summary>Code</summary>

```javascript
    function test_anyone_can_set_password( address randomAddress) public {
        vm.assume(randomAddress != owner);
        vm.prank(randomAddress);
        string memory  expectedPassword = "myNewPassword";
        passwordStore.setPassword(expectedPassword);
        vm.prank(owner);
        string memory actualPassword = passwordStore.getPassword();
        assertEq(actualPassword, expectedPassword);

    }
```

</details>

**Recommended Mitigation:** Add access control to `setPassword` function.

### **[I-1] Incorrect NatSpec Parameter in `PasswordStore::getPassword`**  

#### **Description:**  
The NatSpec documentation for the `PasswordStore::getPassword` function incorrectly references a parameter `newPassword` that does not exist in the function signature. The actual function signature is:  

```solidity
function getPassword() external view returns (string memory)
```
However, the NatSpec comment suggests the function takes a `newPassword` parameter, which is misleading and incorrect.  

#### **Impact:**  
- The documentation misrepresents the function’s behavior.  
- Developers might assume the function requires a parameter, leading to confusion.  
- Incorrect documentation can cause issues during contract integration and maintenance.  

#### **Recommended Mitigation:**  
Remove the incorrect `@param` line from the NatSpec documentation.  

#### **Suggested Correction:**  

```diff
 /*
  * @notice This function allows only the owner to retrieve the password.
- * @param newPassword The new password to set.
  */
 function getPassword() external view returns (string memory);
```  

 