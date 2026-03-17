# base2yyuimport time
from collections import Counter
from web3 import Web3

RPC_URL = "https://mainnet.base.org"

# ERC20 Transfer topic
TRANSFER_TOPIC = "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a5f7b3b1a"


def main():
    w3 = Web3(Web3.HTTPProvider(RPC_URL))

    if not w3.is_connected():
        raise RuntimeError("Cannot connect to Base RPC")

    print("Connected to Base")
    print("Tracking top ERC20 tokens by transfer activity...\n")

    last_block = w3.eth.block_number

    while True:
        try:
            current_block = w3.eth.block_number

            if current_block > last_block:

                logs = w3.eth.get_logs({
                    "fromBlock": last_block + 1,
                    "toBlock": current_block,
                    "topics": [TRANSFER_TOPIC]
                })

                token_counter = Counter()

                for log in logs:
                    token_counter[log["address"]] += 1

                if token_counter:
                    print(f"\nBlocks {last_block+1} → {current_block}")

                    for token, count in token_counter.most_common(5):
                        print(f"{token} → {count} transfers")

                last_block = current_block

            time.sleep(5)

        except Exception as e:
            print("Error:", e)
            time.sleep(5)


if __name__ == "__main__":
    main()
