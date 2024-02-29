# State changes:

- Updates `pool`[Pool]:
	- `pool.shared_escrow_account`[Pubkey, // this points to the shared escrow account PDA (usually M2)]
	- `pool.shared_escrow_count`[u64, // this means that how many times (count) the shared escrow account can be fulfilled, and it can be mutable]

# Notes:

- [x] check validations:
  - [x] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/set_shared_escrow.rs#L18)
  - [x] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/set_shared_escrow.rs#L19)

# Signers:

- owner: owner fo the `pool`[Pool] account
- cosigner: cosigner fo the `pool`[Pool] account

# Handler function parameters:

- args: SetSharedEscrowArgs
```rust
  pub struct SetSharedEscrowArgs {
      pub shared_escrow_count: u64,
  }
```

# Context accounts:

```rust
  pub struct SetSharedEscrow<'info> {
      #[account(mut)]
      pub owner: Signer<'info>,
  
      pub cosigner: Signer<'info>,
  
      #[account(
          mut,
          seeds = [POOL_PREFIX.as_bytes(), owner.key().as_ref(), pool.uuid.as_ref(,
          bump)]
      pub pool: Box<Account<'info, Pool>>,
  
      #[account(
          seeds = [
              M2_PREFIX.as_bytes(),
              M2_AUCTION_HOUSE.as_ref(),
              owner.key().as_ref(),
          ],
          bump,
          seeds::program = M2_PROGRAM)]
      pub shared_escrow_account: UncheckedAccount<'info>,
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

# Miro frame url:

https://miro.com/app/board/uXjVNrXUjBs=/?moveToWidget=3458764579678840716
            