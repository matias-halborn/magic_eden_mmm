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
- Updates `owner`[UncheckedAccount]
- Updates `referral`[UncheckedAccount]
- Updates `buyside_sol_escrow_account`[UncheckedAccount]
- Updates `sellside_escrow_token_account`[UncheckedAccount]
- Updates `owner_token_account`[UncheckedAccount]
- Transfers `COMPLETE_WITH_AMOUNT` tokens from `payer_asset_account`[authority=COMPLETE_WITH_TOKEN_AUTHORITY] to `COMPLETE_WITH_DESTINATION_TOKEN_ACCOUNT`[authority=COMPLETE_WITH_TOKEN_AUTHORITY]
- Delegates `COMPLETE_WITH_AMOUNT` tokens from `payer_asset_account`[authority=COMPLETE_WITH_TOKEN_AUTHORITY] to `COMPLETE_WITH_DESTINATION_TOKEN_ACCOUNT`[authority=COMPLETE_WITH_TOKEN_AUTHORITY]
- `COMPLETE_WITH_THE_REST_OF_STATE_CHANGES`

# Notes:

- [ ] check validations:
  - [ ] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/sol_fulfill_buy.rs#L51)
  - [ ] [has_one = referral @ MMMErrorCode::InvalidReferral](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/sol_fulfill_buy.rs#L52)
  - [ ] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/sol_fulfill_buy.rs#L53)
  - [ ] [constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/sol_fulfill_buy.rs#L54)
  - [ ] [constraint = pool.expiry == 0](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/sol_fulfill_buy.rs#L55)
- COMPLETE_WITH_NOTES

# Signers:

- payer: COMPLETE_WITH_SIGNER_DESCRIPTION
- cosigner: COMPLETE_WITH_SIGNER_DESCRIPTION

# Handler function parameters:

- args: SolFulfillBuyArgs
```rust
  pub struct SolFulfillBuyArgs {
      pub asset_amount: u64,
      pub min_payment_amount: u64,
      pub allowlist_aux: Option<String>, // TODO: use it for future allowlist_aux
      pub maker_fee_bp: i16,             // will be checked by cosigner
      pub taker_fee_bp: i16,             // will be checked by cosigner
  }
```

# Context accounts:

```rust
  pub struct SolFulfillBuy<'info> {
      #[account(mut)]
      pub payer: Signer<'info>,
  
      #[account(mut)]
      pub owner: UncheckedAccount<'info>,
  
      pub cosigner: Signer<'info>,
  
      #[account(mut)]/// CHECK: we will check that the referral matches the pool's referral)]
      pub referral: UncheckedAccount<'info>,
  
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
  
      #[account(
      seeds = [
          "metadata".as_bytes(),
          mpl_token_metadata::ID.as_ref(),
          asset_mint.key().as_ref(),
      ],
      bump,
      seeds::program = mpl_token_metadata::ID)]
      pub asset_metadata: UncheckedAccount<'info>,
  
      pub asset_master_edition: UncheckedAccount<'info>,
  
      pub asset_mint: InterfaceAccount<'info, Mint>,
  
      #[account(
          mut,
          token::mint = asset_mint,
          token::authority = payer,
          token::token_program = token_program)]
      pub payer_asset_account: Box<InterfaceAccount<'info, TokenAccount>>,
  
      #[account(mut)]
      pub sellside_escrow_token_account: UncheckedAccount<'info>,
  
      #[account(mut)]
      pub owner_token_account: UncheckedAccount<'info>,
  
      pub allowlist_aux_account: UncheckedAccount<'info>,
  
      #[account(
          init_if_needed,
          payer = payer,
          seeds = [
              SELL_STATE_PREFIX.as_bytes(),
              pool.key().as_ref(),
              asset_mint.key().as_ref(),
          ],
          space = SellState::LEN,
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
    	has_one = referral @ MMMErrorCode::InvalidReferral,
    	has_one = cosigner @ MMMErrorCode::InvalidCosigner,
    	constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint,
    	constraint = pool.expiry == 0,
    )]
    pub pool: Box<Account<'info, Pool>>,
```
```rust
    assert_valid_fees_bp(args.maker_fee_bp, args.taker_fee_bp)?;
```

# Miro frame url:

COMPLETE_WITH_MIRO_FRAME_URL
            