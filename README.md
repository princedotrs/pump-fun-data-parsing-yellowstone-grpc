# pump-fun-data-parsing-yellowstone-grpc: a real-time Pump.fun mint monitor

A TypeScript service that streams Solana transactions over Yellowstone gRPC (Geyser), picks out every new token created on Pump.fun, and decodes it into clean JSON: mint, bonding curve, creator, name, symbol and metadata URI.

What you'll find here:

- subscribing to a Yellowstone gRPC stream with a transaction filter for one program
- matching an instruction by its 8-byte Anchor discriminator
- mapping instruction account indexes to named accounts
- decoding Borsh-encoded instruction arguments (strings and a public key) by hand
- resolving the token image from IPFS metadata, with fallback gateways

## What it does

For every Pump.fun `create` transaction it prints:

```javascript
{
  signature: 'Keju34ZKVvEHX5BDVDxvigEnXZ8QaEXAW48E5BWzmvyH41Ti6iZATnXQ8CwwrVSQDYC9qFXegvb1pxpdhuC2xJU',
  slot: '351135728',
  mint: '8Bdt4ViKGfj67NZZ5VfVgbhnjVySusXHxTZW5jMHpump',
  bonding_curve: 'gM75mej5DAL6RFeas1v6jRUbJ7e7uo9Er1VJEWuftdj',
  associated_bonding_curve: '4pcQJXCyYF5jJxLvG7NB5kZABSPwrHsPzcDVPRHAtd3Y',
  user: 'DnV3yLQK8FKDeb5F93nfVewCUVrpUqtianCiUxxhH9Zh',
  name: 'Dicken Strange',
  symbol: 'D.STRANGE',
  uri: 'https://ipfs.io/ipfs/bafkreie5wkau7ytjvnmwtsuvnsmprcw7lexxtzkhptzzho7cqdvibyh3rq',
  creator: 'DnV3yLQK8FKDeb5F93nfVewCUVrpUqtianCiUxxhH9Zh'
}
```

## How it works

1. **Subscribe.** The client opens a Yellowstone gRPC stream and subscribes, at `finalized` commitment, to transactions that touch the Pump.fun program (`6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P`).
2. **Match the instruction.** For each transaction it looks for an instruction whose first 8 bytes equal the `create` discriminator `[24, 30, 200, 40, 5, 28, 7, 119]`.
3. **Name the accounts.** Pump.fun's `create` instruction has a fixed account order, so the monitor reads `mint` (index 0), `bonding_curve` (2), `associated_bonding_curve` (3) and `user` (7).
4. **Decode the arguments.** After the discriminator come Borsh-encoded `name`, `symbol` and `uri` (each a 4-byte length plus UTF-8 bytes) and the `creator` public key (32 bytes).
5. **Fetch the image.** `getImageUri` reads the metadata JSON from several IPFS gateways in turn, so one slow gateway doesn't block it.

## Run it

Prerequisites: Node.js 18+ and access to a Yellowstone gRPC endpoint (Triton, Helius LaserStream, QuickNode and others provide one).

```bash
git clone https://github.com/princedotrs/pump-fun-data-parsing-yellowstone-grpc.git
cd pump-fun-data-parsing-yellowstone-grpc
npm install

cp .env.example .env
# set ENDPOINT and TOKEN in .env

npm run dev            # run with ts-node
# or
npm run build && npm start
```

The code currently checks that both `ENDPOINT` and `TOKEN` are set. Uncomment the authenticated client in `index.ts` if your endpoint needs the token.

## Project layout

```
index.ts         # stream subscription, filters, instruction decoding, IPFS lookup
pump-fun.json    # Pump.fun IDL, for reference
tokens.json      # sample raw stream updates
.env.example     # ENDPOINT and TOKEN
```

## Ideas to extend it

- Decode `buy` and `sell` instructions too and track each curve's progress
- Use an IDL-based decoder instead of manual offsets
- Write mints to a database or push them to a webhook
- Reconnect automatically with a resume-from-slot

## License

MIT
