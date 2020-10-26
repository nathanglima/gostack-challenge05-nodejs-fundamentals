<img alt="GoStack" src="https://camo.githubusercontent.com/a869a2aaab296ef925343d7e76518cd213eb0a30/68747470733a2f2f73746f726167652e676f6f676c65617069732e636f6d2f676f6c64656e2d77696e642f626f6f7463616d702d676f737461636b2f6865616465722d6465736166696f732d6e65772e706e67" />

<h2 align="center">
  Challenge 04: NodeJS Fundamentals
</h2>


## :page_facing_up: About this challenge

This challenge, we were challenged to develop the logic of a bank transaction API.

We follow the architecture of development with the routes, services (business rules), repositories (interaction with the model and the data).

We did calculations and some business rules according of values and transaction types, conform the test below.

## :rocket: About of tests
Some rules for evaluating this challenge were proposed, some tests were written using jest and to pass the tests it is necessary to respect all the points below:

#### *should be able to create a new transaction*

In order for this test to pass, your application must allow a transaction be created and return a json with the transaction created.

<b>transaction.routes.ts</b>

```js
transactionRouter.post('/', (request, response) => {
  try {
    const { title, value, type } = request.body;

    const createTransaction = new CreateTransactionService( transactionsRepository ); 

    const transaction = createTransaction.execute({ title, value, type });

    return response.json(transaction);
    
  } catch (err) {
    return response.status(400).json({ error: err.message });
  }
});
```
<b>CreateTransactionService.ts</b>

```js
public execute({ title, value, type }: Request ): Transaction {

  const { total } = this.transactionsRepository.getBalance();

  if(type == 'outcome' && value > total) {
    throw new Error('You do not have enough balance');
  }

  const transaction = this.transactionsRepository.create({ title, value, type });

  return transaction;
}
```

<b>TransactionsRepository.ts</b>

```js
public create({ title, value, type }: CreateTransactionDTO ): Transaction {
  const transaction = new Transaction({ title, value, type });

  this.transactions.push(transaction);

  return transaction;
}
```

#### *should be able to list the transactions*

In order for this test to pass, your application must allow be returned an object containing all transactions together the balance of income, outcome and total of transactions that went created until now.

<b>transaction.routes.ts</b>

```js
transactionRouter.get('/', (request, response) => {
  try {
    const transactions = transactionsRepository.all();
    const balance = transactionsRepository.getBalance();

    return response.json({ transactions, balance });
  } catch (err) {
    return response.status(400).json({ error: err.message });
  }
});
```

<b>TransactionsRepository.ts</b>

```js
public all(): Transaction[] {
  return this.transactions;
}

public getBalance(): Balance {
  const { income, outcome } = this.transactions.reduce( (acumulator: Balance, transaction: Transaction) => {

    switch (transaction.type) {
      case 'income':
        acumulator.income += transaction.value;
        break;
      case 'outcome':
        acumulator.outcome += transaction.value;
        break;
      default:
        break;
    }

    return acumulator;

  }, {
    income: 0,
    outcome: 0,
    total: 0,
  });

  const total = income - outcome;

  return { income, outcome, total };
}
```

#### *should not be able to create outcome transaction without a valid balance*

In order for this test to pass, your application don't be must a transaction of type 'outcome' does not exceed the total value that the user have on account balance, returning a response with HTTP code 400 and a message of error in the following format: {error: string}

<b>CreateTransactionService.ts</b>

```js
const { total } = this.transactionsRepository.getBalance();

if(type == 'outcome' && value > total) {
  throw new Error('You do not have enough balance');
}
```
