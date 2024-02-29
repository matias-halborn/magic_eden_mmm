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
- Updates `asset_metadata`[UncheckedAccount]
- Updates `owner_token_record`[UncheckedAccount]
- Updates `destination_token_record`[UncheckedAccount]
- Transfers `COMPLETE_WITH_AMOUNT` tokens from `asset_token_account`[authority=COMPLETE_WITH_TOKEN_AUTHORITY] to `COMPLETE_WITH_DESTINATION_TOKEN_ACCOUNT`[authority=COMPLETE_WITH_TOKEN_AUTHORITY]
- Delegates `COMPLETE_WITH_AMOUNT` tokens from `asset_token_account`[authority=COMPLETE_WITH_TOKEN_AUTHORITY] to `COMPLETE_WITH_DESTINATION_TOKEN_ACCOUNT`[authority=COMPLETE_WITH_TOKEN_AUTHORITY]
- `COMPLETE_WITH_THE_REST_OF_STATE_CHANGES`

# Notes:

- [ ] check validations:
  - [ ] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_deposit_sell.rs#L30)
  - [ ] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_deposit_sell.rs#L31)
  - [ ] [constraint = asset_mint.supply == 1 && asset_mint.decimals == 0 @ MMMErrorCode::InvalidMip1AssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_deposit_sell.rs#L47)
  - [ ] [constraint = asset_token_account.amount == 1 @ MMMErrorCode::InvalidMip1AssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_deposit_sell.rs#L56)
  - [ ] [constraint = args.asset_amount == 1 @ MMMErrorCode::InvalidMip1AssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_deposit_sell.rs#L57)
  - [ ] [address = mpl_token_metadata::ID](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_deposit_sell.rs#L93)
  - [ ] [address = MPL_TOKEN_AUTH_RULES](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_deposit_sell.rs#L96)
  - [ ] [address = sysvar::instructions::id(](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/mip1_deposit_sell.rs#L99)
- COMPLETE_WITH_NOTES

# Signers:

- owner: COMPLETE_WITH_SIGNER_DESCRIPTION
- cosigner: COMPLETE_WITH_SIGNER_DESCRIPTION

# Handler function parameters:

- args: DepositSellArgs
```rust
  pub struct DepositSellArgs {
      pub asset_amount: u64,
      pub allowlist_aux: Option<String>, // TODO: use it for future allowlist_aux
  }
```

# Context accounts:

```rust
  pub struct Mip1DepositSell<'info> {
      #[account(mut)]
      pub owner: Signer<'info>,
  
      pub cosigner: Signer<'info>,
  
      #[account(
          mut,
          seeds = [POOL_PREFIX.as_bytes(), owner.key().as_ref(), pool.uuid.as_ref(,
          bump
      )]
      pub pool: Box<Account<'info, Pool>>,
  
      #[account(mut,
      seeds = [
          "metadata".as_bytes(),
          mpl_token_metadata::ID.as_ref(),
          asset_mint.key().as_ref(),
      ],
      bump,
      seeds::program = mpl_token_metadata::ID)]
      pub asset_metadata: UncheckedAccount<'info>,
  
      pub asset_mint: InterfaceAccount<'info, Mint>,
  
      pub asset_master_edition: UncheckedAccount<'info>,
  
      #[account(
          mut,
          token::mint = asset_mint,
          token::authority = owner)]
      pub asset_token_account: Box<InterfaceAccount<'info, TokenAccount>>,
  
      #[account(
          init_if_needed,
          payer = owner,
          associated_token::mint = asset_mint,
          associated_token::authority = pool,
          associated_token::token_program = token_program)]
      pub sellside_escrow_token_account: Box<InterfaceAccount<'info, TokenAccount>>,
  
      #[account(
          init_if_needed,
          payer = owner,
          seeds = [
              SELL_STATE_PREFIX.as_bytes(),
              pool.key().as_ref(),
              asset_mint.key().as_ref(),
          ],
          space = SellState::LEN,
          bump
      )]
      pub sell_state: Account<'info, SellState>,
  
      pub allowlist_aux_account: UncheckedAccount<'info>,
  
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
    	constraint = asset_token_account.amount == 1 @ MMMErrorCode::InvalidMip1AssetParams,
    	constraint = args.asset_amount == 1 @ MMMErrorCode::InvalidMip1AssetParams,
    )]
    pub asset_token_account: Box<InterfaceAccount<'info, TokenAccount>>,
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
    assert_is_programmable(&parsed_metadata)?;
```

# Miro frame url:

COMPLETE_WITH_MIRO_FRAME_URL
            