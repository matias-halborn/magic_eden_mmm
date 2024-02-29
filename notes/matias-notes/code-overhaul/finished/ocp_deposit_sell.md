# State changes:

- Updates `pool`[Pool]:
	- `pool.sellside_asset_amount`[u64]

# Notes:

- [ ] check validations:
  - [x] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/ocp_deposit_sell.rs#L26)
  - [x] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/ocp_deposit_sell.rs#L27)
  - [ ] [constraint = asset_mint.supply == 1 && asset_mint.decimals == 0 @ MMMErrorCode::InvalidOcpAssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/ocp_deposit_sell.rs#L43)
  - [ ] [constraint = asset_token_account.amount == 1 @ MMMErrorCode::InvalidOcpAssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/ocp_deposit_sell.rs#L50)
  - [ ] [constraint = args.asset_amount == 1 @ MMMErrorCode::InvalidOcpAssetParams](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/ocp_deposit_sell.rs#L51)
  - [x] [address = open_creator_protocol::id(](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/ocp_deposit_sell.rs#L80)
  - [x] [address = community_managed_token::id(](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/ocp_deposit_sell.rs#L83)
  - [x] [address = sysvar::instructions::id(](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/ocp/ocp_deposit_sell.rs#L86)

# Signers:

- owner: owner fo the `pool`[Pool] account
- cosigner: cosigner fo the `pool`[Pool] account

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
  pub struct OcpDepositSell<'info> {
      #[account(mut)]
      pub owner: Signer<'info>,
  
      pub cosigner: Signer<'info>,
  
      #[account(
          mut,
          seeds = [POOL_PREFIX.as_bytes(), owner.key().as_ref(), pool.uuid.as_ref(,
          bump
      )]
      pub pool: Box<Account<'info, Pool>>,
  
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
          token::mint = asset_mint,
          token::authority = owner)]
      pub asset_token_account: Box<InterfaceAccount<'info, TokenAccount>>,
  
      #[account(mut)]
      pub sellside_escrow_token_account: UncheckedAccount<'info>,
  
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
      pub ocp_mint_state: UncheckedAccount<'info>,
  
      pub ocp_policy: UncheckedAccount<'info>,
  
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
    	has_one = cosigner @ MMMErrorCode::InvalidCosigner,
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
    	constraint = asset_token_account.amount == 1 @ MMMErrorCode::InvalidOcpAssetParams,
    	constraint = args.asset_amount == 1 @ MMMErrorCode::InvalidOcpAssetParams,
    )]
    pub asset_token_account: Box<InterfaceAccount<'info, TokenAccount>>,
```
```rust
    #[account(
    	address = open_creator_protocol::id(),
    )]
    pub ocp_program: UncheckedAccount<'info>,
```
```rust
    #[account(
    	address = community_managed_token::id(),
    )]
    pub cmt_program: UncheckedAccount<'info>,
```
```rust
    #[account(
    	address = sysvar::instructions::id(),
    )]
    pub instructions: UncheckedAccount<'info>,
```

# Miro frame url:

https://miro.com/app/board/uXjVNrXUjBs=/?moveToWidget=3458764579678840412
            