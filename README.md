# CAUS dataset ver1

### Notice  
- Developed using Python version 3.11.2.
- Please note that this code does not reflect the latest updates from the OpenAI and Langchain libraries. For the latest versions, please check each provider's websites.
- This work would be presented at CogSci2024 conference in July 2024.
- For more detailed information, see our paper on arXiv: [CAUS: A Dataset for Question Generation based on Human Cognition Leveraging Large Language Models](https://arxiv.org/abs/2404.11835)

## About the Dataset
- **Overall workflow**
![A workflow diagram illustrating the process](https://github.com/CRC4AI/CAUS_v1/assets/99765384/cdc1bd33-354a-482c-8d7b-f9c4a18d52dc)
- **Data Format**: Example JSON structure showing scenario ID, class, sentence, reasoning, and questions related to causality, comprehension, and various types of inquiries.
  ```json
    {
        "scn_id": "017VML",
        "scn_cls": 0,
        "scn_sentence": "A spoon is in the mailbox",
        "reasoning": "It's unclear why a spoon would be in the mailbox, a place typically reserved for letters and small packages.",
        "question": [
            {
                "qid01": "017VMLQ01",
                "query01": "Why is there a spoon in the mailbox?",
                "ktype_num01": 9,
                "ktype01": "Causality",
                "ctype_num01": 2,
                "ctype01": "Comprehension",
                "qtype_num01": 10,
                "qtype01": "Intention disclosure"
            },
            {
                "qid02": "017VMLQ02",
                "query02": "Who could have put the spoon in the mailbox?",
                "ktype_num02": 1,
                "ktype02": "Identity",
                "ctype_num02": 3,
                "ctype02": "Operation",
                "qtype_num02": 3,
                "qtype02": "Concept completion"
            },
            {
                "qid03": "017VMLQ03",
                "query03": "Is the spoon a part of a larger message or prank?",
                "ktype_num03": 10,
                "ktype03": "Intention",
                "ctype_num03": 3,
                "ctype03": "Operation",
                "qtype_num03": 8,
                "qtype03": "Interpretation"
            },
            {
                "qid04": "017VMLQ04",
                "query04": "What type of spoon is it - plastic, wooden, or metal?",
                "ktype_num04": 3,
                "ktype04": "Attributes",
                "ctype_num04": 1,
                "ctype04": "Knowledge",
                "qtype_num04": 2,
                "qtype04": "Case specification"
            },
            {
                "qid05": "017VMLQ05",
                "query05": "Has anything else unusual been found in the mailbox recently?",
                "ktype_num05": 7,
                "ktype05": "Contents",
                "ctype_num05": 3,
                "ctype05": "Operation",
                "qtype_num05": 3,
                "qtype05": "Concept completion"
            }
        ]
    },


___
## About Codes   
### CAUS_Scene.ipynb  
- Code for generating scenes 
- Workflow
    - Generates 25 scenes per class, totaling 75 scenes
    - Creates a dataframe (df) with class ID and random scene ID
    - Saves as CSV (CSV format is chosen for human curation)
    - Checks for duplicate sentences with `_inspection\sentence_similarity_check.ipynb`

### 5qCAUS_result.ipynb
- Generating 1 reasoning and 5 queries
- Workflow
  - Loads a CSV file containing human-curated scene lists  
  - Uses a GPT model to generate results (reasoning + 5 queries)
  - Saves intermediate files in JSON format

### 5qCAUS_KCQ_short.ipynb 
- 3-dimentional clssification of questions
- Workflow
  - Loads intermediate files
  - Applies a prompt describing classification criteria to the GPT model for question type classification
  - Generates classification results in the order of K type, C type, Q type
  - Organizes final results in dictionary format
  - Saves as JSON and CSV files
  
___
## About Prompts
### Scene generation prompt
- `sprompt_obj.txt`
    - Object uncertainty. Focuses on the incongruity between an object and its environment.
    - **Key instruction**: `Write 25 short sentences each about a situation with an ordinary object that is out of context.`
- `sprompt_int.txt`
    - Intention uncertainty. Deals with unclear intentions of characters.
    - **Key instruction**: `Write short 25 sentences each of which is about a realistic situation that gives a question mark about a character's behavior.`
- `sprompt_plt.txt`
    - Event uncertainty. Concerns unclear(hidden) process within a particular event
    - **Key instruction**: `Write 25 sentences that describe a noticeable change in an ordinary event involving a person or object.`

### Result generation prompt
- `rprompt.txt`
    - Identifies something unclear or uncertain in the scene.
    - `Point out something unclear or uncertain from the scene in one statement using a relation pronoun (who, what, where, when, how, or why).`
- `5qqprompt.txt`
    - Generates 5 questions arising from the scene.
    - `Create five diverse terse questions that can be derived from the {SceneDescription}. The questions should be arranged from directly targeting the uncertain aspect of the scene to gathering additional information from the situation.` 


### Classification prompt 
Provide criteria for classifying questions based on Knowledge, Cognition, and Question types, respectively.

- `ktype_prompt.txt` Focuses on the incongruity between an object and its environment.
```html
Ignore all previous instructions. 
Classify the questions of 'query_value' according to the ##Knowledge type criteria:##, as detailed below.
Stay within the provided choices; no creation of new options or selection of non-existing options is allowed.

##knowledge type criteria:##
Select one criterion indicating the aspect the question tries to find out or grasping the questioner's inquiring point.
They are presented as "criteria_number. knowledge_type: definition".

1. Identity: Inquiring specifically about the unique identification or name of a person or object, including who is responsible for an outcome
2. Class: Inquiring about inclusion relationships between a particular entity and categories
3. Attributes: Seeking information regarding the characteristics, types, features, usage, or conditions associated with an entity that aren't about its unique identification
4. Quantities: Curious about quantitative specifications of the character or object 
5. Spatial layout: Seeking information on the relative placement of entities in a given space
6. Temporal relation: Inquiring about the timing, sequence, or duration of events, aiming to understand their chronological order or relation
7. Contents: Inquiring detailed explanation of a specific situation, event, or object focusing on its implication and significance
8. Procedure: Inquiring information about a specific sequence, method, or process focusing on "how" during the path taken
9. Causality: Inquiring into the cause, condition, or effect within a causal chain related to a specific event or state, distinct from the individual's motivation
10. Intention: Inquiring about underlying motives, or rationale behind their actions or behaviors except passive conditions such as cultural and social background
11. Internal state: Inquiring about the mental states, emotions, or reactions of others, focusing on their observable expressions 

###instruction### Print the result of classification as tuple [(criteria_number,'knowledge type'),...]
Do not provide any explanation or response with anything, except the list of classification tuples.

```

- `ctype_prompt.txt` 
```html
Ignore all previous instructions. 

'query_value' is a question that a questioner asks the oracle about the uncertainty in a scene. 
Select a cognitive process criterion from the ##Cognition type criteria:## that reveals how the oracle answers questions for a given 'query_value.'
Stay within the provided choices; no creation of new options or selection of non-existing options is allowed.

##Cogniton type criteria:##
They presented as "criteria_number. cognition_type: definition."

1. Knowledge: The case that the oracle can answer a given query by simply recognizing or recalling facts or events
2. Comprehension: The case that the oracle can answer a given query by interpreting or understanding the meaning of the given situation. 
3. Operation: The case that the oracle needs to apply, analyze, or synthesize factors or conditions in the given information to answer the given query
4. Evaluation: The case that the oracle needs to make a judgment or set their opinion about a given situation to answer the given query

###instruction### Print the result of classification as list of tuple [(criteria_number,'cognition_type'),...] 
Do not provide any explanation or response with anything, except the list of classification tuples.
```

- `qtype_prompt.txt`  
```html
Ignore all previous instructions.
Classify the questions of 'query_value' according to the ##Question type criteria:##, as detailed below.
Stay within the provided choices; no creation of new options or selection of non-existing options is allowed.

##Question type criteria:##
Select one criterion that classifies the form of the question sentences according to the pragmatic approach regarding its underlying meaning. They are presented as "criteria_number. question_type: definition".

1. Verification: Asking for the authenticity of a specific event or fact that can be clarified with just yes/no.
2. Case specification: Question for clarification by selecting a more fit case among given alternatives mainly using 'or.'
3. Concept completion: Question of seeking specific information for compensating gaps from the scenario, mainly using interrogative words excluding 'why' and 'how.' 
4. Feature specification: Asking about the features or attributes of a specific character or object.
5. Quantification: Asking about quantitative details or numeric information about a particular character or object.
6. Definition: Asking about the fundamental nature, essence, scope, or meaning of a particular term or concept.
7. Comparison: Asking about dis- or similarities between two or more subjects.
8. Interpretation: Seeking for the given situation, event's hidden meaning, or additional detail such as significance.
9. Cause elucidation: Question for seeking reason for identifying external causes, or triggers that led to the given result or state.
10. Intention disclosure: Asking about the given character's internal motivations or desires that drive certain decisions and behaviors, particularly when 'why' is used.
11. Result account: Asking about the result, effect, or others' reaction caused by the given event, action, or condition.
12. Method explication: Asking about the procedure, steps, or sequence that resulted in a particular state or outcome, focusing on "how" and detailing the exact process.
13. Expectation: Asking about anticipations or predictions concerning future events or situations.
14. Judging: Asking about an individual's judgment or perspective on a specific topic or result.

###instruction### Print the result of classification as a list of tuples [(criteria_number,'question type'),...] 
Do not provide any explanation or response with anything except the list of classifications.
```
