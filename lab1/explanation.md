# What `IRLab1-skeleton.ipynb` Does

## The big picture

This notebook asks you to build a mini **search engine**. Given a folder
of webpages, you will:

1. Read all the words in all the pages.
2. Build an **index** — a big lookup table that says, for every word,
   which pages contain it.
3. Use that index to answer searches like `"stanford computer science"`
   (find pages that contain *all* these words).
4. Then do it again, but make the index take up less disk space
   (compression).
5. Answer some questions about speed and size for a short report.

The data is a folder of plain text files (HTML already removed) from
stanford.edu. It's split into 10 sub-folders named `0` to `9`. Each
sub-folder is called a **block**. The rule of the lab is: load and
process **one block at a time**, never the whole dataset at once. This
mimics what real search engines have to do when the data is too big for
memory.

There's also a tiny `toy-data` folder with just a couple of files, so
you can test your code quickly before running it on the full ~170MB
corpus.

> Note: cells starting with `%%tee submission/...` just make a copy of
> that cell into a `submission/` folder for grading. You don't need to
> do anything about that — just write your code normally in the cell.

## Walking through the notebook, step by step

### 1. Setup
Creates the folders you'll need (`lab1-data`, `output_dir`, `tmp`,
`toy_output_dir`, `submission`) and shows you what the data looks like.

### 2. Build a simple (uncompressed) index

- **`IdMap`** — Words and file paths are easier to work with as small
  numbers instead of long strings. `IdMap` is a little tool that
  remembers "word X = number 5" and can go back and forth between the
  two. You fill in the two lookup directions.

- **`UncompressedPostings`** — This is given to you, not an exercise.
  For a given word, the list of page-numbers that contain it is called
  a **postings list**. This class shows how to turn that list into raw
  bytes to save on disk, and back again.

- **`InvertedIndex` and friends** — A family of classes for reading and
  writing the index to disk without loading the whole thing into
  memory:
  - `InvertedIndexWriter` — writes one word's postings list to disk at
    a time, and remembers where each one was written.
  - `InvertedIndexIterator` — lets you walk through an index on disk,
    one word at a time, like flipping pages in a book.
  - `InvertedIndexMapper` — lets you jump straight to one word's
    postings list on disk, instead of reading through the whole file.

- **`BSBIIndex`** — the main class that does the actual work. It's
  built up a little bit at a time across several cells:
  - `parse_block` — reads every file in one block and records which
    words appear in which files.
  - `invert_write` — turns that raw list into a proper postings list
    per word, and writes it to disk.
  - `merge` — combines the 10 per-block indexes into one final index.
  - `retrieve` — given a search query, finds pages that contain
    *every* word in the query.

- **`sorted_intersect`** — a helper that finds the common items between
  two sorted lists efficiently. Used by `retrieve`.

You'll test all of this first on the toy data, then run it on the real
data, and check your answers against provided example queries.

### 3. Build a compressed index

Same idea as before, but now postings lists are squeezed to take less
space, using a technique called **gap + variable-byte encoding**
(store the *difference* between consecutive page numbers, instead of
the page numbers themselves, then pack each difference into as few
bytes as possible). You implement this in **`CompressedPostings`** and
re-run the same index-and-search steps to check it still gives the same
answers.

### 4. Report

Short answer questions about what you built:
- A series of search queries (`q1` to `q5`) where you investigate what
  makes a search fast or slow (timed with `%timeit`), and explain why.
- Figuring out the index's size in bytes directly from its metadata
  (instead of just asking the file system), and comparing the
  compressed vs. uncompressed sizes.
- A few general questions about trade-offs in your design.

### 5. Extra credit

Implement one more compression method of your choice (e.g. gamma
encoding, delta encoding) in **`ECCompressedPostings`**, and briefly
compare it to the method from part 3.

## Your to-do list

Every place you need to write something is marked with:

```
### Begin your code

### End your code
```

Here is the full list, in the order they appear:

**Part 1 — basic index and search**
1. `IdMap._get_str` and `IdMap._get_id` — fill in the two lookup
   directions.
2. Write your own quick tests for `parse_block` on the toy data.
3. `InvertedIndexWriter.append` — write one word's postings list to
   disk and remember where you put it.
4. Write your own tests for `invert_write`.
5. `InvertedIndexIterator._initialization_hook` — set up anything the
   iterator needs before it starts.
6. `InvertedIndexIterator.__next__` — return the next word + postings
   list, reading only a little bit of the file at a time.
7. Write your own tests for `InvertedIndexIterator`.
8. `BSBIIndex.merge` — combine several block-indexes into one, in
   sorted order.
9. `InvertedIndexMapper._get_postings_list` — jump straight to one
   word's postings list on disk (no scanning the whole file).
10. Write your own tests for `_get_postings_list`.
11. `sorted_intersect` — find the common items in two sorted lists.
12. Write your own tests for `sorted_intersect`.
13. `BSBIIndex.retrieve` — answer a search query using the pieces
    above. Must not crash on words that don't exist in the corpus.

**Part 2 — compression**
14. `CompressedPostings.encode` and `.decode` (plus any helper methods
    you want) — gap-encode the postings list, then variable-byte
    encode each gap, and be able to reverse it.
15. Write your own tests for any helper methods.
16. Write your own tests for `encode`/`decode`.

**Part 3 — report (each of these also needs a short written answer in
the blockquote right below it)**
17. `q1` — write a query that runs slower than the example, and time
    it.
18. `q2` — write a query that runs twice as slow as `q1`.
19. `q3` — write query(ies) that prove *why* `q2` was slower (ruling
    out other possible causes).
20. `q4` — add a word to `q2` that makes it faster again.
21. `q5` — shuffle the words in `q4` and time it, to see if order
    matters.
22. Compute the index's size in bytes directly from `postings_dict`
    (one line of code).
23. Work out the smallest number of bits needed to store a single page
    number for this dataset.

**Part 4 — extra credit**
24. `ECCompressedPostings.encode` and `.decode` (plus helpers) — a
    second compression method of your choice, with a short written
    comparison to the one from Part 2.
