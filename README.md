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

To deploy them:

```js
const CA = await ethers.getContractFactory('ContractA')
const CB = await ethers.getContractFactory('ContractB')

ca = CA.deploy()
cb = CB.deploy('arg1','arg2')
```


### Calling external method
To call `methodA` external method in ContractA:
```js
await ca.connect(`signer-wallet`).methodA() // no params
await ca.connect(`signer-wallet`).methodA('param1','param2') // with params
```
If signer is not required, we can just:
```js
await ca.methodA() // no params
await ca.methodA('param1','param2') // with params
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

### Get Accounts
We generally need more than a account/wallet to test. We can get multiple wallets using `ethers.getSigners()`. It returns array of wallets that we can use. All these wallets have some test eth already preloaded for hardhat development environment.

When calling external method, when we didn't specify the signer, the 1st wallet returned by this method is used. This wallet is used to deploy the contracts as well.

When we need other wallets, we can:
```js
let [wallet1, wallet2, wallet3] = await ethers.getSigners() // first 3 wallets are saved in the variables
await ca.connect(wallet2).methodA('paramA')
```