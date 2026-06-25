import hashlib
import json
import time
from typing import List, Dict, Any

class Block:
    def __init__(self, index: int, previous_hash: str, timestamp: float, transactions: List[Dict], nonce: int = 0):
        self.index = index
        self.previous_hash = previous_hash
        self.timestamp = timestamp
        self.transactions = transactions
        self.nonce = nonce
        self.hash = self.calculate_hash()

    def calculate_hash(self) -> str:
        block_string = json.dumps({
            "index": self.index,
            "previous_hash": self.previous_hash,
            "timestamp": self.timestamp,
            "transactions": self.transactions,
            "nonce": self.nonce
        }, sort_keys=True).encode()
        return hashlib.sha256(block_string).hexdigest()

    def mine_block(self, difficulty: int = 4):
        target = "0" * difficulty
        while self.hash[:difficulty] != target:
            self.nonce += 1
            self.hash = self.calculate_hash()
        print(f"Block mined: {self.hash}")

class Blockchain:
    def __init__(self):
        self.chain: List[Block] = []
        self.pending_transactions: List[Dict] = []
        self.create_genesis_block()

    def create_genesis_block(self):
        genesis = Block(0, "0", time.time(), [], 0)
        self.chain.append(genesis)

    def get_latest_block(self) -> Block:
        return self.chain[-1]

    def add_transaction(self, sender: str, recipient: str, amount: float) -> int:
        self.pending_transactions.append({
            "sender": sender,
            "recipient": recipient,
            "amount": amount,
            "timestamp": time.time()
        })
        return self.get_latest_block().index + 1

    def mine_pending_transactions(self, miner_address: str, difficulty: int = 4):
        # پاداش استخراج
        self.add_transaction("System", miner_address, 10.0)  # مثل اتریوم reward

        block = Block(
            index=len(self.chain),
            previous_hash=self.get_latest_block().hash,
            timestamp=time.time(),
            transactions=self.pending_transactions
        )
        block.mine_block(difficulty)
        self.chain.append(block)
        self.pending_transactions = []

    def is_chain_valid(self) -> bool:
        for i in range(1, len(self.chain)):
            current = self.chain[i]
            previous = self.chain[i-1]
            if current.hash != current.calculate_hash():
                return False
            if current.previous_hash != previous.hash:
                return False
        return True

# مثال استفاده
if __name__ == "__main__":
    eth_like = Blockchain()
    print("Genesis block created!")

    eth_like.add_transaction("Alice", "Bob", 50.0)
    eth_like.add_transaction("Bob", "Charlie", 25.0)

    print("Mining new block...")
    eth_like.mine_pending_transactions("Miner1")

    print(f"Blockchain length: {len(eth_like.chain)}")
    print(f"Latest block hash: {eth_like.get_latest_block().hash}")
    print(f"Is valid? {eth_like.is_chain_valid()}")
