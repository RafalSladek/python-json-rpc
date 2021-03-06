# Example usage of json-rpc in Python for Monero

Monero is a secure, private, untraceable cryptocurrency. For more information or questions,
please go to [getmonero.org](https://getmonero.org) and
[r/Monero](https://www.reddit.com/r/Monero), respectively.

The two main components of monero are `simplewallet` and `bitmonerod`. The first one
is the wallet, as the name suggest. The second one is monero deamon, which is responsbile
for interacting with monero blockchain.

Most important functions of Monero's `simplewallet` and
`bitmonreod` can be executed by means of JavaScript Object Notation Remote Procedure Calls ([json-rpc](https://en.wikipedia.org/wiki/JSON-RPC)).

Using these procedures, other applications can be developed
on top of the `simplewallet` and `bitmonerod`. For examples, a GUI wallet,
an web applications allowing for accessing wallet balance
online, and block explorer.

Since there seem to be no tutorials and/or examples of how
to use json-rpc to interact with both `bitmonerod` and `simplewallet`,
 the following examples in Python were created. Hopefully, they will allow others to start developing some python
programs on top of Monero.

## simplewallet
The examples demonstrate how to call the most popular procedures
that `simplewallet` exposes for other applications to use, such as:

 - getbalance
 - query_key
 - get_payments
 - getaddress
 - incoming_transfers
 - transfer

The basic documentaion of the procedures can be found
[here](https://getmonero.org/knowledge-base/developer-guides/wallet-rpc).

**Prerequsits**

Before executing this code make sure that `simplewallet` is
running and listening for the incoming rpc calls. For example, you can run the `simplewallet `in rpc mode as follows:
```
/opt/bitmonero/simplewallet --wallet-file ~/wallet.bin --password <wallet_password> --rpc-bind-port 18082
```

The code was written, tested and executed on Ubuntu 15.10 with
Python 3.4.3 and requires the [Requests package](https://pypi.python.org/pypi/requests).

**Basic example 1: get wallet balance**
```python
import requests
import json

def main():

    # simple wallet is running on the localhost and port of 18082
    url = "http://localhost:18082/json_rpc"

    # standard json header
    headers = {'content-type': 'application/json'}

    # simplewallet' procedure/method to call
    rpc_input = {
           "method": "getbalance"
    }

    # add standard rpc values
    rpc_input.update({"jsonrpc": "2.0", "id": "0"})

    # execute the rpc request
    response = requests.post(
        url,
        data=json.dumps(rpc_input),
        headers=headers)

    # amounts in cryptonote are encoded in a way which is convenient
    # for a computer, not a user. Thus, its better need to recode them
    # to something user friendly, before displaying them.
    #
    # For examples:
    # 4760000000000 is 4.76
    # 80000000000   is 0.08
    #
    # In example 3 "Basic example 3: get incoming transfers" it is
    # shown how to convert cryptonote values to user friendly format.

    # pretty print json output
    print(json.dumps(response.json(), indent=4))

if __name__ == "__main__":
    main()
```

Generated output:
```python
{
    "jsonrpc": "2.0",
    "id": "0",
    "result": {
        "unlocked_balance": 4760000000000,
        "balance": 4760000000000
    }
}
```



**Basic example 2: get a payment information having payment id**
```python
import requests
import json

def main():

    # simple wallet is running on the localhost and port of 18082
    url = "http://localhost:18082/json_rpc"

    # standard json header
    headers = {'content-type': 'application/json'}

    # an example of a payment id
    payment_id = "426870cb29c598e191184fa87003ca562d9e25f761ee9e520a888aec95195912"

    # simplewallet' procedure/method to call
    rpc_input = {
        "method": "get_payments",
        "params": {"payment_id": payment_id}
    }

    # add standard rpc values
    rpc_input.update({"jsonrpc": "2.0", "id": "0"})

    # execute the rpc request
    response = requests.post(
        url,
        data=json.dumps(rpc_input),
        headers=headers)

    # pretty print json output
    print(json.dumps(response.json(), indent=4))

if __name__ == "__main__":
    main()
```

Generated output:
```python
{
    "result": {
        "payments": [
            {
                "tx_hash": "66040ad29f0d780b4d47641a67f410c28cce575b5324c43b784bb376f4e30577",
                "amount": 4800000000000,
                "block_height": 795523,
                "payment_id": "426870cb29c598e191184fa87003ca562d9e25f761ee9e520a888aec95195912",
                "unlock_time": 0
            }
        ]
    },
    "jsonrpc": "2.0",
    "id": "0"
}
```



**Basic example 3: get incoming transfers**

```python
import requests
import json

def main():

    # simple wallet is running on the localhost and port of 18082
    url = "http://localhost:18082/json_rpc"

    # standard json header
    headers = {'content-type': 'application/json'}

    # simplewallet' procedure/method to call
    rpc_input = {
            "method": "incoming_transfers",
            "params": {"transfer_type": "all"}
    }

    # add standard rpc values
    rpc_input.update({"jsonrpc": "2.0", "id": "0"})

    # execute the rpc request
    response = requests.post(
         url,
         data=json.dumps(rpc_input),
         headers=headers)

    # make json dict with response
    response_json = response.json()

    # amounts in cryptonote are encoded in a way which is convenient
    # for a computer, not a user. Thus, its better need to recode them
    # to something user friendly, before displaying them.
    #
    # For examples:
    # 4760000000000 is 4.76
    # 80000000000   is 0.08
    #
    if "result" in response_json:
        if "transfers" in response_json["result"]:
            for transfer in response_json["result"]["transfers"]:
                transfer["amount"] = float(get_money(str(transfer["amount"])))


    # pretty print json output
    print(json.dumps(response_json, indent=4))

def get_money(amount):
    """decode cryptonote amount format to user friendly format. Hope its correct.

    Based on C++ code:
    https://github.com/monero-project/bitmonero/blob/master/src/cryptonote_core/cryptonote_format_utils.cpp#L751
    """

    CRYPTONOTE_DISPLAY_DECIMAL_POINT = 12

    s = amount

    if len(s) < CRYPTONOTE_DISPLAY_DECIMAL_POINT + 1:
        # add some trailing zeros, if needed, to have constant width
        s = '0' * (CRYPTONOTE_DISPLAY_DECIMAL_POINT + 1 - len(s)) + s

    idx = len(s) - CRYPTONOTE_DISPLAY_DECIMAL_POINT

    s = s[0:idx] + "." + s[idx:]

    return s

if __name__ == "__main__":
    main()
```

Generated output:
```python
{
    "jsonrpc": "2.0",
    "result": {
        "transfers": [
            {
                "tx_hash": "<66040ad29f0d780b4d47641a67f410c28cce575b5324c43b784bb376f4e30577>",
                "tx_size": 521,
                "spent": true,
                "global_index": 346865,
                "amount": 0.8
            },
            {
                "tx_hash": "<66040ad29f0d780b4d47641a67f410c28cce575b5324c43b784bb376f4e30577>",
                "tx_size": 521,
                "spent": true,
                "global_index": 177947,
                "amount": 4.0
            },
            {
                "tx_hash": "<79e7eb67b7022a21505fa034388b5e3b29e1ce639d6dec37347fefa612117ce9>",
                "tx_size": 562,
                "spent": false,
                "global_index": 165782,
                "amount": 0.08
            },
            {
                "tx_hash": "<79e7eb67b7022a21505fa034388b5e3b29e1ce639d6dec37347fefa612117ce9>",
                "tx_size": 562,
                "spent": false,
                "global_index": 300597,
                "amount": 0.9
            },
            {
                "tx_hash": "<79e7eb67b7022a21505fa034388b5e3b29e1ce639d6dec37347fefa612117ce9>",
                "tx_size": 562,
                "spent": false,
                "global_index": 214803,
                "amount": 3.0
            },
            {
                "tx_hash": "<e8409a93edeed9f6c67e6716bb180d9593e8beafa63d51facf68bee233bf694d>",
                "tx_size": 525,
                "spent": false,
                "global_index": 165783,
                "amount": 0.08
            },
            {
                "tx_hash": "<e8409a93edeed9f6c67e6716bb180d9593e8beafa63d51facf68bee233bf694d>",
                "tx_size": 525,
                "spent": false,
                "global_index": 375952,
                "amount": 0.7
            }
        ]
    },
    "id": "0"
}
```

**Basic example 4: make a transaction**
```python
import requests
import json
import os
import binascii


def main():
    """DONT RUN IT without changing the destination address!!!"""

    # simple wallet is running on the localhost and port of 18082
    url = "http://localhost:18082/json_rpc"

    # standard json header
    headers = {'content-type': 'application/json'}

    destination_address = "489MAxaT7xXP3Etjk2suJT1uDYZU6cqFycsau2ynCTBacncWVEwe9eYFrAD6BqTn4Y2KMs7maX75iX1UFwnJNG5G88wxKoj"


    # amount of xmr to send
    amount = 0.54321

    # cryptonote amount format is different then
    # that normally used by people.
    # thus the float amount must be changed to
    # something that cryptonote understands
    int_amount = int(get_amount(amount))

    # just to make sure that amount->coversion->back
    # gives the same amount as in the initial number
    assert amount == float(get_money(str(int_amount))), "Amount conversion failed"

    # send specified xmr amount to the given destination_address
    recipents = [{"address": destination_address,
                  "amount": int_amount}]

    # using given mixin
    mixin = 4

    # get some random payment_id
    payment_id = get_payment_id()

    # simplewallet' procedure/method to call
    rpc_input = {
        "method": "transfer",
        "params": {"destinations": recipents,
                   "mixin": mixin,
                   "payment_id" : payment_id}
    }

    # add standard rpc values
    rpc_input.update({"jsonrpc": "2.0", "id": "0"})

    # execute the rpc request
    response = requests.post(
         url,
         data=json.dumps(rpc_input),
         headers=headers)

    # print the payment_id
    print("#payment_id: ", payment_id)

    # pretty print json output
    print(json.dumps(response.json(), indent=4))


def get_amount(amount):
    """encode amount (float number) to the cryptonote format. Hope its correct.

    Based on C++ code:
    https://github.com/monero-project/bitmonero/blob/master/src/cryptonote_core/cryptonote_format_utils.cpp#L211
    """

    CRYPTONOTE_DISPLAY_DECIMAL_POINT = 12

    str_amount = str(amount)

    fraction_size = 0

    if '.' in str_amount:

        point_index = str_amount.index('.')

        fraction_size = len(str_amount) - point_index - 1

        while fraction_size < CRYPTONOTE_DISPLAY_DECIMAL_POINT and '0' == str_amount[-1]:
            print(44)
            str_amount = str_amount[:-1]
            fraction_size = fraction_size - 1

        if CRYPTONOTE_DISPLAY_DECIMAL_POINT < fraction_size:
            return False

        str_amount = str_amount[:point_index] + str_amount[point_index+1:]

    if not str_amount:
        return False

    if fraction_size < CRYPTONOTE_DISPLAY_DECIMAL_POINT:
        str_amount = str_amount + '0'*(CRYPTONOTE_DISPLAY_DECIMAL_POINT - fraction_size)

    return str_amount


def get_money(amount):
    """decode cryptonote amount format to user friendly format. Hope its correct.

    Based on C++ code:
    https://github.com/monero-project/bitmonero/blob/master/src/cryptonote_core/cryptonote_format_utils.cpp#L751
    """

    CRYPTONOTE_DISPLAY_DECIMAL_POINT = 12

    s = amount

    if len(s) < CRYPTONOTE_DISPLAY_DECIMAL_POINT + 1:
        # add some trailing zeros, if needed, to have constant width
        s = '0' * (CRYPTONOTE_DISPLAY_DECIMAL_POINT + 1 - len(s)) + s

    idx = len(s) - CRYPTONOTE_DISPLAY_DECIMAL_POINT

    s = s[0:idx] + "." + s[idx:]

    return s

def get_payment_id():
    """generate random payment_id

    generate some random payment_id for the
    transactions

    payment_id is 32 bytes (64 hexadecimal characters)
    thus we first generate 32 random byte array
    which is then change to string representation, since
    json will not not what to do with the byte array.
    """

    random_32_bytes = os.urandom(32)
    payment_id = "".join(map(chr, binascii.hexlify(random_32_bytes)))

    return payment_id


if __name__ == "__main__":
    main()
```

Generated output:

```python
#payment_id:  4926869b6b5d50b24cb59f08fd76826cacdf76201b2d4648578fe610af7f786e
{
    "id": "0",
    "jsonrpc": "2.0",
    "result": {
        "tx_key": "",
        "tx_hash": "<04764ab4855b8a9f9c42d99e19e1c40956a502260123521ca3f6488dd809797a>"
    }
}
```

Other examples are [here](https://github.com/moneroexamples/python-json-rpc/blob/master/src/simplewallet_rpc_examples.py)

## bitmonreod

The baisc `bitmonerod` rpc calls are as follows:

 - getheight
 - query_key
 - mining_status
 - getlastblockheader
 - getblockheaderbyhash
 - getblockheaderbyheight
 - getblock
 - get_info
 - get_connections

**Prerequsits**
Before executing this code make sure that `bitmonerod` is running.
Just like before, the code was written, tested and executed on Ubuntu 15.10 with
Python 3.4.3 and it requires the [Requests package](https://pypi.python.org/pypi/requests).


**Basic example 1: get a mining status**
```python
import requests
import json

def main():

    # bitmonerod' is running on the localhost and port of 18081
    url = "http://localhost:18081/mining_status"

    # standard json header
    headers = {'content-type': 'application/json'}

    # execute the rpc request
    response = requests.post(
        url,
        headers=headers)

    # pretty print json output
    print(json.dumps(response.json(), indent=4))

if __name__ == "__main__":
    main()
```
Generated output:

```python
{
    "status": "OK",
    "threads_count": 2,
    "speed": 117,
    "active": true,
    "address": "48daf1rG3hE1Txapcsxh6WXNe9MLNKtu7W7tKTivtSoVLHErYzvdcpea2nSTgGkz66RFP4GKVAsTV14v6G3oddBTHfxP6tU"
}
```


**Basic example 2: get block header having a block hash**
```python
import requests
import json

def main():

    # bitmonerod is running on the localhost and port of 18081
    url = "http://localhost:18081/json_rpc"

    # standard json header
    headers = {'content-type': 'application/json'}

    # the block to get
    block_hash = 'd78e2d024532d8d8f9c777e2572623fd0f229d72d9c9c9da3e7cb841a3cb73c6'

    # bitmonerod' procedure/method to call
    rpc_input = {
           "method": "getblockheaderbyhash",
           "params": {"hash": block_hash}
    }

    # add standard rpc values
    rpc_input.update({"jsonrpc": "2.0", "id": "0"})

    # execute the rpc request
    response = requests.post(
        url,
        data=json.dumps(rpc_input),
        headers=headers)

    # pretty print json output
    print(json.dumps(response.json(), indent=4))

if __name__ == "__main__":
    main()
```

Generated output:
```python
{
    "result": {
        "status": "OK",
        "block_header": {
            "difficulty": 756932534,
            "height": 796743,
            "nonce": 8389,
            "depth": 46,
            "orphan_status": false,
            "hash": "d78e2d024532d8d8f9c777e2572623fd0f229d72d9c9c9da3e7cb841a3cb73c6",
            "timestamp": 1445741816,
            "major_version": 1,
            "minor_version": 0,
            "prev_hash": "dff9c6299c84f945fabde9e96afa5d44f3c8fa88835fb87a965259c46694a2cd",
            "reward": 8349972377827
        }
    },
    "jsonrpc": "2.0",
    "id": "0"
}
```


**Basic example 3: get full block information**

```python
import requests
import json

def main():

    # bitmonerod is running on the localhost and port of 18082
    url = "http://localhost:18081/json_rpc"

    # standard json header
    headers = {'content-type': 'application/json'}

    # the block to get
    block_hash = 'd78e2d024532d8d8f9c777e2572623fd0f229d72d9c9c9da3e7cb841a3cb73c6'

    # bitmonerod' procedure/method to call
    rpc_input = {
           "method": "getblock",
           "params": {"hash": block_hash}
    }

    # add standard rpc values
    rpc_input.update({"jsonrpc": "2.0", "id": "0"})

    # execute the rpc request
    response = requests.post(
        url,
        data=json.dumps(rpc_input),
        headers=headers)

    # the response will contain binary blob. For some reason
    # python's json encoder will crash trying to parse such
    # response. Thus, its better to remove it from the response.
    response_json_clean = json.loads(
                            "\n".join(filter(
                                lambda l: "blob" not in l, response.text.split("\n")
                            )))

    # pretty print json output
    print(json.dumps(response_json_clean, indent=4))

if __name__ == "__main__":
    main()
```

Generated output:
```python
{
    "jsonrpc": "2.0",
    "result": {
        "block_header": {
            "difficulty": 756932534,
            "major_version": 1,
            "height": 796743,
            "prev_hash": "dff9c6299c84f945fabde9e96afa5d44f3c8fa88835fb87a965259c46694a2cd",
            "depth": 166,
            "reward": 8349972377827,
            "minor_version": 0,
            "timestamp": 1445741816,
            "nonce": 8389,
            "orphan_status": false,
            "hash": "d78e2d024532d8d8f9c777e2572623fd0f229d72d9c9c9da3e7cb841a3cb73c6"
        },
        "json": "{\n  \"major_version\": 1, \n  \"minor_version\": 0, \n  \"timestamp\": 1445741816, \n  \"prev_id\": \"dff9c6299c84f945fabde9e96afa5d44f3c8fa88835fb87a965259c46694a2cd\", \n  \"nonce\": 8389, \n  \"miner_tx\": {\n    \"version\": 1, \n    \"unlock_time\": 796803, \n    \"vin\": [ {\n        \"gen\": {\n          \"height\": 796743\n        }\n      }\n    ], \n    \"vout\": [ {\n        \"amount\": 9972377827, \n        \"target\": {\n          \"key\": \"aecebf2757be84a2d986052607ec3114969f7c9e128a051f5e13f2304287733d\"\n        }\n      }, {\n        \"amount\": 40000000000, \n        \"target\": {\n          \"key\": \"c3a6d449f3fa837edbbc6beac8bc0405c6340c4e39418164b4aa1fa2202573f2\"\n        }\n      }, {\n        \"amount\": 300000000000, \n        \"target\": {\n          \"key\": \"cfce614b779ab2705fc5f94a022eb983a2960ba9da02d61f430e988128236b0a\"\n        }\n      }, {\n        \"amount\": 8000000000000, \n        \"target\": {\n          \"key\": \"b445b474d19ae555e048762e12ac8c406a4a6d7b0f37993dc8dabe7a31ef65b8\"\n        }\n      }\n    ], \n    \"extra\": [ 1, 243, 56, 214, 120, 176, 255, 133, 1, 251, 134, 27, 135, 49, 198, 55, 249, 146, 222, 116, 48, 103, 249, 229, 195, 120, 162, 127, 62, 35, 57, 231, 51, 2, 8, 0, 0, 0, 0, 25, 79, 41, 47\n    ], \n    \"signatures\": [ ]\n  }, \n  \"tx_hashes\": [ \"cc283dcae267c622d685b3e5f8e72aaba807dad0bb2d4170521af57c50be8165\", \"d2873b1c1800ce04434c663893a16417e8717015e9686914166f7957c5eabd68\"\n  ]\n}",
        "tx_hashes": [
            "cc283dcae267c622d685b3e5f8e72aaba807dad0bb2d4170521af57c50be8165",
            "d2873b1c1800ce04434c663893a16417e8717015e9686914166f7957c5eabd68"
        ],
        "status": "OK"
    },
    "id": "0"
}
```

**Basic example 4: get blockchain information**
```python
import requests
import json

def main():

    # bitmonerod is running on the localhost and port of 18082
    url = "http://localhost:18081/json_rpc"

    # standard json header
    headers = {'content-type': 'application/json'}

    # bitmonerod' procedure/method to call
    rpc_input = {
           "method": "get_info"
    }

    # add standard rpc values
    rpc_input.update({"jsonrpc": "2.0", "id": "0"})

    # execute the rpc request
    response = requests.post(
        url,
        data=json.dumps(rpc_input),
        headers=headers)

    # pretty print json output
    print(json.dumps(response.json(), indent=4))

if __name__ == "__main__":
    main()
```


Generated output:
```python
{
    "jsonrpc": "2.0",
    "result": {
        "status": "OK",
        "alt_blocks_count": 0,
        "difficulty": 692400878,
        "height": 797031,
        "tx_pool_size": 1,
        "grey_peerlist_size": 3447,
        "outgoing_connections_count": 12,
        "tx_count": 492488,
        "white_peerlist_size": 253,
        "target_height": 796995,
        "incoming_connections_count": 0
    },
    "id": "0"
}
```

More examples hopefully coming soon.

## How can you help?

Constructive criticism, code and website edits, and new examples are always welcome.  
They can be made through gihub.

Some Monero are also welcome:
```
48daf1rG3hE1Txapcsxh6WXNe9MLNKtu7W7tKTivtSoVLHErYzvdcpea2nSTgGkz66RFP4GKVAsTV14v6G3oddBTHfxP6tU
```
