# [Stellar SDK for Flutter](https://github.com/Soneso/stellar_flutter_sdk)

![Alpha Version](https://img.shields.io/badge/Alpha-v0.7.8-yellow.svg)
![Dart](https://img.shields.io/badge/Dart-green.svg)
![Flutter](https://img.shields.io/badge/Flutter-blue.svg)
![Supports Stellar Horizon v1.4.0](https://img.shields.io/badge/Horizon-v1.4.0-blue.svg)
![Supports Stellar Core v13](https://img.shields.io/badge/Core-v13-blue.svg)

The Soneso open source stellar SDK for flutter is build with dart and provides APIs to build and sign transactions, connect and query [Horizon](https://github.com/stellar/horizon).

The SDK is currently in alpha stage - v. 0.7.8. 

## Installation

### From pub.dev
1. Add the dependency to your pubspec.yaml file:
```
dependencies:
  stellar_flutter_sdk: ^0.7.8
```
2. Install it (command line or IDE):
```
flutter pub get
```
3. In your source file import the SDK, initialize and use it:
```dart
import 'package:stellar_flutter_sdk/stellar_flutter_sdk.dart';

final StellarSDK sdk = StellarSDK.TESTNET;

String accountId = "GASYKQXV47TPTB6HKXWZNB6IRVPMTQ6M6B27IM5L2LYMNYBX2O53YJAL";
AccountResponse account = await sdk.accounts.account(accountId);
print("sequence number: ${account.sequenceNumber}");
```
   
### Manual

Add the SDK is a flutter dart plugin. Here is a step by step that we recommend:

1. Clone this repo;
2. Open the project in your IDE (e.g. Android Studio).
3. Open the file `pubspec.yaml` and press `Pub get` in your IDE.
4. Go to the project's `test` directory, run a test from there and you are good to go!

Add it to your app:

5. In your flutter app add the local dependency in `pubspec.yaml` and then run `pub get`:
```code
dependencies:
   flutter:
     sdk: flutter
   stellar_flutter_sdk:
     path: ../stellar_flutter_sdk
```
6. In your source file import the SDK, initialize and use it:
```dart
import 'package:stellar_flutter_sdk/stellar_flutter_sdk.dart';

final StellarSDK sdk = StellarSDK.TESTNET;

String accountId = "GASYKQXV47TPTB6HKXWZNB6IRVPMTQ6M6B27IM5L2LYMNYBX2O53YJAL";
AccountResponse account = await sdk.accounts.account(accountId);
print("sequence number: ${account.sequenceNumber}");
```

## Quick Start

### 1. Create a Stellar key pair

#### Random generation
```dart
// create a completely new and unique pair of keys.
KeyPair keyPair = KeyPair.random();

print("${keyPair.accountId}");
// GCFXHS4GXL6BVUCXBWXGTITROWLVYXQKQLF4YH5O5JT3YZXCYPAFBJZB

print("${keyPairB.secretSeed}");
// SAV76USXIJOBMEQXPANUOQM6F5LIOTLPDIDVRJBFFE2MDJXG24TAPUU7
```

### 2. Create an account
After the key pair generation, you have already got the address, but it is not activated until someone transfers at least 1 lumen into it.

#### 2.1 Testnet
If you want to play in the Stellar test network, the sdk can ask Friendbot to create an account for you as shown below:
```dart
bool funded = await FriendBot.fundTestAccount(keyPair.accountId);
print ("funded: ${funded}");
```
#### 2.2 Public net

On the other hand, if you would like to create an account in the public net, you should buy some Stellar Lumens from an exchange. When you withdraw the Lumens into your new account, the exchange will automatically create the account for you. However, if you want to create an account from another account of your own, you may run the following code:

```dart
/// Create a key pair for your existing account.
KeyPair keyA = KeyPair.fromSecretSeed("SAPS66IJDXUSFDSDKIHR4LN6YPXIGCM5FBZ7GE66FDKFJRYJGFW7ZHYF");

/// Load the data of your account from the stellar network.
AccountResponse accA = await sdk.accounts.account(keyA.accountId);

/// Create a keypair for a new account.
KeyPair keyB = KeyPair.random();

/// Create the operation builder.
CreateAccountOperationBuilder createAccBuilder = CreateAccountOperationBuilder(keyB.accountId, "3"); // send 3 XLM (lumen)

// Create the transaction.
Transaction transaction = new TransactionBuilder(accA, Network.PUBLIC)
        .addOperation(createAccBuilder.build())
        .build();

/// Sign the transaction with the key pair of your existing account.
transaction.sign(keyA);

/// Submit the transaction to the stellar network.
SubmitTransactionResponse response = await sdk.submitTransaction(transaction);

if (response.success) {
  print ("account ${keyB.accountId} created");
}
```

### 3. Check account
#### 3.1 Basic info

After creating the account, we may check the basic information of the account.

```dart
String accountId = "GASYKQXV47TPTB6HKXWZNB6IRVPMTQ6M6B27IM5L2LYMNYBX2O53YJAL";

// Request the account data.
AccountResponse account = await sdk.accounts.account(accountId);

// You can check the `balance`, `sequence`, `flags`, `signers`, `data` etc.

for (Balance balance in account.balances) {
  switch (balance.assetType) {
    case Asset.TYPE_NATIVE:
      print("Balance: ${balance.balance} XLM");
      break;
    default:
      print("Balance: ${balance.balance} ${balance
          .assetCode} Issuer: ${balance.assetIssuer}");
  }
}

print("Sequence number: ${account.sequenceNumber}");

for (Signer signer in account.signers) {
  print("Signer public key: ${signer.accountId}");
}

for (String key in account.data.keys) {
  print("Data key: ${key} value: ${account.data[key]}");
}
```

#### 3.2 Check payments

You can check the payments connected to an account:

```dart
Page<OperationResponse> payments = await sdk.payments.forAccount(accountAId).order(RequestBuilderOrder.DESC).execute();

for (OperationResponse response in payments.records) {
  if (response is PaymentOperationResponse) {
    PaymentOperationResponse por = response as PaymentOperationResponse;
    if (por.transactionSuccessful) {
      print("Transaction hash: ${por.transactionHash}");
    }
  }
}
```
You can use:`limit`, `order`, and `cursor` to customize the query. Get the most recent payments for accounts, ledgers and transactions.

#### 3.3 Check others

Just like payments, you you check `assets`, `transactions`, `effects`, `offers`, `operations`, `ledgers` etc. 

```dart
sdk.assets.
sdk.transactions.
sdk.effects.
sdk.offers.
sdk.operations.
sdk.orderBook.
sdk.trades.
// add so on ...
```
### 4. Building and submitting transactions

Example "send multiple native payments at once":

```dart
transaction = new TransactionBuilder(accountA, Network.TESTNET)
        .addOperation(PaymentOperationBuilder(accountCId,Asset.NATIVE, "10").build())
        .addOperation(PaymentOperationBuilder(accountBId,Asset.NATIVE, "10").build())
        .addOperation(PaymentOperationBuilder(accountDId,Asset.NATIVE, "10").build())
        .build();

transaction.sign(keyPairA);

response = await sdk.submitTransaction(transaction);
```
## Documentation and Examples

Will be added asap. Meanwhile you may have a look inti the test folder.


## How to contribute

Please read our [Contribution Guide](https://github.com/Soneso/stellar_flutter_sdk/blob/master/CONTRIBUTING.md).

Then please [sign the Contributor License Agreement](https://goo.gl/forms/hS2KOI8d7WcelI892).

## License

The stellar sdk for flutter is licensed under an MIT license. See the [LICENSE](https://github.com/Soneso/stellar_flutter_sdk/blob/master/LICENSE) file for details.