# TRUSTIFY-VERIFIED-REVIEW-ON-BLOCKCHAIN

```
NAME : JOEL P
REG NO: 212222230057
```

## Title of the Project
Small description about the project like one below The integration of a chatbot within a hostel booking system, aimed at streamlining the reservation process for students and improving the overall user experience.

## About
Tailored Chatbot for Hostel Booking System is a project designed to integrate a chatbot that leverages advanced natural language processing techniques to understand and respond to user queries to the hostel booking system. Traditional hostel booking processes are often time-consuming and involve manual searches and extensive communication with hostel staff. This project seeks to overcome these challenges by creating an easy-to-use chatbot interface that assists students in addressing inquiries.

## Features
Implements advance neural network method.
A framework based application for deployment purpose.
High scalability.
Less time complexity.
A specific scope of Chatbot response model, using json data format.

## Requirements
Operating System: Requires a 64-bit OS (Windows 10 or Ubuntu) for compatibility with deep learning frameworks.
Development Environment: Python 3.6 or later is necessary for coding the sign language detection system.
Deep Learning Frameworks: TensorFlow for model training, MediaPipe for hand gesture recognition.
Image Processing Libraries: OpenCV is essential for efficient image processing and real-time hand gesture recognition.
Version Control: Implementation of Git for collaborative development and effective code management.
IDE: Use of VSCode as the Integrated Development Environment for coding, debugging, and version control integration.
Additional Dependencies: Includes scikit-learn, TensorFlow (versions 2.4.1), TensorFlow GPU, OpenCV, and Mediapipe for deep learning tasks.

## SYSTEM ARCHITECTURE
<img width="1081" height="666" alt="image" src="https://github.com/user-attachments/assets/150832bd-ada9-4f31-8b77-5658677e8f91" />


#### CODE SEGMENTS:
### FRONTEND - INDEX.HTML
```
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Trustify Demo</title>
  <script src="https://cdn.jsdelivr.net/npm/ethers@5.7.2/dist/ethers.umd.min.js"></script>
  <style>body{font-family:Arial;max-width:800px;margin:20px auto;padding:12px}input,button{padding:8px;margin-top:6px;width:100%}</style>
</head>
<body>
  <h2>Trustify — Demo Frontend</h2>

  <div>
    <label>Contract address</label>
    <input id="contractAddress" value="0xC49210F2bd9fcF70B7eC60729012B0110c7A3980" />
  </div>

  <h3>1) Simulate Purchase (Backend mints)</h3>
  <label>Buyer address (Ganache account)</label>
  <input id="buyer" placeholder="0x..." />
  <label>Order ID</label>
  <input id="orderId" value="ORDER001" />
  <button id="mintBackend">Create Order & Mint Token (Backend)</button>
  <pre id="mintResult"></pre>

  <hr/>

  <h3>2) Verify Ownership (MetaMask)</h3>
  <button id="connectMeta">Connect MetaMask</button>
  <div id="connected">Not connected</div>

  <label>Token ID to verify</label>
  <input id="tokenId" value="1" />
  <button id="verifyBtn">Check Ownership</button>

  <pre id="verifyResult"></pre>

<script>
const ABI = [
  "function ownerOf(uint256 tokenId) view returns (address)",
  "function tokenMetadata(uint256 tokenId) view returns (string)"
];

let provider, signer, userAddr;

document.getElementById('connectMeta').onclick = async () => {
  if (!window.ethereum) return alert("Install MetaMask");
  await window.ethereum.request({ method: 'eth_requestAccounts' });
  provider = new ethers.providers.Web3Provider(window.ethereum);
  signer = provider.getSigner();
  userAddr = await signer.getAddress();
  document.getElementById('connected').innerText = "Connected: " + userAddr;
};

document.getElementById('mintBackend').onclick = async () => {
  const buyer = document.getElementById('buyer').value.trim();
  const orderId = document.getElementById('orderId').value.trim();
  if (!buyer || !orderId) return alert("Enter buyer and orderId");
  try {
    const res = await fetch("http://localhost:4000/mint", {
      method: "POST",
      headers: {"Content-Type":"application/json"},
      body: JSON.stringify({ buyer, orderId })
    });
    const data = await res.json();
    document.getElementById('mintResult').innerText = JSON.stringify(data, null, 2);
  } catch(e) {
    document.getElementById('mintResult').innerText = String(e);
  }
};

document.getElementById('verifyBtn').onclick = async () => {
  const contractAddr = document.getElementById('contractAddress').value.trim();
  const tokenId = document.getElementById('tokenId').value.trim();
  if (!contractAddr || !tokenId) return alert("contract and tokenId required");
  try {
    if (!provider) provider = new ethers.providers.Web3Provider(window.ethereum);
    const contract = new ethers.Contract(contractAddr, ABI, provider);
    const owner = await contract.ownerOf(tokenId);
    let metadata = "(not available)";
    try { metadata = await contract.tokenMetadata(tokenId); } catch(e){}
    const accounts = await provider.send("eth_requestAccounts", []);
    const connected = accounts && accounts[0];
    const verified = connected && owner.toLowerCase() === connected.toLowerCase();
    document.getElementById('verifyResult').innerText = 
      `Owner: ${owner}\nMetadata: ${metadata}\nConnected: ${connected}\nVerified: ${verified}`;
  } catch(e) {
    document.getElementById('verifyResult').innerText = String(e);
  }
};
</script>
</body>
</html>
```

### BACKEND - SERVER.JS
```
// SPDX-License-Identifier: MIT
// backend/server.js

import express from "express";
import cors from "cors";
import dotenv from "dotenv";
import { ethers } from "ethers";

dotenv.config();

const app = express();
app.use(cors());
app.use(express.json());

// ---------------------------
// Configuration
// ---------------------------
const RPC = process.env.RPC_URL || "http://127.0.0.1:7545"; // Ganache RPC URL
const OWNER_KEY = process.env.OWNER_PRIVATE_KEY; // Ganache owner private key
const CONTRACT_ADDRESS = process.env.CONTRACT_ADDRESS; // Deployed contract address

if (!OWNER_KEY || !CONTRACT_ADDRESS) {
  console.error("Please set OWNER_PRIVATE_KEY and CONTRACT_ADDRESS in .env file!");
  process.exit(1);
}

// ---------------------------
// Provider & Wallet
// ---------------------------
const provider = new ethers.providers.JsonRpcProvider(RPC);
const wallet = new ethers.Wallet(OWNER_KEY, provider);

// ABI for Trustify smart contract
const ABI = [
  "function mint(address to, string calldata metadata) external returns (uint256)",
  "function ownerOf(uint256 tokenId) view returns (address)",
  "function tokenMetadata(uint256 tokenId) view returns (string memory)",
  "event Transfer(address indexed from, address indexed to, uint256 indexed tokenId)"
];

const contract = new ethers.Contract(CONTRACT_ADDRESS, ABI, wallet);

// ---------------------------
// POST /mint → Create token for a purchase
// ---------------------------
app.post("/mint", async (req, res) => {
  try {
    const { buyer, orderId } = req.body;
    if (!buyer || !orderId)
      return res.status(400).json({ error: "❗ buyer and orderId are required" });

    console.log(`Minting token for buyer: ${buyer}, Order ID: ${orderId}`);

    const tx = await contract.mint(buyer, orderId);
    console.log(`Transaction sent: ${tx.hash}`);

    const receipt = await tx.wait();
    console.log("Transaction confirmed!");

    // ---------------------------
    // Decode tokenId from Transfer event
    // ---------------------------
    let tokenId = null;
    try {
      const transferTopic = ethers.utils.id("Transfer(address,address,uint256)");
      const transferEvent = receipt.logs.find(
        log => log.topics[0] === transferTopic
      );

      if (transferEvent) {
        tokenId = ethers.BigNumber.from(transferEvent.topics[3]).toString();
        console.log(`Extracted tokenId: ${tokenId}`);
      } else {
        console.log(" Transfer event not found in logs!");
      }
    } catch (e) {
      console.log("TokenId extraction failed:", e.message);
    }

    res.json({
      status: "minted",
      txHash: receipt.transactionHash,
      tokenId: tokenId || "unknown"
    });
  } catch (err) {
    console.error("Error in /mint:", err);
    res.status(500).json({ error: err.message || String(err) });
  }
});

// ---------------------------
// GET /verify/:tokenId → Verify token ownership
// ---------------------------
app.get("/verify/:tokenId", async (req, res) => {
  try {
    const { tokenId } = req.params;
    const owner = await contract.ownerOf(tokenId);
    res.json({ tokenId, owner });
  } catch (err) {
    res.status(500).json({ error: "Verification failed", details: err.message });
  }
});

// ---------------------------
// GET /token/:tokenId → Fetch token metadata (optional)
// ---------------------------
app.get("/token/:tokenId", async (req, res) => {
  try {
    const { tokenId } = req.params;
    const metadata = await contract.tokenMetadata(tokenId);
    res.json({ tokenId, metadata });
  } catch (err) {
    res.status(500).json({ error: "Metadata fetch failed", details: err.message });
  }
});

// ---------------------------
// Start backend
// ---------------------------
const PORT = process.env.PORT || 4000;
app.listen(PORT, () =>
  console.log(`Trustify backend running at: http://localhost:${PORT}`)
);
```

### CONTRACT - PURCHASETOKEN.SOL`
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract PurchaseToken is ERC721, Ownable {
    uint256 private _nextTokenId;
    mapping(uint256 => string) public tokenMetadata;

    constructor() ERC721("Trustify PurchaseToken", "TPUR") Ownable(msg.sender) {
        _nextTokenId = 1;
    }

    // onlyOwner (deployer / backend account) can mint
    function mint(address to, string calldata metadata) external onlyOwner returns (uint256) {
        uint256 tokenId = _nextTokenId++;
        _safeMint(to, tokenId);
        tokenMetadata[tokenId] = metadata;
        return tokenId;
    }
}
```
### OUTPUT1
![WhatsApp Image 2025-11-05 at 21 06 12_46333331](https://github.com/user-attachments/assets/4b5595df-bd9f-4df3-a811-107fc03c8bde)
### OUTPUT 2
![WhatsApp Image 2025-11-05 at 21 06 12_20db57d0](https://github.com/user-attachments/assets/6f4e8660-637b-457d-a0f4-9d1c61dd1fb5)


## Results and Impact
The Sign Language Detection System enhances accessibility for individuals with hearing and speech impairments, providing a valuable tool for inclusive communication. The project's integration of computer vision and deep learning showcases its potential for intuitive and interactive human-computer interaction.

This project serves as a foundation for future developments in assistive technologies and contributes to creating a more inclusive and accessible digital environment.

## Articles published / References
N. S. Gupta, S. K. Rout, S. Barik, R. R. Kalangi, and B. Swampa, “Enhancing Heart Disease Prediction Accuracy Through Hybrid Machine Learning Methods ”, EAI Endorsed Trans IoT, vol. 10, Mar. 2024.
A. A. BIN ZAINUDDIN, “Enhancing IoT Security: A Synergy of Machine Learning, Artificial Intelligence, and Blockchain”, Data Science Insights, vol. 2, no. 1, Feb. 2024.
