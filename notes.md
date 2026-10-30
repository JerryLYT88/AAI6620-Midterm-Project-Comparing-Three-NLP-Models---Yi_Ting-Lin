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

