## Escrow Smart Contract (in Ink!)

Escrow is a financial arrangement where a neutral third party temporarily holds funds/assets while two parties complete a transaction. It ensures:  

1. **Security**: Funds are protected until conditions are met.  
2. **Trust**: Neither party can act unilaterally.  
3. **Automation**: Rules are enforced programmatically (via smart contracts).  

### Key Features

| Feature                 | Description                                            |  
|-------------------------|--------------------------------------------------------|  
| **Mutual Approval**     | Both buyer and seller must approve to release funds.   |  
| **Automatic Execution** | Transfers funds instantly when both parties approve.   |  
| **Cancellation**        | Allows refunds (when funded) if the transaction fails. |  
| **Transparency**        | All actions logged as on-chain events                  |  
| **Security Checks**     | Prevents invalid states.                               |  

### Data Structure

| Component          | Type         | Description                                  |
|--------------------|--------------|----------------------------------------------|
| `buyer`            | AccountId    | Initiator/asset depositor                    |
| `seller`           | AccountId    | Recipient of assets                          |
| `amount`           | Balance      | Agreed transaction value                     |
| `buyer_approved`   | bool         | Buyer's confirmation flag                    |
| `seller_approved`  | bool         | Seller's confirmation flag                   |
| `state`            | EscrowState  | Current lifecycle stage (see state diagram)  |

## Functions overview

### `initiate_escrow` - Start Transaction

**Key Points**:

- Buyer initiates by specifying seller/amount
- Prevents self-dealing with `buyer == seller` check
- Auto-increments escrow IDs

### `deposit_assets` - Fund Escrow

**Key Points**:

- Only buyer can deposit
- Exact amount required
- Must be in `Created` state

### `complete_escrow` - Mutual Approval

**Key Points**:

- Buyer and seller must call separately
- Prevents duplicate approvals
- Funds transfer only after mutual consent

### `cancel_escrow` - Abort Transaction

**Key Points**:

- Either party can cancel
- Refund only if funds were deposited
- Completed escrows cannot be canceled

## States

```mermaid
stateDiagram-v2
    [*] --> Created
    Created --> Funded : deposit_assets()\n
    Created --> Canceled : cancel_escrow()\n
    
    Funded --> Completed : complete_escrow()\n
    Funded --> Canceled : cancel_escrow()\n
    
    Completed --> [*]
    Canceled --> [*]
```

## Sequence Diagram

```mermaid
sequenceDiagram
    participant B as Buyer
    participant C as Contract
    participant S as Seller

    B->>C: initiate_escrow(seller, amount)
    activate C
    C-->>B: escrow_id
    deactivate C
    Note right of C: State: Created

    B->>C: deposit_assets(escrow_id) + amount
    activate C
    C-->>B: Ok
    deactivate C
    Note right of C: State: Funded

    B->>C: complete_escrow(escrow_id)
    activate C
    C->>C: Mark buyer_approved = true
    C-->>B: Ok
    deactivate C

    S->>C: complete_escrow(escrow_id)
    activate C
    C->>C: Mark seller_approved = true
    alt Both approved?
        C->>S: Transfer funds
        Note right of C: State: Completed
    end
    C-->>S: Ok
    deactivate C

    alt Cancellation
        B->>C: cancel_escrow(escrow_id)
        activate C
        alt State == Funded
            C->>B: Refund
        end
        Note right of C: State: Canceled
        C-->>B: Ok
        deactivate C
    end
```
