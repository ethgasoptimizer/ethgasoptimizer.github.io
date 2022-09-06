# TOR/Onion RPC endpoints
Together with HTTP(S) and WebSockets endpoints, ZMOK offers endpoints as a TOR hidden service. This way, ZMOK and nobody else knows who sends the API request, from where and how. This means, that no service/user can be blocked by some other authority.

?> **INFO: TOR/Onion RPC endpoints are available for all paid accounts as an extension for free.**

## Sample CURL call
Sample Onion endpoint call via the shared ZMOK SOCKS5 proxy:

```sh
curl "http://zmok2uls65q5ceoxcarpjpa5hlpjxsmeqyapfy3l42ofklmrdbcs4cqd.onion/3c52e46b8527ee6ca27fb41cee87a077e662421a8bb5807f526c666275549175" \
--socks5-hostname "api.zmok.io:9150" \
-X POST \
-v \
-H 'content-type:application/json' \
--data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

## TOR Web3 ProviderEngine
When building safe/untraceable Web3 application, you may find this Web3 ProviderEngine enhanced with [TOR SOCKS5 proxy](https://www.npmjs.com/package/tor-web3-provider-engine) helpful.
