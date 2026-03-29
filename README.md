# INITIA
1.mkdir my-initia-project
2.cd my-initia-project
3.apt-get update
4.apt-get install -y lz4 curl wget tar
5.wget https://go.dev/dl/go1.25.8.linux-amd64.tar.gz
6.sudo rm -rf /usr/local/go
7.sudo tar -C /usr/local -xzf go1.25.8.linux-amd64.tar.gz

8.echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
9.source ~/.bashrc
10.wget https://github.com/initia-labs/weave/releases/download/v0.3.8/weave-0.3.8-linux-amd64.tar.gz
11.tar -xzf weave-0.3.8-linux-amd64.tar.gz
12.chmod +x weave
13.sudo mv weave /usr/local/bin/
14.export TERM=xterm-256color
15.script -q -c "weave version" /dev/null
16.script -q -c "weave init" /dev/null


Stop and remove everything from the previous attempt

1.
docker rm -f weave-relayer 2>/dev/null || true
docker volume rm weave-relayer-data 2>/dev/null || true
docker volume prune -f
rm -rf ~/.weave ~/.initia ~/.minitia ~/.opinit ~/.relayer


This must return a block height number — if it times out, wait and retry
2.
curl -s https://rpc.testnet.initia.xyz/status | jq '.result.sync_info.latest_block_height'

If that times out, the Initia testnet RPC is temporarily down — wait 5–10 minutes and try again. Don't run weave init until this returns a number.

3.
docker volume create test-vol && echo "Docker OK" && docker volume rm test-vol

Must print Docker OK. If not — restart Docker Desktop from Windows tray first.

4.
weave init

Gas Station setup       → "Generate new account (recommended)"

You will see an init1... address on screen.
5.
Copy that address. Open a browser. Go to:
https://app.testnet.initia.xyz/faucet
Paste the init1... address directly into the text box.
Do NOT connect MetaMask. Do NOT use your own wallet address.
Click Submit. Wait for sometime.

Then open a NEW terminal tab and verify the balance arrived:
6.
curl -s "https://rest.testnet.initia.xyz/cosmos/bank/v1beta1/balances/PASTE_YOUR_init1_ADDRESS_HERE" | jq '.balances'
You must see uinit amount > 0 before continuing.
If it shows [] or 0 — go back to the faucet and try again.
(In the first terminal)
7.
Type continue             → continue
What do you want to do    → "Launch a new rollup"
Select L1 network         → "Testnet (initiation-2)"
Select VM                 → "EVM"
Rollup chain ID           → pixelvault-1   ← SAVE THIS
Gas denom                 → press Tab (default: umin)
Node moniker              → press Tab (default: operator)
Submission interval       → press Tab (default: 1m)
Finalization period       → press Tab (default: 168h)
lock data location       → "Initia L1"
Oracle price feed         → "Enable"
System keys               → "Generate new system keys"
Funding option            → "Use the default preset"
Fee whitelist             → press Enter (leave empty)
Add Gas Station to genesis→ "Yes"
Genesis balance           → 1000000000000000000000000
Add genesis accounts      → "No"
Type continue             → continue
Confirm transactions      → y

8.
find ~/.weave -name "minitiad" 2>/dev/null
It will show something like:
/root/.weave/data/minievm@v1.2.15/minitiad
Copy the DIRECTORY part (not the file), e.g: /root/.weave/data/minievm@v1.2.15

Replace the version below with whatever version you see above:
9.
echo 'export PATH=$PATH:/root/.weave/data/minievm@v1.2.15' >> ~/.bashrc
source ~/.bashrc

Verify — must print a version number
10.
minitiad version

11.
weave opinit init executor

12.

INTERACTIVE PROMPTS:
Existing keys detected    → "Yes, use detected keys"
System key for Oracle     → "Generate new system key"
Pre-fill from config      → "Yes, prefill"
Listen address            → press Tab (localhost:3000)
L1 RPC / Chain ID / Denom → press Enter for each
Rollup RPC                → press Tab (http://localhost:26657)

13.
weave opinit start executor -d

14.
STOP HERE — DO THIS ON WINDOWS FIRST 
"Quit Docker Desktop"
Reopen Docker Desktop.
Run the container
Then come back to WSL terminal and continue.

Test Docker volumes work — must complete without errors:
docker volume create test-vol && docker volume rm test-vol
# If this gives "input/output error" → restart Docker Desktop again

15.
weave relayer init


16.
INTERACTIVE PROMPTS:
Select rollup             → "Local Rollup (pixelvault-1)"
RPC endpoint              → press Tab (http://localhost:26657)
REST endpoint             → press Tab (http://localhost:1317)
IBC channel method        → "Subscribe to only transfer and nft-transfer"
Select IBC channels       → press Space to select all → press Enter
Challenger key            → "Yes (recommended)"

17.
weave relayer start -d

18.
MNEMONIC=$(jq -r '.common.gas_station.mnemonic' ~/.weave/config.json)

19.
initiad keys add gas-station \
  --recover \
  --keyring-backend test \
  --coin-type 60 \
  --key-type eth_secp256k1 \
  --source <(echo -n "$MNEMONIC")

20.
minitiad keys add gas-station \
  --recover \
  --keyring-backend test \
  --coin-type 60 \
  --key-type eth_secp256k1 \
  --source <(echo -n "$MNEMONIC")


Verify both show "gas-station" in the list:
21.
initiad keys list --keyring-backend test
minitiad keys list --keyring-backend test


Chain is producing blocks — must return a number > 0:

22.
curl -s http://localhost:26657/status | jq '.result.sync_info.latest_block_height'

Gas Station has L2 balance — must show tokens:
23.
minitiad q bank balances \
  $(minitiad keys show gas-station -a --keyring-backend test) \
  --node http://localhost:26657

EVM JSON-RPC is live — must return a hex block number:
24.
curl -s -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

Relayer is running — check for activity, no crash errors:

25.
weave relayer log

26.
weave rollup start -d
weave opinit start executor -d

## Setup is complete. In this folder only we will be writing our entire codebase.

