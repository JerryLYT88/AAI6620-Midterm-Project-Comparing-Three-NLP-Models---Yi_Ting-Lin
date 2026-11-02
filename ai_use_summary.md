# AI Use Summary

This file explains how I used Claude in this project. I asked Claude to
guide me step by step instead of writing the whole project for me. For
each step, Claude explained the idea, gave me hints or a code skeleton,
and asked me questions. I wrote, ran, and checked the code myself.

## How I Used AI at Each Stage

### 1. Setup
- **Claude:** Explained the difference between Git and GitHub, how to
  install Git, and how to create a virtual environment.
- **Me:** Installed Git, created the environment, and moved my project
  out of OneDrive after Claude explained that OneDrive can cause file
  sync problems.

### 2. Data Cleaning
- **Claude:** Gave me hints for reading the file and printing the
  beginning and the end of the text. Gave me a code skeleton for
  finding the start and end markers.
- **Me:** Wrote the code and found the extra Project Gutenberg text at
  the beginning and the end. I explained why it should be removed.
  I filled in the skeleton to remove it.
- **Claude:** Pointed out the token "6_d_" in my vocabulary, which
  showed the underscore problem.
- **Me:** Decided to fix it in the body text and to replace "_" with a
  space.

### 3. Tokenization and Vocabulary
- **Claude:** Gave me the regular expression for splitting words and
  punctuation, and explained why punctuation should be kept as tokens.
- **Me:** Checked the "holmes" variants before and after tokenization
  (16 variants became 1). Wrote the code to count how many tokens
  appear fewer than 1, 3, and 5 times, and their percentages. Chose the
  threshold of 3 and explained my reason.

### 4. Encoding and Data Split
- **Claude:** Explained stoi, itos, and how to use .get() for <unk>.
- **Me:** Wrote the decoding code and the train/validation split.
  Wrote the loop that prints "context -> target" pairs.

### 5. Models
- **Claude:** Provided the code for the bigram, LSTM, and transformer
  models, the training function, the evaluation function, and the
  generation function. Explained the code line by line.
- **Me:** Tested each part before training. I predicted the output
  shapes and parameter counts, and checked them against the real
  outputs. Ran all the training.

### 6. Experiment 2
- **Me:** Chose to run Experiment 2 and made my predictions before
  running it.
- **Claude:** Suggested changing batch_size to 8 so that each training
  step uses the same number of tokens.

### 7. Writing
- **Me:** Wrote the first drafts of my analysis, some in English and
  some in Chinese.
- **Claude:** Corrected grammar, suggested clearer wording, translated
  my Chinese drafts, and pointed out numbers that did not match my
  notebook.

## Errors I Debugged

| Error | Cause | How I fixed it |
|---|---|---|
| NameError: sherlock is not defined | File name had no quotes | Added quotes |
| FileNotFoundError | Missing the data/ folder in the path | Used os.listdir() to check, then fixed the path |
| Printed almost the whole book | Used text[:-1000] instead of text[-1000:] | Learned how negative slicing works |
| All tokens became <unk> | Used stoi on data that was already numbers | Decoded with itos directly |
| SyntaxError: unmatched ')' | Wrote find"( instead of find(" | Fixed the order |
| TypeError in __init__  | Typo in the __init__ name | Fixed the name and re-ran the class cell |
| IndexError: 0-dim tensor | Wrote train_data[8] instead of train_data[:8] | Added the colon |
| Wrong split size | Wrote len(data * 0.9) instead of 0.9 * len(data) | Fixed the order |
| Float tensor error in generate | Typo in the regular expression | Fixed the regex |
| Experiment 2 results were not fair | Copied cells but forgot to change lr and n_layer | Re-ran with the same settings as Experiment 1 |

## AI Mistakes I Noticed

- Claude predicted that the LSTM would overfit less than the bigram
  model. My results showed the opposite.
- Claude first said that "arthur" became <unk>. My output showed that
  it was "conan" and "doyle".
- Claude first said that the bigram model did not need to be trained
  again in Experiment 2, then corrected this to keep the experiment fair.

## How I Checked the Work

- I compared every number in my README with the final outputs in my
  notebook after running all cells again.
- I used torch.manual_seed so that I could repeat my results. When I
  ran the notebook again, the losses were exactly the same.