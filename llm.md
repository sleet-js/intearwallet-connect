# intearwallet-connect - Quick Usage Guide

## Installation

```bash
npm install intearwallet-connect
```

## Import

```typescript
import IntearWalletConnector, { 
  LocalStorageStorage, 
  INTEAR_NATIVE_WALLET_URL 
} from 'intearwallet-connect';
```

## Initialize Connector

```typescript
// Using localStorage (recommended for web apps)
const storage = new LocalStorageStorage('my-app-prefix');
const connector = await IntearWalletConnector.loadFrom(storage);
```

## Connect Wallet

### Simple Sign-In (No Message, No Access Key)

```typescript
if (!connector.connectedAccount) {
  const result = await connector.requestConnection({
    networkId: 'mainnet',
    walletUrl: 'iframe:https://wallet.intear.tech'
  });
  
  if (result) {
    console.log('Connected account:', result.account.accountId);
  } else {
    console.log('User rejected connection');
  }
}
```

### Sign-In with Message (NEP-413)

```typescript
const result = await connector.requestConnection({
  networkId: 'mainnet',
  walletUrl: 'iframe:https://wallet.intear.tech',
  messageToSign: {
    message: 'Login to my app',
    nonce: crypto.getRandomValues(new Uint8Array(32)),
    recipient: 'your-app.com'
  }
});

console.log('Signed message:', result.signedMessage);
```

### Sign-In with Limited Access Key

```typescript
const result = await connector.requestConnection({
  networkId: 'mainnet',
  walletUrl: 'iframe:https://wallet.intear.tech',
  functionCallKey: {
    publicKey: 'ed25519:...', // Your app's public key
    contractId: 'your-contract.near',
    methodNames: ['method_one', 'method_two'], // or "any"
    gasAllowance: '250000000000000000000000000' // 0.25 NEAR in yoctoNEAR
  }
});
```

## Check Connection Status

```typescript
// Load from storage on app restart
const connector = await IntearWalletConnector.loadFrom(storage);

if (connector.connectedAccount) {
  console.log('Already connected:', connector.connectedAccount.accountId);
}
```

## Send Transactions

```typescript
const result = await connector.connectedAccount.sendTransactions([{
  signerId: 'user.near',
  receiverId: 'contract.near',
  actions: [{
    type: 'FunctionCall',
    params: {
      methodName: 'store_data',
      args: { key: 'value' },
      gas: '30000000000000', // 30 TGas
      deposit: '100000000000000000000000' // 0.001 NEAR in yoctoNEAR
    }
  }]
}]);

console.log('Transaction outcomes:', result.outcomes);
```

### Multiple Transactions

```typescript
const result = await connector.connectedAccount.sendTransactions([
  {
    signerId: 'user.near',
    receiverId: 'contract1.near',
    actions: [{
      type: 'FunctionCall',
      params: { methodName: 'action_one', args: {}, gas: '30000000000000', deposit: '0' }
    }]
  },
  {
    signerId: 'user.near',
    receiverId: 'contract2.near',
    actions: [{
      type: 'Transfer',
      params: { deposit: '1000000000000000000000000' } // 1 NEAR
    }]
  }
]);
```

## Sign Message (After Connection)

```typescript
const signedMessage = await connector.connectedAccount.signMessage({
  message: 'Verify ownership',
  nonce: crypto.getRandomValues(new Uint8Array(32)),
  recipient: 'your-app.com'
});

console.log('Signature:', signedMessage.signature);
```

## Disconnect

```typescript
await connector.disconnect();
```

## Transaction Action Types

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

// Stake
{ 
  type: 'Stake', 
  params: { 
    stake: '10000000000000000000000000', 
    publicKey: 'ed25519:...' 
  } 
}
```

## Wallet Options

```typescript
// Web popup via iframe selector (recommended)
walletUrl: 'iframe:https://wallet.intear.tech'

// Direct web popup
walletUrl: 'https://wallet.intear.tech'

// Native desktop/mobile app
walletUrl: INTEAR_NATIVE_WALLET_URL // "intear://"
```

## Notes

- **Storage**: Required for persisting connection state across page reloads
- **Gas**: Specified in gas units (1 TGas = 1,000,000,000,000)
- **Deposit**: Specified in yoctoNEAR (1 NEAR = 10²⁴ yoctoNEAR)
- **Args**: Pass as JavaScript object, library handles serialization
- **All connection options are optional** - simple sign-in requires only `networkId`
