# javascript implementation of zkSnark

This is a javascript implementation of zkSnarks.

This library allows to do the trusted setup,  generate proofs and verify the proofs.

This library uses the compiled circuits generated by the jaz compiler.

## Install

```
npm install zkSnark
```

## Usage

### import

```
const zkSnark = require("zksnark");
```

### Load a circuit.

```
// "myCircuit.cir" is the output of the jaz compiler

const circuitDef = JSON.parse(fs.readFileSync("myCircuit.cir", "utf8"));
const circuit = new zkSnark.Circuit(circuitDef);
```

### Inspect the circuit.

```
    // `signalId` can always be a number or an alias string

    circuit.nConstrains; // number of constrains
    circuit.nSignals; // number of signals
    circuit.nPublic; // number of public signals (nOutputs + nPublicInputs)

    // The array of signals is always sorted in this order:
    // [ 1, outputs, publicInputs, privedInputs, internalSignals, constants]

    // returns a,b and c coeficients of the `signalId` on a given `constrain`
    circuit.a(constrain, signalId)
    circuit.b(constrain, signalId)
    circuit.c(constrain, signalId)

    circuit.nOutputs           // number of public outputs
    circuit.pubInputs          // number of public inputs
    circuit.nPrvInputs         // number of private inputs
    circuit.nInputs            // number of inputs ( nPublicInputs + nPrivateInputs)
    circuit.nVars              // number of variables ( not including constants (one is a variable) )
    circuit.nSignals           // number of signals ( including constants )

    circuit.outputIdx(i)       // returns the index of the i'th output
    circuit.inputIdx(i)        // returns the index of the i'th input
    circuit.pubInputIdx(i)     // returns the index of the i'th public input
    circuit.prvInputIdx(i)     // returns the index of the i'th private input
    circuit.varIdx(i)          // returns the index of the i'th variable
    circuit.constantIdx(i)     // returns the index of the i'th constant
    circuit.signalIdx(i)       // returns the index of the i'th signal

    // returns signal Idx given a signalId
    // if the idx >= n , it is a constant
    // if the idx == -1, the signal does not exist
    circuit.signalId2idx(signalId);

    // returns an array aliases names for a given signalId
    circuit.signalNames(signalId)

    // input is a key value object where keys are the signal names
    //   of all the inputs (public and private)
    // returns an array of values that represent the witness
    circuit.generateWitness(input)
```

### Trusted setup

```
const setup = zkSnark.setup(circuit);
fs.writeFileSink("myCircuit.vk_proof", JSON.stringify(setup.vk_proof), "utf8");
fs.writeFileSink("myCircuit.vk_verifier", JSON.stringify(setup.vk_verifier), "utf8");
setup.toxic  // Must be discarded.
```

### Generate proof

```
const circuitDef = JSON.parse(fs.readFileSync("myCircuit.cir", "utf8"));
const circuit = new zkSnark.Circuit(circuitDef);
const input = {
    "main.pubIn1": "123",
    "main.out1": "456"
}
const witness = circuit.generateWitness(input);
const vk_proof = JSON.parse(fs.readFileSync("myCircuit.vk_proof", "utf8"));

const {proof, publicSignals} = zkSnark.genProof(vk_proof, witness);
```

### Verifier

```
const vk_verifier = JSON.parse(fs.readFileSync("myCircuit.vk_verifier", "utf8"));

if (zkSnark.isValid(vk_verifier, proof, publicSignals)) {
    console.log("The proof is valid");
} else {
    console.log("The proof is not valid");
}
```
