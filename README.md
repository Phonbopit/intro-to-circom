Intro to circom
---

- [Circom Docs](https://docs.circom.io/)
- [Circomlib](https://github.com/iden3/circomlib/)

## Prerequisite

- [Rust](https://www.rust-lang.org/)
- [snarkjs](https://github.com/iden3/snarkjs)
- [Circom]()

Install Rust with rustup

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Install snarkjs on global using npm

```bash
npm install -g snarkjs
```

Install circom

```bash
git clone https://github.com/iden3/circom.git
cargo build --release

# Then install this binary as follows:
cargo install --path circom
```

## Usage

Create a new empty folder that contains output files:

```bash
mkdir multiplier2
```

Create simple circuit called `Multiplier2`

```circom
pragma circom 2.0.0;

template Multiplier2() {
  signal input a;
  signal input b;
  signal output c;

  // Constraints
  c <== a * b;
}

component main = Multiplier2();
```

and saved with named `circuits/multiplier2.circom`

Then, compile the circuit with syntax:

```bash
circom <CIRCUIT_FILE> --r1cs --wasm --sym --c
```

For example:

```bash
circom circuits/multiplier2.circom --r1cs --wasm --sysm --c --output ./multiplier2
```

- `--r1cs` - generate R1CS constraint system in binary format
- `--wasm` - it generates `multiplier2_js` folder that contains the WebAssembly code and files needed to generate the [witness](https://docs.circom.io/background/background/#witness)
- `--sym` - it generates file `multiplier2.sym`, a symbols file for debugging printing the constraint system.
- `--c` - it generates the `multiplier2_cpp` folder, needed to compile the C code to generate the witness.
- `--output` - generate all files into `output` folder.

## Print info on circuit

```
snarkjs r1cs info multiplier2/multiplier2.r1cs
```

The result should be:

```bash
[INFO]  snarkJS: Curve: bn-128
[INFO]  snarkJS: # of Wires: 4
[INFO]  snarkJS: # of Constraints: 1
[INFO]  snarkJS: # of Private Inputs: 2
[INFO]  snarkJS: # of Public Inputs: 0
[INFO]  snarkJS: # of Labels: 4
[INFO]  snarkJS: # of Outputs: 1
```

## Compute the witness

Go to `multiplier2/multiplier2_js`, which is the javascript program and then we can use snarkjs to compute witness

we have to create an input file named `input.json` with input signals `a` and `b`

```json
{
  "a": 10,
  "b": 5
}
```

The output should be `50` because `10 * 5 = 50`

the syntax to generate witness is

```bash
node generate_witness.js <file.wasm> <input.json> <output.wtns>
```

- `<file.wasm>` - We use `multiplier2.wasm` file.
- `<input.json>` - We use file that we just created `input.json`
- `<output.wtns>` - The output file that we want.

Then run this:

```bash
node generate_witness.js multiplier2.wasm input.json output.wtns
```

## Viewing the witness

Now we have a binary file of witness, we need to convert it to JSON file with snarkjs

```bash
snarkjs wtns export json output.wtns output.json
```

The result is:

```
[
 "1",
 "50",
 "10",
 "5"
]
```

- Line 1 : `1` - is just a content of the contraints system generated.
- Line 2 : `50` - is the public output
- Line 3, 4 : `10` and `5` - are private inputs.

## Scripts

`./bin/run.sh`
