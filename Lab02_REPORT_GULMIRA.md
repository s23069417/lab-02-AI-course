**Lab 02 - Open the box**

**Student: Kabylkhanova Gulmira**

**REPORT**

**Introduction.** The goal of this lab was to verify with my own hands
three statements from the lecture about what happens inside a language
model that the model output is not a word, but a list of 50,257
probabilities, that temperature changes the shape of this distribution,
but cannot change which token is in the first place and that attention
is a table of weights, where each row sums to one. I tested all of this
on a real GPT‑2 small model (124 million parameters) in Google Colab.

**Part 0.** The word "bank" is one token, but "банк" was split into 5
strange bytes (Ð, ±). The reason is that merges are learned based on the
frequency of pairs in the text, not on the language, and there was
almost no Cyrillic in the GPT‑2 training data. For "бөлімшеңізде" I
predicted 16 tokens but got 19 - GPT‑2 performs worse on Kazakh than I
thought.

**Part 1.** For the prompt "The capital of Kazakhstan is Astana. The
capital of France is,". My prediction was wrong. I thought \" Paris\"
will be first, but the real top token is \" Ast\" (22.87%). \" Paris\"
is very close, only 22.08%. This is strange because \" Ast\" is not a
full word, it looks like the model is just repeating \"Astana\" from
earlier in the sentence, not really thinking about geography. Also, I
said top 10 will be more than 90%, but real number is only 64.0%. This
means the model is not very sure about the answer, even when the
sentence pattern is clear. So, my prediction about the top token and
about the probability were both incorrect.

Then, I checked three more of my prompts: "Eiffel Tower..." -\> "Paris"
(6.38%, guessed correctly, but uncertainly); "fridge..." -\> "large"
(3.33%, many options); "opposite of hot..." -\> "cold" (16.27%, highest
confidence - antonyms are a rigid pattern).

**Temperature.** Predicted that the top 1 would not change at different
T, only the probability would drop this was fully confirmed. The top 1
("Ast") remained the same at T=0.25/1/2/5, the probability dropped from
53.45% to 0.04%, and the entropy increased almost to the maximum. A
separate conclusion: T=0 provides stability, not correctness the top 1
token remains incorrect.

**Top-p** turned out to be a fundamentally different operation.
Temperature only "stretches" the distribution even at T=5, 35,021 tokens
still have at least some non‑zero probability. But top-p physically
removes low‑probability tokens at p=0.9 and T=1.0, only 341 tokens
remain out of all 50,257, and all the others get exactly zero. This is
the key difference between "reshape" and "delete" that the lecture
talked about.

**Part 2**

I found two key (layer, head) pairs:

-   Previous-token head: layer 4, head 11, score 1.000 an almost perfect
    head that almost always looks at the previous token. On the heatmap,
    this looks like a diagonal line shifted one position to the left of
    the main diagonal.

-   Token-0 head (attention sink): layer 7, head 10, score 0.964- 96.4%
    of this head's attention is directed at the very first token
    ("The"), regardless of which token is "being looked at".

There was an important trap that the lab warned about: high attention to
the first token doesnt mean that the model "is thinking about the word
'The'" its most likely just a mechanical side effect of the fact that
softmax has to allocate an attention unit somewhere, and if the head
"has nothing to look at," the weight goes to the first token. High
attention weight is not proof of importance its just a picture of the
mechanism.

**Part 3.** I predicted that Kazakh would be noticeably worse than
Russian. Reality: Kazakh 4.25× English, Russian 1.88× English, the
difference is smaller than I expected. The most unexpected thing was the
text generation: for Russian and Kazakh prompts, the top‑1 token often
turned out to be a broken symbol \'?\' (37.79% probability), and the
text would go into repetition or nonsense, while English remained
coherent. The problem isnt with the language, but with the tokenizer,
which barely encountered Cyrillic during training.

**The main conclusion** is that the model doesnt think and doesnt know
facts it predicts the statistically most likely continuation of the
text. Sometimes this coincides with the truth (Eiffel Tower), sometimes
it doesnt (Astana/Paris). Language inequality is not a property of the
language itself, but a consequence of the data on which the tokenizer
was trained.

**Declaration on the Use of AI**

All predictions were formulated and entered by me personally, before
running the corresponding cells. I ran all the code myself in Google
Colab, one cell at a time, following the lab rules.

I used the Claude AI assistant to:

Explain theoretical concepts from the lecture in simple words,

Assistance in interpreting the results after I received them for
example, explanations of why the top‑1 token turned out to be "Ast"
rather than "Paris," as I had expected,

Technical assistance with the GitHub/Colab interface.
