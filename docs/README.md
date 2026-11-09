# COBOL Account Management System


This directory documents the COBOL account-management example in `src/cobol`. The program provides a console menu for viewing a balance, crediting an account, and debiting an account.

## Source files

| File | Purpose | Key behavior |
| --- | --- | --- |
| `src/cobol/main.cob` | Console entry point (`MainProgram`). | Displays the menu, accepts a choice, and calls `Operations` with `TOTAL`, `CREDIT`, or `DEBIT`. Choice 4 exits; other choices display an invalid-choice message. |
| `src/cobol/operations.cob` | Account operation handler (`Operations`). | Reads and displays the balance, accepts credit/debit amounts, applies the requested change, and reports the result. It rejects a debit when the available balance is too low. |
| `src/cobol/data.cob` | Balance storage interface (`DataProgram`). | Handles `READ` by returning the stored balance and `WRITE` by replacing it. The working-storage balance starts at `1000.00`. |

## Student account rules represented in the code

The source code does not define student records, student identifiers, account ownership, or student-specific policies. “Student account” therefore refers only to the generic account balance handled by this example; no student-specific behavior should be inferred.

The implemented balance rules are:

- The balance is initialized to `1000.00` when `DataProgram` starts.
- A credit adds the entered amount to the current balance.
- A debit is allowed only when the current balance is greater than or equal to the entered amount. Otherwise, the program displays an insufficient-funds message and leaves the balance unchanged.
- The balance and amounts use `PIC 9(6)V99`, representing up to six integer digits and two decimal digits. The implied decimal point is not displayed by these fields.
- The balance is held in working storage; the program does not save it to a file or database. It is not durable across program runs.

## Important limitations

The program does not validate that entered amounts are positive, enforce student eligibility or account limits, or check for a balance overflow after a credit. These are not implemented business rules and would need to be specified before adding student-account behavior.

## Sequence diagram

```mermaid
sequenceDiagram
actor User
participant Main as MainProgram
participant Ops as Operations
participant Data as DataProgram

loop Until the user exits
    Main->>User: Display menu and request choice
    User-->>Main: Select 1, 2, 3, or 4
    alt View balance (choice 1)
	Main->>Ops: TOTAL
	Ops->>Data: READ, FINAL-BALANCE
	Data-->>Ops: Return stored balance
	Ops-->>User: Display current balance
    else Credit account (choice 2)
	Main->>Ops: CREDIT
	Ops->>User: Request credit amount
	User-->>Ops: Enter amount
	Ops->>Data: READ, FINAL-BALANCE
	Data-->>Ops: Return stored balance
	Ops->>Ops: Add amount to balance
	Ops->>Data: WRITE, updated balance
	Data-->>Ops: Store updated balance
	Ops-->>User: Display new balance
    else Debit account (choice 3)
	Main->>Ops: DEBIT
	Ops->>User: Request debit amount
	User-->>Ops: Enter amount
	Ops->>Data: READ, FINAL-BALANCE
	Data-->>Ops: Return stored balance
	alt Balance is sufficient
	    Ops->>Ops: Subtract amount from balance
	    Ops->>Data: WRITE, updated balance
	    Data-->>Ops: Store updated balance
	    Ops-->>User: Display new balance
	else Insufficient funds
	    Ops-->>User: Display insufficient-funds message
	end
    else Exit (choice 4)
	Main->>Main: Set continue flag to NO
	Main-->>User: Display goodbye message
    else Invalid choice
	Main-->>User: Display invalid-choice message
    end
end
```
