# Helius SDK
An asynchronous Helius Rust SDK for building the future of Solana

## Documentation
The latest documentation can be found [here on docs.rs](https://docs.rs/helius/latest/helius/)

## Contributions
Interested in contributing to the Helius Rust SDK? Read the following [contributions guide](https://github.com/helius-labs/helius-rust-sdk/blob/dev/CONTRIBUTIONS.md) before opening up a pull request!

## Installation
To start using the Helius Rust SDK in your project, add it as a dependency via `cargo`. Open your project's `Cargo.toml` and add the following line under `[dependencies]`:
```toml
helius = "x.y.z"
```
where `x.y.z` is your desired version. Alternatively, use `cargo add helius` to add the dependency directly via the command line. This will automatically find the latest version compatible with your project and add it to your `Cargo.toml`.

Remember to run `cargo update` regularly to fetch the latest version of the SDK.

## Usage
The SDK provides a [`Helius`](https://github.com/helius-labs/helius-rust-sdk/blob/dev/src/client.rs) instance that can be configured with an API key and a given Solana cluster. Developers can generate a new API key on the [Helius Developer Dashboard](https://dev.helius.xyz/dashboard/app). This instance acts as the main entry point for interacting with the SDK by providing methods to access different Solana and RPC client functionalities. The following code is an example of how to use the SDK to fetch info on [Mad Lad #8420](https://xray.helius.xyz/token/F9Lw3ki3hJ7PF9HQXsBzoY8GyE6sPoEZZdXJBsTTD2rk?network=mainnet):
```rust
use helius::error::HeliusError;
use helius::types::{Cluster, DisplayOptions, GetAssetRequest, GetAssetResponseForAsset};

#[tokio::main]
async fn main() -> Result<(), HeliusError> {
    let api_key: &str = "YOUR_API_KEY";
    let cluster: Cluster = Cluster::MainnetBeta;

    let helius: Helius = Helius::new(api_key, cluster).unwrap();

    let request: GetAssetRequest = GetAssetRequest {
        id: "F9Lw3ki3hJ7PF9HQXsBzoY8GyE6sPoEZZdXJBsTTD2rk".to_string(),
        display_options: Some(DisplayOptions {
            show_unverified_collections: false,
            show_collection_metadata: false,
            show_fungible: false,
            show_inscription: false,
        }),
    };

    let response: Result<Option<GetAssetResponseForAsset>, HeliusError> = helius.rpc().get_asset(request).await;

    match response {
        Ok(Some(asset)) => {
            println!("Asset: {:?}", asset);
        },
        Ok(None) => println!("No asset found."),
        Err(e) => println!("Error retrieving asset: {:?}", e),
    }

    Ok(())
}
```
The `Helius` client has an embedded [Solana client](https://docs.rs/solana-client/latest/solana_client/rpc_client/struct.RpcClient.html) that can be accessed via `helius.connection().request_name()` where `request_name()` is a given [RPC method](https://docs.rs/solana-client/latest/solana_client/rpc_client/struct.RpcClient.html#implementations). A full list of all Solana RPC HTTP methods can be found [here](https://solana.com/docs/rpc/http).

More examples of how to use the SDK can be found in the [`examples`](https://github.com/helius-labs/helius-rust-sdk/tree/dev/examples) directory.

## Error Handling
### Common Error Codes
You may encounter several error codes when working with the Helius Rust SDK. Below is a table detailing some of the common error codes along with additional information to aid with troubleshooting:

| Error Code | Error Message             | More Information                                                                           |
|------------|---------------------------|---------------------------------------------------------------------------------------------|
| 401        | Unauthorized              | This occurs when an invalid API key is provided or access is restricted |
| 429        | Too Many Requests         | This indicates that the user has exceeded the request limit in a given timeframe or is out of credits |
| 5XX        | Internal Server Error     | This is a generic error message for server-side issues. Please contact Helius support for assistance |

If you encounter any of these errors:
- Refer to [`errors.rs`](https://github.com/helius-labs/helius-rust-sdk/blob/dev/src/error.rs) for a list of all possible errors returned by the `Helius` client
- Refer to the [Helius documentation](https://docs.helius.dev/) for further guidance
- Reach out to the Helius support team for more detailed assistance

## Methods
Our SDK is designed to provide a seamless developer experience when building on Solana. We've separated the core functionality into various segments:

### DAS API
- [`get_asset`](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/get-asset) - Gets an asset by its ID
- [`get_asset_batch`](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/get-asset/get-asset-batch) - Gets multiple assets by their ID
- [`get_asset_proof`](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/get-asset-proof) - Gets a merkle proof for a compressed asset by its ID
- [`get_asset_proof_batch`](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/get-asset-proof/get-asset-proof-batch) - Gets multiple asset proofs by their IDs
- [`get_assets_by_owner`](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/get-assets-by-owner) - Gets a list of assets owned by a given address
- [`get_assets_by_authority`](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/get-assets-by-authority) - Gets a list of assets of a given authority
- [`get_assets_by_creator`](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/get-assets-by-creator) - Gets a list of assets of a given creator
- [`get_assets_by_group`](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/get-assets-by-group) - Gets a list of assets by a group key and value
- [`search_assets`](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/search-assets) - Gets assets based on the custom search criteria passed in
- [`get_signatures_for_asset`](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/get-signatures-for-asset) - Gets transaction signatures for a given asset
- [`get_token_accounts`](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/get-token-accounts) - Gets information about all token accounts for a specific mint or owner
- [`get_nft_edition`](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/get-nft-editions) - Gets all the NFT editions  associated with a specific master NFT
- [`get_rwa_asset`](https://github.com/helius-labs/helius-sdk/pull/71) - Gets a Real-World Asset (RWA) based on its mint address

### Mint API
- [`mint_compressed_nft`](https://docs.helius.dev/compression-and-das-api/mint-api/mint-compressed-nft) - The easiest way to mint a compressed NFT (cNFT)

### Enhanced Transactions API
- [`parse_transactions`](https://docs.helius.dev/solana-apis/enhanced-transactions-api/parse-transaction-s) - Parses transactions given an array of transaction IDs
- [`parsed_transaction_history`](https://docs.helius.dev/solana-apis/enhanced-transactions-api/parsed-transaction-history) - Retrieves a parsed transaction history for a specific address

### Webhooks
- [`append_addresses_to_webhook`](https://github.com/helius-labs/helius-rust-sdk/blob/2d161e1ebf6d06df686d9e248ea80de215457b40/src/webhook.rs#L50-L73) - Appends a set of addresses to a given webhook
- [`create_webhook`](https://docs.helius.dev/webhooks-and-websockets/api-reference/create-webhook) - Creates a webhook given account addresses
- [`delete_webhook`](https://docs.helius.dev/webhooks-and-websockets/api-reference/delete-webhook) - Deletes a given Helius webhook programmatically
- [`edit_webhook`](https://docs.helius.dev/webhooks-and-websockets/api-reference/edit-webhook) - Edits a Helius webhook programmatically
- [`get_all_webhooks`](https://docs.helius.dev/webhooks-and-websockets/api-reference/get-all-webhooks) - Retrieves all Helius webhooks programmatically
- [`get_webhook_by_id`](https://docs.helius.dev/webhooks-and-websockets/api-reference/get-webhook) - Gets a webhook config given a webhook ID
- [`remove_addresses_from_webhook`](https://github.com/helius-labs/helius-rust-sdk/blob/bf24259e3333ae93126bb65b342c2c63e80e07a6/src/webhook.rs#L75-L105) - Removes a list of addresses from an existing webhook by its ID

### Helper Methods
- [`get_priority_fee_estimate`](https://docs.helius.dev/solana-rpc-nodes/alpha-priority-fee-api) - Gets an estimate of the priority fees required for a transaction to be processed more quickly
- [`deserialize_str_to_number`](https://github.com/helius-labs/helius-rust-sdk/blob/dev/src/utils/deserialize_str_to_number.rs) - Deserializes a `String` to a `Number`
- [`is_valid_solana_address`](https://github.com/helius-labs/helius-rust-sdk/blob/dev/src/utils/is_valid_solana_address.rs) - Returns whether a given string slice is a valid Solana address
- [`make_keypairs`](https://github.com/helius-labs/helius-rust-sdk/blob/dev/src/utils/make_keypairs.rs) - Generates a specified number of keypairs