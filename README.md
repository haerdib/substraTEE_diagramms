| №  | Title                                                                                        | Depends on | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|----|----------------------------------------------------------------------------------------------|------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1  | Implement Rpc interface                                                                      | -          | Implement a clean worker-api-direct invocation interface: the client calls into enclave via rpc call (place_order, withdraw -> dummy functions only).<br>The following rpc functions should be callable: <br>• place_order<br>• cancel_order<br>• withdraw<br>• get_balance<br><br>remove obsolete rpc functions, including the top pool.<br>                                                                                                                                                                                                                        |
| 2  | Implement Balance Storage in  the enclave                                                    | -          | In a first step, all balances are stored in a HashMap:<br> (CurrencyID, AccountID) -> (balance free, balance reserved)<br>To interact with the storage, an interface should be provided, offering the following interactions: <br>* read balance of a specific account and ID <br> * Mutate balances of a specific account and ID <br> * regular snapshotting to store balances in IPFS with committing the IPFS cid to polkadex chain (ocall to worker) __TODO: IPFS or disc?__ <br> *  (Future work, but should be considered during design phase)update balances from a snapshot            |
| 3  | Proxy register                                                                               | -          | The proxy register needs to be loaded from the onchain state to the local enclave memory (e.g. atomic pointer) at initial start up (init chain relay function or something). <br> The following two interaction function should be provided: <br>* Check if the given AccountID is registered <br>* Check if the given AccountID (proxy) and AccountID (main) are assosciated <br> * add proxy account to list <br> * remove proxy account from list                                                                                                                     |
| 4  | reimplement indirect invocation upon new onchain block                                       | -          | The worker subscribes to polkadex chain and upon new finalized block, it gets the block header and calls into the enclave.                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| 5  | get_balance call via rpc interface                                                           | 1,2,3      | Restructure the current trusted getter get_balance such that the balance will not be called from stf state but from the balance hash map.<br> For call authorisation, the signature and nonce need to be verified (can be more or less copied from current stf checks) <br> Additionally, it needs to be checked if the calling proxy account is registered.                                                                                                                                                                                                             |
| 6  | Trusted operation call withdraw via rpc interface                                            | 1,2,3,5    | Implement a trusted call that allows a proxy account to release a choosabel amount of a currencyID of the main account.<br> For authenticiation, the nonce and signature need to be checked, aswell as the asssociation of the registered (!) proxy and the main account. <br> Proxy account free reserve > amount withdraw <br> Upon succes the enclave will update local balance map (move money from proxy to main) and perform an ocall to the worker to call the ocex pallet::release function to mutate onchain balance of main acc.                             |
| 7  | Implement openFinex jsonrpc api & ws listening to updates on Finex side<br>subscribe matches | -          | upon PolkaDEX GW call, the api needs to be able to create the corresponding json string to call the rpc OpenFinex RPC server. <br> Upon worker start, a websocket client within enclave should be started that is listening to the openFinex server for match updated (subscribe_matches)                                                                                                                                                                                                                                                                                |
| 8  | Add Orderbook mirror storage & interface in worker and enclave                               |            | Workerside: Design the orderbook mirror DB such that it allows the storage of a order and the removal of it. Each order is stored in plain text (human readable) in a file. <br> To lower delay as much as possible: start new thread to store order in mirror db (for each order one file) and do not wait for thread to finish before returning. <br> Enclave side: Implement an interface that allows easy storage and removal of an oder: sign the file & ocalls into the worker to store the file there. <br> __Verify match: Disc call or local memorxy storage?__ |
| 9  | Implement rpc functions place_oder and cancel_order                                          | 1,2,7,8    | Via Trusted Call. Authentication of proxy&assocication to main acc. Verify nonce & signature. & if proxy owns enough money of currencyID<br> Upon successful authentication, store order via storage interface in worker __(in memory storage aswell?)__<br> Then reserve balance of proxy account on local enclave store <br> Then issue corresponing call to finex api. Upon Ok from Finex return OrderID to client                                                                                                                                                    |
| 10 | Implement match_verify upon Finex ws call                                                    | 2,7        | When ws receives a call from openFinex, it calls the Polkadex GW, which verifies the following: <br>* both orders are stored in enclave <br>* authenticates the sender (SSL, certificate matching engine) <br>* orders are matching in terms of currencyID and limits <br> If successful, mutate balances accordingly and remove orders from the order list                                                                                                                                                                                                              |
| 11 | Implement register_proxy and update_balance                                                  | 3,4        | The enclave iterates through all received block. Upon "register proxy" detection, the enclave adds the proxy account to its proxy list. Upon update_balance it mutates the main account accordingly (free -> reserved)                                                                                                                                                                                                                                                                                                                                                   |
| 12 | Demo bash script                                                                             | last       | A demo bash script will be supplied to show usage examples<br>• create account and fund from faucet • register proxy account (with no onchain balance)<br>• deposit funds<br>• send orders<br>• cancel orders<br>• subscribe to matches<br>• withdraw with direct or indirect invocation                                                                                                                                                                                                                                                                                 |


