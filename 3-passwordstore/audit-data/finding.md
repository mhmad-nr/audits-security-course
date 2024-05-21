### [H-1] Variables which stores in storage are public , any one can see the password

**Description:** Variables which stores in storage are public , the `PasswordStore::s_password` variable is tend to be used only visable from the `PasswordStore::getPassword` function which can only call by the owner of the password

**Impact:** The `PasswordStore::s_password` variable to anyone

**Proof of Concept:** ( Proof of Code)
below test shows how anyone can access to the password

1. Create a locally running chain

```bash
make anvil
```

2. Deploy the Contract on the chain

```bash
make deploy
```

3. Read the contract storage a slot 1 because `PasswordStore::s_password` variable located in there

```bash
cast storage "contract-address" 1
```

You will get some output like this:
"0x6d7950617373776f726400000000000000000000000000000000000000000014"

4. Pasre it from bytes to string

```bash
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```

Now you get the password
"myPassword"

**Recommended Mitigation:** Due to this, the overall architecture of the contract should be rethought.
One could encrypt the password off-chain, and then store the encrypted password on-chain. This
would require the user to remember another password on-chain to decrypt the password. However,
you’d also likely want to remove the view function as you wouldn’t want the user to accidentally send
a transaction with the password that decrypts your password.

### [H-2] The `PasswordStore::setPassword` function can call by even a non-owner, which make the passwored changable by anyone

**Description:** set the password can be do only by owner while The `PasswordStore::setPassword` function has no any restriction to prvent other from chaining the password

```javascript
    function setPassword(string memory newPassword) external {
@>      // thres no any access control
        s_password = newPassword;
        emit SetNetPassword();
    }
```

**Impact:** Anyone can set/change the password by calling the `PasswordStore::setPassword` function

**Proof of Concept:** Add the following to the `PasswordStore.t.sol` test file

<details>
<summary>Code</summary>

```javascript

 function test_non_owner_can_set_password(address randomAddress) public {
        vm.assume(randomAddress != owner);
        vm.prank(randomAddress);

        string memory expectedPassword = "myNewPassword";
        passwordStore.setPassword(expectedPassword);
        vm.prank(owner);

        string memory actualPassword = passwordStore.getPassword();
        assertEq(actualPassword, expectedPassword);
    }

```

</details>

**Recommended Mitigation:** To check who is calling this function and prevent non-owners from changing the password only needs to add a contition for checking is msg.sender equal to owner
add the following at the beginig of the `PasswordStord::setPassword` function:

```diff

+    if(msg.sender != owner){
+        revert Error()
+    }

```

### TITLE The natspec indicate that the `PasswordStord::getPassword` function has a parameter like `getPassword(param)`

**Description:** 
```javascript
    /*
     * @notice This allows only the owner to retrieve the password.
@>   * @param newPassword The new password to set.
     */
    function getPassword() external view returns (string memory) {}
```

**Impact:** The natspec is inccorect

**Recommended Mitigation:** Remove the line of coment

```diff
-    * @param newPassword The new password to set.
```
