# HALTT4LLM - Hallucination Trivia Test for Large Language Models
This project is an attempt to create a common metric to test LLM's for
progress in eliminating hallucinations; the most serious current problem
in widespread adoption of LLM's for real world purposes.

## Results (as of March 2023) 

| Model Name            | HQ Trivia | C    | IDK | Fake Questions | C   | NOTA Questions | C   | IDK |
|-----------------------|-----------|------|-----|----------------|-----|----------------|-----|-----|
| **GPT4All**           | **88.47%**| 1243 | 7   | 74.16%         | 310 | **70.32%**     | 109 | 0   |
| **GPT-3.5**           | 59.33%    | 705  | 262 | **81.81%**     | 342 | 51.93%         | 58  | 45  |
| **GPT-3**             | 55.67%    | 776  | 17  | 6.10%          | 26  | 32.25%         | 43  | 14  |
| **Llama-7B-4bit**     | 49.75%    | 701  | 0   | 2.15%          | 18  | 8.38%          | 26  | 0   |
| **Alpaca-7B-4bit**    | 44.32%    | 624  | 1   | 0.00%          | 0   | 0.00%          | 0   | 0   |
| **GPT-4**             |           |      |     |                |     |                |     |     |

* **C** *number of correct answers*
* **IDK** *number of 'I don't know' answers*
* **NOTA** *stands for None of the Above*
* **HQ Trivia** - 1409 questions
* **Fake Questions** - 418 questions
* **NOTA Questions** - 155 questions

## Scoring

The scoring is as follows:

*  **+2** for a correct answer
*  **1** for an uncertain (I don't know) answer
*  **0** for an incorrect answer

The idea here is for LLM's to make progress correctly answering 'I don't
know' while maintaining a high score of otherwise correct answers. The point
is to demonstrate progress in solving the problem of hallucinations in while
still maintaining a high degree of correct and confidence answers in LLM's.

## Strategy

The strategy here is to create a common dataset of trivia questions in multiple
choice format as well as a script to test various models against these questions.
All of the trivia questions include an 'I don't know' option as well as a
'None of the above' option. The trivia questions include a set of fake or
trick questions where 'I don't know' is the correct response as well as a
set of questions where 'None of the above' is the correct response. This
in addition to a large corpus of real trivia questions with objective and 
unambiguous correct real world answers.

The resulting scores across these three sets can serve as a baseline to
test various techniques/methods to mitigate hallucinations in LLMs.

## Trivia datasets

The questions consist of the following trivia sets:

* hq_trivia_questions.json - taken from https://www.kaggle.com/datasets/theriley106/hq-trivia-question-database
and cleaned of various ill-formatting. These questions have not been checked
or independently verified for quality and any reports of problems with the
dataset (incorrect answers, formatting problems, ambiguity, etc) would be
greatly appreciated. Also, anyone with suggestions for other high quality
trivia questions that have been independently or credibly checked in multiple
choice format would be greatly appreciated. Currently 1409 questions.

* fake_trivia_questions.json - these are generated by GPT3.5 with the prompt
in the repository and the script to generate them. Currently 418 questions.

* none_of_the_above_questions.json -  also generated by GPT3.5 with script
and prompt in repository. Currently 155 questions.

Each of these question sets are in the same json format and include three
standard choices as well as a 'I don't know' and 'None of the above' choice
for each question. Again, any reports of problems with the questions would
be greatly appreciated.

## Discussion

While GPT-3.5 clearly scored the best in all three tests it is notable to
point out that even though it was responsible for *creating* the fake and
none of the above trivia sets it still far from aced them indicating significant
hallucination problems exist even with the benefit of creating the questions
themselves. It would be interesting to see what would happen to the score
of GPT-3.5 if the questions were generated with a non-gpt derived models.
That said, it is clear that GPT-3.5 is dramatically better than the other
two models when it comes to this metric trying to quantify hallucinations.
GPT-3.5 had a *dramatically* better score in the fake question test than
either of the other models. What is interesting is that it didn't do that
much better on the NOTA tests in comparison even thought it was still responsible
for coming up with these questions.

Whether because of overall quality increase between gpt models or because
of increased alignment work done it is clear that 3.5 has a much easier
time admitting uncertainty compared to its predacessor. The accuracy actually
decreased though between GPT-3 and GPT-3.5 although not by enough to definitely
say it is because of the hallucination mitigation techniques OpenAI must
have employed between GPT-3 and GPT-3.5.

When it comes to uncertainty and handling hallucinations it is clear that
both gpt models are far and away superior to Llama 7B and Alpaca Lora which
does not admit uncertainty under any circumstance. This is in keeping with
qualitative first hand experience of many human users who report a marked
increase in hallucinations from the Stanford Alpaca derived models in comparison
to the OpenAI models.

On the positive side, the overall score of correct answers on the real HQ
Trivia test for Alpaca Lora 7B was very decent in comparison to GPT-3. Alpaca
also suffers from sometimes not responding correctly to the prompt with
trying to generate new questions instead of answering them in a clear enough
matter that doesn't require a more complex regex or parser. It's possible
that training alpaca on test taking in this format would provide a modest
boost to the score.

The amazing standout is the new GP4All model which was trained on ~800k new
prompts generated from GPT-3.5 output. In fact, this model is outscoring
GPT-3.5 itself on the real trivia set by a wide margin and still managing
to finish second in the fake hallucination test with a very respectable
74.16% correct!

In the future it will be interesting to see how GPT-4 fares in comparison
to GPT-3.5 with this test. Also, would be nice to establish a baseline for
other widespread models such as the Llama based ones and different size
Alpaca and GPT4All.

## Contributing

As mentioned above, we would greatly appreciate any efforts to validate and
check the datasets for correctness. If you find any errors please don't
hesitate to open a PR or ticket in github's bug tracker.

## Setup and install

Install python dependencies.

```
pip install -r requirements.txt
```

Testing yourself against Alpaca Lora 7B (4bit) you need to execute the
following to download the model/lora/weights and put them in the correct
directory for the take_test.py to find the model correctly.

```
python download-model.py --text-only decapoda-research/llama-7b-hf
wget https://huggingface.co/decapoda-research/llama-7b-hf-int4/resolve/main/llama-7b-4bit.pt -P ./weights
python download-model.py samwit/alpaca7B-lora
python download-model.py nomic-ai/gpt4all-lora
```

## Examples for running the tests

Running the HQ Trivia test on OpenAI GPT-3.5

```
python take_test.py --use-gpt3-5 --openai-key <YOUR_OPEN_API_KEY> --trivia hq_trivia_questions.json
```

Running the Fake Trivia test on OpenAI GPT-3

```
python take_test.py --use-gpt3 --openai-key <YOUR_OPEN_API_KEY> --trivia fake_trivia_questions.json
```

Running the NOTA (None of the Above) Trivia test on Alpaca Lora 7B (4bit)

```
python take_test.py --trivia fake_trivia_questions.json
```

These will all produce test result files at the end named according to the test and the model.

## Testing other models

The repository and script are currently setup only to test OpenAI's GPT3
and GPT3.5 as well as Alpaca Lora 7B (4bit). To add a new model it would
only take a bit of work to edit the 'take_test.py' for instance adding
other alpaca lora models or GPT4All and any additions would be greatly
appreciated.

## Process to generate fake and nota (none of the above) questions

The fake and nota questions were generated by a script `generate_trivia.py` that calls OpenAI's server
to generate fake and nota trivia questions according to a prompt in a broad range of trivia categories.
The script `filter_questions.py` was then used to go through each question and discard any that are
clearly wrong. Others are welcome to generate more of these questions and add them to the current datasets
to increase diversity and further checks on correctness.
