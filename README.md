Audit of BABYTOKEN 

Here are the slither findings :



Compilation warnings/errors on ./contracts/BABYTOKEN.sol:
Warning: Contract code size exceeds 24576 bytes (a limit introduced in Spurious Dragon). This contract may not be deployable on mainnet. Consider enabling the optimizer (with a low "runs" value!), turning off revert strings, or using libraries.
    --> ./contracts/BABYTOKEN.sol:2584:1:
     |
2584 | contract BABYTOKEN is ERC20, Ownable, BaseToken {
     | ^ (Relevant source part starts here and spans across multiple lines).



BABYTOKEN.addLiquidity(uint256,uint256) (contracts/BABYTOKEN.sol#3125-3138) sends eth to arbitrary user
        Dangerous calls:
        - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#functions-that-send-ether-to-arbitrary-destinations

Reentrancy in BABYTOKEN._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#2968-3052):
        External calls:
        - swapAndSendToFee(marketingTokens) (contracts/BABYTOKEN.sol#2997)
                - IERC20(rewardToken).transfer(_marketingWalletAddress,newBalance) (contracts/BABYTOKEN.sol#3063)
                - uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3116-3122)
        - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
                - uniswapV2Router.swapExactTokensForETHSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3098-3104)
        - swapAndSendDividends(sellTokens) (contracts/BABYTOKEN.sol#3005)
                - success = IERC20(rewardToken).transfer(address(dividendTracker),dividends) (contracts/BABYTOKEN.sol#3143-3146)
                - dividendTracker.distributeCAKEDividends(dividends) (contracts/BABYTOKEN.sol#3149)
                - uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3116-3122)
        External calls sending eth:
        - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
        State variables written after the call(s):
        - super._transfer(from,address(this),fees) (contracts/BABYTOKEN.sol#3024)
                - _balances[sender] = senderBalance - amount (contracts/BABYTOKEN.sol#379)
                - _balances[recipient] += amount (contracts/BABYTOKEN.sol#381)
        ERC20._balances (contracts/BABYTOKEN.sol#181) can be used in cross function reentrancies:
        - ERC20._mint(address,uint256) (contracts/BABYTOKEN.sol#397-407)
        - ERC20._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#366-386)
        - ERC20.balanceOf(address) (contracts/BABYTOKEN.sol#246-248)
        - super._transfer(from,to,amount) (contracts/BABYTOKEN.sol#3027)
                - _balances[sender] = senderBalance - amount (contracts/BABYTOKEN.sol#379)
                - _balances[recipient] += amount (contracts/BABYTOKEN.sol#381)
        ERC20._balances (contracts/BABYTOKEN.sol#181) can be used in cross function reentrancies:
        - ERC20._mint(address,uint256) (contracts/BABYTOKEN.sol#397-407)
        - ERC20._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#366-386)
        - ERC20.balanceOf(address) (contracts/BABYTOKEN.sol#246-248)
        - swapping = false (contracts/BABYTOKEN.sol#3007)
        BABYTOKEN.swapping (contracts/BABYTOKEN.sol#2592) can be used in cross function reentrancies:
        - BABYTOKEN._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#2968-3052)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities

BABYTOKEN.swapAndSendToFee(uint256) (contracts/BABYTOKEN.sol#3054-3064) ignores return value by IERC20(rewardToken).transfer(_marketingWalletAddress,newBalance) (contracts/BABYTOKEN.sol#3063)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#unchecked-transfer

Reentrancy in DividendPayingToken._withdrawDividendOfUser(address) (contracts/BABYTOKEN.sol#2141-2167):
        External calls:
        - success = IERC20(rewardToken).transfer(user,_withdrawableDividend) (contracts/BABYTOKEN.sol#2151-2154)
        State variables written after the call(s):
        - withdrawnDividends[user] = withdrawnDividends[user].sub(_withdrawableDividend) (contracts/BABYTOKEN.sol#2157-2159)
        DividendPayingToken.withdrawnDividends (contracts/BABYTOKEN.sol#2106) can be used in cross function reentrancies:
        - DividendPayingToken._withdrawDividendOfUser(address) (contracts/BABYTOKEN.sol#2141-2167)
        - DividendPayingToken.withdrawableDividendOf(address) (contracts/BABYTOKEN.sol#2179-2186)
        - DividendPayingToken.withdrawnDividendOf(address) (contracts/BABYTOKEN.sol#2191-2198)
Reentrancy in BABYTOKENDividendTracker.process(uint256) (contracts/BABYTOKEN.sol#2473-2525):
        External calls:
        - processAccount(address(account),true) (contracts/BABYTOKEN.sol#2506)
                - success = IERC20(rewardToken).transfer(user,_withdrawableDividend) (contracts/BABYTOKEN.sol#2151-2154)
        State variables written after the call(s):
        - lastProcessedIndex = _lastProcessedIndex (contracts/BABYTOKEN.sol#2522)
        BABYTOKENDividendTracker.lastProcessedIndex (contracts/BABYTOKEN.sol#2284) can be used in cross function reentrancies:
        - BABYTOKENDividendTracker.getAccount(address) (contracts/BABYTOKEN.sol#2376-2423)
        - BABYTOKENDividendTracker.getLastProcessedIndex() (contracts/BABYTOKEN.sol#2368-2370)
        - BABYTOKENDividendTracker.lastProcessedIndex (contracts/BABYTOKEN.sol#2284)
        - BABYTOKENDividendTracker.process(uint256) (contracts/BABYTOKEN.sol#2473-2525)
Reentrancy in BABYTOKEN.updateDividendTracker(address) (contracts/BABYTOKEN.sol#2728-2751):
        External calls:
        - newDividendTracker.excludeFromDividends(address(newDividendTracker)) (contracts/BABYTOKEN.sol#2743)
        - newDividendTracker.excludeFromDividends(address(this)) (contracts/BABYTOKEN.sol#2744)
        - newDividendTracker.excludeFromDividends(owner()) (contracts/BABYTOKEN.sol#2745)
        - newDividendTracker.excludeFromDividends(address(uniswapV2Router)) (contracts/BABYTOKEN.sol#2746)
        State variables written after the call(s):
        - dividendTracker = newDividendTracker (contracts/BABYTOKEN.sol#2750)
        BABYTOKEN.dividendTracker (contracts/BABYTOKEN.sol#2594) can be used in cross function reentrancies:
        - BABYTOKEN._setAutomatedMarketMakerPair(address,bool) (contracts/BABYTOKEN.sol#2820-2832)
        - BABYTOKEN._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#2968-3052)
        - BABYTOKEN.claim() (contracts/BABYTOKEN.sol#2956-2958)
        - BABYTOKEN.constructor(string,string,uint256,address[4],uint256[3],uint256,address,uint256) (contracts/BABYTOKEN.sol#2658-2720)
        - BABYTOKEN.dividendTokenBalanceOf(address) (contracts/BABYTOKEN.sol#2886-2892)
        - BABYTOKEN.dividendTracker (contracts/BABYTOKEN.sol#2594)
        - BABYTOKEN.excludeFromDividends(address) (contracts/BABYTOKEN.sol#2894-2896)
        - BABYTOKEN.getAccountDividendsInfo(address) (contracts/BABYTOKEN.sol#2906-2921)
        - BABYTOKEN.getAccountDividendsInfoAtIndex(uint256) (contracts/BABYTOKEN.sol#2923-2938)
        - BABYTOKEN.getClaimWait() (contracts/BABYTOKEN.sol#2851-2853)
        - BABYTOKEN.getLastProcessedIndex() (contracts/BABYTOKEN.sol#2960-2962)
        - BABYTOKEN.getMinimumTokenBalanceForDividends() (contracts/BABYTOKEN.sol#2862-2868)
        - BABYTOKEN.getNumberOfDividendTokenHolders() (contracts/BABYTOKEN.sol#2964-2966)
        - BABYTOKEN.getTotalDividendsDistributed() (contracts/BABYTOKEN.sol#2870-2872)
        - BABYTOKEN.isExcludedFromDividends(address) (contracts/BABYTOKEN.sol#2898-2904)
        - BABYTOKEN.processDividendTracker(uint256) (contracts/BABYTOKEN.sol#2940-2954)
        - BABYTOKEN.swapAndSendDividends(uint256) (contracts/BABYTOKEN.sol#3140-3152)
        - BABYTOKEN.updateClaimWait(uint256) (contracts/BABYTOKEN.sol#2847-2849)
        - BABYTOKEN.updateDividendTracker(address) (contracts/BABYTOKEN.sol#2728-2751)
        - BABYTOKEN.updateMinimumTokenBalanceForDividends(uint256) (contracts/BABYTOKEN.sol#2855-2860)
        - BABYTOKEN.withdrawableDividendOf(address) (contracts/BABYTOKEN.sol#2878-2884)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities-1

BABYTOKEN._transfer(address,address,uint256).iterations (contracts/BABYTOKEN.sol#3038) is a local variable never initialized
BABYTOKEN._transfer(address,address,uint256).claims (contracts/BABYTOKEN.sol#3039) is a local variable never initialized
BABYTOKEN._transfer(address,address,uint256).lastProcessedIndex (contracts/BABYTOKEN.sol#3040) is a local variable never initialized
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#uninitialized-local-variables

BABYTOKEN.claim() (contracts/BABYTOKEN.sol#2956-2958) ignores return value by dividendTracker.processAccount(address(msg.sender),false) (contracts/BABYTOKEN.sol#2957)
BABYTOKEN._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#2968-3052) ignores return value by dividendTracker.process(gas) (contracts/BABYTOKEN.sol#3037-3050)
BABYTOKEN.addLiquidity(uint256,uint256) (contracts/BABYTOKEN.sol#3125-3138) ignores return value by uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#unused-return

DividendPayingToken.__DividendPayingToken_init(address,string,string)._name (contracts/BABYTOKEN.sol#2112) shadows:
        - ERC20Upgradeable._name (contracts/BABYTOKEN.sol#1378) (state variable)
DividendPayingToken.__DividendPayingToken_init(address,string,string)._symbol (contracts/BABYTOKEN.sol#2113) shadows:
        - ERC20Upgradeable._symbol (contracts/BABYTOKEN.sol#1379) (state variable)
DividendPayingToken.dividendOf(address)._owner (contracts/BABYTOKEN.sol#2172) shadows:
        - OwnableUpgradeable._owner (contracts/BABYTOKEN.sol#1722) (state variable)
DividendPayingToken.withdrawableDividendOf(address)._owner (contracts/BABYTOKEN.sol#2179) shadows:
        - OwnableUpgradeable._owner (contracts/BABYTOKEN.sol#1722) (state variable)
DividendPayingToken.withdrawnDividendOf(address)._owner (contracts/BABYTOKEN.sol#2191) shadows:
        - OwnableUpgradeable._owner (contracts/BABYTOKEN.sol#1722) (state variable)
DividendPayingToken.accumulativeDividendOf(address)._owner (contracts/BABYTOKEN.sol#2205) shadows:
        - OwnableUpgradeable._owner (contracts/BABYTOKEN.sol#1722) (state variable)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#local-variable-shadowing

BABYTOKEN.setSwapTokensAtAmount(uint256) (contracts/BABYTOKEN.sol#2724-2726) should emit an event for: 
        - swapTokensAtAmount = amount (contracts/BABYTOKEN.sol#2725) 
BABYTOKEN.setTokenRewardsFee(uint256) (contracts/BABYTOKEN.sol#2790-2794) should emit an event for: 
        - totalFees = tokenRewardsFee.add(liquidityFee).add(marketingFee) (contracts/BABYTOKEN.sol#2792) 
BABYTOKEN.setLiquiditFee(uint256) (contracts/BABYTOKEN.sol#2796-2800) should emit an event for: 
        - liquidityFee = value (contracts/BABYTOKEN.sol#2797) 
        - totalFees = tokenRewardsFee.add(liquidityFee).add(marketingFee) (contracts/BABYTOKEN.sol#2798) 
BABYTOKEN.setMarketingFee(uint256) (contracts/BABYTOKEN.sol#2802-2806) should emit an event for: 
        - marketingFee = value (contracts/BABYTOKEN.sol#2803) 
        - totalFees = tokenRewardsFee.add(liquidityFee).add(marketingFee) (contracts/BABYTOKEN.sol#2804) 
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#missing-events-arithmetic

BABYTOKEN.constructor(string,string,uint256,address[4],uint256[3],uint256,address,uint256)._uniswapV2Pair (contracts/BABYTOKEN.sol#2695-2696) lacks a zero-check on :
                - uniswapV2Pair = _uniswapV2Pair (contracts/BABYTOKEN.sol#2698)
BABYTOKEN.constructor(string,string,uint256,address[4],uint256[3],uint256,address,uint256).serviceFeeReceiver_ (contracts/BABYTOKEN.sol#2665) lacks a zero-check on :
                - address(serviceFeeReceiver_).transfer(serviceFee_) (contracts/BABYTOKEN.sol#2719)
BABYTOKEN.updateUniswapV2Router(address)._uniswapV2Pair (contracts/BABYTOKEN.sol#2760-2761) lacks a zero-check on :
                - uniswapV2Pair = _uniswapV2Pair (contracts/BABYTOKEN.sol#2762)
BABYTOKEN.setMarketingWallet(address).wallet (contracts/BABYTOKEN.sol#2786) lacks a zero-check on :
                - _marketingWalletAddress = wallet (contracts/BABYTOKEN.sol#2787)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#missing-zero-address-validation

DividendPayingToken._withdrawDividendOfUser(address) (contracts/BABYTOKEN.sol#2141-2167) has external calls inside a loop: success = IERC20(rewardToken).transfer(user,_withdrawableDividend) (contracts/BABYTOKEN.sol#2151-2154)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation/#calls-inside-a-loop

Variable 'BABYTOKEN._transfer(address,address,uint256).claims (contracts/BABYTOKEN.sol#3039)' in BABYTOKEN._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#2968-3052) potentially used before declaration: ProcessedDividendTracker(iterations,claims,lastProcessedIndex,true,gas,tx.origin) (contracts/BABYTOKEN.sol#3042-3049)
Variable 'BABYTOKEN._transfer(address,address,uint256).lastProcessedIndex (contracts/BABYTOKEN.sol#3040)' in BABYTOKEN._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#2968-3052) potentially used before declaration: ProcessedDividendTracker(iterations,claims,lastProcessedIndex,true,gas,tx.origin) (contracts/BABYTOKEN.sol#3042-3049)
Variable 'BABYTOKEN._transfer(address,address,uint256).iterations (contracts/BABYTOKEN.sol#3038)' in BABYTOKEN._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#2968-3052) potentially used before declaration: ProcessedDividendTracker(iterations,claims,lastProcessedIndex,true,gas,tx.origin) (contracts/BABYTOKEN.sol#3042-3049)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#pre-declaration-usage-of-local-variables

Reentrancy in BABYTOKEN._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#2968-3052):
        External calls:
        - swapAndSendToFee(marketingTokens) (contracts/BABYTOKEN.sol#2997)
                - IERC20(rewardToken).transfer(_marketingWalletAddress,newBalance) (contracts/BABYTOKEN.sol#3063)
                - uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3116-3122)
        - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
                - uniswapV2Router.swapExactTokensForETHSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3098-3104)
        External calls sending eth:
        - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
        State variables written after the call(s):
        - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
                - _allowances[owner][spender] = amount (contracts/BABYTOKEN.sol#458)
Reentrancy in BABYTOKEN._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#2968-3052):
        External calls:
        - swapAndSendToFee(marketingTokens) (contracts/BABYTOKEN.sol#2997)
                - IERC20(rewardToken).transfer(_marketingWalletAddress,newBalance) (contracts/BABYTOKEN.sol#3063)
                - uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3116-3122)
        - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
                - uniswapV2Router.swapExactTokensForETHSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3098-3104)
        - swapAndSendDividends(sellTokens) (contracts/BABYTOKEN.sol#3005)
                - success = IERC20(rewardToken).transfer(address(dividendTracker),dividends) (contracts/BABYTOKEN.sol#3143-3146)
                - dividendTracker.distributeCAKEDividends(dividends) (contracts/BABYTOKEN.sol#3149)
                - uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3116-3122)
        External calls sending eth:
        - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
        State variables written after the call(s):
        - swapAndSendDividends(sellTokens) (contracts/BABYTOKEN.sol#3005)
                - _allowances[owner][spender] = amount (contracts/BABYTOKEN.sol#458)
Reentrancy in BABYTOKENDividendTracker.processAccount(address,bool) (contracts/BABYTOKEN.sol#2527-2541):
        External calls:
        - amount = _withdrawDividendOfUser(account) (contracts/BABYTOKEN.sol#2532)
                - success = IERC20(rewardToken).transfer(user,_withdrawableDividend) (contracts/BABYTOKEN.sol#2151-2154)
        State variables written after the call(s):
        - lastClaimTimes[account] = block.timestamp (contracts/BABYTOKEN.sol#2535)
Reentrancy in BABYTOKEN.swapAndLiquify(uint256) (contracts/BABYTOKEN.sol#3066-3087):
        External calls:
        - swapTokensForEth(half) (contracts/BABYTOKEN.sol#3078)
                - uniswapV2Router.swapExactTokensForETHSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3098-3104)
        - addLiquidity(otherHalf,newBalance) (contracts/BABYTOKEN.sol#3084)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
        External calls sending eth:
        - addLiquidity(otherHalf,newBalance) (contracts/BABYTOKEN.sol#3084)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
        State variables written after the call(s):
        - addLiquidity(otherHalf,newBalance) (contracts/BABYTOKEN.sol#3084)
                - _allowances[owner][spender] = amount (contracts/BABYTOKEN.sol#458)
Reentrancy in BABYTOKEN.updateUniswapV2Router(address) (contracts/BABYTOKEN.sol#2753-2763):
        External calls:
        - _uniswapV2Pair = IUniswapV2Factory(uniswapV2Router.factory()).createPair(address(this),uniswapV2Router.WETH()) (contracts/BABYTOKEN.sol#2760-2761)
        State variables written after the call(s):
        - uniswapV2Pair = _uniswapV2Pair (contracts/BABYTOKEN.sol#2762)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities-2

Reentrancy in BABYTOKEN._setAutomatedMarketMakerPair(address,bool) (contracts/BABYTOKEN.sol#2820-2832):
        External calls:
        - dividendTracker.excludeFromDividends(pair) (contracts/BABYTOKEN.sol#2828)
        Event emitted after the call(s):
        - SetAutomatedMarketMakerPair(pair,value) (contracts/BABYTOKEN.sol#2831)
Reentrancy in BABYTOKEN._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#2968-3052):
        External calls:
        - swapAndSendToFee(marketingTokens) (contracts/BABYTOKEN.sol#2997)
                - IERC20(rewardToken).transfer(_marketingWalletAddress,newBalance) (contracts/BABYTOKEN.sol#3063)
                - uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3116-3122)
        - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
                - uniswapV2Router.swapExactTokensForETHSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3098-3104)
        External calls sending eth:
        - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
        Event emitted after the call(s):
        - Approval(owner,spender,amount) (contracts/BABYTOKEN.sol#459)
                - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
        - SwapAndLiquify(half,newBalance,otherHalf) (contracts/BABYTOKEN.sol#3086)
                - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
Reentrancy in BABYTOKEN._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#2968-3052):
        External calls:
        - swapAndSendToFee(marketingTokens) (contracts/BABYTOKEN.sol#2997)
                - IERC20(rewardToken).transfer(_marketingWalletAddress,newBalance) (contracts/BABYTOKEN.sol#3063)
                - uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3116-3122)
        - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
                - uniswapV2Router.swapExactTokensForETHSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3098-3104)
        - swapAndSendDividends(sellTokens) (contracts/BABYTOKEN.sol#3005)
                - success = IERC20(rewardToken).transfer(address(dividendTracker),dividends) (contracts/BABYTOKEN.sol#3143-3146)
                - dividendTracker.distributeCAKEDividends(dividends) (contracts/BABYTOKEN.sol#3149)
                - uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3116-3122)
        External calls sending eth:
        - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
        Event emitted after the call(s):
        - Approval(owner,spender,amount) (contracts/BABYTOKEN.sol#459)
                - swapAndSendDividends(sellTokens) (contracts/BABYTOKEN.sol#3005)
        - SendDividends(tokens,dividends) (contracts/BABYTOKEN.sol#3150)
                - swapAndSendDividends(sellTokens) (contracts/BABYTOKEN.sol#3005)
        - Transfer(sender,recipient,amount) (contracts/BABYTOKEN.sol#383)
                - super._transfer(from,address(this),fees) (contracts/BABYTOKEN.sol#3024)
        - Transfer(sender,recipient,amount) (contracts/BABYTOKEN.sol#383)
                - super._transfer(from,to,amount) (contracts/BABYTOKEN.sol#3027)
Reentrancy in BABYTOKEN._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#2968-3052):
        External calls:
        - swapAndSendToFee(marketingTokens) (contracts/BABYTOKEN.sol#2997)
                - IERC20(rewardToken).transfer(_marketingWalletAddress,newBalance) (contracts/BABYTOKEN.sol#3063)
                - uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3116-3122)
        - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
                - uniswapV2Router.swapExactTokensForETHSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3098-3104)
        - swapAndSendDividends(sellTokens) (contracts/BABYTOKEN.sol#3005)
                - success = IERC20(rewardToken).transfer(address(dividendTracker),dividends) (contracts/BABYTOKEN.sol#3143-3146)
                - dividendTracker.distributeCAKEDividends(dividends) (contracts/BABYTOKEN.sol#3149)
                - uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3116-3122)
        - dividendTracker.setBalance(address(from),balanceOf(from)) (contracts/BABYTOKEN.sol#3029-3031)
        - dividendTracker.setBalance(address(to),balanceOf(to)) (contracts/BABYTOKEN.sol#3032)
        - dividendTracker.process(gas) (contracts/BABYTOKEN.sol#3037-3050)
        External calls sending eth:
        - swapAndLiquify(swapTokens) (contracts/BABYTOKEN.sol#3002)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
        Event emitted after the call(s):
        - ProcessedDividendTracker(iterations,claims,lastProcessedIndex,true,gas,tx.origin) (contracts/BABYTOKEN.sol#3042-3049)
Reentrancy in BABYTOKENDividendTracker.processAccount(address,bool) (contracts/BABYTOKEN.sol#2527-2541):
        External calls:
        - amount = _withdrawDividendOfUser(account) (contracts/BABYTOKEN.sol#2532)
                - success = IERC20(rewardToken).transfer(user,_withdrawableDividend) (contracts/BABYTOKEN.sol#2151-2154)
        Event emitted after the call(s):
        - Claim(account,amount,automatic) (contracts/BABYTOKEN.sol#2536)
Reentrancy in BABYTOKEN.processDividendTracker(uint256) (contracts/BABYTOKEN.sol#2940-2954):
        External calls:
        - (iterations,claims,lastProcessedIndex) = dividendTracker.process(gas) (contracts/BABYTOKEN.sol#2941-2945)
        Event emitted after the call(s):
        - ProcessedDividendTracker(iterations,claims,lastProcessedIndex,false,gas,tx.origin) (contracts/BABYTOKEN.sol#2946-2953)
Reentrancy in BABYTOKEN.swapAndLiquify(uint256) (contracts/BABYTOKEN.sol#3066-3087):
        External calls:
        - swapTokensForEth(half) (contracts/BABYTOKEN.sol#3078)
                - uniswapV2Router.swapExactTokensForETHSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3098-3104)
        - addLiquidity(otherHalf,newBalance) (contracts/BABYTOKEN.sol#3084)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
        External calls sending eth:
        - addLiquidity(otherHalf,newBalance) (contracts/BABYTOKEN.sol#3084)
                - uniswapV2Router.addLiquidityETH{value: ethAmount}(address(this),tokenAmount,0,0,address(0),block.timestamp) (contracts/BABYTOKEN.sol#3130-3137)
        Event emitted after the call(s):
        - Approval(owner,spender,amount) (contracts/BABYTOKEN.sol#459)
                - addLiquidity(otherHalf,newBalance) (contracts/BABYTOKEN.sol#3084)
        - SwapAndLiquify(half,newBalance,otherHalf) (contracts/BABYTOKEN.sol#3086)
Reentrancy in BABYTOKEN.swapAndSendDividends(uint256) (contracts/BABYTOKEN.sol#3140-3152):
        External calls:
        - swapTokensForCake(tokens) (contracts/BABYTOKEN.sol#3141)
                - uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(tokenAmount,0,path,address(this),block.timestamp) (contracts/BABYTOKEN.sol#3116-3122)
        - success = IERC20(rewardToken).transfer(address(dividendTracker),dividends) (contracts/BABYTOKEN.sol#3143-3146)
        - dividendTracker.distributeCAKEDividends(dividends) (contracts/BABYTOKEN.sol#3149)
        Event emitted after the call(s):
        - SendDividends(tokens,dividends) (contracts/BABYTOKEN.sol#3150)
Reentrancy in BABYTOKEN.updateDividendTracker(address) (contracts/BABYTOKEN.sol#2728-2751):
        External calls:
        - newDividendTracker.excludeFromDividends(address(newDividendTracker)) (contracts/BABYTOKEN.sol#2743)
        - newDividendTracker.excludeFromDividends(address(this)) (contracts/BABYTOKEN.sol#2744)
        - newDividendTracker.excludeFromDividends(owner()) (contracts/BABYTOKEN.sol#2745)
        - newDividendTracker.excludeFromDividends(address(uniswapV2Router)) (contracts/BABYTOKEN.sol#2746)
        Event emitted after the call(s):
        - UpdateDividendTracker(newAddress,address(dividendTracker)) (contracts/BABYTOKEN.sol#2748)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities-3

BABYTOKENDividendTracker.getAccount(address) (contracts/BABYTOKEN.sol#2376-2423) uses timestamp for comparisons
        Dangerous comparisons:
        - nextClaimTime > block.timestamp (contracts/BABYTOKEN.sol#2420-2422)
BABYTOKENDividendTracker.canAutoClaim(uint256) (contracts/BABYTOKEN.sol#2448-2454) uses timestamp for comparisons
        Dangerous comparisons:
        - lastClaimTime > block.timestamp (contracts/BABYTOKEN.sol#2449)
        - block.timestamp.sub(lastClaimTime) >= claimWait (contracts/BABYTOKEN.sol#2453)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#block-timestamp

Clones.clone(address) (contracts/BABYTOKEN.sol#831-840) uses assembly
        - INLINE ASM (contracts/BABYTOKEN.sol#832-838)
Clones.cloneDeterministic(address,bytes32) (contracts/BABYTOKEN.sol#849-858) uses assembly
        - INLINE ASM (contracts/BABYTOKEN.sol#850-856)
Clones.predictDeterministicAddress(address,bytes32,address) (contracts/BABYTOKEN.sol#863-878) uses assembly
        - INLINE ASM (contracts/BABYTOKEN.sol#868-877)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#assembly-usage

Clones.cloneDeterministic(address,bytes32) (contracts/BABYTOKEN.sol#849-858) is never used and should be removed
Clones.predictDeterministicAddress(address,bytes32) (contracts/BABYTOKEN.sol#883-889) is never used and should be removed
Clones.predictDeterministicAddress(address,bytes32,address) (contracts/BABYTOKEN.sol#863-878) is never used and should be removed
Context._msgData() (contracts/BABYTOKEN.sol#140-142) is never used and should be removed
ContextUpgradeable.__Context_init() (contracts/BABYTOKEN.sol#1319-1321) is never used and should be removed
ContextUpgradeable._msgData() (contracts/BABYTOKEN.sol#1329-1331) is never used and should be removed
DividendPayingToken._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#2224-2239) is never used and should be removed
ERC20._burn(address,uint256) (contracts/BABYTOKEN.sol#420-435) is never used and should be removed
ERC20Upgradeable._transfer(address,address,uint256) (contracts/BABYTOKEN.sol#1562-1582) is never used and should be removed
SafeMath.div(uint256,uint256,string) (contracts/BABYTOKEN.sol#768-777) is never used and should be removed
SafeMath.mod(uint256,uint256) (contracts/BABYTOKEN.sol#728-730) is never used and should be removed
SafeMath.mod(uint256,uint256,string) (contracts/BABYTOKEN.sol#794-803) is never used and should be removed
SafeMath.sub(uint256,uint256,string) (contracts/BABYTOKEN.sol#745-754) is never used and should be removed
SafeMath.tryAdd(uint256,uint256) (contracts/BABYTOKEN.sol#599-605) is never used and should be removed
SafeMath.tryDiv(uint256,uint256) (contracts/BABYTOKEN.sol#641-646) is never used and should be removed
SafeMath.tryMod(uint256,uint256) (contracts/BABYTOKEN.sol#653-658) is never used and should be removed
SafeMath.tryMul(uint256,uint256) (contracts/BABYTOKEN.sol#624-634) is never used and should be removed
SafeMath.trySub(uint256,uint256) (contracts/BABYTOKEN.sol#612-617) is never used and should be removed
SafeMathInt.abs(int256) (contracts/BABYTOKEN.sol#1893-1896) is never used and should be removed
SafeMathInt.div(int256,int256) (contracts/BABYTOKEN.sol#1864-1870) is never used and should be removed
SafeMathInt.mul(int256,int256) (contracts/BABYTOKEN.sol#1852-1859) is never used and should be removed
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#dead-code

Pragma version=0.8.4 (contracts/BABYTOKEN.sol#2572) allows old versions
solc-0.8.4 is not recommended for deployment
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-versions-of-solidity

Function IUniswapV2Router01.WETH() (contracts/BABYTOKEN.sol#935) is not in mixedCase
Function ContextUpgradeable.__Context_init() (contracts/BABYTOKEN.sol#1319-1321) is not in mixedCase
Function ContextUpgradeable.__Context_init_unchained() (contracts/BABYTOKEN.sol#1323-1324) is not in mixedCase
Variable ContextUpgradeable.__gap (contracts/BABYTOKEN.sol#1332) is not in mixedCase
Function ERC20Upgradeable.__ERC20_init(string,string) (contracts/BABYTOKEN.sol#1390-1393) is not in mixedCase
Function ERC20Upgradeable.__ERC20_init_unchained(string,string) (contracts/BABYTOKEN.sol#1395-1398) is not in mixedCase
Variable ERC20Upgradeable.__gap (contracts/BABYTOKEN.sol#1697) is not in mixedCase
Function OwnableUpgradeable.__Ownable_init() (contracts/BABYTOKEN.sol#1729-1732) is not in mixedCase
Function OwnableUpgradeable.__Ownable_init_unchained() (contracts/BABYTOKEN.sol#1734-1736) is not in mixedCase
Variable OwnableUpgradeable.__gap (contracts/BABYTOKEN.sol#1778) is not in mixedCase
Function IUniswapV2Pair.DOMAIN_SEPARATOR() (contracts/BABYTOKEN.sol#1801) is not in mixedCase
Function IUniswapV2Pair.PERMIT_TYPEHASH() (contracts/BABYTOKEN.sol#1802) is not in mixedCase
Function IUniswapV2Pair.MINIMUM_LIQUIDITY() (contracts/BABYTOKEN.sol#1819) is not in mixedCase
Function DividendPayingToken.__DividendPayingToken_init(address,string,string) (contracts/BABYTOKEN.sol#2110-2118) is not in mixedCase
Parameter DividendPayingToken.__DividendPayingToken_init(address,string,string)._rewardToken (contracts/BABYTOKEN.sol#2111) is not in mixedCase
Parameter DividendPayingToken.__DividendPayingToken_init(address,string,string)._name (contracts/BABYTOKEN.sol#2112) is not in mixedCase
Parameter DividendPayingToken.__DividendPayingToken_init(address,string,string)._symbol (contracts/BABYTOKEN.sol#2113) is not in mixedCase
Parameter DividendPayingToken.dividendOf(address)._owner (contracts/BABYTOKEN.sol#2172) is not in mixedCase
Parameter DividendPayingToken.withdrawableDividendOf(address)._owner (contracts/BABYTOKEN.sol#2179) is not in mixedCase
Parameter DividendPayingToken.withdrawnDividendOf(address)._owner (contracts/BABYTOKEN.sol#2191) is not in mixedCase
Parameter DividendPayingToken.accumulativeDividendOf(address)._owner (contracts/BABYTOKEN.sol#2205) is not in mixedCase
Constant DividendPayingToken.magnitude (contracts/BABYTOKEN.sol#2090) is not in UPPER_CASE_WITH_UNDERSCORES
Parameter BABYTOKENDividendTracker.getAccount(address)._account (contracts/BABYTOKEN.sol#2376) is not in mixedCase
Variable BABYTOKEN._marketingWalletAddress (contracts/BABYTOKEN.sol#2605) is not in mixedCase
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#conformance-to-solidity-naming-conventions

Variable IUniswapV2Router01.addLiquidity(address,address,uint256,uint256,uint256,uint256,address,uint256).amountADesired (contracts/BABYTOKEN.sol#940) is too similar to IUniswapV2Router01.addLiquidity(address,address,uint256,uint256,uint256,uint256,address,uint256).amountBDesired (contracts/BABYTOKEN.sol#941)
Variable DividendPayingToken.__DividendPayingToken_init(address,string,string)._rewardToken (contracts/BABYTOKEN.sol#2111) is too similar to BABYTOKENDividendTracker.initialize(address,uint256).rewardToken_ (contracts/BABYTOKEN.sol#2303)
Variable DividendPayingToken._withdrawDividendOfUser(address)._withdrawableDividend (contracts/BABYTOKEN.sol#2145) is too similar to BABYTOKENDividendTracker.getAccount(address).withdrawableDividends (contracts/BABYTOKEN.sol#2383)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#variable-names-too-similar

Clones.clone(address) (contracts/BABYTOKEN.sol#831-840) uses literals with too many digits:
        - mstore(uint256,uint256)(ptr_clone_asm_0,0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000) (contracts/BABYTOKEN.sol#834)
Clones.clone(address) (contracts/BABYTOKEN.sol#831-840) uses literals with too many digits:
        - mstore(uint256,uint256)(ptr_clone_asm_0 + 0x28,0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000) (contracts/BABYTOKEN.sol#836)
Clones.cloneDeterministic(address,bytes32) (contracts/BABYTOKEN.sol#849-858) uses literals with too many digits:
        - mstore(uint256,uint256)(ptr_cloneDeterministic_asm_0,0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000) (contracts/BABYTOKEN.sol#852)
Clones.cloneDeterministic(address,bytes32) (contracts/BABYTOKEN.sol#849-858) uses literals with too many digits:
        - mstore(uint256,uint256)(ptr_cloneDeterministic_asm_0 + 0x28,0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000) (contracts/BABYTOKEN.sol#854)
Clones.predictDeterministicAddress(address,bytes32,address) (contracts/BABYTOKEN.sol#863-878) uses literals with too many digits:
        - mstore(uint256,uint256)(ptr_predictDeterministicAddress_asm_0,0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000) (contracts/BABYTOKEN.sol#870)
Clones.predictDeterministicAddress(address,bytes32,address) (contracts/BABYTOKEN.sol#863-878) uses literals with too many digits:
        - mstore(uint256,uint256)(ptr_predictDeterministicAddress_asm_0 + 0x28,0x5af43d82803e903d91602b57fd5bf3ff00000000000000000000000000000000) (contracts/BABYTOKEN.sol#872)
BABYTOKEN.constructor(string,string,uint256,address[4],uint256[3],uint256,address,uint256) (contracts/BABYTOKEN.sol#2658-2720) uses literals with too many digits:
        - gasForProcessing = 300000 (contracts/BABYTOKEN.sol#2683)
BABYTOKEN.updateGasForProcessing(uint256) (contracts/BABYTOKEN.sol#2834-2845) uses literals with too many digits:
        - require(bool,string)(newValue >= 200000 && newValue <= 500000,BABYTOKEN: gasForProcessing must be between 200,000 and 500,000) (contracts/BABYTOKEN.sol#2835-2838)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#too-many-digits

SafeMathInt.MAX_INT256 (contracts/BABYTOKEN.sol#1847) is never used in SafeMathInt (contracts/BABYTOKEN.sol#1845-1902)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#unused-state-variable

BABYTOKEN.rewardToken (contracts/BABYTOKEN.sol#2596) should be immutable 
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#state-variables-that-could-be-declared-immutable
./contracts analyzed (26 contracts with 84 detectors), 109 result(s) found