## Deploy the whitelist and NFT contracts on the testnet.

> Before deploying contracts on the testnet, you can thoroughly test the contracts using `JS VM`, as it doesn't require gas and deployment is quick.

Do you recall that we received test tokens on the `Polygon Mumbai` testnet initially? Next, our task is to deploy the `Whitelist.sol` and `Shield.sol` contracts on this testnet.

Click on the "Connected" button in the upper-right corner and then select "Disconnect JS VM".

![image-20230223133803703](../resource/Images/image-20230223133803703.png)

Select `Injected Web3 Provider`

![image-20230223133842606](../resource/Images/image-20230223133842606.png)

Click `Metamask`

![image-20230223134000225](../resource/Images/image-20230223134000225.png)

Select `Mumbai`

![image-20230223134208657](../resource/Images/image-20230223134208657.png)

### Depoly Whitelist.sol 

Compile `Whitelis.sol` first

![image-20230223134424725](../resource/Images/image-20230223134424725.png)

Navigate to the Deploy panel and enter an array of up to 4 distinct whitelist addresses, such as:`["0xa323A54987cE8F51A648AF2826beb49c368B8bC6","0x4f2249958655e1b78064cc3d1F1b8a0B12D1dbDE","0x03692A0187c6D8Be757Be0f0775b94B484fFC15D","0x643AAe9DA7f3542f370FD87ea1781bD54D541578"]`

Click `Deploy`

![image-20230223135017015](../resource/Images/image-20230223135017015.png)

The next step, which differs from using JS VM, involves verifying the contract. After verifying the contract, it will be open-sourced on platforms like Etherscan, allowing others to review the source code and ensuring the contract's fairness and accuracy.

ChainIDE offers a convenient Verify plugin. Users simply need to acquire the corresponding Scan API Key to quickly verify the contract.

Switch to the Verify page and click on the redirect link next to "Polygon API Key".

![image-20230223135415733](../resource/Images/image-20230223135415733.png)

On the login page, choose either `"Login"` or `"Click to sign up"`.

![image-20230114122015922](../resource/Images/image-20230114122015922.png)

Select `API Keys`

![image-20230114122844649](../resource/Images/image-20230114122844649.png)

Select add

![image-20230114122902739](../resource/Images/image-20230114122902739.png)

App Name, enter a name you like, and then click `Continue`

![image-20230114122923433](../resource/Images/image-20230114122923433.png)

Then, your `API Key` will be generated. (Please be cautious and avoid sharing the key with others. The usage speed of the key is limited. One of my keys is `98TSWD2C57949VSZVFCZ15WKYDVSCMJKQM`). Click `"Copy"`.

![image-20230114122953738](../resource/Images/image-20230114122953738.png)

Paste into ChainIDE

![image-20230223140212926](../resource/Images/image-20230223140212926.png)

Also, in the Deploy panel, copy the corresponding constructor and contract address.

![image-20230223140319580](../resource/Images/image-20230223140319580.png)

Paste them into the Verify panel, then click `"Verify"`.

![image-20230223140526416](../resource/Images/image-20230223140526416.png)

Congratulations, verification successful!

![image-20230223141313801](../resource/Images/image-20230223141313801.png)

### Deploy Shield.sol

Let's start by compiling the `Shield.sol`.

![image-20230223141704965](../resource/Images/image-20230223141704965.png)

Switch to the deploy page. In the constructor:

* `baseURI` should be the IPFS URI generated in the Metadata section. Mine is: `ipfs://bafybeihuwmkxnqban2ukneymhwctxfqec5ywrdiqyc7vmyegftrrllf7gq/`
* `whitelistContract` is the address of the whitelist we just deployed. Mine is:` 0x78dd3EA257535E08BA0Ee5d2eB5E5c8C64304AFf`

![image-20230223142123752](../resource/Images/image-20230223142123752.png)

After deployment, navigate to the `Verify` page again. Input the same `baseURI` and `whitelistContract` for verification. Also, input the address of the just-deployed `Shield.sol` contract. Click "Verify" and wait for successful verification.

![image-20230223142357545](../resource/Images/image-20230223142357545.png)

Cool, the contract-related part is almost concluded. Next, we need to develop a frontend page for users on the whitelist to perform minting operations.