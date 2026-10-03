# Exploring the My Little Pony dialogue dataset

All commands were run from the repo root on the EC2 instance. The dataset is stored at `data/clean_dialog.csv`.

## How big is the dataset?

```
ls -lh data/clean_dialog.csv
wc -l data/clean_dialog.csv
csvtool height data/clean_dialog.csv
```

The file is 4.7 MB. It has 36,860 lines, and `csvtool height` agrees (36,860 rows), so no dialogue field contains an embedded line break. Excluding the header, the dataset contains **36,859 lines of dialogue**.

## What's the structure of the data?

```
head -n 5 data/clean_dialog.csv
csvtool width data/clean_dialog.csv
csvtool col 3 data/clean_dialog.csv | tail -n +2 | sort | uniq -c | sort -rn | head
```

The file has 4 fields, and every value is wrapped in double quotes:

| Field | Contents | Example |
|---|---|---|
| `title` | Episode title | `Friendship is Magic, part 1` |
| `writer` | Episode writer(s) | `Lauren Faust` |
| `pony` | Who speaks the line | `Twilight Sparkle` |
| `dialog` | The spoken text (one line of dialogue per row) | `...sun and moon...` |

The `pony` field has 842 distinct values. The most frequent speakers are Twilight Sparkle (4,745), Rainbow Dash (3,072), Pinkie Pie (2,833), Applejack (2,748) and Rarity (2,660).

## How many episodes does it cover?

```
csvtool col 1 data/clean_dialog.csv | tail -n +2 | sort | uniq | wc -l
```

`tail -n +2` skips the header, and `sort | uniq` keeps one copy of each title. The dataset covers **197 episodes**.

## Unexpected aspects

**1. Multiple speakers in one `pony` field.** Some lines are spoken by several characters at once, for example `Narrator and Twilight Sparkle` or `Twilight Sparkle and Rarity`. 294 rows contain " and " in the speaker field:

```
csvtool col 3 data/clean_dialog.csv | grep -c " and "
```

**2. Inconsistent and misleading names for the same character.** Listing every speaker name that contains "Twilight" shows variants such as `Twilight and Fluttershy` (short name, no "Sparkle"), `Young Twilight Sparkle`, `Future Twilight Sparkle`, and even `All sans Twilight Sparkle`, a line Twilight explicitly does *not* say:

```
csvtool col 3 data/clean_dialog.csv | grep "Twilight" | sort | uniq -c | sort -rn
```

Both issues mean a simple text search for a name will miscount how often a character speaks.

**3. Names also appear inside the dialogue.** Characters often say each other's names, so searching the whole file instead of the `pony` column counts lines *about* a pony, not lines *by* her. For Rarity, a whole-file `grep -c "Rarity"` returns 3,517, versus 2,660 when only the speaker column is searched.
## Speaker frequency (Task 4)

To count only lines a pony speaks alone, I extracted the `pony` column and matched the name exactly. `^` anchors the match to the start of the line and `$` to the end, so combined speakers like `Twilight Sparkle and Rarity` and variants like `Young Twilight Sparkle` are excluded:

```
csvtool col 3 data/clean_dialog.csv | grep -c "^Twilight Sparkle$"
csvtool col 3 data/clean_dialog.csv | grep -c "^Rarity$"
csvtool col 3 data/clean_dialog.csv | grep -c "^Pinkie Pie$"
csvtool col 3 data/clean_dialog.csv | grep -c "^Rainbow Dash$"
csvtool col 3 data/clean_dialog.csv | grep -c "^Fluttershy$"
```

Percentages use all 36,859 dialogue lines as the denominator, computed with `bc`:

```
echo "scale=2; 100 * 4745 / 36859" | bc
```

| Pony | Lines | % of all lines |
|---|---|---|
| Twilight Sparkle | 4,745 | 12.87 |
| Rainbow Dash | 3,072 | 8.33 |
| Pinkie Pie | 2,833 | 7.68 |
| Rarity | 2,660 | 7.21 |
| Fluttershy | 2,109 | 5.72 |

Results are also saved in `Line_percentages.csv`.

**Choice made:** lines shared with other speakers are not counted. Counting any speaker field that contains the name (`grep -c "Twilight Sparkle"` without anchors) would add a few dozen lines per pony, but would also wrongly count cases like `All sans Twilight Sparkle`.
