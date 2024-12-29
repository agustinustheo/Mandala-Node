# Manual Collator Setup Guide for Zombienet

## Prerequisites
- Zombienet already running with at least one collator
- `mandala-130` binary available
- Access to Polkadot.js Apps
- Sudo access for adding invulnerables

## Step 1: Verify Account

First, verify that your seed phrase generates the correct account (`vSiYWYAZp2mYXFrCpPD7kKqeHCRY6X9xH26K6BHpsEgYx6d9q`) in Polkadot.js Apps:

1. Go to Polkadot.js Apps
2. Navigate to Accounts -> Add Account
3. Enter the seed phrase:
   ```
   tape embrace rival cannon lawsuit step maximum carpet narrow involve term iron
   ```
4. Verify the generated address matches your intended collator address

## Step 2: Insert Keys

```bash
export BASE_PATH="/tmp/zombie-06abe17c6545c7c4066b24fe1821c2ff_-7665-60z27YM8rPmX/alice-collator/data"

# Insert AURA key with Sr25519
./mandala-130 key insert \
  --base-path $BASE_PATH \
  --scheme Sr25519 \
  --key-type aura \
  --chain /tmp/zombie-06abe17c6545c7c4066b24fe1821c2ff_-7665-60z27YM8rPmX/mandala-collator-1/cfg/local.json_rococo-local-2000.json \
  --suri "tape embrace rival cannon lawsuit step maximum carpet narrow involve term iron"

# Insert GRANDPA key with Ed25519
./mandala-130 key insert \
  --base-path $BASE_PATH \
  --scheme Ed25519 \
  --key-type gran \
  --chain /tmp/zombie-06abe17c6545c7c4066b24fe1821c2ff_-7665-60z27YM8rPmX/mandala-collator-1/cfg/local.json_rococo-local-2000.json \
  --suri "tape embrace rival cannon lawsuit step maximum carpet narrow involve term iron"
```

## Step 3: Start the Collator Node

```bash
./mandala-130 \
  --name alice-collator \
  --chain /tmp/zombie-06abe17c6545c7c4066b24fe1821c2ff_-7665-60z27YM8rPmX/mandala-collator-1/cfg/local.json_rococo-local-2000.json \
  --base-path $BASE_PATH \
  --listen-addr /ip4/0.0.0.0/tcp/44236/ws \
  --prometheus-external \
  --rpc-cors all \
  --unsafe-rpc-external \
  --rpc-methods unsafe \
  --prometheus-port 45664 \
  --rpc-port 37608 \
  --collator \
  --force-authoring \
  -- \
  --chain /tmp/zombie-06abe17c6545c7c4066b24fe1821c2ff_-7665-60z27YM8rPmX/mandala-collator-1/cfg/rococo-local.json \
  --execution wasm \
  --force-authoring \
  --port 41364 \
  --rpc-port 45038
```

Wait for the node to sync before proceeding.

## Step 4: Generate Session Keys

```bash
curl -H "Content-Type: application/json" -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "author_rotateKeys",
    "params": []
}' http://localhost:37608
```

Save the returned session key for the next step.

## Step 5: Set Session Keys

Using Polkadot.js Apps (connected to your parachain):
1. Navigate to Developer -> Extrinsics
2. Select "session" pallet
3. Choose "setKeys" function
4. Input the session key from Step 4
5. For proof, input "0x"
6. Submit transaction from the collator account

## Step 6: Add to Invulnerables

Still in Polkadot.js Apps:
1. Navigate to Developer -> Sudo
2. Select "invulnerables" pallet
3. Choose "addInvulnerable" function
4. Input your collator's account address
5. Submit using sudo account

## Step 7: Verify Invulnerable Status

Using Polkadot.js Apps:
1. Navigate to Developer -> Chain state
2. Select "invulnerables" pallet
3. Choose "invulnerables()" query
4. Click the "+" button to query
5. You should see your collator's account address in the returned array of invulnerables

## Troubleshooting

- Monitor node logs for block production errors
- Check if collator appears in active set (Network -> Parachains -> your parachain)
- Ensure collator account has sufficient funds
- Verify sudo account was used for adding to invulnerables
- Wait for the next session change to see the collator active

## Important Notes

- Keys must be inserted before starting the node
- All paths should match your Zombienet-generated paths
- Session changes may take some time before the collator becomes active
- Keep the node running throughout the entire process
- Make sure ports don't conflict with other nodes in your Zombienet setup