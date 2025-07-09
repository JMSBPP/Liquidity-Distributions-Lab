### Soldity Functions
```solidity
function query(
    int24 r(i,tickSpacing),
    int24 i(t),
    int24 i(P_{Y/X})
    bytes32 params
    bytes32 LDF{t-1}
    ) returns (
uint256  LDF(tickSpacing, r, [il,iu], params, LDF{t-1})
uint256 a_0(r, L, i(t), i(P_{Y/X}), params,LDF{t-1} )
uint256 a_1(r, L, i(t), i(P_{Y/X}), params,LDF{t-1} )
bytes32 LDF_{t}
bool shouldSurge
//NOTE: Whether the pool should surge. Usually corresponds to whether the LDF has shifted / changed shape.
)
```

## Math Functions

$$
\begin{align*}
    \begin{cases}
      r (i, \Delta_i) =   \\
      a_0 (r) =  \\
      a_1 (r) =\\
    \end{cases}
\end{align*}
$$

<!-- function liquidityDensityX96(
    int24 roundedTick, 
    int24 tickSpacing, 
    int24 tickLower, 
    int24 tickUpper
    )internal pure returns (uint256)
{
    if (roundedTick < tickLower || roundedTick >= tickUpper) {
        // roundedTick is outside of the distribution
        return 0;
    }
    uint256 length = uint24((tickUpper - tickLower) / tickSpacing);
    return Q96 / length;
} -->