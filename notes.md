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