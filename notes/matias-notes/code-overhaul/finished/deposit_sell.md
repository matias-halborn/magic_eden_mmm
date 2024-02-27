# State changes:

- Updates `pool`[Pool]:
	- `pool.sellside_asset_amount`[u64]
- Updates `sell_state`[SellState]:
	- `sell_state.pool`
	- `sell_state.pool_owner`
	- `sell_state.asset_mint`
	- `sell_state.asset_amount`
	- `sell_state.cosigner_annotation`
- Transfers `args.asset_amount` tokens from `asset_token_account`[authority=owner] to `sellside_escrow_token_account`[authority=pool]

# Notes:

- [x] check validations:
  - [x] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/deposit_sell.rs#L29)
  - [x] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/deposit_sell.rs#L30)

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
  pub struct DepositSell<'info> {
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
  
      pub asset_master_edition: UncheckedAccount<'info>,
  
      pub asset_mint: InterfaceAccount<'info, Mint>,
  
      #[account(
          mut,
          associated_token::mint = asset_mint,
          associated_token::authority = owner,
          associated_token::token_program = token_program)]
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
    assert_is_programmable(&parsed_metadata)?;
```

# Miro frame url:

https://miro.com/app/board/uXjVNrXUjBs=/?moveToWidget=3458764579678840266
            