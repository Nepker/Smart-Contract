[H-1] Passwords stored on-chain are visible to anyone, not matter solidity variable visibility

Description: All data stored on-chain is visible to anyone and can be read directly from the blockchain. The `PasswordStorge::s_password` variable is intended to be private variable and only accessed through the `PasswordStore::getPassword` function which is intended to be only called by the owner of the contracts
However, anyone can directly read this using any number of offchain methodology.

Impact: The password is not private

Proof of Concept: The test below case shows how anyone could read the password directlt from the blockchain. We use foundry's cast tool to read directly from the storage of the contract without being the owner.
1. Create a locally running chain
``` 
make anvil
```
2. Deploy the contract to the chain
``` 
make deploy
```
3. Run the storage tool
we use 1 becaude that's the storage slot of `s_password` in the contract
```
cast storgae <Address_here> 1 --rpc-url http://127.0.0.:8545
```
you'll get an output that looks like this:
```
0x6d7950617373776f726400000000000000000000000000000000000000000014
```
you can then parse that hex to a string with:
```
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
````
And get the output of :
```
myPassword
```

Recommende Mitigation: Due to this, the overall architecture of the contract should be rethought. One could encrypt password on-chain. This would require the user to remember another password on-chain to decrypt the password. However you'd also likely want to remeber the view function as you wouldn't want the user to accidently send a transaction with the password that decrypts your password.

[H-2] `PasswordStorge::setPassword` is called by anyone

Description: The `PasswordStore::setPassword` function is set to be an external function. However the natspec of the function and overall purpose of the smart contract is that, this function allows only the owner to set a new password.

```
function setPassword(string memory newPassword) external {
@>	// @audit - There are no access control here
	s_password = newPassword;
	emit SetNewPassword();
}
```

Impact: Anyone can set/change the password

Proof of Concept:
Add the following to the `PasswordStore.t.sol` test suite
```
function test_anyone_can_set_password(address randomAddress) public {
	vm.prank(randomAddress);
	string memory expectedPassword = "myNewPassword";
	passwordStore.setPassword(exceptedPassword);
	vm.prank(owner);
	string memory actualPassword = passwordStore.getPassword();
	assertEq(actualpassword, exceptedPassword)''
}
```

Recommended Mitigation: Add an access control modifier to the `setPassword` function

```
if (msg.sender != s_owner) {
	revert PasswordStore__NotOwner();
}
```
