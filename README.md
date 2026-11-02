## 1. Task

This project solves a language modeling task: given the previous text,
predict the next token. I trained three models on the same task and the
same data: a bigram model, an LSTM, and a small transformer. The goal is
not to build the best model. The goal is to understand how each model
uses the previous text, and why attention helps language modeling. I
used word-level tokens.

## 2. Dataset

This section explains the dataset and how I prepared it.

**Source:** I used "The Adventures of Sherlock Holmes" by Arthur Conan
Doyle from Project Gutenberg (eBook #1661). To run this project,
download the "Plain Text UTF-8" file from
https://www.gutenberg.org/ebooks/1661 and save it as
`data/sherlock.txt`.

**Cleaning:** The file has extra text at the beginning and the end that
is not part of the stories. This includes information about Project
Gutenberg, license terms, and donation details. I removed this text by
keeping only the text between the "*** START OF" and "*** END OF"
markers. If I kept the end text, it would make up about one third of
the validation set. I also replaced the underscore "_" with a space,
because Project Gutenberg uses underscores to mark italic words, such
as "_the_".

**Tokenization:** I changed all text to lowercase. Then I split the
text into words and punctuation marks. Punctuation marks are kept as
separate tokens. For example, "Holmes," becomes "holmes" and ",". This
helps the models learn where sentences end. The text has 128,736 tokens.

**Vocabulary:** I kept tokens that appear at least 3 times. Tokens that
appear fewer than 3 times (4,762 different tokens) were replaced with
<unk>. About 4.68% of all tokens became <unk>. The final vocabulary has
3,142 tokens.

**Split:** I used the first 90% of the tokens for training (115,862
tokens) and the last 10% for validation (12,874 tokens).

## 3. Three Models

**Bigram**
The bigram model only uses the current
token to predict the next token. It learns how often each token comes
after the current token.

**LSTM**
The LSTM reads the tokens one at a time, from left to right. After each
token, it updates its memory. When it predicts the next token, it uses
this memory, which contains information from all the previous tokens in
the context (up to 16 tokens). The bigram model only uses the current 
token.

**Transformer**
The transformer looks at all the previous tokens at the same time. It
does not store the context in one memory like the LSTM. Instead, it uses
self-attention. For each token, attention gives a weight to every
previous token. A higher weight means that token is more important for
the prediction. 

## 4. How does each model work?

**Bigram**
**What information it uses:** The bigram model only uses the current
token to predict the next token. It learns how often each token comes
after the current token.

**What information it ignores:** It ignores all tokens that come before
the current token.

**Why it has limited language understanding:** For example, in the text
“i think,” said ___, the next word should be the person who is speaking.
The bigram model only sees "said". It cannot see the quote or anything
earlier in the story. So it can only guess the word that most often
comes after "said", no matter who is speaking.

**Generated text:** When I look at pairs of words, the text is readable,
such as "mother had", "could be", and "if he". However, the full
sentences are not readable. For instance, "holmes lately throw tobacco
family" is meaningless. This is because the bigram model only uses the
current token to predict the next token. Each pair makes sense, but the
model cannot connect the pairs into a full idea.
-----------------------------------------------------------------------
**LSTM**
**How it uses more context:** The LSTM reads the tokens one at a time,
from left to right. After each token, it updates its memory. When it
predicts the next token, it uses this memory, which contains information
from all the previous tokens in the context (up to 16 tokens). The
bigram model only uses the current token.

**What improvement it provides:** The validation loss went down from
5.16 (bigram) to 4.66, and the lowest was 4.48. The LSTM also generates
longer phrases that make sense. It does this with about 0.94 million
parameters, which is about one tenth of the bigram model.

**Why it may still struggle with longer dependencies:** First, the LSTM
has to store all the previous tokens in one memory vector of 128 numbers.
Information from early tokens can be pushed out by later tokens. Second,
during training, the gradient from early tokens becomes very small
(vanishing gradient), so the model learns long-distance patterns slowly.
The LSTM gates help with this, but they do not fully solve it. For
example, the model wrote "i has" because it did not keep "i" in memory.

**Generated text:** Compared with the bigram model, the LSTM generates
longer phrases that make sense, such as "following his way in silence"
and "against the same position". The bigram model can only generate two
words in a row that read smoothly. The LSTM still makes mistakes. For
example, it writes "i has" instead of "i have". The words "i" and "has"
are several tokens apart, so the LSTM did not keep the subject in its
memory. The full text, such as "tottenham court days from the
mantelpiece", still has no clear meaning.

**Overfitting:** The LSTM reached its lowest validation loss of 4.48 at
step 1500. After that, the validation loss went up to 4.66 while the
training loss kept going down. This shows overfitting.

-----------------------------------------------------------------------
**Transformer**
**How it uses context:** The transformer looks at all previous tokens
at the same time. Attention gives a weight to every previous token, and
a higher weight means the token is more important for the prediction.
Unlike the LSTM, it does not compress the context into one memory
vector. It can look directly at any earlier token, even one that is far
away.

**What improvement it provides:** In Experiment 1, the transformer did
not improve on the LSTM. Its lowest validation loss was 4.50, and the
LSTM's was 4.48. Its final validation loss was 4.81, which is higher
than the LSTM's 4.66. It also has more parameters (1.21 million) and
took longer to train (55.0 seconds compared with 35.0 seconds). Its
training loss was the lowest (2.95), but this means it memorized the
training data more, not that it learned better.

**Generated text:** The transformer closed quotation marks correctly in
"‘ i say , ’". To close a quote, the model has to remember that a quote
was opened earlier, and attention can look back at the opening mark
directly. However, it still made grammar mistakes, such as "did you hot
in", and it generated <unk> 4 times in 50 tokens. Overall, the text
quality was similar to the LSTM.

**Overfitting:** The transformer reached its lowest validation loss of
4.50 at step 1500. After that, the validation loss went up to 4.81
while the training loss kept going down. The final gap between training
and validation loss was 1.86, which is larger than the LSTM's gap of
1.53. The transformer overfit the most.

## 5. Which model performed best?
**Experiment 1 (block_size = 16)**

| Model | Train Loss | Val Loss | Best Val Loss | Training Time | Text Quality |
|---|---|---|---|---|---|
| Bigram | 3.79 | 5.16 | 5.16 (step 2999) | 74.8 s | Low |
| LSTM | 3.13 | 4.66 | 4.48 (step 1500) | 35.0 s | Medium |
| Transformer | 2.95 | 4.81 | 4.50 (step 1500) | 55.0 s | Medium |

**Experiment 2 (block_size = 64)**

| Model | Train Loss | Val Loss | Best Val Loss | Training Time | Text Quality |
|---|---|---|---|---|---|
| Bigram | 3.79 | 5.15 | 5.13 (step 2500) | 75.5 s | Low |
| LSTM | 3.07 | 4.59 | 4.47 (step 1500) | 45.0 s | Median |
| Transformer | 3.21 | 4.61 | 4.48 (step 2000) | 67.2 s | Median | 

**Which model performed best?**
The LSTM had the lowest validation loss in both experiments: 4.48 in
Experiment 1 and 4.47 in Experiment 2. However, the transformer was
very close, with 4.50 and 4.48. The difference is only 0.01 to 0.02,
so the LSTM and the transformer performed almost the same. Both were
much better than the bigram model (5.16 and 5.13).

**Which model trained fastest?**
In my final run, the LSTM was the fastest in both experiments (35.0 s
and 45.0 s), and the bigram model was the slowest (74.8 s and 75.5 s).
However, training time changed a lot between runs. In earlier runs, the
bigram model was sometimes the fastest. The bigram model has the
simplest computation, but it has the most parameters to update (9.9
million), so its speed changed the most. The only result that stayed
the same in every run is that the transformer was slower than the LSTM.
Attention compares every token with every previous token, so it needs
more computation.

**Which model generated the most realistic text?**
In my samples, the LSTM and the transformer generated text of similar
quality. Both wrote phrases that read smoothly, but the full text still
had no clear meaning. The transformer closed a quotation mark correctly
("‘ i say , ’"), and the LSTM wrote a full question in Experiment 2
("did you see it ?"). Both also made mistakes, such as messy quotation
marks (". ’ “ ‘"). The bigram model had the least realistic text,
because only pairs of words made sense.

## 6. What Did Attention Improve?

**What attention improved**
When the models could look at a longer previous context, the
transformer benefited the most. Its final validation loss went down
from 4.81 to 4.61, and its lowest validation loss went down from 4.50
to 4.48. The gap between training and validation loss became smaller,
from 1.86 to 1.40. Overfitting also started later. It started at step
1500 in Experiment 1, but it moved to step 2000 in Experiment 2. This
shows that attention can use a longer context, because it can look
directly at any earlier token.
The LSTM only improved a little in Experiment 2. Its final validation
loss went down from 4.66 to 4.59, and its lowest validation loss only
changed from 4.48 to 4.47. With a longer context, it might be hard for
the LSTM to focus on the important tokens, because it has to store all
of them in one memory. For example, in Experiment 2 the LSTM opened a
quotation mark in "i bowed , “ did you see it ?" but never closed it.
The other tokens in the longer context might have made it forget the
opening quotation mark.

**What attention did not improve**
Attention did not help the transformer beat the LSTM. In both
experiments, their lowest validation losses were almost the same (4.50
and 4.48 in Experiment 1, 4.48 and 4.47 in Experiment 2). Both models
stopped at about 4.47.

**Was it worth the added complexity?**
The transformer had more parameters (1.21 million compared with 0.94
million) and a longer training time (55.0 s compared with 35.0 s in
Experiment 1). However, its validation loss was almost the same as the
LSTM. In my opinion, it was not worth it for a dataset of this size.
I think the transformer can do better with a larger dataset, because
it can learn the connections between tokens well. My dataset has only
115,862 training tokens, which may not be enough for attention to show
its advantage.

## 7. What I Learned

I learned a lot from this project. Our weekly assignments are usually
fill-in-the-blank coding tasks. I did learn more about the code from
them, but what I learned was limited, because I spent most of the time
changing parameters in code that was already written. In this project,
I started from zero. This helped me understand how to write a model and
how to train it in practice.

I also put the ideas from class into practice. For example, during data
cleaning, I had to remove the extra text at the beginning and the end of
the book. If I did not remove it, the models would learn things they
should not learn, and this text could show up in the generated text.
I also saw how each model works. The bigram model only uses the current
token to predict the next token. The LSTM reads the tokens in a limited
range of context, one at a time, and keeps a memory. The transformer
looks at all previous tokens in the context and uses attention to give
each token a weight before it predicts the next token.

The results were different from what I expected. I expected the
transformer to have the best results and to generate the best text.
However, the LSTM was the best model in this project. It beat the
transformer by only 0.02 and 0.01 in validation loss, and it trained
about 20 seconds in experience 1 and 22.2 seconds in experience 
faster. In Experiment 1, overfitting started at the same step (1500) 
for both models. In Experiment 2, the transformer successfully moved 
its overfitting point later, to step 2000. I used a small dataset, 
so the transformer could not show its strength, and it performed 
about the same as the LSTM. This reminded me that choosing a model 
depends on the available resources and the goal of the training.
I should not always use the same model for every task.

## 8. Responsible AI Use

I do not have a strong programming background, so I used Claude to help
me. I asked Claude to guide me step by step through the whole project,
from the details of data cleaning and how to do it correctly, to
building the models and choosing the parameters. I did not ask Claude
to write the whole project at once. Claude explained each idea first,
gave me hints, and asked me questions, and then I wrote and ran the
code myself.

**What I did myself:**
- I chose the dataset and decided to use word-level tokens.
- I wrote the data exploration and cleaning code with hints from Claude.
  I found the extra Project Gutenberg text and decided how to remove it.
- I compared different frequency thresholds and chose to keep tokens
  that appear at least 3 times.
- I chose to run Experiment 2 and made my own predictions before
  running it.
- I ran all the training and checked every output.
- I wrote the first drafts of my analysis and reflection.

**What Claude provided:**
- Explanations of each concept, such as batches, attention, masks,
  positional embeddings, and overfitting.
- The code for the model classes, the training loop, and the text
  generation function. Claude explained this code line by line, and I
  tested each part before using it.
- Help with debugging. When I got errors, Claude explained what the
  error message meant, and I fixed the code myself.
- Grammar corrections and suggestions for my writing. Some parts of
  the README were translated or reorganized from my own notes.

**How I used it responsibly:** I checked every number in this README
against the outputs in my notebook. I did not trust the AI without
checking. For example, Claude predicted that the LSTM would overfit
less than the bigram model, but my results showed the opposite. In
Experiment 2, Claude also helped me notice that I had changed more than
one setting by mistake, so I ran the models again with the correct
settings. More details are in `ai_use_summary.md`.