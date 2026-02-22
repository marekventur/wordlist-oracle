# Scrabble Word List Oracle (multi-language)

A script that checks how well a candidate Scrabble word list covers the
official tournament dictionary for a given language. The reference
dictionaries are not stored in this repository; they are downloaded
automatically from the [Scrabble3D Dictionaries](https://github.com/Scrabble3D/Dictionaries) project on first use.

This oracle is built to safely verify the quality of a candidate word list against 
an authoritative reference list for a given language, even if the license of 
the reference list is not permissive. To achieve that, this script ensures that
no information about individual words in the authoritative reference list 
is returned and that only aggregated metrics are reported.

All languages provided by Scrabble3D are supported: brazilian, catalan,
deutsch, english, english_phonetic, espanol, francais, greek, hebrew, hollands,
hungarian, irish, italiano, latin, persian, polish, portuguese, romana, russian,
scottishgaelic, slovak, suomi, svenska, tamil, turkish.

## Usage

```bash
# German (default)
cat my_wordlist.txt | python wordlist-oracle.py

# Another language
cat my_wordlist.txt | python wordlist-oracle.py --language english
cat espanol_words.txt | python wordlist-oracle.py --language espanol
```

The word list should be plain text, one word per line, uppercase. Words outside
the 2–9 letter range are ignored.

On first run for a given language, the script downloads the corresponding `.dic.zip`
from the Scrabble3D GitHub repository and extracts the `.dic` file next to the
script (e.g. `deutsch.dic`, `english.dic`). Subsequent runs use the local copy
for that language.

### Output

JSON to stdout, progress to stderr:

```json
{
  "language": "deutsch",
  "nonce": "",
  "fraction": 1,
  "reference_total": 184243,
  "reference_sampled": 184243,
  "candidate_total": 95000,
  "candidate_sampled": 95000,
  "true_positives": 91234,
  "false_positives": 3766,
  "false_negatives": 93009,
  "recall_pct": 49.5162,
  "precision_pct": 96.0358
}
```

| Field | Meaning |
|-------|---------|
| `language` | Dictionary language used |
| `reference_total` | Words in the authoritative reference list for that language |
| `reference_sampled` | Reference words considered after fraction filtering |
| `candidate_total` | Words in your list (2–9 letters) |
| `candidate_sampled` | Candidate words considered after fraction filtering |
| `true_positives` | Candidate words that are valid in the reference |
| `false_positives` | Candidate words not in the reference (invalid) |
| `false_negatives` | Reference words missing from your candidate list |
| `recall_pct` | % of reference words covered by your list |
| `precision_pct` | % of your list that is valid |

## Sampling for fast test runs

The `--fraction` and `--nonce` options let you run on a random subset of both
lists simultaneously. Because the same hash filter is applied to both sides,
recall and precision scores remain representative at any fraction.

```bash
# Run on ~1% of both lists — much faster
cat my_wordlist.txt | python wordlist-oracle.py --language deutsch --fraction 100

# Different random subset at the same size
cat my_wordlist.txt | python wordlist-oracle.py --fraction 100 --nonce "run2"
```

The filter is deterministic: `SHA256(nonce + word) < 2²⁵⁶ / fraction`.
The same nonce always produces the same subset.

## Requirements

Python 3.6+, no third-party dependencies.

## License

The oracle script (`wordlist-oracle.py`) is released under the MIT License.

The dictionary data (`.dic` files) is copyright of their respective authors and
distributed under their own terms with the Scrabble3D project. It is not included
here; the script downloads it automatically from the [Scrabble3D Dictionaries](https://github.com/Scrabble3D/Dictionaries) repository.
