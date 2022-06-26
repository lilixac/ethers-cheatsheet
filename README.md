# ETHERS CHEATSHEET

## Testing

### Deploying contract:
Consider we have `ContractA` and `ContractB` in the contracts folder. 
Asssume, Contract B has some args in constructor.

This works even if folders are nested in contracts directory as:
```
 .
├──  A.sol
├──  folder-1
│  ├──  ContractA.sol
│  └──  Token.sol
├──  folder-2
│  └──  ContractB.sol
└──  utils
   └──  SafeMath.sol
```

### Get Accounts
We generally need more than a account/wallet to test. We can get multiple wallets using `ethers.getSigners()`. It returns array of wallets that we can use. All these wallets have some test eth already preloaded for hardhat development environment.

When calling external method, when we didn't specify the signer, the 1st wallet returned by this method is used. This wallet is used to deploy the contracts as well.

When we need other wallets, we can:
```js
let [wallet1, wallet2, wallet3] = await ethers.getSigners() // first 3 wallets are saved in the variables
await ca.connect(wallet2).methodA('paramA')

// for address
let addr1 = wallet1.getAddress()
await c1.transfer(addr1, ethers.BigNumber.from('1000'))
```

To deploy contracts:

```js
const CA = await ethers.getContractFactory('ContractA')
const CB = await ethers.getContractFactory('ContractB')

ca = CA.deploy()
cb = CB.deploy('arg1','arg2')

// deploy using other account
ca1 = CA.connect(wallet2).deploy()
```


### Calling external method
To call `methodA` external method in ContractA:
```js
await ca.connect(`signer-wallet`).methodA() // no params
await ca.connect(`signer-wallet`).methodA('param1','param2') // with params
```
If signer is not required, we can skip connect. It connects with the 1st account.
```js
await ca.methodA() // no params
await ca.methodA('param1','param2') // with params
```

### Calling payable method
To call `payableMethod` external method in ContractA. The concept of signer is same as above:
```js
const amount = ethers.utils.parseUnits("0.1") // 0.1 ETH
await ca.connect(`signer-wallet`).payableMethod({value: amoun }) // no params
await ca.connect(`signer-wallet`).payableMethod('param1','param2', {value: amount}) // with params
```

### Calling readonly method
To call `readonlyMethod` readonly method in ContractB:
```js
await cb.readonlyMethod() //no params
await cb.readonlyMethod('param1','param2') // with params
```

### BigNumber
We can't just use regular integers everywhere, especially when we're working with tokens. We use a library called BigNumber for it.

```js
ethers.BigNumber.from('1000'); // 100 tokens, not 100 * 10 ** decimals
await erc20.transfer(address, ethers.BigNumber.from('10000')) // transfer 10000 erc20 tokens to address
```

## Frontend 
The methods mentioned previously can be used for frontend.

### Connect Metamask
To connect metamask to your web application.
```js
const connectWallet = async () => {
   await window.ethereum.request({ method: 'eth_requestAccounts' });
   const addr = window.ethereum.selectedAddress
}
```

### Readonly methods
Generally used in useEffect hook. ContractABI can be imported from artifacts.
```js
// gets provider from Metamask
// can be ethereum mainnet, polygon testnet, etc, based on what is selected on metamask
const provider = new ethers.providers.Web3Provider(window.ethereum)
const contractInstance = new ethers.Contract(`contract-address`, contractABI, provider)
const data = await contractInstance.myReadOnlyMethod(params)
```

### External methods

This block will be used to call external methods of a contract. It required signing by the wallet, and takes a transaction fee.
```js
const provider = new ethers.providers.Web3Provider(window.ethereum);
// selected address on metamask is now the signer
const signer = await provider.getSigner();
const contractInstance = new ethers.Contract(`contract-address`, contractABI, provider)
// or you can use try catch for this
const txResult = await contractInstance.myExternalMethod(params)
// await contractInstance.myExternalMethod(address, ethers.BigNumber.from("10000000000000"))
const receipt = txResult.wait() 
if (receipt.status === 1) {
	// transaction successful
	// successful message, call other methods if required
} else {
	// transaction failed
	// error handling
}
```

### Handle BigNumbers
Big Numbers can be quite tricky to handle.

To remain safe, the numerical value recieved by calling the readonly method, if you need to display that value in the UI, for example, user's balance.
```js
// 10 ** 18 wei = 1 ETH
export function weiToETH (hexValue) {
    return parseInt(hexValue.toString()) / 10 ** 18;
}
```

And use the data returned to display in the UI.

For transactions,

```js
const userBal = await ERC20Instance.balanceOf(user)
// userBalance, userBal is of type BigNumber
setUserBalance(userBal) // const [userBalance, setUserBalance] = useState(null)

// to add 2 BigNumbers together
const randomBigNumber = ethers.BigNumber.from("1200000")
userBalance.add(randomBigNumber)

// to subtract 2 BigNumbers
userBalance.sub(randomBigNumber)

// to multiply and divide BigNumber with regular number, the outcome is a bigNumber
const percent = parseInt('60')
const amountToSend = userBalance.mul(percent).div(100)

// to use this bignumber as param for calling methods of contract
const amount = ethers.BigNumber.from(amountToSend.toString())
await ERC20Instance.transfer(`toAddress`, amount)
```



