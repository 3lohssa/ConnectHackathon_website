---
layout: default
title: Home
---

# Price Guessing Game

Welcome to the Price Guessing Game.

- **Min Pool Amount**: <span id="totalAmount">0 ETH</span>
- **Mid Pool Amount**: <span id="totalAmountMid">0 ETH</span>
- **Mega Pool Amount**: <span id="totalAmountMega">0 ETH</span>

Latest ETH/USD Price: <span id="latestPrice">0 USD</span>

## Submit Your Guess

Enter your guessed price: <input type="number" class="form-input" id="guessedPrice" placeholder="Enter your guessed price">

<button class="form-submit" onclick="makeGuess()">Submit Min</button>
<button class="form-submit" onclick="makeGuessMid()">Submit Mid</button>
<button class="form-submit" onclick="makeGuessMega()">Submit Mega</button>

<button class="button" id="connectButton">Connect MetaMask</button>

<script src="https://cdn.jsdelivr.net/npm/web3@1.5.2/dist/web3.min.js"></script>
<script>
  let accounts;
  let contract;
  let kjToken;
  const kjFee = '10'; // 手續費數量，這裡設置為10個KJTOKEN

  const contractAddress = '0x39c7ED287F89b4980f2ebbD143E6CAB855cDbc26';
  const kjTokenAddress = '0xEDF2Da81e7132b99B069443E852Cb07102BF9DCf'; // 替換為你實際的KJToken地址
  const contractABI = [
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "_refundAddress",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "_interval",
                        "type": "uint256"
                    },
                    {
                        "internalType": "address",
                        "name": "_priceFeed",
                        "type": "address"
                    },
                    {
                        "internalType": "address",
                        "name": "_kjTokenAddress",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "_kjFee",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "nonpayable",
                "type": "constructor"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "from",
                        "type": "address"
                    },
                    {
                        "indexed": false,
                        "internalType": "uint256",
                        "name": "amount",
                        "type": "uint256"
                    }
                ],
                "name": "Deposit",
                "type": "event"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "user",
                        "type": "address"
                    },
                    {
                        "indexed": false,
                        "internalType": "int256",
                        "name": "guessedPrice",
                        "type": "int256"
                    }
                ],
                "name": "GuessMade",
                "type": "event"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": false,
                        "internalType": "int256",
                        "name": "price",
                        "type": "int256"
                    }
                ],
                "name": "PriceUpdated",
                "type": "event"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "winner",
                        "type": "address"
                    },
                    {
                        "indexed": false,
                        "internalType": "uint256",
                        "name": "amount",
                        "type": "uint256"
                    },
                    {
                        "indexed": false,
                        "internalType": "string",
                        "name": "poolType",
                        "type": "string"
                    }
                ],
                "name": "RefundFailed",
                "type": "event"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": false,
                        "internalType": "bool",
                        "name": "success",
                        "type": "bool"
                    },
                    {
                        "indexed": false,
                        "internalType": "uint256",
                        "name": "timestamp",
                        "type": "uint256"
                    }
                ],
                "name": "UpkeepPerformed",
                "type": "event"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "winner",
                        "type": "address"
                    },
                    {
                        "indexed": false,
                        "internalType": "uint256",
                        "name": "amount",
                        "type": "uint256"
                    },
                    {
                        "indexed": false,
                        "internalType": "string",
                        "name": "poolType",
                        "type": "string"
                    }
                ],
                "name": "WinnerSelected",
                "type": "event"
            },
            {
                "inputs": [
                    {
                        "internalType": "bytes",
                        "name": "",
                        "type": "bytes"
                    }
                ],
                "name": "checkUpkeep",
                "outputs": [
                    {
                        "internalType": "bool",
                        "name": "upkeepNeeded",
                        "type": "bool"
                    },
                    {
                        "internalType": "bytes",
                        "name": "",
                        "type": "bytes"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "ethUsdCurrentPrice",
                "outputs": [
                    {
                        "internalType": "int256",
                        "name": "",
                        "type": "int256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "getLatestPrice",
                "outputs": [
                    {
                        "internalType": "int256",
                        "name": "",
                        "type": "int256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "name": "guesses",
                "outputs": [
                    {
                        "internalType": "address",
                        "name": "user",
                        "type": "address"
                    },
                    {
                        "internalType": "int256",
                        "name": "guessedPrice",
                        "type": "int256"
                    },
                    {
                        "internalType": "uint256",
                        "name": "timestamp",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "name": "guessesMega",
                "outputs": [
                    {
                        "internalType": "address",
                        "name": "user",
                        "type": "address"
                    },
                    {
                        "internalType": "int256",
                        "name": "guessedPrice",
                        "type": "int256"
                    },
                    {
                        "internalType": "uint256",
                        "name": "timestamp",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "name": "guessesMid",
                "outputs": [
                    {
                        "internalType": "address",
                        "name": "user",
                        "type": "address"
                    },
                    {
                        "internalType": "int256",
                        "name": "guessedPrice",
                        "type": "int256"
                    },
                    {
                        "internalType": "uint256",
                        "name": "timestamp",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "interval",
                "outputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "kjFee",
                "outputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "kjToken",
                "outputs": [
                    {
                        "internalType": "contract IERC20",
                        "name": "",
                        "type": "address"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "lastTimestamp",
                "outputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "int256",
                        "name": "_guessedPrice",
                        "type": "int256"
                    }
                ],
                "name": "makeGuess",
                "outputs": [],
                "stateMutability": "payable",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "int256",
                        "name": "_guessedPrice",
                        "type": "int256"
                    }
                ],
                "name": "makeGuessMega",
                "outputs": [],
                "stateMutability": "payable",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "int256",
                        "name": "_guessedPrice",
                        "type": "int256"
                    }
                ],
                "name": "makeGuessMid",
                "outputs": [],
                "stateMutability": "payable",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "bytes",
                        "name": "",
                        "type": "bytes"
                    }
                ],
                "name": "performUpkeep",
                "outputs": [],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "refundAddress",
                "outputs": [
                    {
                        "internalType": "address",
                        "name": "",
                        "type": "address"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "totalAmount",
                "outputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "totalAmountMega",
                "outputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "totalAmountMid",
                "outputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "withdraw",
                "outputs": [],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "stateMutability": "payable",
                "type": "receive"
            }
        ];
  const kjTokenABI = [
            {
                "inputs": [
                    {
                        "internalType": "uint256",
                        "name": "initialSupply",
                        "type": "uint256"
                    },
                    {
                        "internalType": "address",
                        "name": "initialAccount",
                        "type": "address"
                    }
                ],
                "stateMutability": "nonpayable",
                "type": "constructor"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "spender",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "allowance",
                        "type": "uint256"
                    },
                    {
                        "internalType": "uint256",
                        "name": "needed",
                        "type": "uint256"
                    }
                ],
                "name": "ERC20InsufficientAllowance",
                "type": "error"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "sender",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "balance",
                        "type": "uint256"
                    },
                    {
                        "internalType": "uint256",
                        "name": "needed",
                        "type": "uint256"
                    }
                ],
                "name": "ERC20InsufficientBalance",
                "type": "error"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "approver",
                        "type": "address"
                    }
                ],
                "name": "ERC20InvalidApprover",
                "type": "error"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "receiver",
                        "type": "address"
                    }
                ],
                "name": "ERC20InvalidReceiver",
                "type": "error"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "sender",
                        "type": "address"
                    }
                ],
                "name": "ERC20InvalidSender",
                "type": "error"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "spender",
                        "type": "address"
                    }
                ],
                "name": "ERC20InvalidSpender",
                "type": "error"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "owner",
                        "type": "address"
                    },
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "spender",
                        "type": "address"
                    },
                    {
                        "indexed": false,
                        "internalType": "uint256",
                        "name": "value",
                        "type": "uint256"
                    }
                ],
                "name": "Approval",
                "type": "event"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "from",
                        "type": "address"
                    },
                    {
                        "indexed": false,
                        "internalType": "uint256",
                        "name": "value",
                        "type": "uint256"
                    }
                ],
                "name": "Burn",
                "type": "event"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "from",
                        "type": "address"
                    },
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "to",
                        "type": "address"
                    },
                    {
                        "indexed": false,
                        "internalType": "uint256",
                        "name": "value",
                        "type": "uint256"
                    }
                ],
                "name": "Transfer",
                "type": "event"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "owner",
                        "type": "address"
                    },
                    {
                        "internalType": "address",
                        "name": "spender",
                        "type": "address"
                    }
                ],
                "name": "allowance",
                "outputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "spender",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "value",
                        "type": "uint256"
                    }
                ],
                "name": "approve",
                "outputs": [
                    {
                        "internalType": "bool",
                        "name": "",
                        "type": "bool"
                    }
                ],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "account",
                        "type": "address"
                    }
                ],
                "name": "balanceOf",
                "outputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "uint256",
                        "name": "amount",
                        "type": "uint256"
                    }
                ],
                "name": "burn",
                "outputs": [],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "account",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "amount",
                        "type": "uint256"
                    }
                ],
                "name": "burnFrom",
                "outputs": [],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "decimals",
                "outputs": [
                    {
                        "internalType": "uint8",
                        "name": "",
                        "type": "uint8"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "name",
                "outputs": [
                    {
                        "internalType": "string",
                        "name": "",
                        "type": "string"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "symbol",
                "outputs": [
                    {
                        "internalType": "string",
                        "name": "",
                        "type": "string"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "totalSupply",
                "outputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "to",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "value",
                        "type": "uint256"
                    }
                ],
                "name": "transfer",
                "outputs": [
                    {
                        "internalType": "bool",
                        "name": "",
                        "type": "bool"
                    }
                ],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "from",
                        "type": "address"
                    },
                    {
                        "internalType": "address",
                        "name": "to",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "value",
                        "type": "uint256"
                    }
                ],
                "name": "transferFrom",
                "outputs": [
                    {
                        "internalType": "bool",
                        "name": "",
                        "type": "bool"
                    }
                ],
                "stateMutability": "nonpayable",
                "type": "function"
            }
        ];

  async function connectMetaMask() {
    if (typeof window.ethereum !== 'undefined') {
      window.web3 = new Web3(window.ethereum);

      try {
        accounts = await window.ethereum.request({ method: 'eth_requestAccounts' });
        console.log('Connected:', accounts[0]);
        document.getElementById('connectButton').innerText = `Connected: ${accounts[0]}`;

        // Initialize contract instance
        contract = new web3.eth.Contract(contractABI, contractAddress);
        kjToken = new web3.eth.Contract(kjTokenABI, kjTokenAddress);
        console.log('Contract initialized:', contract);

        // Fetch initial game data
        fetchGameData();
      } catch (error) {
        console.error('User rejected the request:', error);
      }
    } else {
      alert('MetaMask is not installed. Please install MetaMask and try again.');
    }
  }

  async function makeGuess() {
    await sendGuess('makeGuess', '0.001');
  }

  async function makeGuessMid() {
    await sendGuess('makeGuessMid', '0.01');
  }

  async function makeGuessMega() {
    await sendGuess('makeGuessMega', '0.1');
  }

  async function sendGuess(methodName, ethAmount) {
    if (!accounts || accounts.length === 0) {
      alert('Please connect MetaMask first.');
      return;
    }

    if (!contract) {
      alert('Contract is not initialized.');
      return;
    }

    const guessedPrice = document.getElementById('guessedPrice').value;

    try {
      await kjToken.methods.approve(contractAddress, kjFee).send({ from: accounts[0] });

      const receipt = await contract.methods[methodName](guessedPrice).send({
        from: accounts[0],
        value: web3.utils.toWei(ethAmount, 'ether'),
        gas: 500000,
        gasPrice: web3.utils.toWei('50', 'gwei')
      });
      console.log('Guess submitted:', receipt);
      alert('Guess submitted successfully!');
    } catch (error) {
      console.error('Error submitting guess:', error);
      alert(`Error: ${error.message}`);
    }
  }

  async function fetchGameData() {
    if (!contract) {
      console.error('Contract is not initialized.');
      return;
    }

    try {
      const totalAmount = await contract.methods.totalAmount().call();
      document.getElementById('totalAmount').innerText = web3.utils.fromWei(totalAmount, 'ether') + ' ETH';
      const totalAmountMid = await contract.methods.totalAmountMid().call();
      document.getElementById('totalAmountMid').innerText = web3.utils.fromWei(totalAmountMid, 'ether') + ' ETH';
      const totalAmountMega = await contract.methods.totalAmountMega().call();
      document.getElementById('totalAmountMega').innerText = web3.utils.fromWei(totalAmountMega, 'ether') + ' ETH';

      const latestPrice = await contract.methods.getLatestPrice().call();
      document.getElementById('latestPrice').innerText = latestPrice + ' USD';
    } catch (error) {
      console.error('Error fetching game data:', error);
    }
  }

  document.addEventListener('DOMContentLoaded', async () => {
    connectMetaMask(); // Automatically connect MetaMask on page load

    document.getElementById('connectButton').addEventListener('click', connectMetaMask);

    if (typeof window.ethereum !== 'undefined') {
      window.web3 = new Web3(window.ethereum);
      window.onload = fetchGameData;
    } else {
      alert('MetaMask is not installed. Please install MetaMask and try again.');
    }
  });
</script>
