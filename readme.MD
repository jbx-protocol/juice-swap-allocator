# juice-swap-allocator

## Summary:
Allocator swapping the token received for another token

## Design
### Flow
For the user, add/remove pools and add this allocator as the split's beneficiary

The logic is, when fund are allocated
- Get a the best quote of x tokenIn in different pools, to a defined token out (immutable)
- Swap with the best amount out
- Send the swapped token to the split beneficiary

### Contracts/interface
- Main contract: Swap-Allocator: called by the terminal during distribution. Handles the pools registered.
When allocate() is called: get a quote, pull the tokenIn (if necessary), pick the best quote, swap and
send the token to the beneficiary
- Peripherals: differents AMM are wrapped into the same interface, used by the SwapAllocator. This interface
exposes 2 function: getQuote(..) and swap(..)

## Rationale
As there is no metadata passed to the allocator, the frontend cannot pass a quote later
used as a slippage/sandwich bot protection. This allocator circumvene this risk by getting
a quote in multiple pools and the swapping in the most interesting one (ie earning the more
tokenOut for the beneficiary).


## Risks/Tradeoff
- This allocator swaps "blindly" and shouldn't be used with fee-on-transfer token (in theory,
this shouldn't be an issue as the quote *should* not take the fee into account, but it is not
granted and some unpredictable behaviour might arises)
- The philosophy is to be as non-blocking as possible (the allocator is triggered during the
distribution of a project, reverting would impeed it, without any choice than having a new
split reconfigured, with the ballot etc). If the swap fails, the tokenIn should be sent to
the beneficiary (who can swap them later on, if needed).
- One main reason of potential swap failure is low liquidity in the pool. On top of reverting,
a low liquidity would create an additional slippage which might or might not be taken into account
while getting the quotes (depending on the AMM) - it is the allocator owner sole resopnsability
to manage the pools used accordingly (in doubt or if only a single pool is available, do not
use this allocator).