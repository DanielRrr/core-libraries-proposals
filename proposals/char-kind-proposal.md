---
author: Daniel Rogozin, Rinat Stryungis
date-accepted: ""
ticket-url: "https://gitlab.haskell.org/ghc/ghc/-/issues/11342"
implemented: ""
---

This proposal is [discussed at this merge request](https://gitlab.haskell.org/ghc/ghc/-/merge_requests/3598).

# The Character Kind

We would like to propose and discuss several changes related to the character kind. Some of those changes has been implemented jointly with Rinat Stryungis.


## Motivation

The purpose of this patch is to provide a possibility of analysing type-level strings (symbols) as term-level ones. This feature allows users to implement such programs as type-level parsers. One needs to have full-fledged support of type-level characters as well as we already have for strings and numbers. In addition to this functionality, it makes sense to introduce the set of type-families, counterparts of functions defined in the Data.Char module in order to work with type-level strings and chars as usual (more or less).

For more convenience, it’s worth having some of the character-related type families as built-ins and generating the rest of ones as type synonyms.


## Proposed Change Specification

The patch fixes #11342, an issue opened by Alexander Vieth several years ago. In this patch, we introduced the Char Kind, a kind of type-level characters, with the additional type-families, type-level counterparts of functions from the `Data.Char` module.

First of all, we overview the additional type families implemented by us in this patch.

```haskell
type family CmpChar (a :: Char) (b :: Char) :: Ordering
```

Comparison of type-level characters, as a type family. A type-level analogue of the function `compare` specified for characters.

```haskell
type family LeqChar (a :: Char) (b :: Char) :: Bool
```

This is a type-level comparison of characters as well. `LeqChar` yields a Boolean value and corresponds to `(<=)`.

```haskell
type family ConsSymbol (a :: Char) (b :: Symbol) :: Symbol
```

This extends a type-level symbol with a type-level character

```haskell
type family UnconsSymbol (a :: Symbol) :: Maybe (Char, Symbol)
```

This type family yields type-level `Just` storing the first character of a symbol and its tail if it is nonempty and `Nothing` otherwise.


Type-level counterparts of the functions `toUpper`, `toLower`, and `toTitle` from 'Data.Char'.

```haskell
type family ToUpper (a :: Char) :: Char

type family ToLower (a :: Char) :: Char

type family ToTitle (a :: Char) :: Char
```

These type families are type-level analogues of the functions `ord` and `chr` from Data.Char respectively.

```haskell
type family CharToNat (a :: Char) :: Nat

type family NatToChar (a :: Nat) :: Char
```

A type-level analogue of the function `generalCategory` from `Data.Kind`.

```haskell
type family GeneralCharCategory (a :: Char) :: GeneralCategory
```


The second group of type families consists of built-in unary predicates. All of them are based on their corresponding term-level analogues from `Data.Char`. The precise list is the following one:

```haskell
type family IsAlpha (a :: Char) :: Bool

type family IsAlphaNum (a :: Char) :: Bool

type family IsControl (a :: Char) :: Bool

type family IsPrint (a :: Char) :: Bool

type family IsUpper (a :: Char) :: Bool

type family IsLower (a :: Char) :: Bool

type family IsSpace (a :: Char) :: Bool

type family IsDigit (a :: Char) :: Bool

type family IsOctDigit (a :: Char) :: Bool

type family IsHexDigit (a :: Char) :: Bool

type family IsLetter (a :: Char) :: Bool
```

We also provide several type-level predicates implemented via the `GeneralCharCategory` type family.

```haskell
type IsMark a = IsMarkCategory (GeneralCharCategory a)

type IsNumber a = IsNumberCategory (GeneralCharCategory a)

type IsPunctuation a = IsPunctuationCategory (GeneralCharCategory a)

type IsSymbol a = IsSymbolCategory (GeneralCharCategory a)

type IsSeparator a = IsSeparatorCategory (GeneralCharCategory a)
```

Built-in type families we described above are supported with the corresponding definitions and functions in the modules `compiler/GHC/Builtin/Names.hs`, `compiler/GHC/Builtin/Types.hs`, and `compiler/GHC/Builtin/Types/Literals.hs`. In addition to type families, our patch contain the following updates:

1. parsing the 'x' syntax
2. type-checking `'x' :: Char`
3. type-checking `Refl :: 'x' :~: 'x'`
4. Typeable / TypeRep support
5. template-haskell support
6. Haddock related updates
7. tests


## Examples

The following code snippets are well-defined with our patch:

The example of type level character comparison:
```haskell
f1 :: CmpChar 'x' 'x' :~: EQ
f1 = Refl

f2 :: CmpChar 'x' 'y' :~: LT
f2 = Refl
```

The following examples are related to type families based on the corresponding functions from `Data.Char`:
```haskell
testIsLower :: '(IsLower 'x', IsLower 'X') :~: '(True, False)
testIsLower = Refl

testIsUpper :: '(IsUpper 'x', IsUpper 'X') :~: '(False, True)
testIsUpper = Refl

testIsAlpha :: '[IsAlpha 'X', IsAlpha 'x', IsAlpha '6', IsAlpha '/']
  :~: '[True, True, False, False]
testIsAlpha = Refl

testIsAlphaNum :: '[IsAlphaNum ',', IsAlphaNum 'x', IsAlphaNum '6', IsAlphaNum '/']
  :~: '[False, True, True, False]
testIsAlphaNum = Refl
```

## Effect and Interactions

Our patch allows one to analyse type-level strings more precisely having promoted chars and helpers that almost entirely cover term-level functions from `Data.Char`. At the moment, we haven't identified any contentious interactions yet.


## Costs and Drawbacks
API is increased with the proposed updates. Moreover, most of those type families will be redundant in the presence of full-fledged dependent types when term-level and type-level syntax coincide.


## Alternatives
Previously, there was a quite similar patch by Vieth, see [here](https://gitlab.haskell.org/ghc/ghc/-/issues/11342#note_173991). In contrast to Vieth’s approach, we use the same Char type and don’t introduce the different `Character` kind. We provide slightly more helpers to work with the Char kind as we described above.


## Unresolved Questions
We suppose that the issue is solved. We would be glad to receive any feedback and comments.
