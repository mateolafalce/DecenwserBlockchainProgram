## main_account()

```rust
use anchor_lang::{
    prelude::*,
    solana_program::pubkey::Pubkey
}; 
use crate::state::accounts::*;
use crate::error::ErrorCode;

pub fn main_account(
    ctx: Context<MainAccountStruct>,
    web_name: String
) -> Result<()> {
    require!(web_name.len() <= 32, ErrorCode::TooLong);
    let main_account: &mut Account<MainAccount> = &mut ctx.accounts.main_account;
    let (_pda, bump) = Pubkey::find_program_address(&
        [&anchor_lang::solana_program::hash::hash(web_name.as_bytes()).to_bytes()
        ], 
        ctx.program_id
    );
    main_account.bump_original = bump;
    main_account.web_name = web_name;
    main_account.authority = ctx.accounts.signer.key();
    main_account.html = 0;
    main_account.js = 0;
    msg!(
        "{} is part of the international decentralized information interchange. Authority: {}",
        main_account.web_name,
        ctx.accounts.signer.key()
    );
    Ok(())
}

#[derive(Accounts)]
#[instruction(web_name: String)]
pub struct MainAccountStruct<'info> {
    #[account(init, 
        seeds = [
            &anchor_lang::solana_program::hash::hash(web_name.as_bytes()).to_bytes()
            ], 
            bump, 
            payer = signer, 
            space = MainAccount::SIZE + 8
        )
    ]
    pub main_account: Account<'info, MainAccount>,
    #[account(mut)]
    pub signer: Signer<'info>,
    pub system_program: Program<'info, System>,
}
```

## html_store()

```rust
use anchor_lang::{
    prelude::*,
    solana_program::pubkey::Pubkey
}; 
use crate::state::accounts::*;
use crate::error::ErrorCode;

pub fn html_store(
    ctx: Context<HtmlStore>,
    content: Vec<u8>
) -> Result<()> {
    require!(ctx.accounts.main_account.authority.key() == ctx.accounts.signer.key(), ErrorCode::AuthorityError);
    let (_pda, bump) = Pubkey::find_program_address(&[
        b"HTML", 
        ctx.accounts.main_account.html.to_le_bytes().as_ref(), 
        ctx.accounts.main_account.key().as_ref()
        ], 
        ctx.program_id);
    let main_account: &mut Account<MainAccount> = &mut ctx.accounts.main_account;
    let store: &mut Account<StoreAccount> = &mut ctx.accounts.store;
    store.content = content;
    store.bump_original = bump;
    msg!(
        "New account stores the HTML content with seed = [HTML, {}, {}]", 
        main_account.html,
        main_account.key()
    );
    Ok(())
}

#[derive(Accounts)]
pub struct HtmlStore<'info> {
    #[account(
        mut,
        seeds = [
            &anchor_lang::solana_program::hash::hash(
                main_account.web_name.as_bytes()
            ).to_bytes()
            ],
        bump = main_account.bump_original
    )]
    pub main_account: Account<'info, MainAccount>,
    #[account(init, seeds = 
        [
        b"HTML", 
        main_account.html.to_le_bytes().as_ref(), 
        main_account.key().as_ref()
        ], 
    bump, payer = signer, 
    space = StoreAccount::SIZE + 8
    )]
    pub store: Account<'info, StoreAccount>,
    #[account(mut)]
    pub signer: Signer<'info>,
    pub system_program: Program<'info, System>,
}
```

## add_html()

```rust
use anchor_lang::{
    prelude::*,
    solana_program::pubkey::Pubkey
}; 
use crate::state::accounts::*;
use crate::error::ErrorCode;

pub fn add_html(
    ctx: Context<AddHtml>,
    content: Vec<u8>
) -> Result<()> {
    require!(
        ctx.accounts.main_account.authority.key() == ctx.accounts.signer.key(), 
        ErrorCode::AuthorityError
    );
    require!(
        ctx.accounts.store.content.len() < 9900, 
        ErrorCode::Max9900
    );
    let main_account: &mut Account<MainAccount> = &mut ctx.accounts.main_account;
    let store: &mut Account<StoreAccount> = &mut ctx.accounts.store;
    store.content.extend(content);
    msg!("The content of the PDA was updated.");
    if store.content.len() == 9900 {
        main_account.html += 1;
    }
    Ok(())
}

#[derive(Accounts)]
pub struct AddHtml<'info> {
    #[account(
        mut,
        seeds = [
            &anchor_lang::solana_program::hash::hash(
                main_account.web_name.as_bytes()
            ).to_bytes()
            ],
        bump = main_account.bump_original
    )]
    pub main_account: Account<'info, MainAccount>,
    #[account(
        mut, 
        seeds = 
            [
            b"HTML", 
            main_account.html.to_le_bytes().as_ref(), 
            main_account.key().as_ref()
            ],
        bump = store.bump_original,
        realloc = 8 + 4 + 1 + store.content.len() + 900,
        realloc::payer = signer,
        realloc::zero = false,
    )]
    pub store: Account<'info, StoreAccount>,
    #[account(mut)]
    pub signer: Signer<'info>,
    pub system_program: Program<'info, System>,
}
```

## js_store()

```rust
use anchor_lang::{
    prelude::*,
    solana_program::pubkey::Pubkey
}; 
use crate::state::accounts::*;
use crate::error::ErrorCode;

pub fn js_store(
    ctx: Context<JsStore>,
    content: Vec<u8>
) -> Result<()> {
    require!(ctx.accounts.main_account.authority.key() == ctx.accounts.signer.key(), ErrorCode::AuthorityError);
    let (_pda, bump) = Pubkey::find_program_address(&[
        b"JS", 
        ctx.accounts.main_account.js.to_le_bytes().as_ref(), 
        ctx.accounts.main_account.key().as_ref()
        ], 
        ctx.program_id);
    let main_account: &mut Account<MainAccount> = &mut ctx.accounts.main_account;
    let store: &mut Account<StoreAccount> = &mut ctx.accounts.store;
    store.content = content;
    store.bump_original = bump;
    msg!(
        "New account stores the JS content with seed = [JS, {}, {}]", 
        main_account.js,
        main_account.key()
    );
    Ok(())
}

#[derive(Accounts)]
pub struct JsStore<'info> {
    #[account(
        mut,
        seeds = [
            &anchor_lang::solana_program::hash::hash(
            main_account.web_name.as_bytes()
        ).to_bytes()
        ],
        bump = main_account.bump_original,
    )]
    pub main_account: Account<'info, MainAccount>,
    #[account(init, seeds = 
        [
        b"JS", 
        main_account.js.to_le_bytes().as_ref(), 
        main_account.key().as_ref()
        ], 
    bump, payer = signer, 
    space = StoreAccount::SIZE + 8
    )]
    pub store: Account<'info, StoreAccount>,
    #[account(mut)]
    pub signer: Signer<'info>,
    pub system_program: Program<'info, System>,
}
```

## add_js()

```rust
use anchor_lang::{
    prelude::*,
    solana_program::pubkey::Pubkey
}; 
use crate::state::accounts::*;
use crate::error::ErrorCode;

pub fn add_js(
    ctx: Context<AddJs>,
    content: Vec<u8>
) -> Result<()> {
    require!(
        ctx.accounts.main_account.authority.key() == ctx.accounts.signer.key(), 
        ErrorCode::AuthorityError
    );
    require!(
        ctx.accounts.store.content.len() < 9900, 
        ErrorCode::Max9900
    );
    let main_account: &mut Account<MainAccount> = &mut ctx.accounts.main_account;
    let store: &mut Account<StoreAccount> = &mut ctx.accounts.store;
    store.content.extend(content);
    msg!("The content of the PDA was updated.");
    if store.content.len() == 9900 {
        main_account.js += 1;
    }
    Ok(())
}

#[derive(Accounts)]
pub struct AddJs<'info> {
    #[account(
        mut,
        seeds = [
            &anchor_lang::solana_program::hash::hash(
                main_account.web_name.as_bytes()
            ).to_bytes()
            ],
        bump = main_account.bump_original
    )]
    pub main_account: Account<'info, MainAccount>,
    #[account(
        mut, 
        seeds = [
            b"JS", 
            main_account.js.to_le_bytes().as_ref(), 
            main_account.key().as_ref()
        ], 
        bump = store.bump_original,
        realloc = 8 + 4 + 1 + store.content.len() + 900,
        realloc::payer = signer,
        realloc::zero = false,
    )]
    pub store: Account<'info, StoreAccount>,
    #[account(mut)]
    pub signer: Signer<'info>,
    pub system_program: Program<'info, System>,
}
```

## delete_html()

```rust
use anchor_lang::{
    prelude::*,
    solana_program::pubkey::Pubkey
}; 
use crate::state::accounts::*;
use crate::error::ErrorCode;

pub fn delete_html(
    ctx: Context<DeleteHtml>
) -> Result<()> {
    require!(ctx.accounts.main_account.authority.key() == ctx.accounts.signer.key(), ErrorCode::AuthorityError);
    let lamport: u64 = ctx.accounts.account.to_account_info().lamports() - 890880; 
    let main_account: &mut Account<MainAccount> = &mut ctx.accounts.main_account;
    main_account.html -= 1;
    **ctx.accounts.account.to_account_info().try_borrow_mut_lamports()? -= lamport;
    **ctx.accounts.signer.to_account_info().try_borrow_mut_lamports()? += lamport;
    msg!(
        "{} account removes HTML content and removes {} lamports from {} account",
        ctx.accounts.signer.key(),
        lamport,
        ctx.accounts.account.key()
    );
    Ok(())
}
#[derive(Accounts)]
pub struct DeleteHtml<'info> {
    #[account(
        mut,
        seeds = [
            &anchor_lang::solana_program::hash::hash(
                main_account.web_name.as_bytes()
            ).to_bytes()
        ],
        bump = main_account.bump_original
    )]
    pub main_account: Account<'info, MainAccount>,
    #[account(
        mut, 
        seeds = [
            b"HTML", 
            main_account.html.to_le_bytes().as_ref(), 
            main_account.key().as_ref()
        ],  
        bump = account.bump_original, 
        close = signer
    )]
    pub account: Account<'info, StoreAccount>,
    #[account(mut)]
    pub signer: Signer<'info>,
    pub system_program: Program<'info, System>,
}
```

## delete_js()

```rust
use anchor_lang::{
    prelude::*,
    solana_program::pubkey::Pubkey
}; 
use crate::state::accounts::*;
use crate::error::ErrorCode;

pub fn delete_js(
    ctx: Context<DeleteJs>
) -> Result<()> {
    require!(ctx.accounts.main_account.authority.key() == ctx.accounts.signer.key(), ErrorCode::AuthorityError);
    let lamport: u64 = ctx.accounts.account.to_account_info().lamports() - 890880; 
    let main_account: &mut Account<MainAccount> = &mut ctx.accounts.main_account;
    main_account.js -= 1;
    **ctx.accounts.account.to_account_info().try_borrow_mut_lamports()? -= lamport;
    **ctx.accounts.signer.to_account_info().try_borrow_mut_lamports()? += lamport;
    msg!(
        "{} account removes JS content and removes {} lamports from {} account",
        ctx.accounts.signer.key(),
        lamport,
        ctx.accounts.account.key()
    );
    Ok(())
}
#[derive(Accounts)]
pub struct DeleteJs<'info> {
    #[account(
        mut,
        seeds = [
            &anchor_lang::solana_program::hash::hash(
                main_account.web_name.as_bytes()
            ).to_bytes()
            ],
        bump = main_account.bump_original
    )]
    pub main_account: Account<'info, MainAccount>,
    #[account(
        mut, 
        seeds = [
            b"JS", 
            main_account.js.to_le_bytes().as_ref(),  
            main_account.key().as_ref()
        ],  
        bump = account.bump_original, 
        close = signer
    )]
    pub account: Account<'info, StoreAccount>,
    #[account(mut)]
    pub signer: Signer<'info>,
    pub system_program: Program<'info, System>,
}
```
