# PoolRouter Contract

A Uniswap V3 pool router implementation that provides direct pool swapping functionality with configurable fee structure. This contract bypasses the standard Uniswap V3 Router and interacts directly with liquidity pools for gas optimization.

## Overview

The PoolRouter contract enables direct token swapping through Uniswap V3 pools while implementing a fee mechanism. It uses the Uniswap V3 swap callback pattern and integrates with the QuoterV2 for price quotes. The contract computes pool addresses deterministically and executes swaps with customizable fee collection.

## Contract Details

- **License**: MIT
- **Solidity Version**: ^0.8.0
- **Architecture**: Implements `IUniswapV3SwapCallback` interface
- **Pool Interaction**: Direct pool swaps without router overhead

## Constructor Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `_factory` | `address` | Address of the Uniswap V3 Factory contract |
| `_quoter` | `address` | Address of the Uniswap V3 QuoterV2 contract |
| `_feeAccount` | `address` | Address that will receive the collected fees |
| `_feeBasicPoints` | `uint256` | Fee percentage in basic points (< 10000) |

## State Variables

| Variable | Type | Visibility | Description |
|----------|------|------------|-------------|
| `factory` | `address` | `public immutable` | Uniswap V3 Factory contract address |
| `quoter` | `IQuoterV2` | `public immutable` | QuoterV2 contract interface |
| `feeAccount` | `address` | `public immutable` | Fee recipient address |
| `feeBasicPoints` | `uint256` | `public immutable` | Fee rate in basic points |
| `BASIS_POINTS_DENOMINATOR` | `uint256` | `public constant` | Denominator for fee calculations (10000) |
| `MIN_SQRT_RATIO` | `uint160` | `private constant` | Minimum sqrt price ratio (4295128739) |
| `MAX_SQRT_RATIO` | `uint160` | `private constant` | Maximum sqrt price ratio |

## Functions

### View Functions

#### `BASIS_POINTS_DENOMINATOR()`
```solidity
function BASIS_POINTS_DENOMINATOR() external view returns (uint256)
```
Returns the basis points denominator (10000).

#### `factory()`
```solidity
function factory() external view returns (address)
```
Returns the Uniswap V3 Factory contract address.

#### `feeAccount()`
```solidity
function feeAccount() external view returns (address)
```
Returns the fee recipient address.

#### `feeBasicPoints()`
```solidity
function feeBasicPoints() external view returns (uint256)
```
Returns the fee percentage in basic points.

#### `quoter()`
```solidity
function quoter() external view returns (address)
```
Returns the QuoterV2 contract address.

#### `getPoolAddress(address tokenA, address tokenB, uint24 fee)`
```solidity
function getPoolAddress(
    address tokenA,
    address tokenB,
    uint24 fee
) external view returns (address)
```
Computes the deterministic pool address for given token pair and fee tier.

| Parameter | Type | Description |
|-----------|------|-------------|
| `tokenA` | `address` | Address of first token |
| `tokenB` | `address` | Address of second token |
| `fee` | `uint24` | Pool fee tier (500, 3000, 10000) |

**Returns:**
- `address`: Computed pool address

### State-Changing Functions

#### `poolSwapWithPoolKey(address tokenIn, address tokenOut, uint24 fee, uint256 amount)`
```solidity
function poolSwapWithPoolKey(
    address tokenIn,
    address tokenOut,
    uint24 fee,
    uint256 amount
) external
```
Executes a token swap through the specified Uniswap V3 pool with fee deduction.

| Parameter | Type | Description |
|-----------|------|-------------|
| `tokenIn` | `address` | Address of input token |
| `tokenOut` | `address` | Address of output token |
| `fee` | `uint24` | Pool fee tier (500, 3000, 10000) |
| `amount` | `uint256` | Amount of input tokens |

**Requirements:**
- `amount > 0`
- User must approve this contract to spend `amount` of `tokenIn`
- Pool must exist for the given token pair and fee tier

#### `getQuote(address tokenIn, address tokenOut, uint24 poolFee, uint256 amountIn)`
```solidity
function getQuote(
    address tokenIn,
    address tokenOut,
    uint24 poolFee,
    uint256 amountIn
) external returns (uint256 amountOut, uint256 feeAmount)
```
Gets a quote for token swap amount after fee deduction.

| Parameter | Type | Description |
|-----------|------|-------------|
| `tokenIn` | `address` | Address of input token |
| `tokenOut` | `address` | Address of output token |
| `poolFee` | `uint24` | Pool fee tier |
| `amountIn` | `uint256` | Amount of input tokens |

**Returns:**
- `amountOut`: Expected output amount after swap
- `feeAmount`: Fee amount that will be charged

**Requirements:**
- `amountIn > 0`
- `tokenIn != tokenOut`

#### `uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata data)`
```solidity
function uniswapV3SwapCallback(
    int256 amount0Delta,
    int256 amount1Delta,
    bytes calldata data
) external override
```
Callback function called by Uniswap V3 pool during swap execution.

| Parameter | Type | Description |
|-----------|------|-------------|
| `amount0Delta` | `int256` | Change in token0 amount |
| `amount1Delta` | `int256` | Change in token1 amount |
| `data` | `bytes` | Encoded callback data |

**Requirements:**
- `amount0Delta > 0 || amount1Delta > 0`
- `msg.sender` must be the pool address
- Called only during swap execution

## Fee Structure

The contract implements a fee mechanism using basic points:
- **Fee Rate**: Configurable up to 99.99% (< 10000 basic points)
- **Calculation**: `feeAmount = (amount * feeBasicPoints) / 10000`
- **Application**: Fees are deducted from input amounts before swap execution
- **Distribution**: All fees are sent to the designated `feeAccount`

### Fee Examples

| Fee Basic Points | Percentage | Example: 1000 tokens input | Fee Amount | Amount After Fee |
|------------------|------------|----------------------------|------------|------------------|
| 10 | 0.1% | 1000 | 1 | 999 |
| 50 | 0.5% | 1000 | 5 | 995 |
| 100 | 1.0% | 1000 | 10 | 990 |
| 250 | 2.5% | 1000 | 25 | 975 |
| 500 | 5.0% | 1000 | 50 | 950 |
| 1000 | 10.0% | 1000 | 100 | 900 |

## Uniswap V3 Pool Fee Tiers

| Fee Tier | Basis Points | Percentage | Typical Use Case |
|----------|--------------|------------|------------------|
| 500 | 0.05% | 0.05% | Stablecoin pairs |
| 3000 | 0.3% | 0.30% | Standard pairs |
| 10000 | 1.0% | 1.00% | Exotic pairs |

## Pool Address Computation

The contract uses deterministic pool address computation:

```solidity
PoolAddress.PoolKey memory poolKey = PoolAddress.getPoolKey(tokenA, tokenB, fee);
address pool = PoolAddress.computeAddress(factory, poolKey);
```

**Pool Key Components:**
- `token0`: Lower address between tokenA and tokenB
- `token1`: Higher address between tokenA and tokenB  
- `fee`: Pool fee tier

## Swap Execution Flow

1. **Fee Calculation**: Deduct fee from input amount
2. **Pool Address**: Compute pool address from token pair and fee tier
3. **Token Transfer**: Transfer input tokens from user to contract
4. **Fee Collection**: Transfer fee amount to fee account
5. **Pool Swap**: Execute swap via `IUniswapV3Pool.swap()`
6. **Callback**: Handle `uniswapV3SwapCallback` for token payment
7. **Output Transfer**: Transfer received tokens to user

## Price Limits

The contract uses conservative price limits for swaps:
- **zeroForOne = true**: `sqrtPriceLimitX96 = MIN_SQRT_RATIO + 1`
- **zeroForOne = false**: `sqrtPriceLimitX96 = MAX_SQRT_RATIO - 1`

These limits prevent extreme price movements during swap execution.

## Security Considerations

1. **Callback Validation**: Ensures callback is called only by authorized pools
2. **Input Validation**: Validates all input parameters and amounts
3. **Transfer Checks**: Verifies all token transfers succeed
4. **Fee Limits**: Constructor prevents fees >= 100% (10000 basis points)
5. **Address Validation**: Validates all constructor addresses are non-zero
6. **Reentrancy Protection**: Uses callback pattern safely

## Interface Dependencies

### `IUniswapV3Pool`
Required methods:
- `swap(address, bool, int256, uint160, bytes)`

### `IUniswapV3SwapCallback`
Required implementation:
- `uniswapV3SwapCallback(int256, int256, bytes)`

### `IQuoterV2`
Required methods:
- `quoteExactInputSingle(QuoteExactInputSingleParams)`

### `IERC20`
Required methods:
- `transferFrom(address, address, uint256)`
- `transfer(address, uint256)`
- `balanceOf(address)`

## Library Dependencies

### `PoolAddress`
Required functions:
- `getPoolKey(address, address, uint24)`
- `computeAddress(address, PoolKey)`

## Deployment Requirements

1. Deploy or reference Uniswap V3 Factory
2. Deploy or reference Uniswap V3 QuoterV2
3. Prepare fee account address
4. Choose fee percentage (in basic points, < 10000)
5. Deploy PoolRouter with constructor parameters

## Gas Considerations

- **Direct Pool Swaps**: ~90,000-120,000 gas (vs ~150,000+ for router)
- **Quote Calls**: ~50,000-80,000 gas
- **Pool Address Computation**: ~2,000-5,000 gas

*Gas estimates may vary based on network conditions, token implementations, and pool states.*

## Error Handling

The contract includes comprehensive error handling:

| Error Message | Trigger Condition |
|---------------|-------------------|
| `"Invalid fee account address"` | Fee account is zero address |
| `"Invalid fee"` | Fee >= 10000 basis points |
| `"Invalid factory address"` | Factory is zero address |
| `"Invalid quoter address"` | Quoter is zero address |
| `"amount can't be zero"` | Swap amount is zero |
| `"tokenIn transfer failed"` | Input token transfer fails |
| `"Fee transfer failed"` | Fee transfer fails |
| `"Amount transfer to caller failed"` | Output token transfer fails |
| `"Amount must be greater than 0"` | Quote amount is zero |
| `"Tokens must be different"` | Quote tokens are identical |
| `"Invalid amounts"` | Callback amounts are invalid |
| `"Unauthorized caller"` | Callback from unauthorized address |

## Integration Notes

1. **Pool Existence**: Ensure pools exist before attempting swaps
2. **Liquidity Check**: Verify sufficient pool liquidity for desired swap amounts
3. **Slippage Protection**: Implement slippage checks in calling contracts
4. **Token Approvals**: Users must approve contract for token spending
5. **Fee Calculation**: Account for both protocol fees and contract fees in UI

## Advantages over Standard Router

1. **Gas Efficiency**: Direct pool interaction reduces gas costs
2. **Customizable Fees**: Implement custom fee structures
3. **Simplified Logic**: Fewer intermediate steps and validations
4. **Pool Control**: Direct control over pool selection and parameters