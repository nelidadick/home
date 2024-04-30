import requests
from web3 import Web3
import json

private_key = ''
arbitrum_rpc = 'https://rpc.ankr.com/arbitrum/'

w3 = Web3(provider=Web3.HTTPProvider(
    endpoint_uri=arbitrum_rpc
))

account = w3.eth.account.from_key(private_key=private_key)

usdc_contract_address = Web3.to_checksum_address('0xFF970A61A04b1cA14834A43f5dE4533eBDDB5CC8')
usdc_contract_abi = json.load(open('data/abis/default_token.json'))
usdc_contract = w3.eth.contract(address=usdc_contract_address, abi=usdc_contract_abi)

woofi_contract_address = Web3.to_checksum_address('0x9aEd3A8896A85FE9a8CAc52C9B402D092B629a30')
woofi_contract_abi = json.load(open('data/abis/woofi.json'))
woofi_contract = w3.eth.contract(address=woofi_contract_address, abi=woofi_contract_abi)

eth_address = Web3.to_checksum_address('0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE')

# todo: данный функционал нужно вынести в функцию, но для упрощения кода оставили здесь 
response = requests.get('https://api.binance.com/api/v3/depth?symbol=ETHUSDT&limit=1')
result = response.json()
eth_price = result['asks'][0][0]

eth_amount = 0.0006
eth_decimals = 18

slippage = 0.5
min_to_amount = eth_price * eth_amount * (1 - slippage / 100)

usdc_decimals = usdc_contract.functions.decimals().call()

eth_amount = int(eth_amount * 10 ** eth_decimals)
usdc_amount = int(min_to_amount * 10 ** usdc_decimals)

tx_args = woofi_contract.encodeABI('swap',
                                   args=(
                                       eth_address,
                                       usdc_contract.address,
                                       eth_amount,
                                       usdc_amount,
                                       account.address,
                                       account.address,
                                   ))

tx_params = {
    'chainId': w3.eth.chain_id,
    'gasPrice': w3.eth.gas_price,
    'nonce': w3.eth.get_transaction_count(account.address),
    'from': account.address,
    'to': woofi_contract.address,
    'data': tx_args,
    'value': eth_amount
}

tx_params['gas'] = w3.eth.estimate_gas(tx_params)

sign = w3.eth.account.sign_transaction(tx_params, account.key)
tx = w3.eth.send_raw_transaction(sign.rawTransaction)

tx_data = w3.eth.wait_for_transaction_receipt(tx, timeout=200)
if 'status' in tx_data and tx_data['status'] == 1:
    print(f'transaction was successful: {tx.hex()}')
else:
    print(f'transaction failed {tx_data["transactionHash"].hex()}')
