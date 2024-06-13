
```json



{


    "bep3": {
      "params": {
        "asset_params": [
          {
            "denom": "btcb",
            "coin_id": "0",
            "supply_limit": {
              "limit": "100000000000",
              "time_limited": false,
              "time_period": "0",
              "time_based_limit": "0"
            },
            "active": true,
            "deputy_address": "kava1kla4wl0ccv7u85cemvs3y987hqk0afcv7vue84",
            "fixed_fee": "2",
            "min_swap_amount": "3",
            "max_swap_amount": "2000000000",
            "min_block_lock": "24686",
            "max_block_lock": "86400"
          },
          {
            "denom": "xrpb",
            "coin_id": "144",
            "supply_limit": {
              "limit": "2000000000000000",
              "time_limited": false,
              "time_period": "0",
              "time_based_limit": "0"
            },
            "active": true,
            "deputy_address": "kava14q5sawxdxtpap5x5sgzj7v4sp3ucncjlpuk3hs",
            "fixed_fee": "100000",
            "min_swap_amount": "100001",
            "max_swap_amount": "250000000000000",
            "min_block_lock": "24686",
            "max_block_lock": "86400"
          },
          {
            "denom": "bnb",
            "coin_id": "714",
            "supply_limit": {
              "limit": "100000000000000",
              "time_limited": false,
              "time_period": "0",
              "time_based_limit": "0"
            },
            "active": true,
            "deputy_address": "kava1agcvt07tcw0tglu0hmwdecsnuxp2yd45f3avgm",
            "fixed_fee": "1000",
            "min_swap_amount": "1001",
            "max_swap_amount": "500000000000",
            "min_block_lock": "24686",
            "max_block_lock": "86400"
          },
          {
            "denom": "busd",
            "coin_id": "727",
            "supply_limit": {
              "limit": "2000000000000000",
              "time_limited": false,
              "time_period": "0",
              "time_based_limit": "0"
            },
            "active": true,
            "deputy_address": "kava1j9je7f6s0v6k7dmgv6u5k5ru202f5ffsc7af04",
            "fixed_fee": "20000",
            "min_swap_amount": "20001",
            "max_swap_amount": "100000000000000",
            "min_block_lock": "24686",
            "max_block_lock": "86400"
          }
        ]
      },
      "atomic_swaps": [],
      "supplies": [],
      "previous_block_time": "1970-01-01T00:00:01Z"
    },
 



```

这段JSON代码是关于"BEP3"的参数配置，它是与跨链资产转移相关的。让我们逐个解释每个字段的含义：

1. **bep3**: 这是一个参数对象，其中包含与BEP3跨链协议相关的所有参数。

2. **params**: 这是BEP3协议的参数。

3. **asset_params**: 这是一个资产参数数组，包含了所有支持的资产的详细信息。

    - **denom**: 资产的标识符。

    - **coin_id**: 资产的唯一标识符，通常是在跨链交易中使用的。

    - **supply_limit**: 这是资产的供应限制参数。

        - **limit**: 资产的总供应限制。

        - **time_limited**: 表示供应是否受时间限制。

        - **time_period**: 时间期限。

        - **time_based_limit**: 基于时间的限制。

    - **active**: 表示资产是否处于活跃状态。

    - **deputy_address**: 跨链交易中的代理地址，用于验证跨链交易的有效性。

    - **fixed_fee**: 固定交易费用。

    - **min_swap_amount**: 最小交换数量。

    - **max_swap_amount**: 最大交换数量。

    - **min_block_lock**: 最小锁定区块数量。

    - **max_block_lock**: 最大锁定区块数量。

在这个配置中，似乎有两种资产：一种是`btcb`，另一种是`xrpb`。每种资产都有自己的参数设置，包括供应限制、活跃状态、交易费用等等。