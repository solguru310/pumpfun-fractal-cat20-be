# pumpfun-fractal-cat20-be
The backend of the Cat Protocol's pump.fun, built on the Fractal Bitcoin Network, leverages Node.js and sCrypt to function as a launchpad for CAT20 tokens, enabling their minting, trading, and transfer.

## Installation

Follow these steps to set up the project:

```
# Clone the repository  
git clone https://github.com/Rezzecup/Fractal-CAT-token-pumpfun-backend.git  

# Navigate to the project directory  
cd Fractal-CAT-token-pumpfun-backend  

# Install dependencies  
npm install
```

## Core Functionality

### Token Minting

Use the **mintToken** function to create CAT20 tokens by providing the token name, symbol, and supply:
```
const { mintToken } = require('./services/tokenService');  

(async () => {  
  try {  
    const result = await mintToken('tokenName', 'tokenSymbol', 1000);  
    console.log('Token minted:', result);  
  } catch (error) {  
    console.error('Error minting token:', error);  
  }  
})();
```

### Token Trading

Use the **tradeToken** function to transfer tokens between addresses:
```
const { tradeToken } = require('./services/tradeService');  

(async () => {  
  try {  
    const result = await tradeToken('tokenId', 'toAddress', 100, 'traderPrivateKey');  
    console.log('Token traded:', result);  
  } catch (error) {  
    console.error('Error trading token:', error);  
  }  
})();
```

### Retrieve Token Information

Fetch token details using the API endpoint:
```
const fetch = require('node-fetch');  

async function getTokenInfo(tokenId) {  
  const response = await fetch(`https://api-pump.fun/info?token_id=${tokenId}`, {  
    method: 'GET',  
    headers: {  
      'Content-Type': 'application/json',  
      'x-api-key': process.env.API_KEY,  
    },  
  });  
  return await response.json();  
}  

(async () => {  
  try {  
    const info = await getTokenInfo('tokenId');  
    console.log('Token Info:', info);  
  } catch (error) {  
    console.error('Error fetching token info:', error);  
  }  
})();
```

## Backend Structure

The backend is organized into the following components:

### 1. Model
The model defines the structure and behavior of CAT20 tokens:

**models/CAT20Token.js**:
```
class CAT20Token {  
  constructor(name, symbol, totalSupply, owner) {  
    this.name = name;  
    this.symbol = symbol;  
    this.totalSupply = totalSupply;  
    this.owner = owner;  
    this.balances = { [owner]: totalSupply };  
  }  

  transfer(from, to, amount) {  
    if (this.balances[from] >= amount) {  
      this.balances[from] -= amount;  
      this.balances[to] = (this.balances[to] || 0) + amount;  
      return true;  
    }  
    throw new Error('Insufficient balance');  
  }  
}  

module.exports = CAT20Token;
```

### 2. Controller
The controller manages token-related business logic, including minting and trading:

**controllers/CAT20TokenController.js**:
```
const CAT20Token = require('../models/CAT20Token');  

const tokens = {};  

exports.createToken = (req, res) => {  
  const { name, symbol, totalSupply, owner } = req.body;  
  if (tokens[symbol]) {  
    return res.status(400).json({ message: 'Token symbol already exists' });  
  }  
  const token = new CAT20Token(name, symbol, totalSupply, owner);  
  tokens[symbol] = token;  
  res.status(201).json({ message: 'Token created', token });  
};  

exports.tradeToken = (req, res) => {  
  const { symbol, from, to, amount } = req.body;  
  const token = tokens[symbol];  
  if (!token) {  
    return res.status(404).json({ message: 'Token not found' });  
  }  
  try {  
    token.transfer(from, to, amount);  
    res.status(200).json({ message: 'Tokens transferred successfully', token });  
  } catch (error) {  
    res.status(400).json({ message: error.message });  
  }  
};
```

### 3. Router
The router maps HTTP requests to controller methods:

**routes/CAT20TokenRoutes.js**:
```
const express = require('express');  
const router = express.Router();  
const CAT20TokenController = require('../controllers/CAT20TokenController');  

router.post('/create', CAT20TokenController.createToken);  
router.post('/trade', CAT20TokenController.tradeToken);  

module.exports = router;
```

### 4. Server Setup
Integrate all components into the server:

**server.js**:
```
const express = require('express');  
const bodyParser = require('body-parser');  
const CAT20TokenRoutes = require('./routes/CAT20TokenRoutes');  

const app = express();  
app.use(bodyParser.json());  

app.use('/api/tokens', CAT20TokenRoutes);  

const PORT = process.env.PORT || 3000;  
app.listen(PORT, () => {  
  console.log(`Server running on port ${PORT}`);  
});
```

## Summary
This backend provides essential features for managing CAT20 tokens, including:

- Minting: Create new tokens with specific details.
- Trading: Transfer tokens between addresses securely.
- Information Retrieval: Access token metadata and details.
  
Contributions and suggestions are welcome!
