----Tạo Validator bao gồm datum và redeemer Plutus V3 để lấy mã CBOR------

use aiken/primitive/string
use cardano/transaction.{OutputReference, Transaction}

pub type Redeemer {
msg: ByteArray,
}

pub type Datum {
msg: ByteArray,
}

// The validator is a function that takes the following arguments:
// - datum: the datum of the UTXO being spent
// - redeemer: the redeemer of the UTXO being spent
// - utxo: the UTXO Reference 
// - self: the transaction being created
// - ctx: the script context
// The validator returns a boolean value indicating whether the transaction is valid or not

validator datum_redeemer {
spend(
datum: Option<Datum>,
redeemer: Redeemer,
_utxo: OutputReference,
_self: Transaction,
) {
// The redeemer is a custom type that contains a message
expect Some(datum_input) = datum
let d: Datum = datum_input
// The datum is a custom type that contains a message

let a = d.msg == redeemer.msg
a?
}

else(_) {
fail
}
}

  // // If needs be, remove any of unneeded handlers above, and use:
  //
  // else(_ctx: ScriptContext) {
  //   todo @"fallback logic if none of the other purposes match"
  // }
  //
  // // You will also need an additional import:
  // //
  // // use cardano/script_context.{ScriptContext}



import { Blockfrost, Lucid, Data } from "https://deno.land/x/lucid/mod.ts";

const lucid = new Lucid({
    provider: new Blockfrost(
        "https://cardano-preview.blockfrost.io/api/v0",
        "previewTN8UXKGlPYoZF3fPyqhtaK4H3jNoGIQc",
    ),
});

// Khởi tạo ví từ seed
const seed = "van slice sheriff camp dash village record cute must camp tail roast aerobic brown regular divert luxury spend feed cause minor saddle goat buddy";
lucid.selectWalletFromSeed(seed);

// Lấy địa chỉ ví
const address = await lucid.wallet.address();
console.log(`Ví gửi: ${address}`);

// Định nghĩa script
const datum_redeemer_scripts = lucid.newScript({
  type: "PlutusV3",
  script: "587b01010029800aba2aba1aab9eaab9dab9a4888896600264646644b30013370e900118031baa00289919912cc004cdc3a400060126ea8006266e1ccdc01bad300b300a37540026eb4c02c0192090038b201030090013009300a00130073754005164014600c600e002600c004600c00260066ea801a29344d9590011",
});

const datum_redeemerAddress = datum_redeemer_scripts.toAddress();
console.log(`datum_redeemer address: ${datum_redeemerAddress}`);

// Định nghĩa Schema cho Datum và Redeemer
const DatumSchema = Data.Integer();
const RedeemerSchema = Data.Integer();

// Tạo Datum với giá trị 13
const Datum = () => Data.to(13n, DatumSchema);

// Tạo Redeemer với giá trị được truyền vào
const Redeemer = (value: bigint) => Data.to(value, RedeemerSchema);

const lovelace_lock = 200_000_000n; // 200 ADA

// Lock UTxO
export async function lockUtxo(lovelace: bigint): Promise<string> {
  const tx = await lucid
    .newTx()
    .payToContract(datum_redeemerAddress, { Inline: Datum() }, { lovelace })
    .commit();
  const signedTx = await tx.sign().commit();
  const txHash = await signedTx.submit();
  return txHash;
}

// Mở khóa UTxO
export async function unlockUtxo(redeemerValue: bigint): Promise<string> {
  const redeemer = Redeemer(redeemerValue);
  const utxos = await lucid.utxosAt(datum_redeemerAddress);
  const utxo = utxos.find((utxo) => utxo.datum === Datum());
  if (!utxo) throw new Error("No UTxO found with the specified Datum");
  const tx = await lucid
    .newTx()
    .collectFrom([utxo], redeemer)
    .attachScript(datum_redeemer_scripts)
    .payTo(
      "addr_test1qz3vhmpcm2t25uyaz0g3tk7hjpswg9ud9am4555yghpm3r770t25gsqu47266lz7lsnl785kcnqqmjxyz96cddrtrhnsdzl228",
      { lovelace: redeemerValue * 1_000_000n } // Sử dụng redeemerValue thay vì redeemer
    )
    .commit();
  const signedTx = await tx.sign().commit();
  const txHash = await signedTx.submit();
  return txHash;
}

async function main() {
  try {
    // Lock UTXO
    // const lockTxHash = await lockUtxo(lovelace_lock);
    // console.log(`Transaction hash for lock: ${lockTxHash}`);
    
    // Wait for the transaction to be confirmed (in practice, you might need to wait for blocks)
    // For simplicity, assume it's confirmed immediately
    
    // Unlock UTXO with redeemer = 187
    const unlockTxHash = await unlockUtxo(187n);
    console.log(`Transaction hash for unlock: ${unlockTxHash}`);
  } catch (error) {
    console.error("Error:", error);
  }
}

main();

