<div align="center">
  <img height="160" src="Decenwser.ico" />
  <h1 id="title">Decenwser program</h1>

---

![decenwser](Decenwser.gif)

---
</div>

Decenwser, a decentralized browser powered by blockchain technology. What makes Decenwser different from other browsers? Well, the answer is simple: Decenwser allows data transmission universally and without restrictions.

What does this mean exactly? It means that with Decenwser, there are no limitations as to who can access the information and where it can be accessed. Blockchain technology ensures that data is stored in a secure and decentralized manner, which means there are no intermediaries involved in data transmission. In addition, the use of blockchain also guarantees privacy and the protection of personal data.

This Smart Contract is a key piece of Decenwser, since it allows the data to be stored and protected in a decentralized way in the blockchain. Not only that, but we have also included in the Smart Contract the ability to store and protect the domains used in the browser. This means that information from websites and associated domains are also safe and secure in a decentralized way on the blockchain.

Below are the explanations of the functions of the program:

<h3 align="center">main_account()</h3>

```rust
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

The main_account function is a public function that creates a new main account on the Solana blockchain to store information about a specific website. The function takes the name of the website as input and uses the "find_program_address" function to generate a unique Public Program Address (PDA) for that account.

The function verifies that the length of the website name is less than or equal to 32 characters, and then initializes a new Solana account using the MainAccount structure defined in the smart contract.

The function also records the authority of the parent account and sets the number of bytes used to store HTML and JS in the account to zero. The function then prints a confirmation message indicating that the account has been created successfully.

The MainAccountStruct structure used in the function defines the parameters needed to create and access Solana's main account. The structure includes the main account main_account, the signatory signer and the system program system_program.

<h3 align="center">html_store()</h3>

```rust
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

This function is intended to store the HTML content in a specific record on the Solana blockchain.

The html_store function accepts two arguments: a context (ctx) containing information about the source account, destination account, and other related variables, and HTML content represented as a byte vector (content: Vec<u8>).

The code implements a validation using the require! to ensure that the account calling the function is the one authorized to do so. The Pubkey::find_program_address function is then used to obtain a public account address associated with the specific HTML content record being stored.

The html_store function defines the HtmlStore data structure using the #[derive(Accounts)] attribute. This data structure specifies the accounts to be used in the function, including the source account main_account, the destination store account, the signer account that provides the signature for the transaction, and the system program account system_program.

The target store account is initialized with the #[account(init)] function and is associated with a unique seed. This account is used to store the HTML content on the Solana blockchain.

<h3 align="center">add_html()</h3>

```rust
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

The add_html function is a function used to add HTML content to the Decenwser Smart Contract on the Solana blockchain. This function takes as parameters the execution context and the HTML content to be added.

First, it is verified that the sender of the transaction is the main contract authority. Then, it is verified that the length of the current content in the contract store is less than 9900. If these two conditions are met, the content of the contract store is updated with the new HTML content provided.

If the length of the stored content reaches 9900, the counter for the amount of HTML stored in the main contract is incremented. At the end, the function returns a result indicating whether or not the operation completed successfully.

The AddHtml structure is used to group all the accounts needed to run the add_html function. This structure includes the main contract account, the storage account, the signatory account, and the system program account.

<h3 align="center">js_store()</h3>

```rust
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

This function is intended to save the content of a JS script to a storage account on the Solana blockchain. The function receives as parameters a JsStore data structure that contains information on the accounts involved in the transaction, and a content byte vector that represents the content of the JS script to be stored.

The function uses an authority check to verify that the signer of the transaction has permission to perform the operation. A find_program_address function is then used to calculate the address of the storage account and get a unique index for the account.

The storage account data is then updated with the new content, and a confirmation message is logged to the console of the user calling the function.

Finally, the function returns an Ok(()) result if the operation completed without errors. The JsStore data structure contains a list of accounts involved in the transaction, such as the main account main_account, the storage account store, the signer account signer, and the system program system_program.

<h3 align="center">add_js()</h3>

```rust
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

The function of this is to allow the addition of JavaScript to content stored on the Solana blockchain.

The function takes two parameters: ctx and content. ctx is an object that represents the context of the function and contains information about the Solana accounts involved in the transaction. content is a byte vector representing the JavaScript content to add to storage.

Within the function, various security checks are performed. First, it is verified that the authority calling the function is the same as the authority set in the main Decenwser account. It is then verified that the total amount of data stored in the storage account does not exceed 9900 bytes.

If both checks are successful, the function proceeds to update the content stored in the storage account. If the storage account now contains exactly 9900 bytes, the parent account's script counter is incremented by 1.

Finally, the function returns a value of type Result that indicates whether the operation was successful or not. In short, the add_js function makes it possible to securely add JavaScript to Decenwser's decentralized storage on the Solana blockchain.

<h3 align="center">delete_html()</h3>

```rust
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

This delete_html function handles deleting HTML content and updating values ​​associated with the user's account. In summary, this function does the following:

- Verifies that the authority key executing the function is equal to the signer's key to ensure that only the authorized owner can perform this action.

- Calculates the number of Lamports (the unit of currency on the Solana blockchain) that should be subtracted from the user's account.

- Subtracts the Lamports from the user's account.

- Adds the subtracted Lamports from the user's account to the authorized signer's account.

- Updates the user's account status, subtracting one of the stored HTML values.

- Records an event log message showing the change made and the number of Lamports subtracted from the user's account and added to the authorized signer's account.

<h3 align="center">delete_js()</h3>

```rust
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

This function is used to remove JavaScript (JS) content stored in a specific account, reducing resource usage and storage space. The function also handles the transfer of funds in lamports (the unit of account on the Solana blockchain) from one account to another.

The function starts by checking that the main account's authority (main_account) is equal to the signer's key (signer), and if it is not, an error is thrown (ErrorCode::AuthorityError). Then, the amount of lamports to be transferred is calculated by subtracting the value 890880 from the current balance of the account provided in the context.

The function then updates the js value of the main account (main_account) and transfers the account's lamports to the signer's account by updating the lamports values ​​in both accounts.

Finally, a message (msg!) is sent informing that the JavaScript content has been removed and the lamports have been transferred. The function returns Ok(()) if it completes without errors.
