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
- `COMPLETE_WITH_THE_REST_OF_STATE_CHANGES`

# Notes:

- [ ] check validations:
  - [ ] [has_one = owner @ MMMErrorCode::InvalidOwner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/update_pool.rs#L28)
  - [ ] [has_one = cosigner @ MMMErrorCode::InvalidCosigner](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/update_pool.rs#L29)
  - [ ] [constraint = args.lp_fee_bp <= MAX_LP_FEE_BP @ MMMErrorCode::InvalidBP](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/update_pool.rs#L30)
  - [ ] [constraint = args.buyside_creator_royalty_bp <= 10000 @ MMMErrorCode::InvalidBP](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/update_pool.rs#L31)
  - [ ] [constraint = args.spot_price > 0 @ MMMErrorCode::InvalidSpotPrice](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/update_pool.rs#L32)
  - [ ] [constraint = args.referral.ne(owner.key) @ MMMErrorCode::InvalidReferral](https://github.com/magicoss/mmm/blob/3e15732061ad03256b2570b78ff8018ba74ce039/programs/mmm/src/instructions/admin/update_pool.rs#L33)
- COMPLETE_WITH_NOTES

# Signers:

- owner: COMPLETE_WITH_SIGNER_DESCRIPTION
- cosigner: COMPLETE_WITH_SIGNER_DESCRIPTION

# Handler function parameters:

- args: UpdatePoolArgs
```rust
  pub struct UpdatePoolArgs {
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
  }
```

# Context accounts:

```rust
  pub struct UpdatePool<'info> {
      #[account(mut)]
      pub owner: Signer<'info>,
  
      pub cosigner: Signer<'info>,
  
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
    	has_one = owner @ MMMErrorCode::InvalidOwner,
    	has_one = cosigner @ MMMErrorCode::InvalidCosigner,
    	constraint = args.lp_fee_bp <= MAX_LP_FEE_BP @ MMMErrorCode::InvalidBP,
    	constraint = args.buyside_creator_royalty_bp <= 10000 @ MMMErrorCode::InvalidBP,
    	constraint = args.spot_price > 0 @ MMMErrorCode::InvalidSpotPrice,
    	constraint = args.referral.ne(owner.key) @ MMMErrorCode::InvalidReferral,
    )]
    pub pool: Box<Account<'info, Pool>>,
```

# Miro frame url:

COMPLETE_WITH_MIRO_FRAME_URL
            