# Regexes

- [Regexes](#regexes)
  - [Character classes](#character-classes)
  - [Quantifiers](#quantifiers)
  - [PCRE flags](#pcre-flags)
  - [Complex cases](#complex-cases)
    - [Capturing sequences of N characters](#capturing-sequences-of-n-characters)
    - [Not matching a line matching pattern](#not-matching-a-line-matching-pattern)
    - [Matching a pattern when it's not at the beginning](#matching-a-pattern-when-its-not-at-the-beginning)
  - [Language incompatibilities](#language-incompatibilities)
    - [Javascript](#javascript)

## Character classes

|   Class   | Extended form |
| :-------: | :-----------: |
| `:alnum:` |  `a-zA-Z0-9`  |
| `:blank:` |     ` \t`     |

## Quantifiers

Form: `{M}`, `{M, N}`, `{M,}`.

They're greedy by default! The non greedy form is `{M, N}}`.

Don't forget that when used with a capturing group, the group will capture only the last occurrence:

```ruby
" 2 3 4".scan /( \d){2,}/ # => [[" 4"]]
```

## PCRE flags

- `m`: match newlines with `.`
  - `s`: match line beginning/end with `^`/`$` when `m` is used

WATCH OUT! There are exceptions on some languages; see the specific language.

## Complex cases

### Capturing sequences of N characters

Use a null-character match, followed by lookahead!

The characters in the lookahead need to be in a group, otherwise, they're not captured; the outermost group is purely lookahead syntax.

```ruby
"01234".scan /(?=(\d{3}))/
# [["012"], ["123"], ["234"]]
```

### Not matching a line matching pattern

Awesome! See https://stackoverflow.com/a/406408.

### Matching a pattern when it's not at the beginning

Use a negative lookahead with the beginning-of-input metacharacter, as it's supported in capturing groups:

```ruby
"Re: Re: Re: Pizza".gsub /(?!^)Re: /, '' # "Re: Pizza"
```

The lookbehind version (`/(?<!^)Re: /`) is semantically more precise, however, it's not supported in all JS versions.

## Language incompatibilities

### Javascript

The negative lookbehind (`?<!`) may not be supported in some Javascript versions. Where possible, use the negative lookahead (`?!`).
