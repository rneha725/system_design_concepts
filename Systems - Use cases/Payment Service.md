Use case: Amazon's payment service, customer and seller flow. Also, implementing waller for sellers.

Pay-in flow: When a customer starts checking out, the items in the cardmwill result in payments to multiple sellers. 
Pay-out flow: When seller's money needs to be credited, this flow starts from Amazon's account to seller's account.

Components:
- Payment service: internal payment service that performs the required transactions.
- PSP: Payment Service provides: payments can happen through different payment methods, credit card, UPI, netbanking etc. Instead of building in the systems(which is really complex), a third party PSP is used.
- Ledger: Ledger contains all the transaction on an account level. In the resources, there is a link to Square's double ledger architecture. The idea is to basically maintain a credit/debit etc such that the sum of all is always zero. They are using Spanner, which is  Google's distributed SQL db.
- CDC, if required: an outbox pattern can be used to implement this.

### Resources:
Double Ledger entry: [Square's Blog](https://developer.squareup.com/blog/books-an-immutable-double-entry-accounting-database-service/)
