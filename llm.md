# intearwallet-connect - Quick Usage Guide

## Installation

```bash
npm install intearwallet-connect
```

## Basic Usage

### Import

```typescript
import IntearWalletConnector, { 
  LocalStorageStorage, 
  InMemoryStorage,
  INTEAR_NATIVE_WALLET_URL 
} from 'intearwallet-connect';
```

### Initialize Connector

```typescript
// Using localStorage (recommended for web apps)
const storage = new LocalStorageStorage('my-app-prefix');
const connector = await IntearWalletConnector.loadFrom(storage);

// Or using in-memory storage
const storage = new InMemoryStorage();
const connector = await IntearWalletConnector.loadFrom(storage);
```

### Connect Wallet

```typescript
if (!connector.connectedAccount) {
  const result = await connector.requestConnection({
    networkId: 'mainnet', // or 'testnet'
    walletUrl: 'https://wallet.intear.tech', // or INTEAR_NATIVE_WALLET_URL for native app
    messageToSign: {
      message: 'Login to my app',
      nonce: crypto.getRandomValues(new Uint8Array(32)),
      recipient: 'your-contract.near'
    }
  });
  
  if (result) {
    console.log('Connected:', result.account.accountId);
  }
}
```

### Sign Message (NEP-413)

```typescript
const signedMessage = await connector.connectedAccount.signMessage({
  message: 'Verify ownership',
  nonce: crypto.getRandomValues(new Uint8Array(32)),
  recipient: 'your-app.com'
});
```

### Send Transactions

```typescript
const result = await connector.connectedAccount.sendTransactions([{
  signerId: 'user.near',
  receiverId: 'contract.near',
  actions: [{
    type: 'FunctionCall',
    params: {
      methodName: 'store_data',
      args: { data: 'value' },
      gas: '30000000000000',
      deposit: '100000000000000000000000' // 0.001 NEAR in yoctoNEAR
    }
  }]
}]);

console.log('Transaction outcomes:', result.outcomes);
```

## NEAR API Integration

### With near-api-js

This library provides **low-level transaction formatting**. Use with `near-api-js` for RPC calls:

```typescript
import { connect, keyStores, Near } from 'near-api-js';
import IntearWalletConnector from 'intearwallet-connect';

// Format transactions using intearwallet-connect
const transactions = [{
  signerId: 'user.near',
  receiverId: 'contract.near',
  actions: [{
    type: 'FunctionCall',
    params: {
      methodName: 'mint',
      args: { token_id: '1' },
      gas: '30000000000000',
      deposit: '0'
    }
  }]
}];

// Send via wallet for signing
const result = await connector.connectedAccount.sendTransactions(transactions);

// Use near-api-js to query results if needed
const near = await connect({
  networkId: 'mainnet',
  nodeUrl: 'https://rpc.mainnet.near.org'
});
const account = await near.account('user.near');
const outcome = await near.connection.provider.txStatus(result.outcomes[0].transaction.hash, 'user.near');
```


### Transaction Action Types

```typescript
// Transfer NEAR
{ type: 'Transfer', params: { deposit: '1000000000000000000000000' } }

// Function Call
{ 
  type: 'FunctionCall', 
  params: { 
    methodName: 'method_name', 
    args: { key: 'value' }, 
    gas: '30000000000000', 
    deposit: '0' 
  } 
}

// Create Account
{ type: 'CreateAccount' }

// Add Key
{ 
  type: 'AddKey', 
  params: { 
    publicKey: 'ed25519:...', 
    accessKey: { permission: 'FullAccess' } 
  } 
}
```

## Wallet Options

```typescript
// Web popup (default)
walletUrl: 'https://wallet.intear.tech'

// Native desktop/mobile app
walletUrl: INTEAR_NATIVE_WALLET_URL // "intear://"

// iframe selector (recommended for most dApps)
walletUrl: 'iframe:https://wallet.intear.tech'
```

## Disconnect

```typescript
await connector.disconnect();
```

## Notes

- **Storage**: Required for persisting connection state. Use `LocalStorageStorage` for web apps.
- **Transaction Format**: Uses NEAR's legacy action format, not near-api-js objects.
- **Gas**: Specified in gas units (1 TGas = 1,000,000,000,000)
- **Deposit**: Specified in yoctoNEAR (1 NEAR = 10²⁴ yoctoNEAR)
