# Universal Turing Machine

Turing's standard description encodes a machine as a string using the following notation:

| Notation | Meaning |
|---|---|
| `DA`, `DAA`, … | States q1, q2, … |
| `D` | Blank |
| `DC`, `DCC`, … | Symbols 0, 1, … |
| `L`, `R` | Move left, right |
| `;` | Row separator |

For example, this description table computes the binary expansion of 1/3 (0.010101…):

```
EE; D A D D C R D A A ; D A A D D C C R D A .
```

| State | Symbol | Print | Move | Next |
|-------|--------|-------|------|------|
| q1 | blank | 0 | R | q2 |
| q2 | blank | 1 | R | q1 |

Programs are stored in `programs/`. To run one against both implementations and print the output as a digit string:

```bash
./run <program> <duration>
```

## Rhetorical Design

### Purpose

The 1936 paper "On Computable Numbers, with an Application to the Entscheidungsproblem" is historically significant as the work in which Turing first defined computation in precise, mechanical terms. The mathematical conventions of the paper, however, present a barrier for readers with a programming rather than logic background. This project translates Turing's construction of the Universal Turing Machine into executable code, making it accessible to readers with a programming background.

### Strategy

The implementation is provided in two functional languages — Common Lisp and Haskell — because the m-configuration definitions in the paper rely heavily on mutual recursion and higher-order operations, which map naturally to functional style. Function names are expanded from single letters to descriptive identifiers. A number of errors in the original paper were discovered in the process.

## Technical Challenges

### Translating m-configurations into executable operations

**Problem.** Turing specifies each m-configuration as a case table: for each symbol the head might be reading, a sequence of tape operations followed by a jump to another m-configuration. His utility functions take the next m-configuration as a parameter — continuation-passing style in the mathematical description itself. This notation describes structure, not execution: there is no concept of calling, returning, or threading state.

**Solution in Lisp.** Each m-configuration becomes a function. A reader macro `^` is defined as shorthand for `(lambda () ...)`, so continuations are passed as zero-argument closures, closely mirroring the structure of the original.

**Solution in Haskell.** Continuations are monadic actions of type `T ()`, where `T` is the transformer stack `ReaderT Handle (StateT Integer IO)`. The tape (file handle) and head position are threaded implicitly through the stack, and each m-configuration is a monadic action passed as an argument to the next.
