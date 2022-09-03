## Ethereum Private RPC
Set up private RPC endpoints (similar to Infura's) by running your own ethereum node. Supports both HTTPS and WSS endpoints.

[![License](http://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/bap2pecs/ethereum-private-rpc/main/LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/bap2pecs/ethereum-private-rpc/pulls)

## Basic Usage
First download necessary files from this repository to your host:
```
wget https://raw.githubusercontent.com/bap2pecs/ethereum-private-rpc/main/docker-compose.yml https://raw.githubusercontent.com/bap2pecs/ethereum-private-rpc/main/nginx.tmpl && wget https://raw.githubusercontent.com/bap2pecs/ethereum-private-rpc/main/.geth.env.example -O .geth.env
```
Then modify `.geth.env` values for:
- `VIRTUAL_HOST` and `LETSENCRYPT_HOST`: the host name for accesing the RPC endpoints. It should be a domain you own and you also need to set the DNS record to point the domain to the IP address of the server you wanted to run the containers.
- `VIRTUAL_PATH`: this is for hiding your endpoints from the public. You should use a long randomized string.
- `VIRTUAL_DEST`: should not be changed. "/" is to prevent passing sub-path to the `geth` RPC endpoints (or you will get 404 error).
- `VIRTUAL_PORT` and `VIRTUAL_WS_PORT`: no need to change this unless you modified the compose file to customize the ports for http and ws RPC endpoints from `geth`.

Finally run `docker-compose up -d` to start the containers. Then you can access the https and wss endpoints at:
- `https://subdomain.yourdomain.tld/{VIRTUAL_PATH}`
- `wss://subdomain.yourdomain.tld/{VIRTUAL_PATH}`

You might need to wait some time for the `geth` node to 1) download the data; 2) find some peers of the network. You can check those info via `docker-compose logs -f`.

## Advanced Usage
The compose file by default runs `geth` node in "light" mode and has some other default configs (e.g. `rpc.evmtimeout` set to 60s to avoid `eth_call` timeout. default is 5s). You are free to customized the way you want to run the node. For exampel, if you want to run a full node, you can modify the `command` section under `geth`. For more options of `geth`, please refer to the official [documentaion](https://geth.ethereum.org/docs/interface/command-line-options).

You can even run a different software (e.g. [nethermind](https://hub.docker.com/r/nethermind/nethermind)) for the ethereum node as long as you config it correctly to expose the ports for the RPC endpoints.

## Notes
Initially, I wanted to use `nginx-proxy` but it does not support multi-port in a single container (see related [discusion](https://github.com/nginx-proxy/nginx-proxy/pull/1157/)). Since `geth` can expose both http and ws ports, I have to use `nginx` and `docker-gen` separately per the instruction at [here](https://github.com/nginx-proxy/nginx-proxy#separate-containers) and [here](https://github.com/nginx-proxy/acme-companion/blob/main/docs/Advanced-usage.md) and do some customization for the [nginx.tmpl](https://github.com/nginx-proxy/nginx-proxy/blob/main/nginx.tmpl) template.

The major modification I did in the template was to support `VIRTUAL_WS_PORT` for the same `VIRTUAL_HOST`. In the generated nginx config file, the server for the `VIRTUAL_HOST` will proxy both https and wss requests. It's achieved by using a map variable in the location of `proxy_pass` to dynamically route requests to different upstreams based on the `Upgrade` request header (see [more details](https://nginx.org/en/docs/http/websocket.html)). The modified template will correctly put the http RPC `address:port` URLs into one `upstream` directive and the ws RPC `address:port` URLs into another `upstream` directive.

## Contribution
Thank you for considering contributing to the repo! Anyone is welcomed to submit pull request, even for the smallest fixes or typos.

If you'd like to contribute, please create a new branch and submit a pull request for review.

## License
[MIT](https://raw.githubusercontent.com/bap2pecs/ethereum-private-rpc/main/LICENSE)