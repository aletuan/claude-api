# Claude API Experiments

Advanced prompt evaluation framework using the Anthropic Claude API for automated test case generation and evaluation.

## Project Structure

- `001_prompting.ipynb` - Advanced prompt evaluation framework with automated test case generation
- `dataset.json` - Generated test dataset for prompt evaluation (nutrition/meal planning tasks)

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

## Detailed Implementation Logic

### 1. Dataset Generation Logic (`generate_dataset`)

The dataset generation process follows a two-stage approach:

#### Stage 1: Idea Generation (`generate_unique_ideas`)
```python
# Generates diverse scenario ideas based on task description
ideas = self.generate_unique_ideas(task_description, prompt_inputs_spec, num_cases)
```

**Process:**
1. **Prompt Construction**: Creates a structured prompt asking Claude to generate `num_cases` unique testing ideas
2. **Template Rendering**: Uses the task description and input specifications to create context-aware scenarios
3. **JSON Extraction**: Parses Claude's response to extract idea list using stop sequences
4. **Diversity Enforcement**: System prompt emphasizes generating distinct, non-overlapping scenarios

**Example Output:**
```json
[
  "High-intensity athlete nutrition (Olympic weightlifter)",
  "Endurance athlete meal planning (marathon runner)",
  "Budget-conscious student athlete meals"
]
```

#### Stage 2: Test Case Generation (`generate_test_case`)
```python
# Converts each idea into a structured test case with inputs and criteria
test_case = self.generate_test_case(task_description, idea, prompt_inputs_spec)
```

**Process:**
1. **Input Validation**: Ensures only allowed input keys from `prompt_inputs_spec` are used
2. **Scenario Expansion**: Takes each idea and creates realistic input values
3. **Criteria Definition**: Generates 1-4 specific, measurable evaluation criteria
4. **Structure Enforcement**: Uses JSON schema validation to ensure consistent output format

**Example Test Case:**
```json
{
  "prompt_inputs": {
    "height": "185",
    "weight": "110",
    "goal": "muscle gain",
    "restriction": "vegetarian"
  },
  "solution_criteria": [
    "Includes protein-rich vegetarian options",
    "Provides caloric surplus for muscle gain",
    "Specifies meal timing around workouts"
  ],
  "task_description": "Write a compact, concise 1 day meal plan for a single athlete",
  "scenario": "High-intensity athlete nutrition (Olympic weightlifter)"
}
```

#### Concurrent Processing
- Uses `ThreadPoolExecutor` with configurable `max_concurrent_tasks`
- Processes multiple test cases simultaneously for faster generation
- Reports progress at 20% milestones
- Handles exceptions gracefully without stopping the entire process

### 2. Evaluation Logic (`run_evaluation`)

The evaluation system processes each test case through your prompt function and grades the results:

#### Execution Phase (`run_test_case`)
```python
# Runs your prompt function and grades the output
result = self.run_test_case(test_case, run_prompt_function, extra_criteria)
```

**Process:**
1. **Prompt Execution**: Calls your `run_prompt_function` with test case inputs
2. **Output Capture**: Stores the raw model response for evaluation
3. **Grading**: Passes output through structured evaluation process

#### Grading Phase (`grade_output`)
```python
# Uses Claude to evaluate output quality against criteria
model_grade = self.grade_output(test_case, output, extra_criteria)
```

**Grading Process:**
1. **Context Assembly**: Combines original task, inputs, output, and criteria into evaluation prompt
2. **Scoring Guidelines**: Uses 1-10 scale with specific thresholds:
   - 1-3: Fails mandatory requirements
   - 4-6: Meets mandatory but has significant deficiencies
   - 7-8: Meets most criteria with minor issues
   - 9-10: Meets all criteria comprehensively
3. **Structured Feedback**: Returns JSON with strengths, weaknesses, reasoning, and numeric score
4. **Extra Criteria Enforcement**: Optional mandatory requirements that trigger automatic failure if violated

**Example Evaluation:**
```json
{
  "strengths": ["Includes all required macronutrients", "Provides specific portions"],
  "weaknesses": ["Missing exact caloric calculations", "Lacks meal timing details"],
  "reasoning": "Good nutritional content but missing precision in caloric planning",
  "score": 6
}
```

#### Progress Tracking
- Concurrent execution using thread pools
- Real-time progress reporting at 20% intervals
- Calculates and displays average score upon completion
- Handles errors without stopping evaluation of other test cases

### 3. HTML Report Construction (`generate_prompt_evaluation_report`)

The HTML report provides comprehensive visualization of evaluation results:

#### Report Structure
```html
<!DOCTYPE html>
<html>
<head>
  <!-- Responsive CSS styling for professional appearance -->
</head>
<body>
  <div class="header">
    <!-- Summary statistics and key metrics -->
  </div>
  <table>
    <!-- Detailed results for each test case -->
  </table>
</body>
</html>
```

#### Summary Statistics Generation
```python
total_tests = len(evaluation_results)
avg_score = mean([result["score"] for result in evaluation_results])
pass_rate = 100 * len([s for s in scores if s >= 7]) / total_tests
```

**Metrics Calculated:**
- **Total Test Cases**: Count of all evaluated scenarios
- **Average Score**: Mean of all numeric scores (1-10 scale)
- **Pass Rate**: Percentage of cases scoring ≥7 (considered passing)

#### Visual Elements
1. **Color-Coded Scoring**:
   - Green (8-10): High performance
   - Yellow (6-7): Medium performance
   - Red (1-5): Low performance

2. **Responsive Layout**:
   - Flexbox summary statistics
   - Mobile-friendly table design
   - Proper text wrapping for long outputs

3. **Code Formatting**:
   - Syntax highlighting for code outputs
   - Preserved whitespace and formatting
   - Monospace font for technical content

#### Data Processing
```python
# Converts test case data into HTML table rows
for result in evaluation_results:
    prompt_inputs_html = "<br>".join([
        f"<strong>{key}:</strong> {value}"
        for key, value in result["test_case"]["prompt_inputs"].items()
    ])
    criteria_string = "<br>• ".join(result["test_case"]["solution_criteria"])
    # ... additional formatting
```

**Table Columns:**
- **Scenario**: High-level description of the test case
- **Prompt Inputs**: Key-value pairs used as input
- **Solution Criteria**: Bullet-pointed evaluation criteria
- **Output**: Your prompt's actual response (formatted)
- **Score**: Color-coded numeric score
- **Reasoning**: Detailed explanation of the grade

#### File Output
- Saves as `output.html` with UTF-8 encoding
- Self-contained file with embedded CSS
- Can be opened directly in any web browser
- Includes all data needed for analysis and sharing

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

### Adding New Test Cases

You can modify the dataset generation prompt to create different types of evaluation tasks, or manually edit `dataset.json` to add specific test cases.

## Example Output

The framework generates diverse test scenarios like:
- High-intensity athlete nutrition (Olympic weightlifter)
- Endurance athlete meal planning (marathon runner)
- Budget-conscious student athlete meals

Each task is evaluated comprehensively with detailed scoring and feedback.

## Contributing

Feel free to add new evaluation methods, expand the test dataset, or improve the scoring algorithms.