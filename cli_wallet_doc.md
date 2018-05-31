# Description #
    cli_wallet is a console wallet, which provides tools to work with GOLOS blockchain (Not API calls)

## How to start work with cli_wallet ##
    1. First of all you need `golosd` to be running
    2. Next go to `programs/cli_wallet` and run `./cli_wallet`
    3. If it is your first execution of `cli_wallet`, then you have to set password for your wallet.
    4. Run `set_password <your_password>` to set_password
    5. Now unlock the wallet with command `unlock <your_password>`
    6. Work with wallet.


## What's next? ##
    TODO Можно привести кучу примеров из репозитория [Qa](https://github.com/GolosChain/Qa/tree/master/TestCase)
    [Proposal тесты](https://github.com/GolosChain/Qa/blob/master/TestCase/ProposalTransactions/633-cli_wallet.md), которые хорошо подходят для объяснения как пользоваться proposal.
    [Optional Reputation](https://github.com/GolosChain/Qa/blob/master/TestCase/674-OptionalReputationField.md) --  как получать GBG.

## Non-interactive режим ##
    Cli_wallet предоставляет возможность при помощи ключа `--commands` можно передать последовательный список команд через `&&`. Например:
    ```
    ./cli_wallet --server-rpc-endpoint="ws://127.0.0.1:8091" --commands="unlock 1 && create_account cyberfounder ginger \"{}\" \"10.000 GOLOS\" true && list_my_accounts"
    ```
    Таким образом, на данный момент это единственный способ писать полноценные скрипты для работы с cli_wallet. 

# Methods description #

- `about` 
    Returns info such as client version, git version of graphene/fc, version of boost, openssl.
    ```
    /** Returns info such as client version, git version of graphene/fc, version of boost, openssl.
     * @returns compile time info and client and dependencies versions
     */
    variant_object                      about() const;
    ```

    Execution example:

    ```
    >>> about 
    {
      "client_version": "v0.17.2-188-geb6bc299",
      "steem_revision": "eb6bc2993b28e31b751c24c77e5081997e85dc8e",
      "steem_revision_age": "7 days ago",
      "fc_revision": "d9e526e6e7b8d39fa40adecbf4407fa6fd6366c1",
      "fc_revision_age": "28 days ago",
      "compile_date": "compiled on May 17 2018 at 09:40:32",
      "boost_version": "1.58",
      "openssl_version": "OpenSSL 1.0.2g  1 Mar 2016",
      "build": "linux 64-bit"
    }
    ```

- `add_operation_copy_to_builder_transaction`
    Копирует операцию из одного конструктора в другой. Идентифицируются `builder_transaction` (конструктор) по handler'у,
    который возвращает метод `begin_builder_transaction()`. 
    ```
    void add_operation_copy_to_builder_transaction(
    	transaction_handle_type src_handle, transaction_handle_type dst_handle, uint32_t op_index
    );

    ```

- `add_operation_to_builder_transaction` 
    Чтобы создать операцию (с помощью add_operation_to_builder_transaction) в данный момент,
    сначала нужно получить шаблон для этой операции, а потом его корректно заполнить.
    Да, это не идеальное решение, но всё же лучше, чем ничего.
    ```
    /** Returns an uninitialized object representing a given blockchain operation.
     *
     * This returns a default-initialized object of the given type; it can be used
     * during early development of the wallet when we don't yet have custom commands for
     * creating all of the operations the blockchain supports.
     *
     * Any operation the blockchain supports can be created using the transaction builder's
     * \c add_operation_to_builder_transaction() , but to do that from the CLI you need to
     * know what the JSON form of the operation looks like.  This will give you a template
     * you can fill in.  It's better than nothing.
     *
     * @param operation_type the type of operation to return, must be one of the
     *                       operations defined in `steem/chain/operations.hpp`
     *                       (e.g., "global_parameters_update_operation")
     * @return a default-constructed operation of the given type
     */
    operation get_prototype_operation(string operation_type);

    ```
    Пример:
    ```
    get_prototype_operation account_metadata_operation
    ```

- `approve_proposal`
    Валидирует и подписывает proposal транзакцию
    ```
    /**
     * Approve or disapprove a proposal.
     *
     * @param author The author of the proposal
     * @param title The proposal to modify.
     * @param delta Members contain approvals to create or remove.  In JSON you can leave empty members undefined.
     * @param broadcast true if you wish to broadcast the transaction
     * @return the signed version of the transaction
     */
    signed_transaction approve_proposal(
        const std::string& author,
        const std::string& title,
        const approval_delta& delta,
        bool broadcast /* = false */
    );

    ```
    ```
    approve_proposal alice test {"active_approvals_to_add":["alice"]} true
    ```

- `begin_builder_transaction`

    Создаёт новый конструктор транзакций. Возвращает его идентификатор. 
    ```
    transaction_handle_type begin_builder_transaction();
    ```

    Пример запуска:

    ```
    begin_builder_transaction 
    2

    ```

- `cancel_order`
    ```
    /**
     * Cancel an order created with create_order
     *
     * @param owner The name of the account owning the order to cancel_order
     * @param orderid The unique identifier assigned to the order by its creator
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction cancel_order(string owner, uint32_t orderid, bool broadcast);
    ```
    #TODO добавить пример запуска 

- `cancel_transfer_from_savings`
    ```
    /**
     *  @param from the account that initiated the transfer
     *  @param request_id the id used in transfer_from_savings
     *  @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction cancel_transfer_from_savings( string from, uint32_t request_id, bool broadcast = false );

    ```

    - `change_recovery_account`
    ```
    /**
     * Change your recovery account after a 30 day delay.
     *
     * @param owner The name of your account
     * @param new_recovery_account The name of the recovery account you wish to have
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction change_recovery_account( string owner, string new_recovery_account, bool broadcast );

    ```

- `convert_sbd`
    ```
    /**
     *  This method will convert SBD to STEEM at the current_median_history price one
     *  week from the time it is executed. This method depends upon there being a valid price feed.
     *
     *  @param from The account requesting conversion of its SBD i.e. "1.000 SBD"
     *  @param amount The amount of SBD to convert
     *  @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction convert_sbd( string from, asset amount, bool broadcast = false );

    ```

- `create_account`
    This method will generate new owner, active, posting and memo keys for the new account
    which will be controlable by this wallet.
    Аргументы: имя_акк_создателя, имя_нового_аккаунта, json_meta, fee (в формате asset'а), true\false (broadcast or not)
    ```
    create_account cyberfounder alice "{}" "3.000 GOLOS" true
    ```


- `create_account_delegated`
    ```
    /**
     *  This method will genrate new owner, active, posting and memo keys for the new account which
     *  will be controlable by this wallet. There is a fee associated with account creation
     *  that is paid by the creator. The current account creation fee can be found with the
     *  'info' wallet command.
     *
     *  These accounts are created with combination of GOLOS and delegated GP
     *
     *  @param creator The account creating the new account
     *  @param steem_fee The amount of the fee to be paid with GOLOS
     *  @param delegated_vests The amount of the fee to be paid with delegation
     *  @param new_account_name The name of the new account
     *  @param json_meta JSON Metadata associated with the new account
     *  @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction create_account_delegated(
        string creator, asset steem_fee, asset delegated_vests, string new_account_name, string json_meta, bool broadcast);
    ```
    Пример:
    ```
    create_account_delegated cyberfounder "3.000 GOLOS" "0.100000 GESTS" jim "{}" true
    ```

- `create_account_with_keys`
    ```
    /**
     * This method is used by faucets to create new accounts for other users which must
     * provide their desired keys. The resulting account may not be controllable by this
     * wallet. There is a fee associated with account creation that is paid by the creator.
     * The current account creation fee can be found with the 'info' wallet command.
     *
     * @param creator The account creating the new account
     * @param newname The name of the new account
     * @param json_meta JSON Metadata associated with the new account
     * @param fee The amount of the fee to be paid with GOLOS
     * @param owner public owner key of the new account
     * @param active public active key of the new account
     * @param posting public posting key of the new account
     * @param memo public memo key of the new account
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction create_account_with_keys(
        string creator,
        string newname,
        string json_meta,
        asset fee,
        public_key_type owner,
        public_key_type active,
        public_key_type posting,
        public_key_type memo,
        bool broadcast )const;
    ```

- `create_account_with_keys_delegated`
    ```
    /**
     * This method is used by faucets to create new accounts for other users which must
     * provide their desired keys. The resulting account may not be controllable by this
     * wallet. There is a fee associated with account creation that is paid by the creator.
     * The current account creation fee can be found with the 'info' wallet command.
     *
     * These accounts are created with combination of GOLOS and delegated GP
     *
     * @param creator The account creating the new account
     * @param steem_fee The amount of the fee to be paid with GOLOS
     * @param delegated_vests The amount of the fee to be paid with delegation
     * @param newname The name of the new account
     * @param json_meta JSON Metadata associated with the new account
     * @param owner public owner key of the new account
     * @param active public active key of the new account
     * @param posting public posting key of the new account
     * @param memo public memo key of the new account
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction create_account_with_keys_delegated(
        string creator,
        asset steem_fee,
        asset delegated_vests,
        string newname,
        string json_meta,
        public_key_type owner,
        public_key_type active,
        public_key_type posting,
        public_key_type memo,
        bool broadcast) const;

    ```

- `create_order`
    ```
    /**
     *  Creates a limit order at the price amount_to_sell / min_to_receive and will deduct amount_to_sell from account
     *
     *  @param owner The name of the account creating the order
     *  @param order_id is a unique identifier assigned by the creator of the order, it can be reused after the order has been filled
     *  @param amount_to_sell The amount of either SBD or STEEM you wish to sell
     *  @param min_to_receive The amount of the other asset you will receive at a minimum
     *  @param fill_or_kill true if you want the order to be killed if it cannot immediately be filled
     *  @param expiration the time the order should expire if it has not been filled
     *  @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction create_order( string owner, uint32_t order_id, asset amount_to_sell, asset min_to_receive, bool fill_or_kill, uint32_t expiration, bool broadcast );
    ```

- `database_info`
    Возвращает информацию оь объектах базы.

- `decline_voting_rights`
    Аккаунт с именем `account` будет иметь или не иметь права на голосование в зависимости от значения `decline`. 
    ```
    annotated_signed_transaction decline_voting_rights( string account, bool decline, bool broadcast );
    ```

- `decrypt_memo`
    ```
    /**
     * Returns the decrypted memo if possible given wallet's known private keys
     */
    string decrypt_memo( string memo );
    ```

- `delegate_vesting_shares`
    ```
    /**
     * This method delegates GESTS from one account to another.
     *
     * @param delegator The name of the account delegating GESTS
     * @param delegatee The name of the account receiving GESTS
     * @param vesting_shares The amount of GESTS to delegate
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction delegate_vesting_shares(string delegator, string delegatee, asset vesting_shares, bool broadcast);
    ```
Пример:
    ```
    delegate_vesting_shares alice bob "1026635.005622 GESTS" true
    ```

- `escrow_approve`
    ```
    /**
     * Approve a proposed escrow transfer. Funds cannot be released until after approval. This is in lieu of requiring
     * multi-sig on escrow_transfer
     *
     * @param from The account that funded the escrow
     * @param to The destination of the escrow
     * @param agent The account acting as the agent in case of dispute
     * @param who The account approving the escrow transfer (either 'to' or 'agent)
     * @param escrow_id A unique id for the escrow transfer
     * @param approve true to approve the escrow transfer, otherwise cancels it and refunds 'from'
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction escrow_approve(
            string from,
            string to,
            string agent,
            string who,
            uint32_t escrow_id,
            bool approve,
            bool broadcast = false
    );

    ```

- `escrow_dispute`
    ```
    /**
     * Raise a dispute on the escrow transfer before it expires
     *
     * @param from The account that funded the escrow
     * @param to The destination of the escrow
     * @param agent The account acting as the agent in case of dispute
     * @param who The account raising the dispute (either 'from' or 'to')
     * @param escrow_id A unique id for the escrow transfer
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction escrow_dispute(
            string from,
            string to,
            string agent,
            string who,
            uint32_t escrow_id,
            bool broadcast = false
    );
    ```

- `escrow_release`
    ```
    /**
     * Release funds help in escrow
     *
     * @param from The account that funded the escrow
     * @param to The account the funds are originally going to
     * @param agent The account acting as the agent in case of dispute
     * @param who The account authorizing the release
     * @param receiver The account that will receive funds being released
     * @param escrow_id A unique id for the escrow transfer
     * @param sbd_amount The amount of SBD that will be released
     * @param steem_amount The amount of STEEM that will be released
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction escrow_release(
            string from,
            string to,
            string agent,
            string who,
            string receiver,
            uint32_t escrow_id,
            asset sbd_amount,
            asset steem_amount,
            bool broadcast = false
    );
    ```
    Пример:
    ```
    escrow_release cyberfounder bob smith cyberfounder bob 11 "1.000 GBG" "10.000 GOLOS" true
    ```

- `escrow_transfer`

    ```
    /**
     * Transfer funds from one account to another using escrow. STEEM and SBD can be transferred.
     *
     * @param from The account the funds are coming from
     * @param to The account the funds are going to
     * @param agent The account acting as the agent in case of dispute
     * @param escrow_id A unique id for the escrow transfer. (from, escrow_id) must be a unique pair
     * @param sbd_amount The amount of SBD to transfer
     * @param steem_amount The amount of STEEM to transfer
     * @param fee The fee paid to the agent
     * @param ratification_deadline The deadline for 'to' and 'agent' to approve the escrow transfer
     * @param escrow_expiration The expiration of the escrow transfer, after which either party can claim the funds
     * @param json_meta JSON encoded meta data
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction escrow_transfer(
            string from,
            string to,
            string agent,
            uint32_t escrow_id,
            asset sbd_amount,
            asset steem_amount,
            asset fee,
            time_point_sec ratification_deadline,
            time_point_sec escrow_expiration,
            string json_meta,
            bool broadcast = false
    );
    ```
    Пример:
    ```
    escrow_transfer cyberfounder bob smith 11 "1.000 GBG" "10.020 GOLOS" "10.000 GOLOS" "2018-05-31T10:40:59" "2018-05-31T11:00:00" "{}" true
    ```

- `get_account`
    По имени аккаунта возвращает всю информацию по этому аккаунту
    Пример запуска:
     ```
    >>> get_account cyberfounder

    {
      "id": 3,
      "name": "cyberfounder",
      "owner": {
        "weight_threshold": 1,
        "account_auths": [],
        "key_auths": [[
            "GLS58g5rWYS3XFTuGDSxLVwiBiPLoAyCZgn6aB9Ueh8Hj5qwQA3r6",
            1
          ]
        ]
      },
      "active": {
        "weight_threshold": 1,
        "account_auths": [],
        "key_auths": [[
            "GLS58g5rWYS3XFTuGDSxLVwiBiPLoAyCZgn6aB9Ueh8Hj5qwQA3r6",
            1
          ]
        ]
      },
      "posting": {
        "weight_threshold": 1,
        "account_auths": [],
        "key_auths": [[
            "GLS58g5rWYS3XFTuGDSxLVwiBiPLoAyCZgn6aB9Ueh8Hj5qwQA3r6",
            1
          ]
        ]
      },
      "memo_key": "GLS58g5rWYS3XFTuGDSxLVwiBiPLoAyCZgn6aB9Ueh8Hj5qwQA3r6",
      "json_metadata": "",
      "proxy": "",
      "last_owner_update": "1970-01-01T00:00:00",
      "last_account_update": "1970-01-01T00:00:00",
      "created": "1970-01-01T00:00:00",
      "mined": true,
      "owner_challenged": false,
      "active_challenged": false,
      "last_owner_proved": "1970-01-01T00:00:00",
      "last_active_proved": "1970-01-01T00:00:00",
      "recovery_account": "",
      "last_account_recovery": "1970-01-01T00:00:00",
      "reset_account": "null",
      "comment_count": 0,
      "lifetime_vote_count": 0,
      "post_count": 0,
      "can_vote": true,
      "voting_power": 10000,
      "last_vote_time": "1970-01-01T00:00:00",
      "balance": "43305009.000 GOLOS",
      "savings_balance": "0.000 GOLOS",
      "sbd_balance": "0.000 GBG",
      "sbd_seconds": "0",
      "sbd_seconds_last_update": "1970-01-01T00:00:00",
      "sbd_last_interest_payment": "1970-01-01T00:00:00",
      "savings_sbd_balance": "0.000 GBG",
      "savings_sbd_seconds": "0",
      "savings_sbd_seconds_last_update": "1970-01-01T00:00:00",
      "savings_sbd_last_interest_payment": "1970-01-01T00:00:00",
      "savings_withdraw_requests": 0,
      "vesting_shares": "2455935.529482 GESTS",
      "delegated_vesting_shares": "0.000000 GESTS",
      "received_vesting_shares": "0.000000 GESTS",
      "vesting_withdraw_rate": "0.000001 GESTS",
      "next_vesting_withdrawal": "1969-12-31T23:59:59",
      "withdrawn": 0,
      "to_withdraw": 0,
      "withdraw_routes": 0,
      "curation_rewards": 0,
      "posting_rewards": 0,
      "proxied_vsf_votes": [
        0,
        0,
        0,
        0
      ],
      "witnesses_voted_for": 0,
      "average_bandwidth": 148070876,
      "lifetime_bandwidth": 2512000000,
      "last_bandwidth_update": "2018-05-17T06:59:57",
      "average_market_bandwidth": 113999960,
      "last_market_bandwidth_update": "2018-05-17T06:59:57",
      "last_post": "1970-01-01T00:00:00",
      "last_root_post": "1970-01-01T00:00:00",
      "post_bandwidth": 10000,
      "new_average_bandwidth": 2510708380,
      "new_average_market_bandwidth": 3399615482,
      "witness_votes": []
    }
    ```

- `get_account_history`
    ```
    /**
     *  Account operations have sequence numbers from 0 to N where N is the most recent operation. This method
     *  returns operations in the range [from-limit, from]
     *
     *  @param account - account whose history will be returned
     *  @param from - the absolute sequence number, -1 means most recent, limit is the number of operations before from.
     *  @param limit - the maximum number of items that can be queried (0 to 1000], must be less than from
     */
    map< uint32_t, golos::plugins::operation_history::applied_operation >
        get_account_history( string account, uint32_t from, uint32_t limit );
    ```
    Пример:
    ```
    get_account_history cyberfoudner -1 10
    get_account_history cyberfoudner -1 10
    2104186ms th_0       websocket_api.cpp:86          on_message           ] message: {"jsonrpc":"2.0","result":[],"id":1} 
    #     BLOCK #    TRX ID          OPERATION            DETAILS                                           
    -------------------------------------------------------------------------------
    ```

- `get_active_witnesses`
    Возвращает список активных делегатов (при недостаточном количестве выводятся все, что есть -- меньше 21)
    ```
    /**
     * Returns the list of witnesses producing blocks in the current round (21 Blocks)
     */
    vector< account_name_type > get_active_witnesses()const;
    ```

- `get_block`
    По номеру блока возращает некоторую информацию о нём.
    Пример:
    ```
    unlocked >>> get_block 500
    get_block 500
    {
      "previous": "000001f3bc47032de379e4645ad7fef36ef0f98c",
      "timestamp": "2018-05-24T14:18:42",
      "witness": "cyberfounder",
      "transaction_merkle_root": "0000000000000000000000000000000000000000",
      "extensions": [],
      "witness_signature": "1f3ab190e32b5e6db73279ecd09a6fb140fb029497801fa67963701b25f6acb752711e1c90758a1c582d9e418190d9db228ecfe4713829bfa9682ca3e55124a6da",
      "transactions": [],
      "block_id": "000001f42b036c326e585e1918319985bf82c125",
      "signing_key": "GLS58g5rWYS3XFTuGDSxLVwiBiPLoAyCZgn6aB9Ueh8Hj5qwQA3r6",
      "transaction_ids": []
    }
    ```

- `get_conversion_requests`
    ```
    /** Returns conversion requests by an account
     *
     * @param owner Account name of the account owning the requests
     *
     * @returns All pending conversion requests by account
     */
    vector< database_api::convert_request_api_object > get_conversion_requests( string owner );

    ```

- `get_encrypted_memo`
    ```
    /**
     *  Returns the encrypted memo if memo starts with '#' otherwise returns memo
     */
    string get_encrypted_memo( string from, string to, string memo );

    ```

- `get_feed_history`
    ```
    /** Return the current price feed history
     *
     * @returns Price feed history data on the blockchain
     */
    witness_api::feed_history_api_object get_feed_history()const;
    ```

- `get_inbox`

- `get_miner_queue`
    Returns the queue of pow miners waiting to produce blocks

- `get_open_orders`
    Возвращает открытые ордеры по имени аккаунта 
    ```
    vector< database_api::extended_limit_order > get_open_orders( string accountname );
    ```

- `get_ops_in_block`
    ```
                /** Returns sequence of operations included/generated in a specified block
                 *
                 * @param block_num Block height of specified block
                 * @param only_virtual Whether to only return virtual operations
                 */
                vector<golos::plugins::operation_history::applied_operation> get_ops_in_block( uint32_t block_num, bool only_virtual = true );
    ```

- `get_order_book`
    ```
    /**
     * Gets the current order book for STEEM:SBD
     *
     * @param limit Maximum number of orders to return for bids and asks. Max is 1000.
     */
    market_history::order_book get_order_book( uint32_t limit );
    ```

- `get_outbox`

- `get_owner_history`
    ```
    vector< database_api::owner_authority_history_api_object > get_owner_history( string account )const;
    ```

- `get_private_key`
    Возращает по публичному ключу приватный, если он был ранее добавлен в кошелёк.
    ```
    /**
     * Get the WIF private key corresponding to a public key.  The
     * private key must already be in the wallet.
     */
    string                            get_private_key( public_key_type pubkey )const;
    ```

- `get_private_key_from_password`
    ```
    /**
     *  @param account  - the name of the account to retrieve key for
     *  @param role     - active | owner | posting | memo
     *  @param password - the password to be used at key generation
     *  @return public key corresponding to generated private key, and private key in WIF format.
     */
    pair<public_key_type,string>  get_private_key_from_password( string account, string role, string password )const;
    ```

- `get_proposed_transactions`
    ```
    /**
     * Returns proposals for the account
     */
     std::vector<database_api::proposal_api_object> get_proposed_transactions(
         std::string account, uint32_t from, uint32_t limit
    );
    ```
    Пример:
    ```
    get_proposed_transactions alice 0 10
    ```

- `get_prototype_operation`
    Чтобы создать операцию (с помощью add_operation_to_builder_transaction) в данный момент,
    сначала нужно получить шаблон для этой операции, а потом его корректно заполнить.
    Да, это не идеальное решение, но всё же лучше, чем ничего.
    ```
    /** Returns an uninitialized object representing a given blockchain operation.
     *
     * This returns a default-initialized object of the given type; it can be used
     * during early development of the wallet when we don't yet have custom commands for
     * creating all of the operations the blockchain supports.
     *
     * Any operation the blockchain supports can be created using the transaction builder's
     * \c add_operation_to_builder_transaction() , but to do that from the CLI you need to
     * know what the JSON form of the operation looks like.  This will give you a template
     * you can fill in.  It's better than nothing.
     *
     * @param operation_type the type of operation to return, must be one of the
     *                       operations defined in `steem/chain/operations.hpp`
     *                       (e.g., "global_parameters_update_operation")
     * @return a default-constructed operation of the given type
     */
    operation get_prototype_operation(string operation_type);

    ```
    Пример:
    ```
    add_operation_to_builder_transaction 0 ["account_metadata_operation", {"account":"profile", "json_metadata":"{\"name\":\"builder\"}"}]
    ```

- `get_transaction`
    ```
    /**
     * Returns transaction by ID.
     */
    annotated_signed_transaction get_transaction( transaction_id_type trx_id )const;
    ```

- `get_withdraw_routes`
    ```
    /**
     * Returns vesting withdraw routes for an account.
     *
     * @param account Account to query routes
     * @param type Withdraw type type [incoming, outgoing, all]
     */
    vector< database_api::withdraw_vesting_route_api_object > get_withdraw_routes( string account, database_api::withdraw_route_type type = database_api::all )const;
    ```

- `get_witness`
    Возвращает информацию о переданном в качестве аргумента делегате.
    ```
    /** Returns information about the given witness.
     * @param owner_account the name or id of the witness account owner, or the id of the witness
     * @returns the information about the witness stored in the block chain
     */
    optional< witness_api::witness_api_object > get_witness(string owner_account);
    ```

- `gethelp`
    Должен возвращать справку по методу, но по факту она всегда пустая. Не работает.

- `help`
    Возвращает список всех доступных методов.

- `import_key`
    Импортирует приватный ключ, чтобы дать возможность проводить транзкации.
    Пример:
    `>>> import_key <your_private_key>`
    Например в тестнете это можно вытащить при запуске golosd. См initminer private key
    ```
    ------------------------------------------------------

                STARTING TEST NETWORK

    ------------------------------------------------------
    initminer public key: xxx
    initminer private key: xxx
    chain id: xxx
    blockchain version: 0.18.0
    ------------------------------------------------------
    ```

- `info`
    Returns info about the current state of the blockchain.
    Пример запуска:
    ```
    {
      "id": 0,
      "head_block_number": 522,
      "head_block_id": "0000020a10c9aa73f1ca9de6aac4b7e712b9dd47",
      "time": "2018-05-24T14:19:48",
      "current_witness": "cyberfounder",
      "total_pow": 10,
      "num_pow_witnesses": 1,
      "virtual_supply": "43306574.009 GOLOS",
      "current_supply": "43306574.009 GOLOS",
      "confidential_supply": "0.000 GOLOS",
      "current_sbd_supply": "0.000 GBG",
      "confidential_sbd_supply": "0.000 GBG",
      "total_vesting_fund_steem": "153.810 GOLOS",
      "total_vesting_shares": "362505.491356 GESTS",
      "total_reward_fund_steem": "212.199 GOLOS",
      "total_reward_shares2": "0",
      "sbd_interest_rate": 1000,
      "sbd_print_rate": 10000,
      "average_block_size": 34,
      "maximum_block_size": 131072,
      "current_aslot": 16794396,
      "recent_slots_filled": "340282366881324382206242438636069978111",
      "participation_count": 63,
      "last_irreversible_block_num": 522,
      "max_virtual_bandwidth": "713451110400000000",
      "current_reserve_ratio": 27,
      "vote_regeneration_per_day": 40,
      "witness_majority_version": "0.18.0",
      "hardfork_version": "0.18.0",
      "head_block_num": 522,
      "head_block_age": "0 second old",
      "participation": "49.21875000000000000",
      "median_sbd_price": {
        "base": "0.000 GOLOS",
        "quote": "0.000 GOLOS"
      },
      "account_creation_fee": "0.000 GOLOS"
    }
    ```

- `is_locked`
    Возвращает true или false в зависимости от того заблокирован кошелёк или нет.

- `is_new`
    Возвращает true или false в зависимости от того новый кошелёк или нет.

- `list_accounts`
    ```
    /** Lists all accounts registered in the blockchain.
     * This returns a list of all account names and their account ids, sorted by account name.
     *
     * Use the \c lowerbound and limit parameters to page through the list.  To retrieve all accounts,
     * start by setting \c lowerbound to the empty string \c "", and then each iteration, pass
     * the last account name returned as the \c lowerbound for the next \c list_accounts() call.
     *
     * @param lowerbound the name of the first account to return.  If the named account does not exist,
     *                   the list will start at the account that comes after \c lowerbound
     * @param limit the maximum number of accounts to return (max: 1000)
     * @returns a list of accounts mapping account names to account ids
     */
    vector< account_name_type > list_accounts(const string& lowerbound, uint32_t limit);
    ```

- `list_keys`
    ```
    /** Dumps all private keys owned by the wallet.
     *
     * The keys are printed in WIF format.  You can import these keys into another wallet
     * using \c import_key()
     * @returns a map containing the private keys, indexed by their public key
     */
    map<public_key_type, string> list_keys();
    ```

- `list_my_accounts`
    ```
    /**
     *  Gets the account information for all accounts for which this wallet has a private key
     */
    vector< golos::api::account_api_object > list_my_accounts();
    ```
    Пример:
    ```
    alice                   0.000 GOLOS        986361.909527 GESTS        0.000 GBG
    bob                     0.000 GOLOS        985906.905222 GESTS        0.000 GBG
    cyberfounder     43305208.000 GOLOS        522742.463893 GESTS        0.000 GBG
    -------------------------------------------------------------------------
    TOTAL            43305208.000 GOLOS       2495011.278642 GESTS        0.000 GBG
    ```

- `list_witnesses`
    ```
    /** Lists all witnesses registered in the blockchain.
     * This returns a list of all account names that own witnesses, and the associated witness id,
     * sorted by name.  This lists witnesses whether they are currently voted in or not.
     *
     * Use the \c lowerbound and limit parameters to page through the list.  To retrieve all witnesss,
     * start by setting \c lowerbound to the empty string \c "", and then each iteration, pass
     * the last witness name returned as the \c lowerbound for the next \c list_witnesss() call.
     *
     * @param lowerbound the name of the first witness to return.  If the named witness does not exist,
     *                   the list will start at the witness that comes after \c lowerbound
     * @param limit the maximum number of witnesss to return (max: 1000)
     * @returns a list of witnesss mapping witness names to witness ids
     */
    vector< account_name_type > list_witnesses(const string& lowerbound, uint32_t limit);
    ```

- `load_wallet_file`
    ```
    /** Loads a specified Graphene wallet.
     *
     * The current wallet is closed before the new wallet is loaded.
     *
     * @warning This does not change the filename that will be used for future
     * wallet writes, so this may cause you to overwrite your original
     * wallet unless you also call \c set_wallet_filename()
     *
     * @param wallet_filename the filename of the wallet JSON file to load.
     *                        If \c wallet_filename is empty, it reloads the
     *                        existing wallet file
     * @returns true if the specified wallet is loaded
     */
    bool    load_wallet_file(string wallet_filename = "");
    ```

- `lock`
    Блокирует кошелёк. Потом требуется снова вводить пароль для разблокировки и последующей работы. 

- `normalize_brain_key`
    ```
    /** Transforms a brain key to reduce the chance of errors when re-entering the key from memory.
     *
     * This takes a user-supplied brain key and normalizes it into the form used
     * for generating private keys.  In particular, this upper-cases all ASCII characters
     * and collapses multiple spaces into one.
     * @param s the brain key as supplied by the user
     * @returns the brain key in its normalized form
     */
    string normalize_brain_key(string s) const;
    ```

- `post_comment`
    Один из самый важных методов. С его помощью можно создавать посты и коментарии.
    С точки зрения ноды почти нет разницы между постом и коментарием. Посмотрим на параметры:
    ```
    /**
     *  Post or update a comment.
     *
     *  @param author the name of the account authoring the comment
     *  @param permlink the accountwide unique permlink for the comment
     *  @param parent_author can be null if this is a top level comment
     *  @param parent_permlink becomes category if parent_author is ""
     *  @param title the title of the comment
     *  @param body the body of the comment
     *  @param json the json metadata of the comment
     *  @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction post_comment( string author, string permlink, string parent_author, string parent_permlink, string title, string body, string json, bool broadcast );
    ```
    Видим, что для того, чтобы создать пост не нужно заполнять `parent_author`, если `parent_permlink` пустой, то создаётся категория 
    ```
    post_comment alice test "" ptest "Test post!" "<h1>title!</h1><p>hello!</p><hr>test..." "{}" true
    ```

    А чтобы создать коментарий к посту, нужно передать автора поста (`parent_author`) и  `parent_permlink`
    ```
    post_comment bob "re-test" alice test "comment title" "comment body" "{}" true
    ```

- `preview_builder_transaction`
    Нужно для того, чтобы по идентификатору получить builder_transaction из списка builder_transactions
    ```
    transaction preview_builder_transaction(transaction_handle_type handle);

    ```
    Пример:
    ```
    preview_builder_transaction 0
    ```

- `propose_builder_transaction`
    Создаёт конструктор для proposal транзакций. 
    Аргументы: 
    	- handle от обычного `builder_transaction`
    	- автор proposal транзакции
    	- название
    	- memo data
    	- время, когда транзакция протухнет, если не будет одобрена
    	- время на подпись

    ```
    signed_transaction propose_builder_transaction(
        transaction_handle_type handle,
        std::string author,
        std::string title,
        std::string memo,
        time_point_sec expiration = time_point::now() + fc::minutes(1),
        time_point_sec review_period_time = time_point::min(),
        bool broadcast = true
    );
    ```
    Пример:
    ```
    propose_builder_transaction 0 alice test "memo" "2018-05-22T22:15:00" "1970-01-01T00:00:00" true

    ```

- `publish_feed`
    В описании ниже стоит заменить STEEM на GOLOS, а SBD на GBG
    ```
    /**
     * A witness can public a price feed for the STEEM:SBD market. The median price feed is used
     * to process conversion requests from SBD to STEEM.
     *
     * @param witness The witness publishing the price feed
     * @param exchange_rate The desired exchange rate
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction publish_feed(string witness, price exchange_rate, bool broadcast );
    ```

    Пример:
    ```
    publish_feed cyberfounder {"quote":"1.000 GOLOS", "base":"2.000 GBG"} true
    ```

- `quit`
    Выходит из кошелька

- `recover_account`

- `remove_builder_transaction`
    Удаляет экземпляр `builder_transaction` по его идентификатору
    ```
    void remove_builder_transaction(transaction_handle_type handle);

    ```

- `replace_operation_in_builder_transaction`
    Заменяет операцию номер `op_index` в `builder_transaction` (конструктор) на операцию `op`.
    ```
    void replace_operation_in_builder_transaction(
        transaction_handle_type handle, unsigned op_index, const operation& op
    );

    ```

- `request_account_recovery`
    ```
    /**
     * Create an account recovery request as a recover account. The syntax for this command contains a serialized authority object
     * so there is an example below on how to pass in the authority.
     *
     * request_account_recovery "your_account" "account_to_recover" {"weight_threshold": 1,"account_auths": [], "key_auths": [["new_public_key",1]]} true
     *
     * @param recovery_account The name of your account
     * @param account_to_recover The name of the account you are trying to recover
     * @param new_authority The new owner authority for the recovered account. This should be given to you by the holder of the compromised or lost account.
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction request_account_recovery( string recovery_account, string account_to_recover, authority new_authority, bool broadcast );
    ```

- `save_wallet_file`
    ```
    /** Saves the current wallet to the given filename.
     *
     * @warning This does not change the wallet filename that will be used for future
     * writes, so think of this function as 'Save a Copy As...' instead of
     * 'Save As...'.  Use \c set_wallet_filename() to make the filename
     * persist.
     * @param wallet_filename the filename of the new wallet JSON file to create
     *                        or overwrite.  If \c wallet_filename is empty,
     *                        save to the current filename.
     */
    void    save_wallet_file(string wallet_filename = "");
    ```

- `serialize_transaction`
    ```
    /** Converts a signed_transaction in JSON form to its binary representation.
     *
     * TODO: I don't see a broadcast_transaction() function, do we need one?
     *
     * @param tx the transaction to serialize
     * @returns the binary form of the transaction.  It will not be hex encoded,
     *          this returns a raw string that may have null characters embedded
     *          in it
     */
    string serialize_transaction(signed_transaction tx) const;
    ```

- `set_password`
    Cтавит пароль на кошелёк.
    Пример:
    ```
    >>> set_password <password>
    ```

- `set_transaction_expiration`
    ```
    /**
     * Sets the amount of time in the future until a transaction expires.
     */
    void set_transaction_expiration(uint32_t seconds);
    ```

- `set_voting_proxy`
    ```
    /** Set the voting proxy for an account.
     *
     * If a user does not wish to take an active part in voting, they can choose
     * to allow another account to vote their stake.
     *
     * Setting a vote proxy does not remove your previous votes from the blockchain,
     * they remain there but are ignored.  If you later null out your vote proxy,
     * your previous votes will take effect again.
     *
     * This setting can be changed at any time.
     *
     * @param account_to_modify the name or id of the account to update
     * @param proxy the name of account that should proxy to, or empty string to have no proxy
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction set_voting_proxy(string account_to_modify,
                                                  string proxy,
                                                  bool broadcast = false);
    ```

- `set_withdraw_vesting_route`
    ```
    /**
     * Set up a vesting withdraw route. When vesting shares are withdrawn, they will be routed to these accounts
     * based on the specified weights.
     *
     * @param from The account the VESTS are withdrawn from.
     * @param to   The account receiving either VESTS or STEEM.
     * @param percent The percent of the withdraw to go to the 'to' account. This is denoted in hundreths of a percent.
     *    i.e. 100 is 1% and 10000 is 100%. This value must be between 1 and 100000
     * @param auto_vest Set to true if the from account should receive the VESTS as VESTS, or false if it should receive
     *    them as STEEM.
     * @param broadcast true if you wish to broadcast the transaction.
     */
    annotated_signed_transaction set_withdraw_vesting_route( string from, string to, uint16_t percent, bool auto_vest, bool broadcast = false );

    ```

- `sign_builder_transaction`
    Подписывает все транзакции в builder_transaction.
    ```
    signed_transaction sign_builder_transaction(transaction_handle_type handle, bool broadcast = true);

    ```
    Пример:
    ```
    sign_builder_transaction 0 true
    ```

- `sign_transaction`
    ```
    /** Signs a transaction.
     *
     * Given a fully-formed transaction that is only lacking signatures, this signs
     * the transaction with the necessary keys and optionally broadcasts the transaction
     * @param tx the unsigned transaction
     * @param broadcast true if you wish to broadcast the transaction
     * @return the signed version of the transaction
     */
    annotated_signed_transaction sign_transaction(signed_transaction tx, bool broadcast = false);

    ```

- `suggest_brain_key`
    ```
    /** Suggests a safe brain key to use for creating your account.
     * \c create_account_with_brain_key() requires you to specify a 'brain key',
     * a long passphrase that provides enough entropy to generate cyrptographic
     * keys.  This function will suggest a suitably random string that should
     * be easy to write down (and, with effort, memorize).
     * @returns a suggested brain_key
     */
    brain_key_info suggest_brain_key()const;
    ```

- `transfer`
    ```
    /**
     * Transfer funds from one account to another. STEEM and SBD can be transferred.
     *
     * @param from The account the funds are coming from
     * @param to The account the funds are going to
     * @param amount The funds being transferred. i.e. "100.000 STEEM"
     * @param memo A memo for the transactionm, encrypted with the to account's public memo key
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction transfer(string from, string to, asset amount, string memo, bool broadcast = false);
    ```
    Пример:
    ```
    transfer cyberfounder alice "3000.000 GOLOS" true
    ```

- `transfer_from_savings`
    ```
    /**
     *  @param from       - the account that initiated the transfer
     *  @param request_id - an unique ID assigned by from account, the id is used to cancel the operation and can be reused after the transfer completes
     *  @param to         - the account getting the transfer
     *  @param amount     - the amount of assets to be transfered
     *  @param memo A memo for the transactionm, encrypted with the to account's public memo key
     *  @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction transfer_from_savings( string from, uint32_t request_id, string to, asset amount, string memo, bool broadcast = false );
    ```

- `transfer_to_savings`
    ```
    /**
     *  Transfers into savings happen immediately, transfers from savings take 72 hours
     */
    annotated_signed_transaction transfer_to_savings( string from, string to, asset amount, string memo, bool broadcast = false );
    ```

- `transfer_to_vesting`
    ```
    /**
     * Transfer STEEM into a vesting fund represented by vesting shares (VESTS). VESTS are required to vesting
     * for a minimum of one coin year and can be withdrawn once a week over a two year withdraw period.
     * VESTS are protected against dilution up until 90% of STEEM is vesting.
     *
     * @param from The account the STEEM is coming from
     * @param to The account getting the VESTS
     * @param amount The amount of STEEM to vest i.e. "100.00 STEEM"
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction transfer_to_vesting(string from, string to, asset amount, bool broadcast = false);
    ```
    Пример:
    ```
    transfer_to_vesting cyberfounder bob "500.000 GESTS" true
    ```

- `unlock`
    Разблокирует cli_wallet при вводе пароля
    Пример:
    ```
    >>> unlock <password>
    ```

- `update_account`
    ```
    /**
     * This method updates the keys of an existing account.
     *
     * @param accountname The name of the account
     * @param json_meta New JSON Metadata to be associated with the account
     * @param owner New public owner key for the account
     * @param active New public active key for the account
     * @param posting New public posting key for the account
     * @param memo New public memo key for the account
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction update_account( string accountname,
                                             string json_meta,
                                             public_key_type owner,
                                             public_key_type active,
                                             public_key_type posting,
                                             public_key_type memo,
                                             bool broadcast )const;

    ```

- `update_account_auth_account`
    ```
    /**
     * This method updates the account of an authority for an exisiting account.
     * Warning: You can create impossible authorities using this method. The method
     * will fail if you create an impossible owner authority, but will allow impossible
     * active and posting authorities.
     *
     * @param account_name The name of the account whose authority you wish to update
     * @param type The authority type. e.g. owner, active, or posting
     * @param auth_account The account to add the the authority
     * @param weight The weight the account should have in the authority. A weight of 0 indicates the removal of the account.
     * @param broadcast true if you wish to broadcast the transaction.
     */
    ```

- `update_account_auth_key`

    ```
    /**
     * This method updates the key of an authority for an exisiting account.
     * Warning: You can create impossible authorities using this method. The method
     * will fail if you create an impossible owner authority, but will allow impossible
     * active and posting authorities.
     *
     * @param account_name The name of the account whose authority you wish to update
     * @param type The authority type. e.g. owner, active, or posting
     * @param key The public key to add to the authority
     * @param weight The weight the key should have in the authority. A weight of 0 indicates the removal of the key.
     * @param broadcast true if you wish to broadcast the transaction.
     */
    annotated_signed_transaction update_account_auth_key( string account_name, authority_type type, public_key_type key, weight_type weight, bool broadcast );

    ```

- `update_account_auth_threshold`
    ```
    /**
     * This method updates the weight threshold of an authority for an account.
     * Warning: You can create impossible authorities using this method as well
     * as implicitly met authorities. The method will fail if you create an implicitly
     * true authority and if you create an impossible owner authoroty, but will allow
     * impossible active and posting authorities.
     *
     * @param account_name The name of the account whose authority you wish to update
     * @param type The authority type. e.g. owner, active, or posting
     * @param threshold The weight threshold required for the authority to be met
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction update_account_auth_threshold( string account_name, authority_type type, uint32_t threshold, bool broadcast );
    ```

- `update_account_memo_key`
    ```
    /**
     * This method updates the memo key of an account
     *
     * @param account_name The name of the account you wish to update
     * @param key The new memo public key
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction update_account_memo_key( string account_name, public_key_type key, bool broadcast );
    ```

- `update_account_meta`
    ```
    /**
     * This method updates the account JSON metadata
     *
     * @param account_name The name of the account you wish to update
     * @param json_meta The new JSON metadata for the account. This overrides existing metadata
     * @param broadcast ture if you wish to broadcast the transaction
     */
    annotated_signed_transaction update_account_meta(string account_name, string json_meta, bool broadcast);
    ```
    Пример:
    ```
    update_account_meta profile "{\"name\":\"update\"}" true
    ```

- `update_witness`
    ```
    /**
     * Update a witness object owned by the given account.
     *
     * @param witness_name The name of the witness account.
     * @param url A URL containing some information about the witness.  The empty string makes it remain the same.
     * @param block_signing_key The new block signing public key.  The empty string disables block production.
     * @param props The chain properties the witness is voting on.
     * @param broadcast true if you wish to broadcast the transaction.
     */
    annotated_signed_transaction update_witness(string witness_name,
                                                string url,
                                                public_key_type block_signing_key,
                                                const chain_properties& props,
                                                bool broadcast = false);
    ```
    Пример:
    ```
    update_witness cyberfounder "http://url" GLS58g5rWYS3XFTuGDSxLVwiBiPLoAyCZgn6aB9Ueh8Hj5qwQA3r6 {"account_creation_fee": "1.000 GOLOS"} true
    ```

- `vote`
    ```
    /**
     * Vote on a comment to be paid STEEM
     *
     * @param voter The account voting
     * @param author The author of the comment to be voted on
     * @param permlink The permlink of the comment to be voted on. (author, permlink) is a unique pair
     * @param weight The weight [-100,100] of the vote
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction vote( string voter, string author, string permlink, int16_t weight, bool broadcast );
    ```
    Пример:
    ```
    vote cyberfounder cyberfounder plink 100 true
    ```

- `vote_for_witness`
    ```
    /**
     * Vote for a witness to become a block producer. By default an account has not voted
     * positively or negatively for a witness. The account can either vote for with positively
     * votes or against with negative votes. The vote will remain until updated with another
     * vote. Vote strength is determined by the accounts vesting shares.
     *
     * @param account_to_vote_with The account voting for a witness
     * @param witness_to_vote_for The witness that is being voted for
     * @param approve true if the account is voting for the account to be able to be a block produce
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction vote_for_witness(string account_to_vote_with,
                                                  string witness_to_vote_for,
                                                  bool approve = true,
                                                  bool broadcast = false);
    ```
    Пример:
    ```
    vote_for_witness alice cyberfounder true true
    ```

- `withdraw_vesting`
    ```
    /**
     * Set up a vesting withdraw request. The request is fulfilled once a week over the next two year (104 weeks).
     *
     * @param from The account the VESTS are withdrawn from
     * @param vesting_shares The amount of VESTS to withdraw over the next two years. Each week (amount/104) shares are
     *    withdrawn and deposited back as STEEM. i.e. "10.000000 VESTS"
     * @param broadcast true if you wish to broadcast the transaction
     */
    annotated_signed_transaction withdraw_vesting( string from, asset vesting_shares, bool broadcast = false );
    ```



# List of all cli_wallet commands returned by `help()` command #


    N2fc14static_variantIJNS_14variant_objectEEEE about()
    char                                     add_operation_copy_to_builder_transaction(uint16_t, uint16_t, uint32_t)
    char                                     add_operation_to_builder_transaction(uint16_t, golos::protocol::operation)
    golos::protocol::signed_transaction      approve_proposal(std::string, std::string, golos::wallet::approval_delta, bool)
    uint16_t                                 begin_builder_transaction()
    golos::protocol::annotated_signed_transaction cancel_order(std::string, uint32_t, bool)
    golos::protocol::annotated_signed_transaction cancel_transfer_from_savings(std::string, uint32_t, bool)
    golos::protocol::annotated_signed_transaction change_recovery_account(std::string, std::string, bool)
    golos::protocol::annotated_signed_transaction convert_sbd(std::string, golos::protocol::asset, bool)
    golos::protocol::annotated_signed_transaction create_account(std::string, std::string, std::string, bool)
    golos::protocol::annotated_signed_transaction create_account_delegated(std::string, golos::protocol::asset, golos::protocol::asset, std::string, std::string, bool)
    golos::protocol::annotated_signed_transaction create_account_with_keys(std::string, std::string, std::string, golos::protocol::public_key_type, golos::protocol::public_key_type, golos::protocol::public_key_type, golos::protocol::public_key_type, bool)
    golos::protocol::annotated_signed_transaction create_account_with_keys_delegated(std::string, golos::protocol::asset, golos::protocol::asset, std::string, std::string, golos::protocol::public_key_type, golos::protocol::public_key_type, golos::protocol::public_key_type, golos::protocol::public_key_type, bool)
    golos::protocol::annotated_signed_transaction create_order(std::string, uint32_t, golos::protocol::asset, golos::protocol::asset, bool, uint32_t, bool)
    N2fc14static_variantIJNS_14variant_objectEEEE database_info()
    golos::protocol::annotated_signed_transaction decline_voting_rights(std::string, bool, bool)
    std::string                              decrypt_memo(std::string)
    golos::protocol::annotated_signed_transaction delegate_vesting_shares(std::string, std::string, golos::protocol::asset, bool)
    golos::protocol::annotated_signed_transaction escrow_approve(std::string, std::string, std::string, std::string, uint32_t, bool, bool)
    golos::protocol::annotated_signed_transaction escrow_dispute(std::string, std::string, std::string, std::string, uint32_t, bool)
    golos::protocol::annotated_signed_transaction escrow_release(std::string, std::string, std::string, std::string, std::string, uint32_t, golos::protocol::asset, golos::protocol::asset, bool)
    golos::protocol::annotated_signed_transaction escrow_transfer(std::string, std::string, std::string, uint32_t, golos::protocol::asset, golos::protocol::asset, golos::protocol::asset, fc::time_point_sec, fc::time_point_sec, std::string, bool)
    golos::api::account_api_object           get_account(std::string)
    std::map<uint32_t,golos::plugins::operation_history::applied_operation> get_account_history(std::string, uint32_t, uint32_t)
    std::vector<N2fc14static_variantIJNS_12fixed_stringINS_9uint128_tEEEEEE> get_active_witnesses()
    optional<golos::wallet::signed_block_with_info> get_block(uint32_t)
    std::vector<golos::chain::convert_request_object> get_conversion_requests(std::string)
    std::string                              get_encrypted_memo(std::string, std::string, std::string)
    golos::plugins::witness_api::feed_history_api_object get_feed_history()
    std::vector<golos::plugins::private_message::extended_message_object> get_inbox(std::string, fc::time_point, uint16_t, uint64_t)
    std::vector<N2fc14static_variantIJNS_12fixed_stringINS_9uint128_tEEEEEE> get_miner_queue()
    std::vector<golos::plugins::database_api::extended_limit_order> get_open_orders(std::string)
    std::vector<golos::plugins::operation_history::applied_operation> get_ops_in_block(uint32_t, bool)
    golos::plugins::market_history::order_book get_order_book(uint32_t)
    std::vector<golos::plugins::private_message::extended_message_object> get_outbox(std::string, fc::time_point, uint16_t, uint64_t)
    std::vector<golos::plugins::database_api::owner_authority_history_api_object> get_owner_history(std::string)
    std::string                              get_private_key(golos::protocol::public_key_type)
    N2fc14static_variantIJSt4pairIN5golos8protocol15public_key_typeENSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEEEEEE get_private_key_from_password(std::string, std::string, std::string)
    std::vector<golos::plugins::database_api::proposal_api_object> get_proposed_transactions(std::string, uint32_t, uint32_t)
    golos::protocol::operation               get_prototype_operation(std::string)
    golos::protocol::annotated_signed_transaction get_transaction(uint160_t)
    std::vector<golos::chain::withdraw_vesting_route_object> get_withdraw_routes(std::string, golos::plugins::database_api::withdraw_route_type)
    optional<golos::plugins::witness_api::witness_api_object> get_witness(std::string)
    std::string                              gethelp(std::string)
    std::string                              help()
    bool                                     import_key(std::string)
    fc::variant                              info()
    bool                                     is_locked()
    bool                                     is_new()
    std::vector<N2fc14static_variantIJNS_12fixed_stringINS_9uint128_tEEEEEE> list_accounts(std::string, uint32_t)
    std::map<golos::protocol::public_key_type,std::string> list_keys()
    std::vector<golos::api::account_api_object> list_my_accounts()
    std::vector<N2fc14static_variantIJNS_12fixed_stringINS_9uint128_tEEEEEE> list_witnesses(std::string, uint32_t)
    bool                                     load_wallet_file(std::string)
    char                                     lock()
    std::string                              normalize_brain_key(std::string)
    golos::protocol::annotated_signed_transaction post_comment(std::string, std::string, std::string, std::string, std::string, std::string, std::string, bool)
    golos::protocol::transaction             preview_builder_transaction(uint16_t)
    golos::protocol::signed_transaction      propose_builder_transaction(uint16_t, std::string, std::string, std::string, fc::time_point_sec, fc::time_point_sec, bool)
    golos::protocol::annotated_signed_transaction publish_feed(std::string, golos::protocol::price, bool)
    char                                     quit()
    golos::protocol::annotated_signed_transaction recover_account(std::string, golos::protocol::authority, golos::protocol::authority, bool)
    char                                     remove_builder_transaction(uint16_t)
    char                                     replace_operation_in_builder_transaction(uint16_t, uint32_t, golos::protocol::operation)
    golos::protocol::annotated_signed_transaction request_account_recovery(std::string, std::string, golos::protocol::authority, bool)
    char                                     save_wallet_file(std::string)
    std::string                              serialize_transaction(golos::protocol::signed_transaction)
    char                                     set_password(std::string)
    char                                     set_transaction_expiration(uint32_t)
    golos::protocol::annotated_signed_transaction set_voting_proxy(std::string, std::string, bool)
    golos::protocol::annotated_signed_transaction set_withdraw_vesting_route(std::string, std::string, uint16_t, bool, bool)
    golos::protocol::signed_transaction      sign_builder_transaction(uint16_t, bool)
    golos::protocol::annotated_signed_transaction sign_transaction(golos::protocol::signed_transaction, bool)
    golos::wallet::brain_key_info            suggest_brain_key()
    golos::protocol::annotated_signed_transaction transfer(std::string, std::string, golos::protocol::asset, std::string, bool)
    golos::protocol::annotated_signed_transaction transfer_from_savings(std::string, uint32_t, std::string, golos::protocol::asset, std::string, bool)
    golos::protocol::annotated_signed_transaction transfer_to_savings(std::string, std::string, golos::protocol::asset, std::string, bool)
    golos::protocol::annotated_signed_transaction transfer_to_vesting(std::string, std::string, golos::protocol::asset, bool)
    char                                     unlock(std::string)
    golos::protocol::annotated_signed_transaction update_account(std::string, std::string, golos::protocol::public_key_type, golos::protocol::public_key_type, golos::protocol::public_key_type, golos::protocol::public_key_type, bool)
    golos::protocol::annotated_signed_transaction update_account_auth_account(std::string, golos::wallet::authority_type, std::string, uint16_t, bool)
    golos::protocol::annotated_signed_transaction update_account_auth_key(std::string, golos::wallet::authority_type, golos::protocol::public_key_type, uint16_t, bool)
    golos::protocol::annotated_signed_transaction update_account_auth_threshold(std::string, golos::wallet::authority_type, uint32_t, bool)
    golos::protocol::annotated_signed_transaction update_account_memo_key(std::string, golos::protocol::public_key_type, bool)
    golos::protocol::annotated_signed_transaction update_account_meta(std::string, std::string, bool)
    golos::protocol::annotated_signed_transaction update_witness(std::string, std::string, golos::protocol::public_key_type, golos::protocol::chain_properties, bool)
    golos::protocol::annotated_signed_transaction vote(std::string, std::string, std::string, int16_t, bool)
    golos::protocol::annotated_signed_transaction vote_for_witness(std::string, std::string, bool, bool)
    golos::protocol::annotated_signed_transaction withdraw_vesting(std::string, golos::protocol::asset, bool)

