# State changes:

- Closes `pool`[Pool]. Rent exemption goes to `owner`

# Notes:

- [ ] check validations:
  - [ ] [constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/sol_close_pool.rs#L11)
  - [ ] [constraint = pool.sellside_asset_amount == 0 @ MMMErrorCode::NotEmptySellsideAssetAmount](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/sol_close_pool.rs#L12)
  - [x] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/sol_close_pool.rs#L14)
  - [x] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/sol_close_pool.rs#L15)
  - [ ] [constraint = buyside_sol_escrow_account.lamports() == 0 @ MMMErrorCode::NotEmptyEscrowAccount](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/sol_close_pool.rs#L22)
- [ ] this function only closes the `pool` account, but the `buyside_sol_escrow_account` requires the lamports to be 0?
- [ ] where is the `buyside_sol_escrow_account` account created?
- [ ] why this function requires both the owner and the cosigner to be signers?

# Signers:

- owner: owner of the `Pool` account
- cosigner: cosigner of the `Pool` account

# Handler function parameters:

NO_HANDLER_FUNCTION_PARAMETERS_DETECTED

# Context accounts:

```rust
  pub struct SolClosePool<'info> {
      #[account(mut)]
      pub owner: Signer<'info>,
  
      pub cosigner: Signer<'info>,
  
      #[account(
          mut,
          seeds = [POOL_PREFIX.as_bytes(), owner.key().as_ref(), pool.uuid.as_ref(,
          bump,
          close = owner
      )]
      pub pool: Box<Account<'info, Pool>>,
  
      #[account(
          seeds = [BUYSIDE_SOL_ESCROW_ACCOUNT_PREFIX.as_bytes(), pool.key().as_ref(,
          bump)]
      pub buyside_sol_escrow_account: AccountInfo<'info>,
  
      pub system_program: Program<'info, System>,
  }
```

# Validations:

```rust
    #[account(
    	constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint,
    	constraint = pool.sellside_asset_amount == 0 @ MMMErrorCode::NotEmptySellsideAssetAmount,
    	has_one = owner @ MMMErrorCode::InvalidOwner,
    	has_one = cosigner @ MMMErrorCode::InvalidCosigner,
    )]
    pub pool: Box<Account<'info, Pool>>,
```
```rust
    #[account(
    	constraint = buyside_sol_escrow_account.lamports() == 0 @ MMMErrorCode::NotEmptyEscrowAccount,
    )]
    pub buyside_sol_escrow_account: AccountInfo<'info>,
```

# Miro frame url:

https://miro.com/app/board/uXjVNrXUjBs=/?moveToWidget=3458764579678840018
            