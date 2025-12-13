# Test Task: Implement Recursive Chunking (Recursive Splitting)

## Goal

Build the simplest working implementation of **recursive chunking**: a function that takes a text input and produces a list of chunks where **each chunk is at most `max_chunk_size` characters**.

You can use **any language**. You can ignore overlap (no need to add or handle overlap between chunks).

---

## What is recursive chunking?

When you feed long text into an AI model/agent, you often need to split it into smaller pieces (“chunks”) so each piece fits within a size limit (tokens/characters). A naïve approach is to cut the text every *N* characters. That works, but it often splits in awkward places (mid-sentence, mid-word), which hurts retrieval and downstream reasoning.

**Recursive chunking** is a simple strategy that tries to split text at “nice” boundaries first (like paragraphs), and only falls back to more granular boundaries (like sentences or words) when necessary. It is called *recursive* because when a piece is still too large, you split that piece again using the next “smaller” separator rule.

### Intuition

You define an ordered list of separators from “largest boundary” to “smallest boundary”, e.g.:

1. Paragraph boundary: `"\n\n"`
2. Line boundary: `"\n"`
3. Sentence-ish boundary: `". "`
4. Word boundary: `" "`
5. Fallback: character-level splitting (no separator)

Then:

- Try splitting the text by the first separator (`"\n\n"`).
- If any resulting piece is still larger than `max_chunk_size`, recursively split *that piece* using the next separator (`"\n"`), and so on.
- If you run out of separators and the piece is still too big, do a final fallback: split by fixed width characters.

This produces chunks that tend to respect structure when possible, and only gets “brutal” (character splitting) when needed.

---

## Requirements

Implement a function (name is up to you) with this signature:

- Input:
  - `text: string`
  - `max_chunk_size: int` (in **characters**)
  - `separators: list<string>` (ordered from coarse → fine)

- Output:
  - `chunks: list<string>` where:
    - Each chunk length is `<= max_chunk_size`
    - Concatenating the chunks in order reconstructs the original text **exactly** (including whitespace), or reconstructs it exactly *up to* the chosen separator behavior (see “Separator handling” below).  
      **Choose one approach and document it clearly.**

### Separator handling (choose one)

You can choose either of the following (both are acceptable, just be consistent):

1. **Keep separators attached**: when splitting by a separator, keep the separator at the end of the previous piece (or beginning of the next).  
2. **Drop separators but preserve reconstruction**: split in a way that the original text can be reconstructed exactly (e.g., by reinserting separators during joining, or by splitting while retaining delimiter tokens).

The simplest approach in many languages is “keep separators attached” by splitting in a way that retains delimiters.

### Minimal behavior expectations

- If `len(text) <= max_chunk_size`, return `[text]`.
- If a separator does not occur in the current piece, move to the next separator.
- If splitting by a separator produces many small fragments, **pack them back** into chunks up to `max_chunk_size` (greedy packing is fine).
- Final fallback: if the piece is still too big at the last separator, split by fixed character width.

---

## Suggested algorithm

### High-level outline

1. **Recursive split**:
   - `split_recursively(text, sep_index)`:
     - If `len(text) <= max_chunk_size`: return `[text]`
     - If `sep_index >= len(separators)`: return fixed-width chunks
     - Let `sep = separators[sep_index]`
     - If `sep` not found in `text`: return `split_recursively(text, sep_index + 1)`
     - Otherwise:
       - Split `text` by `sep` **while deciding how to keep/handle `sep`**
       - For each piece:
         - If `len(piece) <= max_chunk_size`: keep it
         - Else: recursively split it with the next separator
       - After that, **pack** the resulting pieces into chunks (greedy) up to `max_chunk_size`
       - Return packed chunks

2. **Greedy pack**:
   - Combine consecutive pieces until adding the next would exceed `max_chunk_size`, then start a new chunk.

### Pseudocode (language-agnostic)

```text
function recursive_chunk(text, max_size, separators):
    return split_rec(text, 0)

function split_rec(text, i):
    if length(text) <= max_size:
        return [text]

    if i == length(separators):
        return fixed_width_split(text, max_size)

    sep = separators[i]
    if sep not in text:
        return split_rec(text, i + 1)

    parts = split_text_retaining_separator(text, sep)  // your chosen separator strategy

    out = []
    for p in parts:
        if length(p) <= max_size:
            out.append(p)
        else:
            out.extend(split_rec(p, i + 1))

    return greedy_pack(out, max_size)

function greedy_pack(parts, max_size):
    chunks = []
    current = ""
    for p in parts:
        if current is empty:
            current = p
        else if length(current) + length(p) <= max_size:
            current = current + p
        else:
            chunks.append(current)
            current = p
    if current not empty:
        chunks.append(current)
    return chunks
