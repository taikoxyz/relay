# Gwyneth AA Pool Mode (Router-Delegated `eth_sendRawTransaction`)

When running the Porto relay against a Gwyneth node that uses the in-node AA pool (“alternate
mempool”), raw Porto intents should be submitted to the **Gwyneth RPC router** rather than the
canonical L1 txpool.

## Config

Keep reads/calls/traces pointed at the L1 RPC, but delegate *only* raw tx submission:

```yaml
chains:
  160010:
    endpoint: "http://host.docker.internal:32002/"
    eth_send_raw_delegates:
      - "http://host.docker.internal:32005/"
```

## Receipt semantics (important)

In AA pool mode, the hash returned from `eth_sendRawTransaction` is a **user-operation hash** (the
intent tx hash). It will **not** appear as an L1 transaction receipt under that hash.

The relay therefore confirms intents by polling:
- `eth_getUserOperationByHash(<intent_hash>)` on the router, until `state == Included`, then
- `eth_getTransactionReceipt(<propose_tx_hash>)` on L1 for the returned `transaction_hash`.

