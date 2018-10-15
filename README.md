# Simple-Eos-Api
# Synopsis

- Powered By [Liberty Block](https://libertyblock.io/), the [Byzantine API gateway](http://api.byzanti.ne:8902/docs/) enables EOS on-chain integration of DEX's, Centralized Exchanges, Wallets and DAPPs, in a secure, scalable and reliable manner, without the need to run mainnet for EOS chain locally
- Provides a simplified abstraction from individual token contract integrations and transfers while preventing replay attacks, validation of tokens with their respective contract hashes and ensures the transaction is secure and used the least latency route to chain
- The API gateway runs its own mainnet and also load balances across 21 block producers when the local mainnet blocks are delayed by >100ms
- For high throughput, the gateway also provides an optional request/response compression using Zstandard
- Provides access to eos-rpc api, history-api as well as custom api collection to simplify interacting with complex actions such as namebids,rambuys,getAllTokenBalances etc.
- Provides NPI action, data traceability to transactions made through the API gateway
- Provides ELK style Kibana Analytics Dashboard for monitoring your transactions, throughput etc
- Security is enabled through a combination of nonce, a private salt and the api-security-key. Using the latter 2 arguments, the client creates a 256-bit cipher which is used for signature. This prevents both replay attacks as well as a secure exchange of keys for signature
- Provides an easy abstraction for:
- - Get Airdrop/Airgrab Tokens with verifiable hashcodes, precision, contracts
- - Create Account, (Un)Delegate, Vote, RAM price, Buy/Sell RAM, Refunds, Account history
- - Transfer across any token with/without scatter
- - A web embeddable ['stripe' wallet for EOS](http://api.byzanti.ne:8902/wallet), that can be customized to work with QR codes on web, mobile or POS systems for accepting EOS/derivative digital assets
- - A DEX market aggregation data across EOS/Derivative assets
- - Reduces the complexity of rampup on EOS chain and prevents issues like the following:
- - [EosBetDice hacked using eosio.token transfer exploit](https://www.zdnet.com/article/blockchain-betting-app-mocks-competitor-for-getting-hacked-gets-hacked-four-days-later/)

- - [NewDEX hacked with fake EOS tokens](https://thenextweb.com/hardfork/2018/09/18/eos-hackers-exchange-fake/)

# [EOS 'Stripe' Wallet](http://api.byzanti.ne:8902/wallet)

```sh
//To embed wallet on your web site to accept any EOS/Derivative assets copy this snippet
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>EOS Wallet</title>
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
<div id="wallet" style="transform: translateY(150px); margin: 0 auto; width:700px"></div>
<script type="text/javascript" src="http://api.byzanti.ne:8902/main.js"></script>
</html>
```

# Design
The high-level design shown below provides an unified interface for all on-chain EOS operations.

- The key components are the API itself, which encapsulates the EOS-chain rpc/api/table methods and encapsulates the nuances by using eosjs and ecc.
- It uses cryptographic nonce to secure customers from replay attack.
- The design necessitates customers to aquire an API-KEY/SALT which is a one time excercise. The api-key is then used to fetch nonce for signing write-transactions and transmit the "sig" attribute as shown in curl examples.

![Alt text](/images/byzapi.png?raw=true "Byzantine API Gateway")

# API cheat sheet
*** The apikey used to illustrate the methods is for test purpose only. If you require a key for industrial-grade calls/low latency please reach out to us with a few bytes on intended purpose at hello@byzanti.ne

### EOS API Endpoint
- [API Explorer](http://api.byzanti.ne:8902/docs)
- [API json Docs](http://api.byzanti.ne:8902/api-docs)

### EOS API Analytics
[OpenAPI Analytics](http://api.byzanti.ne:8902/swagger-stats/ui#sws_summary)

### Curl Samples

```sh
// Base API Operations
// create EOS key sets - owner and active keys
curl -X GET --header 'Accept: application/json' --header 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' 'http://api.byzanti.ne:8902/getKeyset' | json_pp

//create account
curl -X POST -H "Content-Type:application/json" -H 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' -d '{"creator":"gi3dcnjshege","name":"randomgooppy","owner":"EOS7m36vdT6WbE6JA25z9ePGhyWuqMYSLuCxLicMa1eLZ2YqSQqfh","active":"EOS59eusHMqbvJsPsdBKMNbuVHLz8kiif9NW27HQxiuge5iupvZec","sig":"6EF0AEFBFD50850D70366D5B7A6F04346BC81B2BDE0615CED49D803F1C2F042FAA42FF33723ADCC0E73CA4616603D29C4BF544FA515FB4BC1ECD55C9CE6DCF9E"}' http://api.byzanti.ne:8902/createAccount | json_pp

//get info (for testing)
curl -X GET --header 'Content-Type: application/json'  --header 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' 'http://api.byzanti.ne:8902/info' | json_pp

//get all *legit EOS tokens
curl -X GET --header 'Content-Type: application/json' --header 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' 'http://api.byzanti.ne:8902/tokens' | json_pp

//get tokens by account
curl -X GET --header 'Content-Type: application/json' --header 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' 'http://api.byzanti.ne:8902/tokensByAccount/gi3dcnjshege' | json_pp

//get tokens by account
curl -X GET --header 'Content-Type: application/json' --header 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' 'http://api.byzanti.ne:8902/tokensByAccount/randomgooppy' | json_pp

//get account
curl -X GET --header 'Content-Type: application/json' --header 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' 'http://api.byzanti.ne:8902/getAccount/gi3dcnjshege' | json_pp

//get account
curl -X GET --header 'Content-Type: application/json' --header 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' 'http://api.byzanti.ne:8902/getAccount/randomgooppy' | json_pp

//get actions
curl -X GET --header 'Content-Type: application/json' --header 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' 'http://api.byzanti.ne:8902/getActions?account=gi3dcnjshege' | json_pp

//transfer with pki
curl -X POST -H "Content-Type:application/json" -H 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' -d '{"from":"gi3dcnjshege","to":"randomgooppy","amount":"0.0001 EOS","memo":"random test","sig":"6EF0AEFBFD50850D70366D5B7A6F04346BC81B2BDE0615CED49D803F1C2F042FAA42FF33723ADCC0E73CA4616603D29C4BF544FA515FB4BC1ECD55C9CE6DCF9E"}' http://api.byzanti.ne:8902/transfer | json_pp

//get transaction
curl -X GET --header 'Content-Type: application/json' --header 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' 'http://api.byzanti.ne:8902/transaction/5a309398bc60d7f2849080d3b88646a22d8e9f682a5d257f1ac7672d5122688d' | json_pp

//get refunds
curl -X GET --header 'Accept: application/json'  --header 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' 'http://api.byzanti.ne:8902/getRefunds/gi3dcnjshege' | json_pp

//get name bids
curl -X GET --header 'Accept: application/json' --header 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' 'http://api.byzanti.ne:8902/getNameBids/reddy' | json_pp


//Block - Producer operations
//Get EOS producers
curl -X GET --header 'Content-Type: application/json' --header 'Accept: application/json'  --header 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' 'http://api.byzanti.ne:8902/getProducers' | json_pp

//Vote for a producer libertyblock
curl -X POST -H "Content-Type:application/json"  -H 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' -d '{"voter":"gi3dcnjshege","producer":"libertyblock","sig":"XFBEk+="}' http://api.byzanti.ne:8902/voteProducer | json_pp

//Vote for a producer eosfishrocks
curl -X POST -H "Content-Type:application/json"  -H 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' -d '{"voter":"gi3dcnjshege","producer":"eosfishrocks","sig":"XFBEk+="}' http://api.byzanti.ne:8902/voteProducer | json_pp

/Delegate CPU/BW
curl -X POST -H "Content-Type:application/json"  -H 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' -d '{"from":"gi3dcnjshege","receiver":"gi3dcnjshege","net":"0.0001 EOS","cpu":"0.0001 EOS","sig":"6EF0AEFBFD50850D70366D5B7A6F04346BC81B2BDE0615CED49D803F1C2F042FAA42FF33723ADCC0E73CA4616603D29C4BF544FA515FB4BC1ECD55C9CE6DCF9E"}' http://api.byzanti.ne:8902/delegate | json_pp

//Undelegate CPU/BW
curl -X POST -H "Content-Type:application/json"  -H 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' -d '{"from":"gi3dcnjshege","receiver":"gi3dcnjshege","net":"0.9188 EOS","cpu":"0.9188 EOS","sig":"XFBEk+="}' http://api.byzanti.ne:8902/undelegate | json_pp

//get bandwidth
curl -X GET --header 'Content-Type: application/json'  --header 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N'  'http://api.byzanti.ne:8902/getBandwidth/gi3dcnjshege' | json_pp

//Buy Ram
curl -X POST -H "Content-Type:application/json"  -H 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' -d '{"payer":"gi3dcnjshege","receiver":"radomgoopy","quant":"0.0001 EOS","sig":"XFBEk+="}' http://api.byzanti.ne:8902/undelegate | json_pp

//Buy Ram in bytes
curl -X POST -H "Content-Type:application/json"  -H 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' -d '{"payer":"gi3dcnjshege","receiver":"gi3dcnjshege","bytes":240,"sig":"XFBEk+="}' http://api.byzanti.ne:8902/buyRamBytes | json_pp

//Sell RAM in bytes
curl -X POST -H "Content-Type:application/json"  -H 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' -d '{"account":"gi3dcnjshege","bytes":84,"sig":"XFBEk+="}' http://api.byzanti.ne:8902/sellRamBytes | json_pp

//Get RAM price
curl -X GET --header 'Content-Type: application/json'  --header 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N'  'http://api.byzanti.ne:8902/getRamPrice' | json_pp

//Scatter based transfer
curl -X POST -H "Content-Type:application/json"  -H 'api_key: FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' -d '{"from":"gi3dcnjshege","to":"randomgooppy","amount":"0.0001 EOS","memo":"offline test","sig":"c77ac47879b2a8e622f9f301c98959cce5b97a53e4d42f5038d0d2d7cb78a0c3e3a135728fb5f5969a81f92cb0412727a040b143e12f57b533c7e0cc595ce965a6318cab00710549c3bc8984ec22b1c9c38f2db7e7e4cb6ba3bb48a3211db082c5315913977262004a4b8e0c052a8ee2","transactionHeaders":{"expiration": "2018-09-19T00:20:40", "ref_block_num": 19055, "ref_block_prefix": 4239914415}}' http://api.byzanti.ne:8902/transferWithScatter | json_pp

// DEX Functions
//get active trading symbols
curl -X GET --header 'Accept: application/json' 'http://api.byzanti.ne:8902/symbols?api_key=FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' | json_pp

//get ticker (market data + all tokens)
curl -X GET --header 'Accept: application/json' 'http://api.byzanti.ne:8902/ticker?api_key=FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' | json_pp

//get ticker (market data for specific token)
curl -X GET --header 'Accept: application/json' 'http://api.byzanti.ne:8902/ticker?symbol=IQ&api_key=FQK0SYR-W4H4NP2-HXZ2PKH-3J8797N' | json_pp
```

### Todos
- Add Synopsis, design aspects esp security, self-service api-key, the inner workings, etc for maintainability & supportability
