# 18 HF cli_wallet interface changes #
## Основные изменения. ##
1. Added method `create_account_delegated`.
This method will generate new owner, active, posting and memo keys for the new account
which will be controllable by this wallet.
There is a fee associated with account creation that is paid by the creator.
The current account creation fee can be found with the 'info' wallet command.
These accounts are created with combination of GOLOS and delegated GP
    

2. Added method `create_account_with_keys_delegated`.
This method is used by faucets to create new accounts for other users which must provide their desired keys.
The resulting account may not be controllable by this wallet.
There is a fee associated with account creation that is paid by the creator.
The current account creation fee can be found with the 'info' wallet command.
These accounts are created with combination of GOLOS and delegated GP

3. Added method `delegate_vesting_shares`.
This method delegates GESTS from one account to another.

4. Group of API methods Transaction Builder API.
Добавлена возможность вручную, через cli_wallet, создавать любые транзакции.
Все нижеперечисленные методы нужны для того, чтобы предоставить инструмент для конструирования транзакций. 
Also: [bitshares_doc](http://docs.bitshares.org/tutorials/construct-transaction.html)
Так как в 18 ХФ были добавлены proposal транзакции, то часть методов нужна для работы напрямую с ними.
Большая часть работы по proposal'ам была проделана в задаче [#542](https://github.com/GolosChain/golos/issues/542).
Соответствующий pull request [#597](https://github.com/GolosChain/golos/pull/597)
- `begin_builder_transaction`
- `add_operation_to_builder_transaction`
- `add_operation_copy_to_builder_transaction`
- `replace_operation_in_builder_transaction`
- `preview_builder_transaction`
- `sign_builder_transaction`
- `propose_builder_transaction`
- `remove_builder_transaction`
- `approve_proposal`
- `get_proposed_transactions`
(Ниже идёт краткое описание каждого метода)


### Краткое описание процесса создания транзакции с помощью конструктора  ENG ###

1. Create an instance of the transaction builder
2. Add arbitrary operation types
3. Add the required amount of fees
4. Sign and broadcast your transaction

### Краткое описание процесса создания транзакции с помощью конструктора  RUS ###
1. Создать instance конструктора транзакций. (просто вызвать метод begin_builder_transaction, 
который возвращает уникальный handle)
2. Добавить транзакции произвольных типов
3. Добавить требуемое значение fee
4. Подписать и забродкастить транзкацию

## Список всех изменений подробно ##

1. API Add the parameter `fee` to `create_account`. 
Now current signature of this method look this way:
```
annotated_signed_transaction create_account(
    string creator, string new_account_name, string json_meta,
    asset fee = asset(), bool broadcast = false
);
```
[Pull request](https://github.com/GolosChain/golos/pull/658)

2. API Added method `get_proposed_transactions` to `cli_wallet`.  
```
std::vector<proposal_api_object> get_proposed_transactions(account_name_type); 
```
[Pull request](https://github.com/GolosChain/golos/pull/597)

3. API Added method `begin_builder_transaction` to `cli_wallet`.  
```
transaction_handle_type begin_builder_transaction();
```

[Pull request](https://github.com/GolosChain/golos/pull/597)

4. API Added method `get_prototype_operation` to `cli_wallet`. 
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
[Pull request](https://github.com/GolosChain/golos/pull/597)

5. API Added method `add_operation_to_builder_transaction` to `cli_wallet`.  
```
void add_operation_to_builder_transaction(transaction_handle_type handle, const operation& op);

```
[Pull request](https://github.com/GolosChain/golos/pull/597)


6. API Added method `add_operation_copy_to_builder_transaction` to `cli_wallet`.  
Копирует операцию из одного конструктора в другой. Идентифицируются `builder_transaction` (конструктор) по handler'у,
который возвращает метод `begin_builder_transaction()`. 
```
void add_operation_copy_to_builder_transaction(
	transaction_handle_type src_handle, transaction_handle_type dst_handle, uint32_t op_index
);

```
[Pull request](https://github.com/GolosChain/golos/pull/597)


7. API Added method `replace_operation_in_builder_transaction` to `cli_wallet`.  
Заменяет операцию номер `op_index` в `builder_transaction` (конструктор) на операцию `op`.
```
void replace_operation_in_builder_transaction(
    transaction_handle_type handle, unsigned op_index, const operation& op
);

```
[Pull request](https://github.com/GolosChain/golos/pull/597)


8. API Added method `preview_builder_transaction` to `cli_wallet`.
Нужно для того, чтобы по идентификатору получить builder_transaction из списка builder_transactions
```
transaction preview_builder_transaction(transaction_handle_type handle);

```
[Pull request](https://github.com/GolosChain/golos/pull/597)


9. API Added method `sign_builder_transaction` to `cli_wallet`.
Подписывает все транзакции в builder_transaction.
```
signed_transaction sign_builder_transaction(transaction_handle_type handle, bool broadcast = true);

```
[Pull request](https://github.com/GolosChain/golos/pull/597)


10. API Added method `propose_builder_transaction` to `cli_wallet`.
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
[Pull request](https://github.com/GolosChain/golos/pull/597)


11. API Added method `remove_builder_transaction` to `cli_wallet`.
Удаляет экземпляр `builder_transaction` по его идентификатору
```
void remove_builder_transaction(transaction_handle_type handle);

```
[Pull request](https://github.com/GolosChain/golos/pull/597)


12. Added method `approve_proposal` to `cli_wallet`.
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
[Pull request](https://github.com/GolosChain/golos/pull/597)

13. API Add the parameter `fee` to `create_account_with_keys`. 
Now current signature of this method looks this way:
```
annotated_signed_transaction create_account_with_keys(
    string creator,
    string newname,
    string json_meta,
    asset fee,
    public_key_type owner,
    public_key_type active,
    public_key_type posting,
    public_key_type memo,
    bool broadcast 
) const;
```
[Pull request](https://github.com/GolosChain/golos/pull/658)


14. API Added method `create_account_delegated` to `cli_wallet`. 
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
[Pull request](https://github.com/GolosChain/golos/pull/658)


## МЕЛОЧЬ ##

- Изменился возвращаемый результат `get_ops_in_block`
- Было:
`vector<operation_api_object> get_ops_in_block( uint32_t <blockquote></blockquote>_num, bool only_virtual = true );`
- Стало:
`vector<golos::plugins::operation_history::applied_operation> get_ops_in_block( uint32_t block_num, bool only_virtual = true );`
