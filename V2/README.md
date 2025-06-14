# UniswapRouter V2 

A comprehensive smart contract wrapper for Uniswap V2 Router that provides token swapping, ETH wrapping/unwrapping functionality with configurable fee structure.

## Overview

The UniswapRouter contract acts as an intermediary for Uniswap V2 operations, implementing a fee mechanism on all transactions. It supports direct token-to-token swaps, ETH-to-token swaps, token-to-ETH swaps, and WETH wrapping/unwrapping operations.

## Contract Details

- **License**: MIT
- **Solidity Version**: ^0.8.0
- **Dependencies**: OpenZeppelin ERC20 interface

## Constructor Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `_router` | `address` | Address of the Uniswap V2 Router contract |
| `_feeAccount` | `address` | Address that will receive the collected fees |
| `_weth` | `address` | Address of the WETH (Wrapped Ether) contract |
| `_feeBasicPoints` | `uint256` | Fee percentage in basic points (max 1000 = 10%) |

## State Variables

| Variable | Type | Visibility | Description |
|----------|------|------------|-------------|
| `router` | `IUniswapV2Router` | `public immutable` | Uniswap V2 Router interface |
| `WETH` | `IWETH` | `public immutable` | WETH token interface |
| `feeAccount` | `address` | `public immutable` | Fee recipient address |
| `feeBasicPoints` | `uint256` | `public immutable` | Fee rate in basic points |
| `BASIS_POINTS_DENOMINATOR` | `uint256` | `public constant` | Denominator for fee calculations (10000) |

## Functions

### View Functions

#### `BASIS_POINTS_DENOMINATOR()`
```solidity
function BASIS_POINTS_DENOMINATOR() external view returns (uint256)
```
Returns the basis points denominator (10000).

#### `WETH()`
```solidity
function WETH() external view returns (address)
```
Returns the WETH contract address.

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

#### `router()`
```solidity
function router() external view returns (address)
```
Returns the Uniswap V2 Router contract address.

#### `getQuote(address tokenIn, address tokenOut, uint256 amountIn)`
```solidity
function getQuote(
    address tokenIn,
    address tokenOut,
    uint256 amountIn
) external view returns (uint256 amountOut, uint256 feeAmount)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `tokenIn` | `address` | Address of input token |
| `tokenOut` | `address` | Address of output token |
| `amountIn` | `uint256` | Amount of input tokens |

**Returns:**
- `amountOut`: Expected output amount after swap
- `feeAmount`: Fee amount that will be charged

#### `getWrapQuote(uint256 ethAmount, bool chargeFee)`
```solidity
function getWrapQuote(
    uint256 ethAmount,
    bool chargeFee
) external view returns (uint256 wethAmount, uint256 feeAmount)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `ethAmount` | `uint256` | Amount of ETH to wrap |
| `chargeFee` | `bool` | Whether to charge fee on this operation |

**Returns:**
- `wethAmount`: WETH amount user will receive
- `feeAmount`: Fee amount (0 if chargeFee is false)

#### `getUnwrapQuote(uint256 wethAmount, bool chargeFee)`
```solidity
function getUnwrapQuote(
    uint256 wethAmount,
    bool chargeFee
) external view returns (uint256 ethAmount, uint256 feeAmount)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `wethAmount` | `uint256` | Amount of WETH to unwrap |
| `chargeFee` | `bool` | Whether to charge fee on this operation |

**Returns:**
- `ethAmount`: ETH amount user will receive
- `feeAmount`: Fee amount (0 if chargeFee is false)

### State-Changing Functions

#### `swap(address tokenIn, address tokenOut, uint256 amountIn, uint256 minAmountOut)`
```solidity
function swap(
    address tokenIn,
    address tokenOut,
    uint256 amountIn,
    uint256 minAmountOut
) external
```
Performs a token-to-token swap with fee deduction.

| Parameter | Type | Description |
|-----------|------|-------------|
| `tokenIn` | `address` | Address of input token |
| `tokenOut` | `address` | Address of output token |
| `amountIn` | `uint256` | Amount of input tokens |
| `minAmountOut` | `uint256` | Minimum acceptable output amount |

**Requirements:**
- User must approve this contract to spend `amountIn` of `tokenIn`
- Swap must result in at least `minAmountOut` tokens

#### `swapETHForTokens(address tokenOut, uint256 minAmountOut)`
```solidity
function swapETHForTokens(
    address tokenOut,
    uint256 minAmountOut
) external payable
```
Swaps ETH for tokens with fee deduction.

| Parameter | Type | Description |
|-----------|------|-------------|
| `tokenOut` | `address` | Address of output token |
| `minAmountOut` | `uint256` | Minimum acceptable output amount |

**Requirements:**
- Must send ETH with transaction (`msg.value > 0`)
- Swap must result in at least `minAmountOut` tokens

#### `swapTokensForETH(address tokenIn, uint256 amountIn, uint256 minAmountOut)`
```solidity
function swapTokensForETH(
    address tokenIn,
    uint256 amountIn,
    uint256 minAmountOut
) external
```
Swaps tokens for ETH with fee deduction.

| Parameter | Type | Description |
|-----------|------|-------------|
| `tokenIn` | `address` | Address of input token |
| `amountIn` | `uint256` | Amount of input tokens |
| `minAmountOut` | `uint256` | Minimum acceptable ETH output |

**Requirements:**
- User must approve this contract to spend `amountIn` of `tokenIn`
- Swap must result in at least `minAmountOut` ETH

#### `wrapETH(bool chargeFee)`
```solidity
function wrapETH(bool chargeFee) external payable
```
Wraps ETH into WETH with optional fee.

| Parameter | Type | Description |
|-----------|------|-------------|
| `chargeFee` | `bool` | Whether to charge fee on this operation |

**Requirements:**
- Must send ETH with transaction (`msg.value > 0`)

#### `unwrapWETH(uint256 amount, bool chargeFee)`
```solidity
function unwrapWETH(uint256 amount, bool chargeFee) external
```
Unwraps WETH into ETH with optional fee.

| Parameter | Type | Description |
|-----------|------|-------------|
| `amount` | `uint256` | Amount of WETH to unwrap |
| `chargeFee` | `bool` | Whether to charge fee on this operation |

**Requirements:**
- `amount > 0`
- User must approve this contract to spend `amount` of WETH

## Events

### `Swap`
```solidity
event Swap(
    address indexed user,
    address indexed tokenIn,
    address indexed tokenOut,
    uint256 amountIn,
    uint256 amountOut,
    uint256 feeAmount
)
```
Emitted when a swap operation is completed.

| Parameter | Indexed | Description |
|-----------|---------|-------------|
| `user` | ✓ | Address of the user performing the swap |
| `tokenIn` | ✓ | Address of input token (address(0) for ETH) |
| `tokenOut` | ✓ | Address of output token (address(0) for ETH) |
| `amountIn` | ✗ | Amount of input tokens |
| `amountOut` | ✗ | Amount of output tokens received |
| `feeAmount` | ✗ | Fee amount charged |

### `FeeTransferred`
```solidity
event FeeTransferred(
    address indexed token,
    uint256 amount,
    address indexed feeAccount
)
```
Emitted when fees are transferred to the fee account.

| Parameter | Indexed | Description |
|-----------|---------|-------------|
| `token` | ✓ | Address of the token (address(0) for ETH) |
| `amount` | ✗ | Amount of fees transferred |
| `feeAccount` | ✓ | Address receiving the fees |

### `Wrapped`
```solidity
event Wrapped(
    address indexed user,
    uint256 ethAmount,
    uint256 wethReceived,
    uint256 feeAmount,
    bool feeCharged
)
```
Emitted when ETH is wrapped to WETH.

| Parameter | Indexed | Description |
|-----------|---------|-------------|
| `user` | ✓ | Address of the user |
| `ethAmount` | ✗ | Original ETH amount sent |
| `wethReceived` | ✗ | WETH amount received by user |
| `feeAmount` | ✗ | Fee amount charged |
| `feeCharged` | ✗ | Whether fee was charged |

### `Unwrapped`
```solidity
event Unwrapped(
    address indexed user,
    uint256 wethAmount,
    uint256 ethReceived,
    uint256 feeAmount,
    bool feeCharged
)
```
Emitted when WETH is unwrapped to ETH.

| Parameter | Indexed | Description |
|-----------|---------|-------------|
| `user` | ✓ | Address of the user |
| `wethAmount` | ✗ | Original WETH amount |
| `ethReceived` | ✗ | ETH amount received by user |
| `feeAmount` | ✗ | Fee amount charged |
| `feeCharged` | ✗ | Whether fee was charged |

## Fee Structure

The contract implements a fee mechanism using basic points:
- **Fee Rate**: Configurable up to 10% (1000 basic points)
- **Calculation**: `feeAmount = (amount * feeBasicPoints) / 10000`
- **Application**: Fees are deducted from input amounts before processing
- **Distribution**: All fees are sent to the designated `feeAccount`

### Fee Examples

| Fee Basic Points | Percentage | Example: 1000 tokens input | Fee Amount | Amount After Fee |
|------------------|------------|----------------------------|------------|------------------|
| 10 | 0.1% | 1000 | 1 | 999 |
| 50 | 0.5% | 1000 | 5 | 995 |
| 100 | 1.0% | 1000 | 10 | 990 |
| 250 | 2.5% | 1000 | 25 | 975 |
| 500 | 5.0% | 1000 | 50 | 950 |

## Security Considerations

1. **Approval Requirements**: Users must approve the contract to spend their tokens before calling swap functions
2. **Slippage Protection**: All swap functions include `minAmountOut` parameters for slippage protection
3. **Fee Validation**: Constructor validates that fees cannot exceed 10%
4. **Address Validation**: Constructor validates that critical addresses are not zero
5. **Deadline Protection**: All Uniswap interactions use a 5-minute deadline (`block.timestamp + 300`)

## Interface Dependencies

### `IUniswapV2Router`
Required methods:
- `swapExactTokensForTokens()`
- `getAmountsOut()`

### `IWETH`
Required methods:
- `deposit()` (payable)
- `withdraw(uint256)`
- Standard ERC20 methods (`transfer`, `transferFrom`, `approve`)

## Deployment Requirements

1. Deploy Uniswap V2 Router (or use existing)
2. Deploy WETH contract (or use existing)
3. Prepare fee account address
4. Choose fee percentage (in basic points, max 1000)
5. Deploy UniswapRouter with constructor parameters

## Gas Considerations

- Token-to-token swaps: ~150,000-200,000 gas
- ETH-to-token swaps: ~120,000-150,000 gas
- Token-to-ETH swaps: ~150,000-180,000 gas
- WETH wrapping: ~50,000-70,000 gas
- WETH unwrapping: ~50,000-70,000 gas

*Gas estimates may vary based on network conditions and token implementations.*
