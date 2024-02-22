# State changes:

- Updates `pool`[Pool]:
	- `pool.buyside_payment_amount`[u64]
- Transfers `amount_to_withdraw` from `buyside_sol_escrow_account`[UncheckedAccount] to `owner`

# Notes:

- [x] check validations:
  - [x] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/sol_withdraw_buy.rs#L23)
  - [x] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/sol_withdraw_buy.rs#L24)
  - [x] [constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/sol_withdraw_buy.rs#L25)

# Signers:

- owner: owner of the `pool`[Pool] account
- cosigner: cosigner of the `pool`[Pool] account

# Handler function parameters:

- args: SolWithdrawBuyArgs
```rust
  pub struct SolWithdrawBuyArgs {
      payment_amount: u64,
  }
```

# Context accounts:

```rust
  pub struct SolWithdrawBuy<'info> {
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
          mut,
          seeds = [BUYSIDE_SOL_ESCROW_ACCOUNT_PREFIX.as_bytes(), pool.key().as_ref(,
          bump)]
      pub buyside_sol_escrow_account: UncheckedAccount<'info>,
  
      pub system_program: Program<'info, System>,
  }
```

# Validations:

```rust
    #[account(
    	has_one = owner @ MMMErrorCode::InvalidOwner,
    	has_one = cosigner @ MMMErrorCode::InvalidCosigner,
    	constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint,
    )]
    pub pool: Box<Account<'info, Pool>>,
```

# Miro frame url:

https://miro.com/app/board/uXjVNrXUjBs=/?moveToWidget=3458764579678840083
            