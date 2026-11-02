**Data cleaning**
The raw data has extra text at the beginning and the end. 
That is not part of the stories. This text includes Project Gutenberg
information, license terms, and donation details. There are 844
characters at the beginning and 18,465 characters at the end.

This text should be removed for two reasons. First, the model could
learn to generate license or donation text. Second, the end text would
make up about one third of the validation set if I use the last 10% of
the data for validation. Then the validation loss would not show how
well the model learned the stories.

**Vocabulary**
I replaced tokens that appear fewer than 3 times with <unk>.
The vocabulary size is 3,142. About 4.68% of all tokens become <unk>.
I did not use a lower threshold because the bigram model would have
almost 20 million parameters. I did not use a higher threshold because
7.53% of tokens would become <unk>, which is almost one per sentence.

**Data Split**
The train data has 115,862 tokens (90%). The validation data has
12,874 tokens (10%). I used the first 90% of the book for training
and the last 10% for validation. I did not shuffle the tokens, because
the models need to learn from text in its original order. Because I
removed the license text at the end of the file, the validation data
only contains story text.

----------------------------------------------------
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

|Model|Training Loss|Validation Loss|Training Time|Text Quality|Notes|
|Bigram|3.70| 5.16|89.1s|Low|Overfits after step 2,000|

--------------------------------------------------------
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

---------------------------------------------------------


---------------------------------------------------------
**Results:** The bigram model did not change (5.16 to 5.15). This is
because it only uses the current token to predict the next token.

For the LSTM, the lowest validation loss was at step 1,500 in both
experiments. With block_size 64, it was only slightly lower (4.47
compared with 4.48). The final validation loss went down from 4.66 to
4.59.

For the transformer, overfitting started later. With block_size 16, the
lowest validation loss was 4.50 at step 1,500. With block_size 64, it
was 4.48 at step 2,000. The final validation loss went down from 4.81
to 4.61, and the gap between training and validation loss became
smaller (1.86 to 1.40).

**Compared with my prediction:** My prediction for the bigram model was
correct. My prediction for the transformer was partly correct. It
improved more than the LSTM in final validation loss (0.20 compared
with 0.07). But its lowest validation loss only improved by 0.02, and it
still did not beat the LSTM.

**What I learned:** Both models stopped at about 4.47 in both
experiments. I think the main limit is the dataset size, not the model
or the context length. The training data has only 115,862 tokens. A
longer context mainly helped the transformer overfit less, but it did
not help it learn much more.


