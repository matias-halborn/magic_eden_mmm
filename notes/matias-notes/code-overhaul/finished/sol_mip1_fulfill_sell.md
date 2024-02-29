# State changes:

- Updates `pool`[Pool]:
- Updates `sell_state`[SellState]:

# Notes:

- [ ] check validations:
  - [x] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/sol_mip1_fulfill_sell.rs#L51)
  - [x] [has_one = referral @ MMMErrorCode::InvalidReferral](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/sol_mip1_fulfill_sell.rs#L52)
  - [x] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/sol_mip1_fulfill_sell.rs#L53)
  - [x] [constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/sol_mip1_fulfill_sell.rs#L54)
  - [ ] [constraint = pool.expiry == 0](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/sol_mip1_fulfill_sell.rs#L55)
  - [ ] [constraint = asset_mint.supply == 1 && asset_mint.decimals == 0 @ MMMErrorCode::InvalidMip1AssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/sol_mip1_fulfill_sell.rs#L78)
  - [ ] [constraint = sellside_escrow_token_account.amount == 1 @ MMMErrorCode::InvalidMip1AssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/sol_mip1_fulfill_sell.rs#L89)
  - [ ] [constraint = args.asset_amount == 1 @ MMMErrorCode::InvalidMip1AssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/sol_mip1_fulfill_sell.rs#L90)
  - [x] [address = mpl_token_metadata::ID](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/sol_mip1_fulfill_sell.rs#L123)
  - [x] [address = MPL_TOKEN_AUTH_RULES](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/sol_mip1_fulfill_sell.rs#L126)
  - [x] [address = sysvar::instructions::id(](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/mip1/sol_mip1_fulfill_sell.rs#L129)

# Signers:

- owner: owner fo the `pool`[Pool] account
- cosigner: cosigner fo the `pool`[Pool] account

# Handler function parameters:

- args: SolMip1FulfillSellArgs
```rust
  pub struct SolMip1FulfillSellArgs {
      pub asset_amount: u64,
      pub max_payment_amount: u64,
      pub allowlist_aux: Option<String>, // TODO: use it for future allowlist_aux
      pub maker_fee_bp: i16,             // will be checked by cosigner
      pub taker_fee_bp: i16,             // will be checked by cosigner
  }
```

# Context accounts:

```rust
  pub struct SolMip1FulfillSell<'info> {
      #[account(mut)]
      pub payer: Signer<'info>,
  
      #[account(mut)]
      pub owner: UncheckedAccount<'info>,
  
      pub cosigner: Signer<'info>,
  
      #[account(mut)]
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
      pub buyside_sol_escrow_account: AccountInfo<'info>,
  
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
          associated_token::mint = asset_mint,
          associated_token::authority = pool,
          associated_token::token_program = token_program)]
      pub sellside_escrow_token_account: Box<InterfaceAccount<'info, TokenAccount>>,
  
      #[account(
          init_if_needed,
          associated_token::mint = asset_mint,
          associated_token::authority = payer,
          associated_token::token_program = token_program,
          payer = payer
      )]
      pub payer_asset_account: Box<InterfaceAccount<'info, TokenAccount>>,
  
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
    	has_one = referral @ MMMErrorCode::InvalidReferral,
    	has_one = cosigner @ MMMErrorCode::InvalidCosigner,
    	constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint,
    	constraint = pool.expiry == 0,
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
    assert_is_programmable(&parsed_metadata)?;
```
```rust
    assert_valid_fees_bp(args.maker_fee_bp, args.taker_fee_bp)?;
```

# Miro frame url:

https://miro.com/app/board/uXjVNrXUjBs=/?moveToWidget=3458764579678840679
            