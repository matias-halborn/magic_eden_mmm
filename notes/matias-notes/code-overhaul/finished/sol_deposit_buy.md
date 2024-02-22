# State changes:

- Updates `pool`[Pool]:
	- `pool.buyside_payment_amount`[u64]
- Transfers `SolDepositBuyArgs.payment_amount` from `owner` to `buyside_sol_escrow_account`[UncheckedAccount]

# Notes:

- [x] check validations:
  - [x] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/sol_deposit_buy.rs#L20)
  - [x] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/sol_deposit_buy.rs#L21)
  - [x] [constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/vanilla/sol_deposit_buy.rs#L22)

# Signers:

- owner: owner of the `pool`[Pool] account
- cosigner: cosigner of the `pool`[Pool] account

# Handler function parameters:

- args: SolDepositBuyArgs
```rust
  pub struct SolDepositBuyArgs {
      payment_amount: u64,
  }
```

# Context accounts:

```rust
  pub struct SolDepositBuy<'info> {
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

```rust
    if pool.using_shared_escrow() {
        return Err(MMMErrorCode::InvalidAccountState.into());
    }
```

# Miro frame url:

https://miro.com/app/board/uXjVNrXUjBs=/?moveToWidget=3458764579678840039
            