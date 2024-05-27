# Installation

## dependencies

- Clone the project `git clone https://github.com/OwshenNetwork/owshen --recurse-submodules`
- If you already cloned the project without the cloning submodules first running: `git submodule update --init --recursive`
- The option `--remote` was added to support updating to the latest tips of remote branches: `git submodule update --recursive --remote`
- Install Rust language: `curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh`
- Install Foundry: `https://book.getfoundry.sh/getting-started/installation`
- Install dependencies: `sudo apt-get install nodejs npm libgmp3-dev nasm nlohmann-json3-dev`
- Install Circom/SnarkJS: `npm i -g snarkjs circom`
- Install Owshen: `cd owshen && make install`
- For installing client dependencies we need to go to client route and: `yarn` or `npm install`
- Running proper Ganache localhost network: `ganache-cli -d --db chain`
  (We need to import first account from Ganache to metamask for local testing)
- Initialize your pub/priv keys and deploying dependencies by running `cargo run -- init --wallet test.json` (Your keys will be saved in `~/.owshen-wallet.json` - also you can running this command multiple times for testing purpose)
- For deploying the contracts and hash functions and also test tokens `cargo run --release -- deploy --endpoint "http://127.0.0.1:8545" --from "4f3edf983ac636a65a842ce7c78d9aa706d3b113bce9c46f30d7d21715b23b1d"  --name Localhost --config Localhost.json --chain-id "1337" --deploy-owshen --deploy-dive --deploy-hash-function`
  (The `--from` flag is private key that we use to deploying contract its set to first ganache private key)
- Make sure the client is built in root: `cd client/ && yarn build`
- For generating supported network file and also serve .zkey files in root of project run: `make assets` (This makes a combination of directories that are needed for windows mode)
- Run the wallet (GUI): `cargo run -- wallet --port 9000 --db test.json --mode test`

For testing purpose you should add localhost ganache-cli config to your metamask networks:

- chain_id: `1337`
- rpc_url: `http://127.0.0.1:8545`

(After that you need to import the first private-key in you metamask of ganache-cli for testing purpose)

- Beside `~/.owshen-wallet.json` we also have `.owshen-wallet-cache` which is cache for owshen contract events for more efficient event processing and reading data from it
