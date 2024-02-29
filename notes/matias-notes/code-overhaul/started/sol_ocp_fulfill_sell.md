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
- Updates `owner`[UncheckedAccount]
- Updates `referral`[UncheckedAccount]
- Updates `buyside_sol_escrow_account`[AccountInfo]
- Updates `payer_asset_account`[UncheckedAccount]
- Updates `ocp_mint_state`[UncheckedAccount]
- Transfers `COMPLETE_WITH_AMOUNT` tokens from `sellside_escrow_token_account`[authority=COMPLETE_WITH_TOKEN_AUTHORITY] to `COMPLETE_WITH_DESTINATION_TOKEN_ACCOUNT`[authority=COMPLETE_WITH_TOKEN_AUTHORITY]
- Delegates `COMPLETE_WITH_AMOUNT` tokens from `sellside_escrow_token_account`[authority=COMPLETE_WITH_TOKEN_AUTHORITY] to `COMPLETE_WITH_DESTINATION_TOKEN_ACCOUNT`[authority=COMPLETE_WITH_TOKEN_AUTHORITY]
- `COMPLETE_WITH_THE_REST_OF_STATE_CHANGES`

# Notes:

- [ ] check validations:
  - [ ] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_sell.rs#L50)
  - [ ] [has_one = referral @ MMMErrorCode::InvalidReferral](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_sell.rs#L51)
  - [ ] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_sell.rs#L52)
  - [ ] [constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_sell.rs#L53)
  - [ ] [constraint = pool.expiry == 0](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_sell.rs#L54)
  - [ ] [constraint = asset_mint.supply == 1 && asset_mint.decimals == 0 @ MMMErrorCode::InvalidOcpAssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_sell.rs#L77)
  - [ ] [constraint = sellside_escrow_token_account.amount == 1 @ MMMErrorCode::InvalidOcpAssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_sell.rs#L84)
  - [ ] [constraint = args.asset_amount == 1 @ MMMErrorCode::InvalidOcpAssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_sell.rs#L85)
  - [ ] [address = open_creator_protocol::id(](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_sell.rs#L112)
  - [ ] [address = community_managed_token::id(](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_sell.rs#L115)
  - [ ] [address = sysvar::instructions::id(](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/sol_ocp_fulfill_sell.rs#L118)
- COMPLETE_WITH_NOTES

# Signers:

- payer: COMPLETE_WITH_SIGNER_DESCRIPTION
- cosigner: COMPLETE_WITH_SIGNER_DESCRIPTION

# Handler function parameters:

- args: SolOcpFulfillSellArgs
```rust
  pub struct SolOcpFulfillSellArgs {
      asset_amount: u64,
      max_payment_amount: u64,
      allowlist_aux: Option<String>, // TODO: use it for future allowlist_aux
      maker_fee_bp: i16,             // will be checked by cosigner
      taker_fee_bp: i16,             // will be checked by cosigner
  }
```

# Context accounts:

```rust
  pub struct SolOcpFulfillSell<'info> {
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
  
      #[account(
      seeds = [
          "metadata".as_bytes(),
          mpl_token_metadata::ID.as_ref(),
          asset_mint.key().as_ref(),
      ],
      bump,
      seeds::program = mpl_token_metadata::ID)]
      pub asset_metadata: UncheckedAccount<'info>,
  
      pub asset_mint: InterfaceAccount<'info, Mint>,
  
      #[account(
          mut,
          associated_token::mint = asset_mint,
          associated_token::authority = pool)]
      pub sellside_escrow_token_account: Box<InterfaceAccount<'info, TokenAccount>>,
  
      #[account(mut)]
      pub payer_asset_account: UncheckedAccount<'info>,
  
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
    	constraint = sellside_escrow_token_account.amount == 1 @ MMMErrorCode::InvalidOcpAssetParams,
    	constraint = args.asset_amount == 1 @ MMMErrorCode::InvalidOcpAssetParams,
    )]
    pub sellside_escrow_token_account: Box<InterfaceAccount<'info, TokenAccount>>,
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

COMPLETE_WITH_MIRO_FRAME_URL
            