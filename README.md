# Don-key Finance Bounty Challenge

**Intro**: Don-key is a social trading platform for crypto yield farming. Investors have the option to copy proven-to-work and profitable yield farming Strategies from well-known farmers. 

**Cubes**: A cube is a specific action, an investment step that runs on a specific protocol like [Ellipsis](https://www.ellipsis.finance/), [Pancakeswap](https://pancakeswap.finance/) etc. 
- Cube example 1: Deposit BUSD on Alpacafinance to get ibBUSD as a reward.
- Cube example 2: Stake ibBUSD  on Alpacafinance to get ALPACA rewards

Note: You can perform the examples steps above as a user by visiting the Alpacafinance website.

**Strategy**:
The strategy accepts a list of cubes and runs them one after another in order. 
In code it looks like below.
You will not need to change the Solidity code below, it's included here to help you understand how a Strategy is run:

    // in a .sol file
    contract Strategy {
      enum cubesType {
        INVEST, 
    }

    struct cubesDetails{
        address protocolAddress;
        bytes abis;
    }

    mapping(cubesType => cubesDetails[]) private cubes;

    /**
     * @dev add new multiple cubes to strategy, based on cubes further
     *      investment will be done.
     * @param _protocolAddress address of tos
     * @param _abis json data of abicall
     * @param _cType Cubes Type(currently only: INVEST)
     */
    function addCubes(
        address[] memory _protocolAddress,
        bytes[] memory _abis,
        cubesType  _cType
    )
        external
    {
        require(
            locked == false,
            "Strategy is locked and cannot be changed”
        );

        require(
            _protocolAddress.length == _abis.length,
            "Tos and datas length inconsistent"
        );

        cubesDetails memory cube;
        for (uint256 i = 0; i < _protocolAddress.length; i++)
        {
            
            cube.protocolAddress = _protocolAddress[i];
            cube.abis = _abis[i];

            cubes[_cType].push(cube);
        }
    }


     /**
         * @dev When this function is called it will run strategy
         *      based on cubes added to strategy.
         * @param _cType Cubes Type(currently only: INVEST)
         */
        function runStrategy(
            cubesType _cType
        )
            public payable
        {
            for (uint256 i = 0; i < cubes[_cType].length; i++) {
                    executeCube(cubes[_cType][i].protocolAddress, cubes[_cType][i].abis);
            }
        }
    }

    /**
     * @dev This function executes cubes one by one
     * @param _target Protocol Address
     * @param _data Protocol function callData Abi
     */
    function executeCube(
        address _target,
        bytes memory _data
    ) 
        internal 
    {
        require(_target != address(0), "target-invalid");

        assembly {
            let succeeded := call(
                gas(),
                _target,
                0,
                add(_data, 0x20),
                mload(_data),   
                0,
                0
            )

            switch iszero(succeeded)
                case 1 {
                    // throw if delegatecall failed
                    let size := returndatasize()
                    returndatacopy(0x00, 0x00, size)
                    revert(0x00, "Transcation failed")
            }
        }
    }


## Challenge: 
Given the **Strategy** below, write the code needed for the Cubes, so that when the **Cubes** are passed in to the **Strategy**, the Strategy runs successfully 
1. Go to Alpacafinance and deposit BUSD. Receive ibBUSD
2. Still on Alpacafinance take the ibBUSD from step 1 and stake it to get ALPACA as a reward
3. When the ALPACA balance reaches $1000 value, transfer the rewarded ALPACA to Pancakeswap
4. On Pancakeswap swap ALPACA for BUSD and start over from step 1

### The code has to be written in Web3 and be capable of being run from the frontend. Example code below from another, sample strategy:

    // in a .js file
    import abi from "{PATH_TO_PANCAKESWAP_ABI}";
    
    var Amount = web3.utils.toWei('1');
    var BDOaddress="0x190b589cf9Fb8DDEabBFeae36a813FFb2A702454";
    var BUSDaddress = "0xe9e7CEA3DedcA5984780Bafc599bD69ADd087D56";

    // Declarations of the strategy contract address, the pool contract address and the address of every protocol we would like to get a cube of
    var poolAddress = "0x20650450B725AE64763b88aD95AD8e9D11028C0b";
    var strategyAddress = "0xB885aF37aDb11e200747Ae9E8f693d0E44751c09";
    var PancakeRouteraddress="0x05fF2B0DB69458A0750badebc4f9e13aDd608C7F";

    PancakeRouter=[abi,PancakeRouteraddress];
    var data= PancakeRouter.methods.swapExactTokensForTokens(Amount,0,[BUSDaddress,BDOaddress],strategyAddress,blockData.timestamp+10000).encodeABI();
    var addCubes = await Strategy.methods.addCubes([PancakeRouteraddress,PancakeRouteraddress,PancakeRouter],[data]).send({ from: accounts[0] });


