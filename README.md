# Claude API Experiments

A collection of Jupyter notebooks for experimenting with the Anthropic Claude API, focusing on prompt evaluation and testing various AWS-related tasks.

## Project Structure

- `001_prompt_evals.ipynb` - Automated prompt evaluation system with syntax validation
- `001-requests.ipynb` - Basic Claude API request examples
- `dataset.json` - Generated test dataset for AWS-related tasks

## Features

### Prompt Evaluation System

The main evaluation system (`001_prompt_evals.ipynb`) provides:

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

### Running Evaluations

Open `001_prompt_evals.ipynb` and run the cells to:

1. Generate a new test dataset
2. Run prompts against the dataset
3. Evaluate outputs with both model and syntax scoring
4. View aggregated results and scores

### Adding New Test Cases

You can modify the dataset generation prompt to create different types of evaluation tasks, or manually edit `dataset.json` to add specific test cases.

## Example Output

The system generates tasks like:
- Creating Python functions for AWS resource manipulation
- Writing JSON policies and configurations
- Developing regex patterns for AWS resource validation

Each task is evaluated on both correctness and code quality, providing comprehensive feedback on prompt performance.

## Contributing

Feel free to add new evaluation methods, expand the test dataset, or improve the scoring algorithms.