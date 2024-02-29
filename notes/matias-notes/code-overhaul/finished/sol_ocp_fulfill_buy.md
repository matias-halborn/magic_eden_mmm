# State changes:

- Updates `pool`[Pool]:
	- `pool.sellside_asset_amount`[u64]
	- `pool.lp_fee_earned`[u64]
	- `pool.buyside_payment_amount`[u64]

# Notes:

- [ ] check validations:
  - [x] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_buy.rs#L46)
  - [x] [has_one = referral @ MMMErrorCode::InvalidReferral](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_buy.rs#L47)
  - [x] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_buy.rs#L48)
  - [x] [constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_buy.rs#L49)
  - [ ] [constraint = pool.expiry == 0](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_buy.rs#L50)
  - [ ] [constraint = asset_mint.supply == 1 && asset_mint.decimals == 0 @ MMMErrorCode::InvalidOcpAssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_buy.rs#L73)
  - [ ] [constraint = payer_asset_account.amount == 1 @ MMMErrorCode::InvalidOcpAssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_buy.rs#L81)
  - [ ] [constraint = args.asset_amount == 1 @ MMMErrorCode::InvalidOcpAssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_buy.rs#L82)
  - [x] [address = open_creator_protocol::id(](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_buy.rs#L114)
  - [x] [address = community_managed_token::id(](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_buy.rs#L117)
  - [x] [address = sysvar::instructions::id(](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_buy.rs#L120)

# Signers:

- owner: owner fo the `pool`[Pool] account
- cosigner: cosigner fo the `pool`[Pool] account

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
  pub struct SolOcpFulfillBuy<'info> {
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
  
      #[account(mint::token_program = token_program)]
      pub asset_mint: InterfaceAccount<'info, Mint>,
  
      #[account(
          mut,
          token::mint = asset_mint,
          token::authority = payer)]
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
  
      #[account(mut)]
      pub ocp_mint_state: UncheckedAccount<'info>,
  
      pub ocp_policy: Box<Account<'info, Policy>>,
  
      pub ocp_freeze_authority: UncheckedAccount<'info>,
  
      #[account(address = open_creator_protocol::id())]
      pub ocp_program: UncheckedAccount<'info>,
  
      #[account(address = community_managed_token::id())]
      pub cmt_program: UncheckedAccount<'info>,
  
      #[account(address = sysvar::instructions::id())]
      pub instructions: UncheckedAccount<'info>,
  
      pub system_program: Program<'info, System>,
  
      pub token_program: Program<'info, Token>,
  
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
    	constraint = asset_mint.supply == 1 && asset_mint.decimals == 0 @ MMMErrorCode::InvalidOcpAssetParams,
    )]
    pub asset_mint: InterfaceAccount<'info, Mint>,
```
```rust
    #[account(
    	constraint = payer_asset_account.amount == 1 @ MMMErrorCode::InvalidOcpAssetParams,
    	constraint = args.asset_amount == 1 @ MMMErrorCode::InvalidOcpAssetParams,
    )]
    pub payer_asset_account: Box<InterfaceAccount<'info, TokenAccount>>,
```
```rust
    #[account(
    	address = open_creator_protocol::id(,
    )]
    pub ocp_program: UncheckedAccount<'info>,
```
```rust
    #[account(
    	address = community_managed_token::id(,
    )]
    pub cmt_program: UncheckedAccount<'info>,
```
```rust
    #[account(
    	address = sysvar::instructions::id(,
    )]
    pub instructions: UncheckedAccount<'info>,
```
```rust
    assert_valid_fees_bp(args.maker_fee_bp, args.taker_fee_bp)?;
```

# Miro frame url:

https://miro.com/app/board/uXjVNrXUjBs=/?moveToWidget=3458764579678840467
            