# IoT-Blockchain-Based-Secure-Communication-Framework
his project integrates blockchain technology into IoT ecosystems to secure device communication, ensure data integrity, and prevent unauthorized access. Blockchain's decentralized and immutable nature enhances IoT security
from web3 import Web3
import hashlib
import json

# Connect to Ethereum blockchain (local Ganache or Infura)
blockchain_url = "http://127.0.0.1:7545"  # Update with your blockchain provider
web3 = Web3(Web3.HTTPProvider(blockchain_url))

# Contract ABI and Address (replace with your smart contract's details)
contract_address = "0xYourContractAddressHere"
contract_abi = json.loads("""
[
    {
        "constant": false,
        "inputs": [
            {"name": "deviceId", "type": "string"},
            {"name": "dataHash", "type": "string"}
        ],
        "name": "logData",
        "outputs": [],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
    },
    {
        "constant": true,
        "inputs": [
            {"name": "deviceId", "type": "string"}
        ],
        "name": "getLogs",
        "outputs": [{"name": "", "type": "string[]"}],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
    }
]
""")

# Interact with the contract
contract = web3.eth.contract(address=contract_address, abi=contract_abi)

# User wallet configuration
user_wallet = "0xYourWalletAddressHere"
private_key = "0xYourPrivateKeyHere"

def hash_data(data):
    """
    Compute SHA-256 hash of the given data.
    """
    return hashlib.sha256(data.encode()).hexdigest()

def log_data(device_id, data):
    """
    Log IoT device data on the blockchain.
    """
    try:
        data_hash = hash_data(data)
        nonce = web3.eth.getTransactionCount(user_wallet)

        # Build transaction
        transaction = contract.functions.logData(device_id, data_hash).buildTransaction({
            'from': user_wallet,
            'nonce': nonce,
            'gas': 3000000,
            'gasPrice': web3.toWei('20', 'gwei')
        })

        # Sign transaction
        signed_tx = web3.eth.account.signTransaction(transaction, private_key)
        tx_hash = web3.eth.sendRawTransaction(signed_tx.rawTransaction)

        print(f"Data logged on blockchain. Transaction Hash: {web3.toHex(tx_hash)}")
    except Exception as e:
        print(f"Error logging data: {e}")

def get_logs(device_id):
    """
    Retrieve logs for a specific device.
    """
    try:
        logs = contract.functions.getLogs(device_id).call()
        print(f"Logs for Device {device_id}: {logs}")
    except Exception as e:
        print(f"Error retrieving logs: {e}")

if __name__ == "__main__":
    print("IoT Blockchain-Based Security System")
    option = input("Choose: [1] Log Data [2] Get Logs: ")

    if option == "1":
        device_id = input("Enter Device ID: ")
        data = input("Enter Data to Log: ")
        log_data(device_id, data)
    elif option == "2":
        device_id = input("Enter Device ID to Retrieve Logs: ")
        get_logs(device_id)
    else:
        print("Invalid option.")
