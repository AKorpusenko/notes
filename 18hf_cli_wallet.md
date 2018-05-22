# 18 HF cli_wallet interface changes #
## Основные изменения. ##
1. Added method `create_account_delegated`.
This method will generate new owner, active, posting and memo keys for the new account which will be controllable by this wallet. There is a fee associated with account creation that is paid by the creator. The current account creation fee can be found with the 'info' wallet command.
These accounts are created with combination of GOLOS and delegated GP
    

2. Added method `create_account_with_keys_delegated`.
This method is used by faucets to create new accounts for other users which must provide their desired keys. The resulting account may not be controllable by this wallet. There is a fee associated with account creation that is paid by the creator. The current account creation fee can be found with the 'info' wallet command.
These accounts are created with combination of GOLOS and delegated GP

3. Added method `delegate_vesting_shares`.
This method delegates GESTS from one account to another.

4. Group of API methods Transaction Builder API.
Добавлена возможность вручную, через cli_wallet, создавать любые транзакции. Все нижеперечисленные методы нужны для того, чтобы предоставить инструмент для конструирования транзакций. 
Also: [bitshares_doc](http://docs.bitshares.org/tutorials/construct-transaction.html)
Так как в 18 ХФ были добавлены proposal транзакции, то часть методов нужна для работы напрямую с ними. Большая часть работы по proposal'ам была проделана в задаче [#542](https://github.com/GolosChain/golos/issues/542). Соответствующий pull request [#597](https://github.com/GolosChain/golos/pull/597)
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
### Краткое описание процесса создания транзакции с помощью конструктора ###

1. Create an instance of the transaction builder
2. Add arbitrary operation types
3. Add the required amount of fees
4. Sign and broadcast your transaction

1. Создать instance конструктора транзакций. (просто вызвать метод begin_builder_transaction, который возвращает уникальный handle)
2. Добавить транзакции произвольных типов
3. Добавить требуемое значение fee
4. Подписать и забродкастить транзкацию

## Список всех изменений подробно ##

1. Add the parameter `fee` to `create_account`. 
Now current signature of this method look this way:
```
annotated_signed_transaction create_account(
    string creator, string new_account_name, string json_meta,
    asset fee = asset(), bool broadcast = false
);
```
[Pull request](https://github.com/GolosChain/golos/pull/658)

2. Added method `get_proposed_transactions` to `cli_wallet`.  
```
std::vector<proposal_api_object> get_proposed_transactions(account_name_type); 
```
[Pull request](https://github.com/GolosChain/golos/pull/597)

3. Added method `begin_builder_transaction` to `cli_wallet`.  
```
transaction_handle_type begin_builder_transaction();
```

[Pull request](https://github.com/GolosChain/golos/pull/597)

4. Added method `get_proposed_transactions` to `cli_wallet`. 
Чтобы создать операцию (с помощью add_operation_to_builder_transaction) в данный момент, сначала нужно получить шаблон для этой операции, а потом его корректно заполнить. Да, это не идеальное решение, но всё же лучше, чем ничего.
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

5. Added method `add_operation_to_builder_transaction` to `cli_wallet`.  
```
void add_operation_to_builder_transaction(transaction_handle_type handle, const operation& op);

```
[Pull request](https://github.com/GolosChain/golos/pull/597)


6. Added method `add_operation_copy_to_builder_transaction` to `cli_wallet`.  
```
void add_operation_to_builder_transaction(transaction_handle_type handle, const operation& op);

```
[Pull request](https://github.com/GolosChain/golos/pull/597)


- Изменился возвращаемый результат `get_ops_in_block`
Было:
`vector<operation_api_object> get_ops_in_block( uint32_t <blockquote></blockquote>_num, bool only_virtual = true );`
Стало:
`vector<golos::plugins::operation_history::applied_operation> get_ops_in_block( uint32_t block_num, bool only_virtual = true );`