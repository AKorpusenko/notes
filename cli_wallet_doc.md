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

## Non-interactive режим ##
TODO написать про то, что можно писать скрипты, потому что ключ --comands="" позволяет последовтельно
исполнять команды в неинтерактивном режиме


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
#TODO добавить пример запуска 

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
#TODO добавить пример запуска 


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

- `change_recovery_account`

- `convert_sbd`

- `create_account`
This method will generate new owner, active, posting and memo keys for the new account
which will be controlable by this wallet.
Аргументы: имя_акк_создателя, имя_нового_аккаунта, json_meta 

- `create_account_delegated`

- `create_account_with_keys`

- `create_account_with_keys_delegated`

- `create_order`

- `database_info`

- `decline_voting_rights`

- `decrypt_memo`

- `delegate_vesting_shares`

- `escrow_approve`

- `escrow_dispute`

- `escrow_release`

- `escrow_transfer`

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

- `get_active_witnesses`

- `get_block`

- `get_conversion_requests`

- `get_encrypted_memo`

- `get_feed_history`

- `get_inbox`

- `get_miner_queue`

- `get_open_orders`

- `get_ops_in_block`

- `get_order_book`

- `get_outbox`

- `get_owner_history`

- `get_private_key`

- `get_private_key_from_password`

- `get_proposed_transactions`

- `get_prototype_operation`

- `get_transaction`

- `get_withdraw_routes`

- `get_witness`

- `gethelp`

- `help`

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

