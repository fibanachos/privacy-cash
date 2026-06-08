# Privacy Cash — Cookie Chain fork

Transfer COOK privately on [Cookie Chain](https://rpc.cookiescan.io).

> **Fork notice.** This is a fork of [Privacy Cash](https://github.com/Privacy-Cash) ported to Cookie Chain, with COOK (the native token) in place of SOL. SPL-token support is disabled in this build (COOK-only).
>
> The original program was audited by Accretion, HashCloak, Zigtur, Kriko, Sherlock and Veridise **as deployed on Solana mainnet** (onchain hash `c6f1e5336f2068dc1c1e1c64e92e3d8495b8df79f78011e2620af60aa43090c5`). **Those audits and the verified-onchain hash do NOT apply to this fork** — it uses a different program ID, a different authority, and runs on a different chain. Treat this deployment as unaudited.

## Overview

This project implements a privacy protocol on Cookie Chain that allows users to:

1. **Shield COOK**: Deposit COOK into a privacy pool, generating a commitment that is added to a Merkle tree.
2. **Withdraw COOK**: Withdraw COOK from the privacy pool to any recipient address using zero-knowledge proofs.

The implementation uses zero-knowledge proofs to ensure that withdrawals cannot be linked to deposits, providing privacy for Cookie Chain transactions.

## Fork configuration

- **Program ID**: `CASHcHkM2PHpCHaEhksmQrv6C9YRu3csxY2eyKKydHnv` (keypair: `anchor/zkcash-keypair.json`)
- **RPC**: `https://rpc.cookiescan.io`
- **Admin authority**: ⚠️ placeholder (`11111111111111111111111111111111`) in `anchor/programs/zkcash/src/lib.rs` — **replace with your own pubkey before deploying**. Until you do, `initialize` will fail (the placeholder has no private key).
- **Tokens**: COOK (native) only. `ALLOWED_TOKENS` is empty; SPL deposits are rejected.
- **Default deposit limit**: 1000 COOK (changeable on-chain via `update_deposit_limit`).

## Project Structure

- **program/**: Solana on-chain program (smart contract)
  - **src/**: Rust source code for the program
  - **test/**: Tests
  - **Cargo.toml**: Rust dependencies and configuration

## Prerequisites

- Solana CLI 2.1.18 or later (Cookie Chain runs Agave 3.x; the required `alt_bn128` and `poseidon` syscalls are active on the cluster)
- Rust 1.79.0 or compatible version
- Anchor 0.31.1
- Node.js 16 or later
- npm or yarn
- Circom v2.2.2 https://docs.circom.io/getting-started/installation/#installing-dependencies

## SDK
If you want to integrate Privacy Cash into your project, use the [Privacy Cash SDK](https://github.com/Privacy-Cash/privacy-cash-sdk) here.

## Anchor Program
1. Navigate to the program directory:
   ```bash
   cd anchor
   ```

2. Build the program:
   ```bash
   anchor build
   ```

3. Run unit test:
   ```bash
   cargo test
   ```

4. Run integration test:
   ```bash
   npm run test:sol
   npm run test:spl
   npm run test:mint-checked
   ```

5. Deploy the program to Cookie Chain:
   ```bash
   # Set ADMIN_PUBKEY in anchor/programs/zkcash/src/lib.rs to your authority key first.
   anchor build --verifiable
   cp zkcash-keypair.json target/deploy/zkcash-keypair.json

   anchor deploy --verifiable --provider.cluster https://rpc.cookiescan.io

   # or, with the raw CLI:
   solana program deploy target/deploy/zkcash.so \
     --program-id zkcash-keypair.json \
     --upgrade-authority ./deploy-keypair.json \
     --url https://rpc.cookiescan.io
   ```

6. Initialize the pool (creates the Merkle tree + global config). Run this from the
   `scripts/` directory with `anchor/deploy-keypair.json` set to your admin authority:
   ```bash
   cd scripts && npm install && npx ts-node initialize_devnet.ts
   ```

7. Transfer the upgrade authority to your multisig (replace with your own keys):
   ```bash
   solana program set-upgrade-authority CASHcHkM2PHpCHaEhksmQrv6C9YRu3csxY2eyKKydHnv \
   --new-upgrade-authority <YOUR_MULTISIG_PUBKEY> \
   --upgrade-authority deploy-keypair.json \
   --skip-new-upgrade-authority-signer-check \
   --url https://rpc.cookiescan.io
   ```