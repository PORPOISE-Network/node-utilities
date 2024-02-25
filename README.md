# 🐬 PORPOISE Network Node Utilities

This is a super-simple typescript implementation of the functions needed to generate survey roots and prediction commitments
for PORPOISE that are compatible with the [PORPOISE oracle contracts](https://github.com/PORPOISE-Network/oracle-v1). This package
was designed to have no external dependences. 

See the [PORPOISE survey commitment protocol](https://info.porpoise.network/whitepaper/survey-commitment-protocol) for more information. 

## Installation

```sh
npm install --save-dev @zkporpoise/node-utilities
```

## Import

```typescript
import {
  ethHexString,
  padArrayToPowerOfTwo,
  mapBigIntTo256BitNumber,
  computeMerkleRoot,
  convertProofToHex,
} from "@zkporpoise/node-utilities"
```

## Methods

- `computeMerkleRoot(leafs: Buffer[], proof: Buffer[], tracker)`: takes output of `padArrayToPowerOfTwo` for leafs, an empty array, and the index of the leaf for which the proof is needed and returns an array containing the proof array, Merkle root, and root index. 

- `padArrayToPowerOfTwo(arr: string[], paddingValue: string)`: given an array of string elements representing a PORPOISE survey, returns sha256-hashed leaf values. Note, that leafs must be given in the order 🐬, ⏰, 🔮, 🗳️1, ... Additionally, the deadline value (⏰) is encoded as 'hex' while all other inputs are 'binary' encoded.

- `mapBigIntTo256BitNumber(bigIntValue: bigint)`: Converts a BigInt into a hex string so that hashes computed offchain will match hashes computed on-chain. 

- `ethHexString(muBuffer: Buffer)`: returns a `0x${string}` type needed for viem

- `convertProofToHex`: converts a Buffer array and Buffer and converts to `0x${string}` types for use with 

## Example

```typescript
    it("Register a survey only once.", async function () {
      const { porpacle, owner } = await loadFixture(deployPorpacle);

      const dolphin: string = "When Moon?";
      const alarmclock: number = new Date().getTime() + 86400000;
      const hexAlarmClock: string = mapBigIntTo256BitNumber(BigInt(alarmclock));
      const oracle: string = "0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266";
      const option1: string = "Soon";
      const option2: string = "NGMI";

      // the order of the array elements is important, pad values MUST be `0` string
      const paddedComponents: Buffer[] = padArrayToPowerOfTwo([dolphin, hexAlarmClock, oracle, option1, option2], '0');

      const bufferMerkleProof: [Buffer, Buffer[], number] = computeMerkleRoot(paddedComponents, [], 1);
      const stringMerkleProof: [`0x${string}`[], `0x${string}`] = convertProofToHex(bufferMerkleProof[1], bufferMerkleProof[0]);
      const proof: `0x${string}`[] = stringMerkleProof[0];
      const root: `0x${string}` = stringMerkleProof[1];

      await porpacle.write.registerSurvey([proof, root, BigInt(alarmclock)]);
      const surveyTimeouts: bigint = await porpacle.read.surveyTimeouts([root]);
      expect(surveyTimeouts).to.equal(BigInt(alarmclock));
      await expect(porpacle.write.registerSurvey([proof, root, BigInt(alarmclock)])).to.be.rejectedWith("Survey already registered");
    });
```