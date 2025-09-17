# Claude API Experiments

A collection of Jupyter notebooks for experimenting with the Anthropic Claude API, focusing on prompt evaluation and testing various tasks.

## Project Structure

- `001_prompt_evals.ipynb` - Automated prompt evaluation system with syntax validation (AWS-focused)
- `001_prompting.ipynb` - Advanced prompt evaluation framework with automated test case generation
- `001_requests.ipynb` - Basic Claude API request examples
- `dataset.json` - Generated test dataset (currently nutrition/meal planning tasks)

## Features

### Advanced Prompt Evaluation Framework (`001_prompting.ipynb`)

A comprehensive system for automatically generating, running, and evaluating prompts:

```
┌─────────────────────────────────────────────────────────────────────┐
│                     001_prompting.ipynb Workflow                    │
└─────────────────────────────────────────────────────────────────────┘

┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│   TASK SETUP     │    │  DATASET GEN     │    │  EVALUATION      │
│                  │    │                  │    │                  │
│ • task_desc      │───▶│ • Generate ideas │───▶│ • Run prompts    │
│ • input_spec     │    │ • Create cases   │    │ • Grade outputs  │
│ • num_cases      │    │ • Save JSON      │    │ • Score results  │
└──────────────────┘    └──────────────────┘    └──────────────────┘
         │                        │                        │
         ▼                        ▼                        ▼
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│ Define inputs:   │    │ Auto-generated   │    │ HTML Report +    │
│ • height: "185"  │    │ test scenarios   │    │ JSON results     │
│ • weight: "110"  │    │ with criteria    │    │ • Scores 1-10    │
│ • goal: "muscle" │    │ for evaluation   │    │ • Pass/fail rate │
│ • restriction    │    │                  │    │ • Detailed feedback
└──────────────────┘    └──────────────────┘    └──────────────────┘

Flow Details:
1. SETUP: Define task & input structure
2. IDEAS: AI generates diverse test scenarios
3. CASES: Convert ideas → structured test cases with criteria
4. RUN: Execute your prompt function on each case
5. GRADE: AI evaluates outputs against criteria
6. REPORT: Generate comprehensive evaluation report
```

**Key Components:**
- **Automated Test Generation**: Creates diverse scenarios based on task description
- **Concurrent Processing**: Parallel test case generation and evaluation
- **Structured Evaluation**: Solution criteria matching for accurate scoring
- **Rich Reporting**: HTML reports with visual scoring and detailed feedback

### Basic Prompt Evaluation System (`001_prompt_evals.ipynb`)

The original evaluation system for code generation tasks:

- **Automated Dataset Generation**: Creates test cases for Python, JSON, and regex tasks
- **Dual Scoring System**: Combines model-based evaluation with syntax validation
- **Format Validation**: Checks if outputs are valid Python, JSON, or regex patterns
- **AWS-Focused Tasks**: Generates tasks related to AWS services and resources

### Evaluation Components

1. **Model Grading**: Uses Claude to evaluate solution quality with structured feedback
2. **Syntax Validation**: Programmatically validates code syntax and JSON structure
3. **Combined Scoring**: Averages model scores with syntax validation scores

## Setup

1. Clone the repository
2. Install dependencies:
   ```bash
   pip install anthropic python-dotenv jupyter
   ```
3. Create a `.env` file with your Anthropic API key:
   ```
   ANTHROPIC_API_KEY=your_api_key_here
   ```
4. Start Jupyter:
   ```bash
   jupyter notebook
   ```

## Usage

### Using the Advanced Framework (`001_prompting.ipynb`)

1. **Define your task and inputs:**
   ```python
   evaluator = PromptEvaluator()
   dataset = evaluator.generate_dataset(
       task_description="Write a meal plan for an athlete",
       prompt_inputs_spec={
           "height": "Athlete's height in cm",
           "weight": "Athlete's weight in kg",
           "goal": "Training goal",
           "restriction": "Dietary restrictions"
       },
       num_cases=3
   )
   ```

2. **Create your prompt function:**
   ```python
   def run_prompt(prompt_inputs):
       prompt = f"Create meal plan for: {prompt_inputs}"
       return chat_with_claude(prompt)
   ```

3. **Run evaluation:**
   ```python
   results = evaluator.run_evaluation(
       run_prompt_function=run_prompt,
       dataset_file="dataset.json"
   )
   ```

4. **View results in HTML report** with scores, feedback, and detailed analysis

### Using Basic Evaluation (`001_prompt_evals.ipynb`)

Open `001_prompt_evals.ipynb` and run the cells to:

1. Generate a new test dataset
2. Run prompts against the dataset
3. Evaluate outputs with both model and syntax scoring
4. View aggregated results and scores

### Adding New Test Cases

You can modify the dataset generation prompt to create different types of evaluation tasks, or manually edit `dataset.json` to add specific test cases.

## Example Output

**Advanced Framework** generates tasks like:
- High-intensity athlete nutrition (Olympic weightlifter)
- Endurance athlete meal planning (marathon runner)
- Budget-conscious student athlete meals

**Basic System** generates tasks like:
- Creating Python functions for AWS resource manipulation
- Writing JSON policies and configurations
- Developing regex patterns for AWS resource validation

Each task is evaluated comprehensively with detailed scoring and feedback.

## Contributing

Feel free to add new evaluation methods, expand the test dataset, or improve the scoring algorithms.