# Simple-Eos-Api
# Synopsis

- Powered By [Liberty Block](https://libertyblock.io/), Simple EOS API enables on-chain integration of DEX's, Centralized Exchanges, Wallets and DAPPs, in a secure, scalable and reliable manner, without the need to run mainnet for EOS chain. In short:
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
- - A web embeddable ['stripe' wallet for EOS](https://api.byzanti.ne/wallet), that can be customized to work with QR codes on web, mobile or POS systems for accepting EOS/derivative digital assets
- - Reduces the complexity of rampup on EOS chain and prevents issues like the following:
- - [EosBetDice hacked using eosio.token transfer exploit](https://www.zdnet.com/article/blockchain-betting-app-mocks-competitor-for-getting-hacked-gets-hacked-four-days-later/)

- - [NewDEX hacked with fake EOS tokens](https://thenextweb.com/hardfork/2018/09/18/eos-hackers-exchange-fake/)

# [EOS 'Stripe' Wallet](some address)

# Design
The high-level design shown below provides an unified interface for all on-chain EOS operations.

- The key components are the API itself, which encapsulates the EOS-chain rpc/api/table methods and encapsulates the nuances by using eosjs and ecc.
- It uses cryptographic nonce to secure customers from replay attack.
- The design necessitates customers to aquire an API-KEY/SALT which is a one time excercise. The api-key is then used to fetch nonce for signing write-transactions and transmit the "sig" attribute as shown in curl examples.

![Alt text](/images/byzapi.png?raw=true "Byzantine API Gateway")

# API cheat sheet
*** The MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF apikey used to illustrate the methods is for test purpose only. If you require a key for industrial-grade calls/low latency please reach out to us with a few bytes on intended purpose at hello@byzanti.ne


### Curl Samples

```sh
// Base API Operations
// create EOS key sets - owner and active keys
curl -X GET --header 'Accept: application/json' --header 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' 'https://api.byzanti.ne/getKeyset' | json_pp

//create account
curl -X POST -H "Content-Type:application/json" -H 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' -d '{"creator":"gi3dcnjshege","name":"randomgooppy","owner":"EOS7m36vdT6WbE6JA25z9ePGhyWuqMYSLuCxLicMa1eLZ2YqSQqfh","active":"EOS59eusHMqbvJsPsdBKMNbuVHLz8kiif9NW27HQxiuge5iupvZec","sig":"6EF0AEFBFD50850D70366D5B7A6F04346BC81B2BDE0615CED49D803F1C2F042FAA42FF33723ADCC0E73CA4616603D29C4BF544FA515FB4BC1ECD55C9CE6DCF9E"}' https://api.byzanti.ne/createAccount | json_pp

//get info (for testing)
curl -X GET --header 'Content-Type: application/json'  --header 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' 'https://api.byzanti.ne/info' | json_pp

//get all *legit EOS tokens
curl -X GET --header 'Content-Type: application/json' --header 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' 'https://api.byzanti.ne/tokens' | json_pp

//get tokens by account
curl -X GET --header 'Content-Type: application/json' --header 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' 'https://api.byzanti.ne/tokensByAccount/gi3dcnjshege' | json_pp

//get tokens by account
curl -X GET --header 'Content-Type: application/json' --header 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' 'https://api.byzanti.ne/tokensByAccount/randomgooppy' | json_pp

//get account
curl -X GET --header 'Content-Type: application/json' --header 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' 'https://api.byzanti.ne/getAccount/gi3dcnjshege' | json_pp

//get account
curl -X GET --header 'Content-Type: application/json' --header 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' 'https://api.byzanti.ne/getAccount/randomgooppy' | json_pp

//get actions
curl -X GET --header 'Content-Type: application/json' --header 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' 'https://api.byzanti.ne/getActions?account=gi3dcnjshege' | json_pp

//transfer with pki
curl -X POST -H "Content-Type:application/json" -H 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' -d '{"from":"gi3dcnjshege","to":"randomgooppy","amount":"0.0001 EOS","memo":"random test","sig":"6EF0AEFBFD50850D70366D5B7A6F04346BC81B2BDE0615CED49D803F1C2F042FAA42FF33723ADCC0E73CA4616603D29C4BF544FA515FB4BC1ECD55C9CE6DCF9E"}' https://api.byzanti.ne/transfer | json_pp

//get transaction
curl -X GET --header 'Content-Type: application/json' --header 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' 'https://api.byzanti.ne/transaction/5a309398bc60d7f2849080d3b88646a22d8e9f682a5d257f1ac7672d5122688d' | json_pp

//get refunds
curl -X GET --header 'Accept: application/json'  --header 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' 'https://api.byzanti.ne/getRefunds/gi3dcnjshege' | json_pp

//get name bids
curl -X GET --header 'Accept: application/json' --header 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' 'https://api.byzanti.ne/getNameBids/reddy' | json_pp


//Block - Producer operations
//Get EOS producers
curl -X GET --header 'Content-Type: application/json' --header 'Accept: application/json'  --header 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' 'https://api.byzanti.ne/getProducers' | json_pp

//Vote for a producer libertyblock
curl -X POST -H "Content-Type:application/json"  -H 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' -d '{"voter":"gi3dcnjshege","producer":"libertyblock","sig":"XFBEk+="}' https://api.byzanti.ne/voteProducer | json_pp

//Vote for a producer eosfishrocks
curl -X POST -H "Content-Type:application/json"  -H 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' -d '{"voter":"gi3dcnjshege","producer":"eosfishrocks","sig":"XFBEk+="}' https://api.byzanti.ne/voteProducer | json_pp

/Delegate CPU/BW
curl -X POST -H "Content-Type:application/json"  -H 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' -d '{"from":"gi3dcnjshege","receiver":"gi3dcnjshege","net":"0.0001 EOS","cpu":"0.0001 EOS","sig":"6EF0AEFBFD50850D70366D5B7A6F04346BC81B2BDE0615CED49D803F1C2F042FAA42FF33723ADCC0E73CA4616603D29C4BF544FA515FB4BC1ECD55C9CE6DCF9E"}' https://api.byzanti.ne/delegate | json_pp

//Undelegate CPU/BW
curl -X POST -H "Content-Type:application/json"  -H 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' -d '{"from":"gi3dcnjshege","receiver":"gi3dcnjshege","net":"0.9188 EOS","cpu":"0.9188 EOS","sig":"XFBEk+="}' https://api.byzanti.ne/undelegate | json_pp

//get bandwidth
curl -X GET --header 'Content-Type: application/json'  --header 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF'  'https://api.byzanti.ne/getBandwidth/gi3dcnjshege' | json_pp

//Buy Ram
curl -X POST -H "Content-Type:application/json"  -H 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' -d '{"payer":"gi3dcnjshege","receiver":"radomgoopy","quant":"0.0001 EOS","sig":"XFBEk+="}' https://api.byzanti.ne/undelegate | json_pp

//Buy Ram in bytes
curl -X POST -H "Content-Type:application/json"  -H 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' -d '{"payer":"gi3dcnjshege","receiver":"gi3dcnjshege","bytes":240,"sig":"XFBEk+="}' https://api.byzanti.ne/buyRamBytes | json_pp

//Sell RAM in bytes
curl -X POST -H "Content-Type:application/json"  -H 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' -d '{"account":"gi3dcnjshege","bytes":84,"sig":"XFBEk+="}' https://api.byzanti.ne/sellRamBytes | json_pp

//Get RAM price
curl -X GET --header 'Content-Type: application/json'  --header 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF'  'https://api.byzanti.ne/getRamPrice' | json_pp

//Scatter based transfer
curl -X POST -H "Content-Type:application/json"  -H 'api_key: MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' -d '{"from":"gi3dcnjshege","to":"randomgooppy","amount":"0.0001 EOS","memo":"offline test","sig":"c77ac47879b2a8e622f9f301c98959cce5b97a53e4d42f5038d0d2d7cb78a0c3e3a135728fb5f5969a81f92cb0412727a040b143e12f57b533c7e0cc595ce965a6318cab00710549c3bc8984ec22b1c9c38f2db7e7e4cb6ba3bb48a3211db082c5315913977262004a4b8e0c052a8ee2","transactionHeaders":{"expiration": "2018-09-19T00:20:40", "ref_block_num": 19055, "ref_block_prefix": 4239914415}}' https://api.byzanti.ne/transferWithScatter | json_pp

// Basic DEX Functions
//get active trading symbols
curl -X GET --header 'Accept: application/json' 'https://api.byzanti.ne/symbols?api_key=MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' | json_pp

//get ticker (market data + all tokens)
curl -X GET --header 'Accept: application/json' 'https://api.byzanti.ne/ticker?api_key=MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' | json_pp

//get ticker (market data for specific token)
curl -X GET --header 'Accept: application/json' 'https://api.byzanti.ne/ticker?symbol=IQ&api_key=MHH0A3F-5NH4SHS-GEVSWJJ-654P7ZF' | json_pp

```

### Todos
- Add Synopsis, design aspects esp security, self-service api-key, the inner workings, etc for maintainability & supportability
