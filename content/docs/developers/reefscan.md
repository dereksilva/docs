---
title: "Reefscan"
description: "Reefscan is more than just a block explorer. It contains useful resources that developers can leverage for their dapps, such as the PostgreSQL database layer, GraphQL server and smart contract APIs."
lead: "Reefscan is more than just a block explorer. It contains useful resources that developers can leverage for their dapps, such as the PostgreSQL database layer, GraphQL server and smart contract APIs."
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu:
  docs:
    parent: "developers"
weight: 270
toc: true
---

## Reef Chain data
Most apps require a fast and efficient data source for things that are not easily available over raw
blockchain RPC, for example:
 - balances of all tokens that a user has
 - list of token holders for a particular coin
 - history of token trades on a DEX
 - history of user's interaction with a smart contract

Reefscan offers 3 solutions for various use cases:
 - GraphQL server for web apps
 - HTTP API for developers

### GraphQL
Reef uses a public GraphQL service from Subsquid that indexes all onchain events at blazing speeds.

It is available for both mainnet and testnet, under following URIs:

- Mainnet: [https://squid.subsquid.io/reef-explorer/graphql](https://squid.subsquid.io/reef-explorer/graphql)
- Testnet: [https://squid.subsquid.io/reef-explorer-testnet/graphql](https://squid.subsquid.io/reef-explorer-testnet/graphql)

The URIs provide a GraphQL Playground interface for testing queries as well.

Larger dApps in production can deploy their own indexer by cloning and creating new subsquid.io service from source code.
For fully featured indexer you can use https://github.com/reef-chain/subsquid-processor.
Dex indexer is available at https://github.com/reef-chain/subsquid-processor-dex.
You can also check an example of contract events indexer https://github.com/reef-chain/subsquid-evm-event-processor.
Each repo also has its setup description.

![](/docs/developers/the_graph.png)

#### Examples

Here is an example query for getting chain stats:
```
query ChainInfosQuery {
  chainInfos(limit: 10) {
    count
    id
  }
}
```
We can get contract events:
```
import { gql } from '@apollo/client';

export const CONTRACT_EVENTS_GQL = gql`
  query evmEvent(
      $address: String_comparison_exp!
      $blockId: bigint_comparison_exp!
      $topic0: String_comparison_exp
    ) {
      evm_event(
        order_by: [
          { block_id: desc }
          { extrinsic_index: desc }
          { event_index: desc }
        ]
        where: {
          _and: [
            { contract_address: $address }
            { topic_0: $topic0 }
            { method: { _eq: "Log" } }
            { block_id: $blockId }
          ]
        }
      ) {
        contract_address
        data_parsed
        data_raw
        topic_0
        topic_1
        topic_2
        topic_3
        block_id
        extrinsic_index
        event_index
      }
    }
`;
```

If you want to subscribe to contract events you can use subsquid.io service to easily run your own indexer and GQL subscriptions.
Another way is to use https://www.npmjs.com/package/@reef-chain/util-lib and subscribe to `getEvmEvents$` rx-js Observable.

We can also find tokens and their balances for a specific account:
```
query MyQuery($accountId: String) {
  tokenHolders(
    where: {
    signer: {id_eq: $accountId}},
    orderBy: balance_DESC, limit: 10
  ) {
    evmAddress
    balance
  }
}
```

## Smart contracts

### Querying Reefscan API
The querying API allows one to obtain the contract sources and ABI (if verified).
```
curl -s 'https://testnet.reefscan.com/api/contract/0xc12532e256D63F9A2C3b7Cc750ed5C136035AEe9' | jq .
```

Response:
```
{
  "address": "0xc12532e256d63f9a2c3b7cc750ed5c136035aee9",
  "bytecode": "0x608060405234801561001057600080fd5b5061012f806100206000396000f3fe6080604052348015600f57600080fd5b506004361060325760003560e01c80632e64cec11460375780636057361d146051575b600080fd5b603d6069565b6040516048919060c2565b60405180910390f35b6067600480360381019060639190608f565b6072565b005b60008054905090565b8060008190555050565b60008135905060898160e5565b92915050565b60006020828403121560a057600080fd5b600060ac84828501607c565b91505092915050565b60bc8160db565b82525050565b600060208201905060d5600083018460b5565b92915050565b6000819050919050565b60ec8160db565b811460f657600080fd5b5056fea26469706673582212206967f80107a3b8ab68d5b6075e84b822cbd472d325fe716af1eaa2a4fa2548f364736f6c63430008040033"
}
```

### Verification API
The verification API allows developers to automatically upload and verify the source code of their smart contracts.
```
curl 'https://testnet.reefscan.com/api/verificator/submit-verification' \
  --data-raw $'{"address":"0xc12532e256D63F9A2C3b7Cc750ed5C136035AEe9","arguments":"[]","name":"Storage","filename":"contracts/1_Storage.sol","target":"london","source":"{\\"contracts/1_Storage.sol\\":\\"// SPDX-License-Identifier: GPL-3.0\\\\n\\\\npragma solidity >=0.7.0 <0.9.0;\\\\n\\\\n/**\\\\n * @title Storage\\\\n * @dev Store & retrieve value in a variable\\\\n */\\\\ncontract Storage {\\\\n\\\\n    uint256 number;\\\\n\\\\n    /**\\\\n     * @dev Store value in variable\\\\n     * @param num value to store\\\\n     */\\\\n    function store(uint256 num) public {\\\\n        number = num;\\\\n    }\\\\n\\\\n    /**\\\\n     * @dev Return value\\\\n     * @return value of \'number\'\\\\n     */\\\\n    function retrieve() public view returns (uint256){\\\\n        return number;\\\\n    }\\\\n}\\"}","optimization":"false","compilerVersion":"v0.8.4+commit.c7e474f2","license":"GPL-3.0","runs":200}' \
  --compressed
```

[Remix](https://remix.reefscan.com) and [HardHat](https://github.com/reef-chain/hardhat-reef-examples) already support the verification API, and should be the default methods for
deploying smart contracts onto Reef chain.
