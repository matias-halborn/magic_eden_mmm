# State changes:

- Updates `pool`[Pool]:
	- `pool.allowlists`[Allowlist; ALLOWLIST_MAX_LEN]

# Notes:

- [ ] check validations:
  - [x] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/update_allowlists.rs#L19)
- [ ] why it is not requiring owner to sign, but is necessary to update the pool config?

# Signers:

- cosigner: cosigner of the `Pool` account

# Handler function parameters:

- args: UpdateAllowlistsArgs
```rust
  pub struct UpdateAllowlistsArgs {
      pub allowlists: [Allowlist; ALLOWLIST_MAX_LEN],
  }
```

# Context accounts:

```rust
  pub struct UpdateAllowlists<'info> {
      #[account(mut)]
      pub cosigner: Signer<'info>,
  
      pub owner: UncheckedAccount<'info>,
  
      #[account(
          mut,
          seeds = [POOL_PREFIX.as_bytes(), owner.key().as_ref(), pool.uuid.as_ref(,
          bump)]
      pub pool: Box<Account<'info, Pool>>,
  }
```

# Validations:

```rust
    #[account(
    	has_one = cosigner @ MMMErrorCode::InvalidCosigner,
    )]
    pub pool: Box<Account<'info, Pool>>,
```

```rust 
	check_allowlists(&args.allowlists)?;
```

# Miro frame url:

https://miro.com/app/board/uXjVNrXUjBs=/?moveToWidget=3458764579678839986
            