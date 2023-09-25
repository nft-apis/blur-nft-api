
Today we will discuss getting NFT-related data from BLUR NFT Marketplace. [BLUR](https://blur.io/) recently beamed into the Web3 scenario with their famous token airdrop and no fees model.

Additionally, BLUR also allows NFT Loans through its novel Blend protocol. We will also discuss how to get NFT loan data using Bitquery Graphql API from the Blur market.

If you want to learn more about Bitquery NFT data capabilities, you can read our article about [Opensea API](https://bitquery.io/blog/opensea-nft-api) and [NFT API Guide](https://community.bitquery.io/t/nft-api-complete-guide/1509).

Note — We are using [Streaming APIs (v2)](https://docs.bitquery.io/) to get the following data; you can also turn them into WebSocket simply using [Graphql Subscription](https://docs.bitquery.io/docs/start/getting-updates/).

> You can run the following queries [here](https://streaming.bitquery.io/).

[![Telegram Badge](https://badgen.net/static/Join/Bloxy_info?icon=telegram)](https://t.me/Bloxy_info) [![Diiscord Bage](https://badgen.net/discord/members/qHMBkJ8gDk)](https://discord.gg/qHMBkJ8gDk)

> This tutorial was originally published on Bitquery Community. Please check out original post: [BLUR NFT Marketplace API](https://bitquery.io/blog/blur-nft-marketplace-api)

## Table of Contents

- [Getting Started](#getting-started)  
- [Latest Trades on Blur](#latest-trades-on-blur)
- [Most traded NFTs on Blur Marketplace](#most-traded-nfts-on-blur-marketplace)
- [Total buy-sell of an NFT token on BLUR](#total-buy-sell-of-an-nft-token-on-blur)
- [Top buyers of NFTs on BLUR](#top-buyers-of-nfts-on-blur)
- [Specific buyer stats for an NFT on BLUR](#specific-buyer-stats-for-an-nft-on-blur)
- [Latest Loans taken on Blur](#latest-loans-taken-on-blur)
- [Latest loans for specific NFT token](#latest-loans-for-specific-nft-token)
- [Latest Loans for a specific lender](#latest-loans-for-a-specific-lender)
- [Loans above a specific amount on the Blur NFT marketplace](#loans-above-a-specific-amount-on-the-blur-nft-marketplace)
- [Loan history for specific NFT ID](#loan-history-for-specific-nft-id)
- [Get loan details for specific LienId](#get-loan-details-for-specific-lienid)
- [Latest Loan Refinances on Blur](#latest-loan-refinances-on-blur)
- [Loans refinanced by specific Address](#loans-refinanced-by-specific-address)
- [All refinance loans for specific NFT](#all-refinance-loans-for-specific-nft)
- [Loan Repayments](#loan-repayments)
- [Loan repayment for specific NFT collection](#loan-repayment-for-specific-nft-collection)
- [Auction Events](#auction-events)
- [New Auctions for specific NFT Collection](#new-auctions-for-specific-nft-collection)
- [Latest Locked NFTs Buy Trades](#latest-locked-nfts-buy-trades)
- [Locked NFTs bought by a buyer](#locked-nfts-bought-by-a-buyer)
- [Get Cancelled Offers](#get-cancelled-offers)
- [Get Seize Offers](#get-seize-offers)
- [About Bitquery](#about-bitquery)

## Getting Started

To get started for free, please create an account with your email: [GraphQL IDE ](https://ide.bitquery.io/).Once you create the account, check out our docs to learn [how to create your first query](https://docs.bitquery.io/docs/ide/query/).

To learn more about how to use the Bitquery API, please see the following resources:
- Historical / Near-Realtime Chain Data: [Blockchain API Documentation (V1 Graphql Docs) | Blockchain Graphql API (V1  API Docs)](https://docs.bitquery.io/v1/) 
- Realtime Data , Websocket and Cloud Product: [Blockchain Streaming API (V2 Graphql Docs) | Streaming API (V2  API Docs)](https://docs.bitquery.io/) 

## Latest Trades on Blur

BLUR marketplace supports [Seaport protocol](https://opensea.io/blog/articles/introducing-seaport-protocol); we will use it to get the latest Blur trades.

In this [query](https://ide.bitquery.io/Latest-10-Trades-on-Blur), we get NFT trades on Blur by setting the To address in the transaction to [Blur Marketplace contract](https://explorer.bitquery.io/ethereum/smart_contract/0x39da41747a83aee658334415666f3ef92dd0d541).

```
query MyQuery {
  EVM {
    DEXTrades(
      limit: { offset: 0, count: 10 }
      orderBy: { descendingByField: "Block_Time" }
      where: {
        Trade: { Dex: { ProtocolName: { is: "seaport_v1.4" } } }
        Transaction: {
          To: { is: "0x39da41747a83aeE658334415666f3EF92DD0D541" }
        }
      }
    ) {
      Trade {
        Dex {
          ProtocolName
        }
        Buy {
          Price
          Seller
          Buyer
          Currency {
            HasURI
            Name
            Fungible
            SmartContract
          }
        }
        Sell {
          Price
          Amount
          Currency {
            Name
          }
          Buyer
          Seller
        }
      }
      Transaction {
        Hash
      }
      Block {
        Time
      }
    }
  }
}
```

## Most traded NFTs on Blur Marketplace

Let’s figure out the most traded NFT on the Blur marketplace. In the following [query](https://ide.bitquery.io/Most-traded-NFT-on-Blur-marketplace), we aggregate based on buyers, sellers, nfts, and trade volume and sorting based on count (Trade count).

```
query MyQuery {
  EVM(dataset: combined, network: eth) {
    DEXTrades(
      where: {
        Trade: { Dex: { ProtocolName: { in: "seaport_v1.4" } } }
        Transaction: {
          To: { is: "0x39da41747a83aeE658334415666f3EF92DD0D541" }
        }
      }
      orderBy: { descendingByField: "count" }
      limit: { count: 10 }
    ) {
      tradeVol: sum(of: Trade_Buy_Amount)
      count
      buyers: count(distinct: Trade_Buy_Buyer)
      seller: count(distinct: Trade_Buy_Seller)
      nfts: count(distinct: Trade_Buy_Ids)
      Trade {
        Buy {
          Currency {
            Name
            ProtocolName
            Symbol
            Fungible
            SmartContract
          }
        }
      }
    }
  }
}
```

## Total buy-sell of an NFT token on BLUR

In the following [query](https://ide.bitquery.io/Total-buy-sell-of-an-NFT-token-onBLUR), we are getting total trades, trade volume, buyers, and sellers for [Nakamigos](https://explorer.bitquery.io/ethereum/token/0xd774557b647330c91bf44cfeab205095f7e6c367) NFT token.

```
query MyQuery {
  EVM(dataset: combined, network: eth) {
    DEXTrades(
      where: {
        Trade: {
          Dex: { ProtocolName: { in: "seaport_v1.4" } }
          Buy: {
            Currency: {
              Fungible: false
              SmartContract: {
                is: "0xd774557b647330c91bf44cfeab205095f7e6c367"
              }
            }
          }
        }
        Transaction: {
          To: { is: "0x39da41747a83aeE658334415666f3EF92DD0D541" }
        }
      }
      orderBy: { descendingByField: "count" }
      limit: { count: 10 }
    ) {
      tradeVol: sum(of: Trade_Buy_Amount)
      count
      buyer: count(distinct: Trade_Buy_Buyer)
      seller: count(distinct: Trade_Buy_Seller)
      nfts: count(distinct: Trade_Buy_Ids)
      Trade {
        Buy {
          Currency {
            Name
            ProtocolName
            Symbol
            Fungible
            SmartContract
          }
        }
      }
    }
  }
}
```

## Top buyers of NFTs on BLUR

If we want to know the top buyer on the Blur marketplace, you can use the following query. In this [query](https://ide.bitquery.io/Top-buyers-of-NFTs-onBLUR), we ate aggregating NFTs bought or sold, unique transactions for top 10 buyers, and sorting them based on no. of trades.

```
query MyQuery {
  EVM(dataset: combined, network: eth) {
    DEXTrades(
      where: {
        Trade: {
          Dex: { ProtocolName: { in: "seaport_v1.4" } }
          Buy: { Currency: { Fungible: false } }
        }
        Transaction: {
          To: { is: "0x39da41747a83aeE658334415666f3EF92DD0D541" }
        }
      }
      orderBy: { descendingByField: "count" }
      limit: { count: 10 }
    ) {
      count
      uniq_tx: count(distinct: Transaction_Hash)
      Block {
        first_date: Time(minimum: Block_Date)
        last_date: Time(maximum: Block_Date)
      }
      nfts: count(distinct: Trade_Buy_Ids)
      difffernt_nfts: count(distinct: Trade_Buy_Currency_SmartContract)
      total_money_paid: sum(of: Trade_Sell_Amount)
      Trade {
        Buy {
          Buyer
        }
      }
    }
  }
}
```

## Specific buyer stats for an NFT on BLUR

In the following [query](https://ide.bitquery.io/Specific-buyer-stats-for-an-NFT-onBLUR), we are getting details for a specific address on Blur nft marketplace. We are also getting the first and last trade dates for the address.

```
query MyQuery {
  EVM(dataset: combined, network: eth) {
    DEXTrades(
      where: {
        Trade: {
          Dex: { ProtocolName: { in: "seaport_v1.4" } }
          Buy: {
            Currency: {
              SmartContract: {
                is: "0xd774557b647330c91bf44cfeab205095f7e6c367"
              }
            }
            Buyer: { is: "0x9ba58eea1ea9abdea25ba83603d54f6d9a01e506" }
          }
        }
        Transaction: {
          To: { is: "0x39da41747a83aeE658334415666f3EF92DD0D541" }
        }
      }
      orderBy: { descendingByField: "count" }
      limit: { count: 10 }
    ) {
      count
      uniq_tx: count(distinct: Transaction_Hash)
      Block {
        first_date: Time(minimum: Block_Date)
        last_date: Time(maximum: Block_Date)
      }
      nfts: count(distinct: Trade_Buy_Ids)
      Trade {
        Buy {
          Buyer
          Currency {
            Name
            ProtocolName
            Symbol
            Fungible
            SmartContract
          }
        }
      }
    }
  }
}
```

## Latest Loans taken on Blur

Blur uses the [Blend protocol](https://www.paradigm.xyz/2023/05/blend) to enable NFT loans. We will query Blur’s [Blend smart contract events](https://explorer.bitquery.io/ethereum/smart_contract/0x29469395eaf6f95920e59f858042f0e28d98a20b/events) to get different loans related data.

In this [query](https://ide.bitquery.io/Latest-Loans-taken-onBlur), we look for the “LoanOfferTaken” events and set the smart contract to the Blur: [Blend Contract](https://explorer.bitquery.io/ethereum/smart_contract/0x29469395eaf6f95920e59f858042f0e28d98a20b) to get loan events on the marketplace.

We are using Logheader to query smart contract events and not Log → smart contract because it’s a [delegated proxy contract](https://medium.com/coinmonks/proxy-pattern-and-upgradeable-smart-contracts-45d68d6f15da).

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "LoanOfferTaken" } } }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Index
        Type
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## Latest loans for specific NFT token

Now we can filter arguments in smart contract events; in this [API](https://ide.bitquery.io/Latest-loans-for-specific-NFTtoken), we are getting all loans for the [MutantApeYachtClub NFT collection](https://explorer.bitquery.io/ethereum/token/0x60e4d786628fea6478f785a6d7e704777c86a7c6) sorted based on block time on the Blur marketplace.

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "LoanOfferTaken" } } }
        Arguments: {
          includes: [
            {
              Name: { is: "collection" }
              Value: {
                Address: { is: "0x60e4d786628fea6478f785a6d7e704777c86a7c6" }
              }
            }
          ]
        }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## Latest Loans for a specific lender

Using the same technique of filtering arguments in the following [API](https://ide.bitquery.io/Latest-Loans-for-a-specificlender), we are getting the latest loans for specific lender address. Similarly, you can use this [API](https://ide.bitquery.io/Latest-Loans-for-a-specificborrower-on-Blur-marketplace) to get the lastest loans for specific borrower address.

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "LoanOfferTaken" } } }
        Arguments: {
          includes: [
            {
              Name: { is: "lender" }
              Value: {
                Address: { is: "0xfa0e027fcb7ce300879f3729432cd505826eaabc" }
              }
            }
          ]
        }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## Loans above a specific amount on the Blur NFT marketplace

If we want to track loans above a specific amount on the Blur marketplace, we can use the following [API](https://ide.bitquery.io/Loans-above-a-specific-amount-on-the-Blur-NFT-marketplace).

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "LoanOfferTaken" } } }
        Arguments: {
          includes: [
            {
              Name: { is: "loanAmount" }
              Value: { BigInteger: { gt: "3000000000000000000" } }
            }
          ]
        }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## Loan history for specific NFT ID

Let’s say you want to know the loan history for a specific NFT ID on the Blur marketplace; you can use the following [API](https://ide.bitquery.io/Loan-history-for-specific-NFTID) to get this result.

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "LoanOfferTaken" } } }
        Arguments: {
          includes: [
            {
              Name: { is: "collection" }
              Value: {
                Address: { is: "0x49cf6f5d44e70224e2e23fdcdd2c053f30ada28b" }
              }
            }
            { Name: { is: "tokenId" }, Value: { BigInteger: { eq: "2662" } } }
          ]
        }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## Get loan details for specific LienId

Blur’s Blend protocol uses LienID as the primary key throughout to track details of specific loans.

We will use [this query](https://ide.bitquery.io/Get-loan-details-for-specificlienId) to track loan details for specific LienID through different events.

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "LoanOfferTaken" } } }
        Arguments: {
          includes: [
            { Name: { is: "lienId" }, Value: { BigInteger: { eq: "40501" } } }
          ]
        }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## Latest Loan Refinances on Blur

Refinance means taking out a new loan to pay off an existing loan. In the case of NFTs, refinance can be used to take out a new loan using an NFT as collateral. In this [query](https://ide.bitquery.io/loan-refinance-on-Blur) will get the latest refinance events.

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "Refinance" } } }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Index
        Type
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## Loans refinanced by specific Address

If you want to track loans refinanced by a specific address on Blur marketplace, use the following [query](https://ide.bitquery.io/Loans-refinanced-by-specificAddress).

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "Refinance" } } }
        Arguments: {
          includes: [
            {
              Name: { is: "newLender" }
              Value: {
                Address: { is: "0xaaac34d30d6938787c653aafb922bc20bfa9c512" }
              }
            }
          ]
        }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Index
        Type
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## All refinance loans for specific NFT

Use the following [query](https://ide.bitquery.io/All-refinance-loans-for-specificNFT-collection) to filter Refinance event arguments to get all refinance loans for specific NFT collection.

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "Refinance" } } }
        Arguments: {
          includes: [
            {
              Name: { is: "collection" }
              Value: {
                Address: { is: "0xed5af388653567af2f388e6224dc7c4b3241c544" }
              }
            }
          ]
        }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Index
        Type
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## Loan Repayments

In this [query](https://ide.bitquery.io/Loan-repayment-of-blur-marketplace), we get loan repayment transactions by filtering for “Repay” events and setting the smart contract address to Blur: Blend address.

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "Repay" } } }
        Arguments: {
          includes: [
            { Name: { is: "lienId" }, Value: { BigInteger: { eq: "43662" } } }
          ]
        }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Index
        Type
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## Loan repayment for specific NFT collection

To get loan repayments for specific NFT collections, you can filter “Repay” smart contract event arguments. Check the following [API](https://ide.bitquery.io/loan-repayment-for-specific-NFT-collection) to get the latest loan repayments for specific NFT collections on the Blur marketplace.

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "Repay" } } }
        Arguments: {
          includes: [
            {
              Name: { is: "collection" }
              Value: {
                Address: { is: "0xbc4ca0eda7647a8ab7c2061c2e118a18a936f13d" }
              }
            }
          ]
        }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Index
        Type
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## Auction Events

The StartAuction event is emitted when an auction is started for an NFT on the Blur: Blend smart contract. You can find the query [here](https://ide.bitquery.io/Auction-on-blur-marketplace). Similarly, you can also get Auctions for specific Lien ID using this [query](https://ide.bitquery.io/Auctions-for-specific-lienID).

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "StartAuction" } } }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Index
        Type
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## New Auctions for specific NFT Collection

Similarly, you can get all the latest actions for specific NFT collections. See the following query in which we are getting the latest auction for [Milady NFT collection](https://explorer.bitquery.io/ethereum/token/0x5af0d9827e0c53e4799bb226655a1de152a425a5) on the Blur marketplace.

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "StartAuction" } } }
        Arguments: {
          includes: [
            {
              Name: { is: "collection" }
              Value: {
                Address: { is: "0x5af0d9827e0c53e4799bb226655a1de152a425a5" }
              }
            }
          ]
        }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Index
        Type
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## Latest Locked NFTs Buy Trades

A locked NFT is an NFT that is temporarily unable to be transferred or sold. It will be sold once the lock period has ended. The price of a locked NFT may be lower than the price of a non-locked NFT because the buyer cannot access the NFT until the lock period has expired.

To get locked NFT trades, we filter for the “buylocked” smartcontract event. You can find the query [here](https://ide.bitquery.io/Locked-NFT-bought-on-Blur-marketplace).

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "BuyLocked" } } }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Index
        Type
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## Locked NFTs bought by a buyer

Using the following [query](https://ide.bitquery.io/Locked-NFTs-bought-by-abuyer), we can also track all the Locked NFTs bought by specific buyers by tracking the [BuyLocked event](https://explorer.bitquery.io/ethereum/smart_contract/0x29469395eaf6f95920e59f858042f0e28d98a20b/events) and filtering it using buyer argument.

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "BuyLocked" } } }
        Arguments: {
          includes: [
            {
              Name: { is: "buyer" }
              Value: {
                Address: { is: "0x96a7021972646bb05f9b544b13036a4872796fb0" }
              }
            }
          ]
        }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Index
        Type
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## Get Cancelled Offers

To get this data, we filter by the OfferCancelled event emitted when an offer is canceled. You can find the query [here](https://ide.bitquery.io/Latest-Cancelled-offers-on-Blur-NFT-marketplace).

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "OfferCancelled" } } }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Index
        Type
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## Get Seize Offers

When a seizure event occurs, the NFT is typically transferred to the control of the third party who is seizing it. To get this data, we filter transactions by the “seize” event on the Blur: Blend contract. You can get the query [here](https://ide.bitquery.io/Latest-Seized-NFTs-on-Blur-marketplace).

```
{
  EVM(dataset: combined, network: eth) {
    Events(
      where: {
        LogHeader: {
          Address: { is: "0x29469395eaf6f95920e59f858042f0e28d98a20b" }
        }
        Log: { Signature: { Name: { is: "Seize" } } }
      }
      limit: { count: 10 }
      orderBy: { descending: Block_Time }
    ) {
      Block {
        Number
      }
      Transaction {
        Hash
      }
      Log {
        SmartContract
        Signature {
          Name
          Signature
        }
      }
      Arguments {
        Name
        Index
        Type
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## About Bitquery

[**Bitquery**](https://bitquery.io/?source=blog&utm_medium=about_coinpath) is your comprehensive toolkit designed with developers in mind, simplifying blockchain data access. Our products offer practical advantages and flexibility.

-   **APIs** - [Explore API](https://ide.bitquery.io/streaming): Easily retrieve precise real-time and historical data for over 40 blockchains using GraphQL. Seamlessly integrate blockchain data into your applications, making data-driven decisions effortless.

-   **Coinpath®** - [Try Coinpath](https://bitquery.io/products/coinpath?utm_source=blog&utm_medium=about): Streamline compliance and crypto investigations by tracing money movements across 40+ blockchains. Gain insights for efficient decision-making.

-   **Data in Cloud** - [Try Demo Bucket](https://bitquery.io/products/data-in-cloud?utm_source=blog&utm_medium=about): Access indexed blockchain data cost-effectively and at scale for your data pipeline. We currently support Ethereum, BSC, Solana, with more blockchains on the horizon, simplifying your data access.

-   **Explorer** - [Try Explorer](http://explorer.bitquery.io): Discover an intuitive platform for exploring data from 40+ blockchains. Visualize data, generate queries, and integrate effortlessly into your applications.

Bitquery empowers developers with straightforward blockchain data tools. If you have questions or need assistance, connect with us on our [Telegram channel](https://t.me/Bloxy_info) or via email at <hello@bitquery.io>. Stay updated on the latest in cryptocurrency by subscribing to our newsletter below.
