# Multi-Conditional Plutus Validator

A Cardano smart contract project using [Aiken](https://aiken-lang.org) and [Mesh SDK](https://meshjs.dev) to demonstrate multi-conditional Plutus validators. This project includes scripts for locking and unlocking ADA with custom on-chain logic, and is structured for easy extension and testing.

---

## Features

- Write and test Plutus validators in Aiken (`validators/`).
- Lock and unlock ADA using TypeScript scripts powered by Mesh SDK.
- Easily generate and manage wallet credentials.
- Modular structure for validators and supporting libraries.

---

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) (v18+ recommended)
- [Aiken](https://aiken-lang.org)
- [Mesh SDK](https://meshjs.dev)
- Cardano testnet (preprod) tADA for testing

### Installation

Clone the repository and install dependencies:

```sh
git clone <your-repo-url>
cd multi-conditional-validator
npm install
```

### Environment Setup

Create a `.env` file with your Blockfrost project ID:

```
BLOCKFROST_PROJECT_ID=your_preprod_project_id
```

Or set it in your shell before running scripts:

```sh
$env:BLOCKFROST_PROJECT_ID="your_preprod_project_id" # PowerShell
export BLOCKFROST_PROJECT_ID=your_preprod_project_id # Bash
```

---

## Wallet Setup

Generate a new wallet and address for testing:

```sh
npx tsx generate-credentials.ts
```

This will create `me.sk` (secret key) and `me.addr` (address) files.

Fund your testnet address (`me.addr`) with tADA from the [Cardano Testnet Faucet](https://docs.cardano.org/cardano-testnet/tools/faucet/).

---

## Building the Validator

Compile your Aiken validator:

```sh
aiken build
```

---

## Locking Funds

Lock 1 tADA into the contract:

```sh
npx tsx lock.ts
```

---

## Unlocking Funds

Unlock the funds by providing the correct redeemer and signature:

```sh
npx tsx unlock.ts <tx_hash_from_lock from the output log of lock.ts>
```

To send the unlocked funds to a different address, edit `unlock.ts` and set `.changeAddress()` to your desired recipient.

---

## Writing Validators

Write your validators in the `validators/` folder using `.ak` files. Example:

```aiken
validator hello_world {
  spend(
    datum: Option<Datum>,
    redeemer: Redeemer,
    _own_ref: OutputReference,
    self: Transaction,
  ) {
    trace @"redeemer": string.from_bytearray(redeemer.msg)
    expect Some(Datum { owner }) = datum
    let must_say_hello = redeemer.msg == "Hello, World!"
    let must_be_signed = list.has(self.extra_signatories, owner)
    must_say_hello? && must_be_signed?
  }
}
```

---

## Testing

Write tests in any `.ak` module using the `test` keyword. Example:

```aiken
test hello_world_example() {
  let datum = Datum { owner: #"00000000000000000000000000000000000000000000000000000000" }
  let redeemer = Redeemer { msg: "Hello, World!" }
  let placeholder_utxo = OutputReference { transaction_id: "", output_index: 0 }
  hello_world.spend(
    Some(datum),
    redeemer,
    placeholder_utxo,
    Transaction { ..transaction.placeholder, extra_signatories: [datum.owner] },
  )
}
```

Run all tests:

```sh
aiken check
```

Run only tests matching a string:

```sh
aiken check -m hello_world
```

---

## Documentation

Generate HTML documentation for your library:

```sh
aiken docs
```

---

## Project Structure

```
multi-conditional-validator/
├── validators/                 # Aiken validator scripts
├── lib/                        # Supporting Aiken libraries
├── build/                      # Build artifacts
├── env/                        # Environment-specific configs
├── common.ts                   # Shared TypeScript utilities
├── lock.ts                     # Script to lock funds
├── unlock.ts                   # Script to unlock funds
├── redeemer-msg.ts             # script to provide redeemer-msg
├── generate-credentials.ts     # Wallet generation script
├── me.sk                       # Wallet secret key (auto-generated)
├── me.addr                     # Wallet address (auto-generated)
├── plutus.json                 # Compiled Plutus scripts
├── package.json
├── aiken.toml
├── .env
└── README.md
```

---

## Resources

- [Aiken User Manual](https://aiken-lang.org)
- [Mesh SDK Documentation](https://meshjs.dev)
- [Cardano Testnet Faucet](https://docs.cardano.org/cardano-testnet/tools/faucet/)

---
