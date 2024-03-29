# State changes:

- Initializes `pool`[Pool], funded by `owner`

# Notes:

- [ ] check validations:
  - [x] [constraint = args.lp_fee_bp <= MAX_LP_FEE_BP @ MMMErrorCode::InvalidBP](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/create_pool.rs#L36)
  - [x] [constraint = args.buyside_creator_royalty_bp <= 10000 @ MMMErrorCode::InvalidBP](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/create_pool.rs#L37)
  - [x] [constraint = args.spot_price > 0 @ MMMErrorCode::InvalidSpotPrice](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/create_pool.rs#L38)
  - [x] [constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/create_pool.rs#L39)
  - [ ] [constraint = args.referral.ne(owner.key) @ MMMErrorCode::InvalidReferral](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/create_pool.rs#L40)

# Signers:

- owner: creator of the pool
- cosigner: CHECK: the cosigner can be set as owner if you want optional cosigner

# Handler function parameters:

- args: CreatePoolArgs
```rust
  pub struct CreatePoolArgs {
      // mutable
      pub spot_price: u64,
      pub curve_type: u8,
      pub curve_delta: u64,
      pub reinvest_fulfill_buy: bool,
      pub reinvest_fulfill_sell: bool,
      pub expiry: i64,
      pub lp_fee_bp: u16,
      pub referral: Pubkey,
      pub cosigner_annotation: [u8; 32],
      pub buyside_creator_royalty_bp: u16,
  
      // immutable
      pub uuid: Pubkey, // randomly generated keypair
      pub payment_mint: Pubkey,
      pub allowlists: [Allowlist; ALLOWLIST_MAX_LEN],
  }
```

# Context accounts:

```rust
  pub struct CreatePool<'info> {
      #[account(mut)]
      pub owner: Signer<'info>,
  
      pub cosigner: Signer<'info>,
  
      #[account(
          init,
          payer = owner,
          seeds = [POOL_PREFIX.as_bytes(), owner.key().as_ref(), args.uuid.as_ref(,
          bump,
          space = Pool::LEN)]
      pub pool: Box<Account<'info, Pool>>,
  
      pub system_program: Program<'info, System>,
  }
```

# Validations:

```rust
    #[account(
    	constraint = args.lp_fee_bp <= MAX_LP_FEE_BP @ MMMErrorCode::InvalidBP,
    	constraint = args.buyside_creator_royalty_bp <= 10000 @ MMMErrorCode::InvalidBP,
    	constraint = args.spot_price > 0 @ MMMErrorCode::InvalidSpotPrice,
    	constraint = pool.payment_mint.eq(&Pubkey::default()) @ MMMErrorCode::InvalidPaymentMint,
    	constraint = args.referral.ne(owner.key) @ MMMErrorCode::InvalidReferral,
    )]
    pub pool: Box<Account<'info, Pool>>,
```

```rust
    check_allowlists(&args.allowlists)?;
    check_curve(args.curve_type, args.curve_delta)?;
```

# Miro frame url:

https://miro.com/app/board/uXjVNrXUjBs=/?moveToWidget=3458764579678624909
            