# LDF
- $\mathcal{L}_i^O \bigg (\big [ i_l, i_u\big]\bigg)$: is restrictive
- $\mathcal{L}_i^O$ not differentiable on $P_{Y/X}$
- $\partial_{\Delta X} \, C^{\text{\texttt{Tx}}} > \, 0$: This is because for large swaps the swap has to go through different ticks where liqudiity levels might differ.
  

If one knows how liquidity demand $\mathcal{L}_{\Delta i}^D$ is distributed across all ticks $\big [ i_{\text{\texttt{min}}}, i_{\text{\texttt{max}}}  \big ], \, \mathcal{L}^D_{\Delta i} \bigg ( \big [ i_{\text{\texttt{min}}}, i_{\text{\texttt{max}}}  \big ] \bigg )$. Then defining **ricks** as:
$$
\begin{align*}
    r \big (i, \Delta i) = \big \lfloor \frac{i}{\Delta i} \big \rfloor \cdot \Delta i
\end{align*}
$$

We have that:

$$
\begin{align*}
\mathcal{L}^O \bigg ( r\bigg) :=\bigg ( \sum_{r=r_{\text{\texttt{min}}}}^{r_{\text{\texttt{max}}}} \, \mathcal{L}^O \big ( r\big) \bigg )\cdot \mathcal{L}^D_{\Delta r} \bigg ( r \bigg ) \, , \, r \in \mathbb{R}_{Y/X} \, \\
 \sum_{r} \, \mathcal{L}^D_{\Delta r} \bigg ( r \bigg ) =1
\end{align*}
$$
Here is the interface: 
```cpp
interface ILiquidityDensityFunction {
	+🔍query() -> L^O_t (M,r,r_t ,r (P_t); params, L^O_{t-1} )
	+🔍computeSwap()
	+🔍cumulativeAmount0()
	+🔍cumulativeAmount1()
	+🔍isValidParams()
}
```

## BunniHub ==> Liquidity Actions Flow
- The main contract LPs interact with. 

- Each BunniKey corresponds to a BunniToken,
which is the ERC20 LP token (`IBunniToken`) for the Uniswap V3 position specified by the BunniKey.

```cpp
class BunniHub{
    #[[WETH]] weth -> Quote token
	#[[IPermit2]] permit2 -> For transfers
	#[[IPoolManager]] poolManager -> UV4 PoolManager
	#[[IBunniToken]] bunniTokenImplementation -> Represents an LP Position 
    interface IBunniToken{
            //==QUERIES===
            +🔍hub()
	        +🔍token0()
	        +🔍token1()
	        +🔍poolManager()
	        +🔍poolKey()
	        +🔍hooklet()
	        +🔍metadataURI()
            // ==COMMANDS==
            +mint()
	        +burn()
	        +initialize()
	        +setMetadataURI()
	        +incrementNonce()
    } 
	#[[HubStorage]] s
    struct HubStorage {
        mapping(PoolId poolId => RawPoolState) poolState;
                        struct RawPoolState {
                            address immutableParamsPointer;
                            uint256 rawBalance0;
                            uint256 rawBalance1;
                        }
        mapping(PoolId poolId => uint256) reserve0;
        mapping(PoolId poolId => uint256) reserve1;
        mapping(PoolId poolId => IdleBalance) idleBalance;
        mapping(bytes32 bunniSubspace => uint24) nonce;
        mapping(IBunniToken bunniToken => PoolId) poolIdOfBunniToken;
        mapping(PoolId poolId => mapping(address => QueuedWithdrawal)) queuedWithdrawals;
        mapping(address guy => bool) isPauser;
        mapping(IBunniHook hook => bool) hookWhitelist;
        uint8 pauseFlags;
        bool unpauseFuse;
    }
}

interface IBunniHub {
    //====QUERIES=====
    +🔍poolState()
	+🔍poolParams()
	+🔍bunniTokenOfPool()
	+🔍hookletOfPool()
	+🔍hookParams()
	+🔍nonce()
	+🔍poolIdOfBunniToken()
	+🔍poolBalances()
	+🔍idleBalance()
	+🔍isPauser()
	+🔍getPauseStatus()
	+🔍poolInitData()
	+🔍hookIsWhitelisted()
    //====COMMANDS====
    +💰deposit()
	+queueWithdraw()
	+withdraw()
	+deployBunniToken()
	+hookHandleSwap()
	+hookSetIdleBalance()
	+hookGive()
	+setPauser()
	+setPauseFlags()
	+burnPauseFuse()
	+setHookWhitelist()
	+lockForRebalance()

}
```
- Now This are the services and clients of the liqudiity workflow (BunniHub). What is the logic?
```cpp
abstract BunniHubLogic{
	//====QUERIES======
	#🔍_validateVault()
	//====ADDING LIQUIDITY====
	+deposit()
	-_depositLogic()
	#_depositVaultReserve()
	#_mintShares()
	//====REMOVING LIQUIDITY====
	+withdraw()
	#_withdrawVaultReserve()
	//====INITIATE LIQUIDITY====
	+deployBunniToken()

}

```

## BunniHook ==> Swaps Action Flow
```cpp
class BunniHook {
    #[[WETH]] weth --> Base Token
	#[[IBunniHub]] hub --> Liquidity Management reference
	#[[address]] permit2 --> For transfers
	#[[IFloodPlain]] floodPlain --> ??
	#{static}[[uint256]] REBALANCE_OUTPUT_BALANCE_SLOT
	#{static}[[uint256]] REBALANCE_ACTIVE_SLOT
	#[[HookStorage]] s 
            struct HookStorage {
                mapping(PoolId => Oracle.Observation[MAX_CARDINALITY]) observations;
                mapping(PoolId => IBunniHook.ObservationState) states;
                mapping(PoolId id => bytes32) rebalanceOrderHash;
                mapping(PoolId id => bytes32) rebalanceOrderPermit2Hash;
                mapping(PoolId id => uint256) rebalanceOrderDeadline;
                mapping(PoolId => VaultSharePrices) vaultSharePricesAtLastSwap;
                mapping(PoolId => bytes32) ldfStates;
                mapping(PoolId => Slot0) slot0s;
                mapping(Currency => uint256) totalCuratorFees;
                mapping(PoolId => CuratorFees) curatorFees;
            }  
	#[[address]] hookFeeRecipient
	#[[uint32]] hookFeeModifier
	#[[IZone]] floodZone
	#[[mapping PoolId=>bool ]] withdrawalUnblocked
	#[[uint48]] _K
	#[[uint48]] _pendingK
	#[[uint160]] _pendingKActiveBlock
	#[[address]] hookFeeRecipientController
	#[[uint256]] deployTimestamp
}
interface IBunniHook {
	//====QUERIES======
	+🔍observe()
	+🔍isValidParams()
	+🔍ldfStates()
	+🔍slot0s()
	+🔍getAmAmmEnabled()
	+🔍canWithdraw()
	//====COMMANDS======
	+increaseCardinalityNext()
	+claimProtocolFees()
	+updateLdfState()
	+setZone()
	+setHookFeeRecipient()
	+setHookFeeModifier()
	+setWithdrawalUnblocked()
	+scheduleKChange()
	+curatorSetFeeRate()
	+curatorClaimFees()
	+rebalanceOrderHook()
}

```

 