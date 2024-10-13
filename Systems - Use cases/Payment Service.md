Reference: System Design interview an insider's guide [volume 2](https://github.com/rneha725/Books/blob/main/System%20Design%20Interview%20An%20Insider%E2%80%99s%20Guide%20by%20Alex%20Xu%20Volume%202%20(2).pdf)

Use case: Amazon's payment service, customer and seller flow. Also, implementing waller for sellers.

Pay-in flow: When a customer starts checking out, the items in the cardmwill result in payments to multiple sellers. 
Pay-out flow: When seller's money needs to be credited, this flow starts from Amazon's account to seller's account.

Components:
- Payment service: internal payment service that performs the required transactions.
- PSP: Payment Service provides: payments can happen through different payment methods, credit card, UPI, netbanking etc. Instead of building in the systems(which is really complex), a third party PSP is used. A PSP can also provide all the transtion happened on a day. A PSP can help in two ways:
  - Only facilitate the payment and not store the sensitive information of the user, in that case the company has to do it.
  - To also, store the sensitive info and facilitate the payment.
- Ledger: Ledger contains all the transaction on an account level. In the resources, there is a link to Square's double ledger architecture. The idea is to basically maintain a credit/debit etc such that the sum of all is always zero. They are using Spanner, which is  Google's distributed SQL db.
  - Double entry ledger has these 3 properties:
    - Immutability: once data is written cannot be changed, if there is any error, a corrective entry is added.
    - Auditability
    - Zero-sum entries are getting added.  
- CDC, if required: an outbox pattern can be used to implement this.

### Pay-in flow
![image](https://github.com/user-attachments/assets/14671642-2538-4928-ac72-a52bb798d747)

### Payout flow
- ![image](https://github.com/user-attachments/assets/e583250c-7230-4441-99e2-226f65287e50)
Each the step defined in the diagram can fail, to make it right, system performs reconciliation, say, each day. PSP can provide the daily transactions can it can be match against the ledger. According to the mismatch either the system handles it or it is handled manually by the finance team. 

### Resources:
- Double Ledger entry: [Square's Blog](https://developer.squareup.com/blog/books-an-immutable-double-entry-accounting-database-service/)
- Uber's payment service using kafka: [Third Party blog based on talks](https://underhood.blog/uber-payments-platform)
