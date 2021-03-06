
#### **ALEART** This repo could't have glov6B.300d.txt.
## General idea
The idea is to generate multiple choice answers from text, by splitting this complex problem to simpler steps:

 - **Identify keywords** from the text and use them as answers to the questions.
 - **Replace the answer** from the sentence with *blank space* and use it as the base for the question.
 - **Transform the sentence** with a blank space for answer to a more *question-like sentence*.
 - **Generate distractors**, words that are similar to the answer, as *incorrect answers*.

![Question generation step by step gif](https://media.giphy.com/media/1n4JPydITD3mGvTZBZ/giphy.gif)

## Execution

### Data Exploration
Before I could to anything, I wanted to understand more about how questions are made and what kind of words are it's answers.

I used the [SQuAD 1.0](https://rajpurkar.github.io/SQuAD-explorer/) dataset which has about 100 000 questions generated from Wikipedia articles.

You can read about the insights I've found in the *Data Exploration* jupyter notebook.

### Identifying answers
My assumption was that **words from the text would be great answers for questions**. All I needed to do was to decide which words, or short phrases, are good enough to become answers.

I decided to do a binary classification on each word from the text. [spaCy](https://spacy.io/) really helped me with the word tagging.

#### Feature engineering
I pretty much needed to create the entire dataset for the binary classification. 
I extracted each non-stop word from the paragraphs of each question in the SQuAD dataset and added some features on it like:

 - **Part of speech**
 - Is it a **Named entity**
 - Are only **alpha characters** used
 - **Shape** - whether it's only alpha characters, digits, has punctuation (xxxx, dddd, Xxx X. Xxxx)
 - **Word count**

And the label **isAnswer** - whether the word extracted from the paragraph is the same and in the same place as the answer of the SQuAD question. 

Some other features like **TF-IDF** score and **cosine similarity** *to the title* would be the great, but I didn't have the time to add them.

Other than those, it's up to our imagination to create new features - maybe whether it's in the start, middle or end of a sentence,  information about the words surrounding it and more... Though before adding more feature it would be nice to have a metric to assess whether the feature is going to be useful or not.

#### Model training
I found the problem similar to *spam filtering*, where a common approach is to tag each word of an email as coming from a spam or not a spam email.

I used scikit-learn's **Gaussian Naive Bayes** algorithm to classify each word whether it's an answer.

The results were surprisingly good - at a quick glance, the algorithm classified most of the words as answers. The ones it didn't were in fact unfit.

The cool thing about *Naive Bayes* is that you get the **probability** for each word. In the demo I've used that to order the words from the most likely answer to the least likely.

### Creating questions
Another assumption I had was that **the sentence of an answer could easily be turned to a question**. Just by placing a *blank space* in the position of the answer in the text I get a **"cloze" question** *(sentence with a blank space for the missing word)*

**Answer:** 
Oxygen

**Question:**
 \_____ is a chemical element with symbol O and atomic number 8.

I decided it wasn't worth it to transform the cloze question to a more question-looking sentence, but I imagine it could be done with a **seq2seq neural network**, similarly to the way text is translated from one language to another.

### Generating incorrect answers
The part turned out really well. 

For each answer I generate it's most similar words using **word embeddings** and **cosine similarity**.

![Most similar words to oxygen](https://i.gyazo.com/175b9f86b3defc0798800cb06169cc3f.png)

Most of the words are just fine and could easily be mistaken for the correct answer. But there are some which are obviously not appropriate.

Since I didn't have a dataset with incorrect answers I fell back on a more classical approach.

I removed the words that **weren't the same part of speech** or **the same named entity** as the answer, and added some more context from the question.

I would like to find a dataset with multiple choice answers and see if I can create a *ML model* for generating better incorrect answers.

## Results
After adding a Demo project, the generated questions aren't really fit to go into a classroom instantly, but they are't bad either. 

The cool thing is the **simplicity** and **modularity** of the approach, where you could find where it's doing bad (*say it's classifying verbs*) and plug a fix into it. 

