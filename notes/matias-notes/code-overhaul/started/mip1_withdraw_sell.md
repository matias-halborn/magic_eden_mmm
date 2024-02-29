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
- Updates `asset_metadata`[UncheckedAccount]
- Updates `buyside_sol_escrow_account`[UncheckedAccount]
- Updates `owner_token_record`[UncheckedAccount]
- Updates `destination_token_record`[UncheckedAccount]
- Transfers `COMPLETE_WITH_AMOUNT` tokens from `sellside_escrow_token_account`[authority=COMPLETE_WITH_TOKEN_AUTHORITY] to `COMPLETE_WITH_DESTINATION_TOKEN_ACCOUNT`[authority=COMPLETE_WITH_TOKEN_AUTHORITY]
- Delegates `COMPLETE_WITH_AMOUNT` tokens from `sellside_escrow_token_account`[authority=COMPLETE_WITH_TOKEN_AUTHORITY] to `COMPLETE_WITH_DESTINATION_TOKEN_ACCOUNT`[authority=COMPLETE_WITH_TOKEN_AUTHORITY]
- `COMPLETE_WITH_THE_REST_OF_STATE_CHANGES`

# Notes:

- [ ] check validations:
  - [ ] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_withdraw_sell.rs#L31)
  - [ ] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_withdraw_sell.rs#L32)
  - [ ] [constraint = asset_mint.supply == 1 && asset_mint.decimals == 0 @ MMMErrorCode::InvalidMip1AssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_withdraw_sell.rs#L37)
  - [ ] [constraint = sellside_escrow_token_account.amount == 1 @ MMMErrorCode::InvalidMip1AssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_withdraw_sell.rs#L66)
  - [ ] [constraint = args.asset_amount == 1 @ MMMErrorCode::InvalidMip1AssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_withdraw_sell.rs#L67)
  - [ ] [address = mpl_token_metadata::ID](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_withdraw_sell.rs#L99)
  - [ ] [address = MPL_TOKEN_AUTH_RULES](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_withdraw_sell.rs#L102)
  - [ ] [address = sysvar::instructions::id(](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_withdraw_sell.rs#L105)
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
  pub struct Mip1WithdrawSell<'info> {
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
  
      #[account(mut,
      seeds = [
          "metadata".as_bytes(),
          mpl_token_metadata::ID.as_ref(),
          asset_mint.key().as_ref(),
      ],
      bump,
      seeds::program = mpl_token_metadata::ID)]
      pub asset_metadata: UncheckedAccount<'info>,
  
      #[account(
          init_if_needed,
          associated_token::mint = asset_mint,
          associated_token::authority = owner,
          associated_token::token_program = token_program,
          payer = owner
      )]
      pub asset_token_account: Box<InterfaceAccount<'info, TokenAccount>>,
  
      #[account(
          mut,
          associated_token::mint = asset_mint,
          associated_token::authority = pool,
          associated_token::token_program = token_program)]
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
  
      #[account(mut)]
      pub owner_token_record: UncheckedAccount<'info>,
  
      #[account(mut)]
      pub destination_token_record: UncheckedAccount<'info>,
  
      pub authorization_rules: UncheckedAccount<'info>,
  
      #[account(address = mpl_token_metadata::ID)]
      pub token_metadata_program: UncheckedAccount<'info>,
  
      #[account(address = MPL_TOKEN_AUTH_RULES)]
      pub authorization_rules_program: UncheckedAccount<'info>,
  
      #[account(address = sysvar::instructions::id())]
      pub instructions: UncheckedAccount<'info>,
  
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
    #[account(
    	constraint = asset_mint.supply == 1 && asset_mint.decimals == 0 @ MMMErrorCode::InvalidMip1AssetParams,
    )]
    pub asset_mint: InterfaceAccount<'info, Mint>,
```
```rust
    #[account(
    	constraint = sellside_escrow_token_account.amount == 1 @ MMMErrorCode::InvalidMip1AssetParams,
    	constraint = args.asset_amount == 1 @ MMMErrorCode::InvalidMip1AssetParams,
    )]
    pub sellside_escrow_token_account: Box<InterfaceAccount<'info, TokenAccount>>,
```
```rust
    #[account(
    	address = mpl_token_metadata::ID,
    )]
    pub token_metadata_program: UncheckedAccount<'info>,
```
```rust
    #[account(
    	address = MPL_TOKEN_AUTH_RULES,
    )]
    pub authorization_rules_program: UncheckedAccount<'info>,
```
```rust
    #[account(
    	address = sysvar::instructions::id(,
    )]
    pub instructions: UncheckedAccount<'info>,
```
```rust
    assert_is_programmable(&Metadata::safe_deserialize(&asset_metadata.data.borrow())?)?;
```

# Miro frame url:

COMPLETE_WITH_MIRO_FRAME_URL
            