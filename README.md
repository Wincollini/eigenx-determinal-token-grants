## Grant API Integration

The backend provides a grant-based authentication system that allows users to access the chat completions API using a wallet signature instead of an API key. Here's how to integrate with an EOA (Externally Owned Account) address.

### Server URL

```
https://determinal-api.eigenarcade.com
```

### Authentication Flow

1. **Get a Grant Message**: Call `/message` to get a message to sign
2. **Sign the Message**: Use your private key to sign the message
3. **Make API Calls**: Use the signed message to authenticate requests

### Step-by-Step Integration

#### Step 1: Get a Grant Message

```bash
curl "https://determinal-api.eigenarcade.com/message?address=YOUR_ETHEREUM_ADDRESS"
```

**Example Response:**
```json
{
  "success": true,
  "message": "Sign this message to authenticate your grant request for: 0x...",
  "address": "YOUR_ETHEREUM_ADDRESS"
}
```

#### Step 2: Sign the Message

Use your private key to sign the message. Here's how to do it in TypeScript with `viem`:

> **Security**: Never hardcode a private key. Set it as an environment variable before running.
> ```bash
> export PRIVATE_KEY=0x...   # your wallet private key
> export ACCOUNT_ADDRESS=0x... # your wallet address
> ```

```typescript
import { privateKeyToAccount } from 'viem/accounts';

const privateKey = process.env.PRIVATE_KEY as `0x${string}`;
if (!privateKey) throw new Error('PRIVATE_KEY env var required');

const account = privateKeyToAccount(privateKey);

const message = "Sign this message to authenticate your grant request for: 0x...";
const signature = await account.signMessage({ message });
```

Or with `curl` and `cast` (from foundry):

```bash
MESSAGE="Sign this message to authenticate your grant request for: 0x..."
SIGNATURE=$(cast wallet sign --private-key $PRIVATE_KEY "$MESSAGE")
```

#### Step 3: Make Chat Completions API Calls

Call `/api/chat/completions` with the signed message:

```bash
curl -X POST https://determinal-api.eigenarcade.com/api/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "user", "content": "Hello!"}
    ],
    "model": "gpt-oss-120b-f16",
    "max_tokens": 150,
    "seed": 42,
    "grantMessage": "Sign this message to authenticate your grant request for: 0x...",
    "grantSignature": "0x...",
    "walletAddress": "YOUR_ETHEREUM_ADDRESS"
  }'
```

### Complete TypeScript Example

```typescript
import { privateKeyToAccount } from 'viem/accounts';
import { createWalletClient, http } from 'viem';
import { mainnet } from 'viem/chains';

const SERVER_URL = 'https://determinal-api.eigenarcade.com';

const PRIVATE_KEY = process.env.PRIVATE_KEY as `0x${string}`;
if (!PRIVATE_KEY) throw new Error('PRIVATE_KEY env var required');

const ACCOUNT_ADDRESS = process.env.ACCOUNT_ADDRESS;
if (!ACCOUNT_ADDRESS) throw new Error('ACCOUNT_ADDRESS env var required');

async function getGrantMessage(address: string) {
  const response = await fetch(`${SERVER_URL}/message?address=${address}`);
  const data = await response.json();
  return data.message;
}

async function signMessage(message: string, account: any) {
  return await account.signMessage({ message });
}

async function callChatCompletions(
  message: string,
  grantMessage: string,
  grantSignature: string,
  walletAddress: string
) {
  const response = await fetch(`${SERVER_URL}/api/chat/completions`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      messages: [
        { role: 'user', content: message }
      ],
      model: 'gpt-oss-120b-f16',
      max_tokens: 150,
      seed: 42,
      grantMessage,
      grantSignature,
      walletAddress
    })
  });

  return await response.json();
}

// Complete flow
async function main() {
  const account = privateKeyToAccount(PRIVATE_KEY);
  const walletAddress = ACCOUNT_ADDRESS;

  // Step 1: Get grant message
  const grantMessage = await getGrantMessage(walletAddress);
  console.log('Grant message:', grantMessage);

  // Step 2: Sign the message
  const grantSignature = await signMessage(grantMessage, account);
  console.log('Signature:', grantSignature);

  // Step 3: Make API call
  const response = await callChatCompletions(
    'Hello, how are you?',
    grantMessage,
    grantSignature,
    walletAddress
  );
  
  console.log('Response:', response);
}

main();
```

### Parameters

#### `/message` Endpoint
- **Method**: GET
- **Query Parameters**: `address` (Ethereum address)
- **Response**: `{ success: true, message: string, address: string }`

#### `/api/chat/completions` Endpoint
- **Method**: POST
- **Grant Parameters**:
  - `grantMessage`: The message returned from `/message`
  - `grantSignature`: The signed grant message
  - `walletAddress`: Your Ethereum address
- **Chat Parameters**:
  - `messages`: Array of chat messages
  - `model`: Model to use (e.g., `gpt-oss-120b-f16`)
  - `max_tokens`: Maximum tokens to generate
  - `seed`: Random seed for reproducibility
  - `temperature`: (Optional) Sampling temperature

### Checking Grant Status

You can check if you have an active grant:

```bash
curl "https://determinal-api.eigenarcade.com/checkGrant?address=YOUR_ETHEREUM_ADDRESS"
```

**Response:**
```json
{
  "success": true,
  "tokenCount": 500,
  "address": "YOUR_ETHEREUM_ADDRESS",
  "hasGrant": true
}
```

### Error Handling

- **Invalid Address**: Returns 400 with error message
- **No Grant**: `tokenCount` will be 0 in the response
- **Insufficient Tokens**: API will return an error if you don't have enough tokens
