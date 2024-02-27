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
- Updates `sell_state`[SellState]:
	- `sell_state.pool`[Pubkey]
	- `sell_state.pool_owner`[Pubkey]
	- `sell_state.asset_mint`[Pubkey]
	- `sell_state.asset_amount`[u64]
- Updates `buyside_sol_escrow_account`[UncheckedAccount]
- Transfers `COMPLETE_WITH_AMOUNT` tokens from `sellside_escrow_token_account`[authority=COMPLETE_WITH_TOKEN_AUTHORITY] to `COMPLETE_WITH_DESTINATION_TOKEN_ACCOUNT`[authority=COMPLETE_WITH_TOKEN_AUTHORITY]
- Delegates `COMPLETE_WITH_AMOUNT` tokens from `sellside_escrow_token_account`[authority=COMPLETE_WITH_TOKEN_AUTHORITY] to `COMPLETE_WITH_DESTINATION_TOKEN_ACCOUNT`[authority=COMPLETE_WITH_TOKEN_AUTHORITY]
- `COMPLETE_WITH_THE_REST_OF_STATE_CHANGES`

# Notes:

- [ ] check validations:
  - [ ] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/withdraw_sell.rs#L29)
  - [ ] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/withdraw_sell.rs#L30)
- COMPLETE_WITH_NOTES

# Signers:

- owner: COMPLETE_WITH_SIGNER_DESCRIPTION
- cosigner: COMPLETE_WITH_SIGNER_DESCRIPTION

# Handler function parameters:

- args: WithdrawSellArgs
```rust
  pub struct WithdrawSellArgs {
      pub asset_amount: u64,
      pub allowlist_aux: Option<String>, // TODO: use it for future allowlist_aux
  }
```

# Context accounts:

```rust
  pub struct WithdrawSell<'info> {
      #[account(mut)]
      pub owner: Signer<'info>,
  
      pub cosigner: Signer<'info>,
  
      #[account(
          mut,
          seeds = [POOL_PREFIX.as_bytes(), owner.key().as_ref(), pool.uuid.as_ref(,
          bump
      )]
      pub pool: Box<Account<'info, Pool>>,
  
      pub asset_mint: InterfaceAccount<'info, Mint>,
  
      #[account(
          init_if_needed,
          payer = owner,
          associated_token::mint = asset_mint,
          associated_token::authority = owner,
          associated_token::token_program = token_program
      )]
      pub asset_token_account: Box<InterfaceAccount<'info, TokenAccount>>,
  
      #[account(
          mut,
          associated_token::mint = asset_mint,
          associated_token::authority = pool,
          associated_token::token_program = token_program
      )]
      pub sellside_escrow_token_account: Box<InterfaceAccount<'info, TokenAccount>>,
  
      #[account(
          mut,
          seeds = [BUYSIDE_SOL_ESCROW_ACCOUNT_PREFIX.as_bytes(), pool.key().as_ref(,
          bump)]
      pub buyside_sol_escrow_account: UncheckedAccount<'info>,
  
      pub allowlist_aux_account: UncheckedAccount<'info>,
  
      #[account(
          mut,
          seeds = [
              SELL_STATE_PREFIX.as_bytes(),
              pool.key().as_ref(),
              asset_mint.key().as_ref(),
          ],
          bump
      )]
      pub sell_state: Account<'info, SellState>,
  
      pub system_program: Program<'info, System>,
  
      pub token_program: Interface<'info, TokenInterface>,
  
      pub associated_token_program: Program<'info, AssociatedToken>,
  
      pub rent: Sysvar<'info, Rent>,
  }
```

# Validations:

```rust
    #[account(
    	has_one = owner @ MMMErrorCode::InvalidOwner,
    	has_one = cosigner @ MMMErrorCode::InvalidCosigner,
    )]
    pub pool: Box<Account<'info, Pool>>,
```
```rust
    assert_is_programmable(&Metadata::safe_deserialize(&asset_metadata.data.borrow())?)?;
```

# Miro frame url:

COMPLETE_WITH_MIRO_FRAME_URL
            