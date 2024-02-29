# State changes:

- Updates `pool`[Pool]:
	- `pool.spot_price`[u64]
	- `pool.curve_type`[u8]
	- `pool.curve_delta`[u64]
	- `pool.reinvest_fulfill_buy`[bool]
	- `pool.reinvest_fulfill_sell`[bool]
	- `pool.expiry`[i64]
	- `pool.lp_fee_bp`[u16]
	- `pool.referral`[Pubkey]
	- `pool.referral_bp`[u16, // deprecated]
	- `pool.buyside_creator_royalty_bp`[u16]
	- `pool.sellside_asset_amount`[u64]
	- `pool.lp_fee_earned`[u64]
	- `pool.owner`[Pubkey]
	- `pool.cosigner`[Pubkey]
	- `pool.uuid`[Pubkey, // randomly generated keypair]
	- `pool.payment_mint`[Pubkey]
	- `pool.buyside_payment_amount`[u64]
	- `pool.shared_escrow_account`[Pubkey, // this points to the shared escrow account PDA (usually M2)]
	- `pool.shared_escrow_count`[u64, // this means that how many times (count) the shared escrow account can be fulfilled, and it can be mutable]
- Updates `owner`[UncheckedAccount]
- Updates `buyside_sol_escrow_account`[UncheckedAccount]
- `COMPLETE_WITH_THE_REST_OF_STATE_CHANGES`

# Notes:

- [ ] check validations:
  - [ ] [address = CANCEL_AUTHORITY](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/close_if_balance_invalid.rs#L12)
  - [ ] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/close_if_balance_invalid.rs#L20)
  - [ ] [constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/close_if_balance_invalid.rs#L21)
- COMPLETE_WITH_NOTES

# Signers:

- authority: COMPLETE_WITH_SIGNER_DESCRIPTION

# Handler function parameters:

NO_HANDLER_FUNCTION_PARAMETERS_DETECTED

# Context accounts:

```rust
  pub struct CloseIfBalanceInvalid<'info> {
      #[account(address = CANCEL_AUTHORITY)]
      pub authority: Signer<'info>,
  
      #[account(mut)]
      pub owner: UncheckedAccount<'info>,
  
      #[account(
          mut,
          seeds = [POOL_PREFIX.as_bytes(), owner.key().as_ref(), pool.uuid.as_ref(,
          bump
      )]
      pub pool: Box<Account<'info, Pool>>,
  
      #[account(
          mut,
          seeds = [BUYSIDE_SOL_ESCROW_ACCOUNT_PREFIX.as_bytes(), pool.key().as_ref(,
          bump)]
      pub buyside_sol_escrow_account: UncheckedAccount<'info>,
  
      pub system_program: Program<'info, System>,
  }
```

# Validations:

```rust
    #[account(
    	address = CANCEL_AUTHORITY,
    )]
    pub authority: Signer<'info>,
```
```rust
    #[account(
    	has_one = owner @ MMMErrorCode::InvalidOwner,
    	constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint,
    )]
    pub pool: Box<Account<'info, Pool>>,
```

# Miro frame url:

COMPLETE_WITH_MIRO_FRAME_URL
            