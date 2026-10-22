# Financial Audit AI Tool

A desktop app that audits travel and expense reports for Cal Poly's financial audit office. It reads Concur Excel exports, sends each employee's expenses to Claude 3 Sonnet on Amazon Bedrock along with the Cal Poly and CSU travel policy, and writes out flagged reports and charts.

Our team of five built this during the Cal Poly DxHub AI Summer Camp (sponsored by AWS) in August 2025. This is a fork of the [team repository](https://github.com/flwrsfralgernn/Financial-Audit-AI-Tool).

**Note:** The tool was built around a confidential dataset that is not included here, so the code will not run as is. In demo mode it also audits only 5 randomly chosen employee reports. You can change this in `services/auditor.py`.

## How it works

1. Load one or more Concur exports and combine them into a master report.
2. Clean the data and group expenses by employee and report.
3. Build a prompt with each group's expenses and the policy text from `config/policies/policy_rules.txt`.
4. Send the groups to Claude 3 Sonnet in parallel. The model returns the rows that violate policy or need an exception.
5. Merge those flags back into the data and write the reports and charts.

## Output

All files are saved in `audit_reports/` and `summary_charts/`.

- `Audited_Expenses_<timestamp>.xlsx`: every row with an Audit Flag column (red for violations, yellow for exceptions)
- `Violations_Report_<timestamp>.xlsx` and `Exceptions_Report_<timestamp>.xlsx`: flagged rows only
- A text summary for each employee report
- Bar and pie charts of the results

## Design notes

**RAG vs. putting the policy in the prompt.** We tried two ways of giving the model the travel policy. The first stored the Cal Poly and CSU policies in an Amazon Bedrock Knowledge Base and used `retrieve_and_generate` to pull the relevant sections for each trip. The second puts the full policy text directly in the prompt. We went with the second one because it was simpler, the policy fits in the prompt, and there is no knowledge base to set up. The Knowledge Base version is still in the commit history.

**Parallel processing.** Excel files load in parallel with a `ProcessPoolExecutor`, and the Bedrock calls run in parallel with a `ThreadPoolExecutor`.

## My part

I built the data pipeline that combines and formats the Concur exports (`combine_and_format.py`), the Knowledge Base (RAG) prototype and trip-based evaluation, and a large update to `main.py` that added features and sped up execution.

## Running it

Requirements: Python 3.10+ and an AWS account with Bedrock access to Claude 3 Sonnet.

```bash
git clone https://github.com/ashtonaliu/Financial-Audit-AI-Tool.git
cd Financial-Audit-AI-Tool
pip install -r requirements.txt
```

Set up AWS credentials with `aws configure`, and create a local `config/config.py` with your keys (it is not committed). Then run:

```bash
python app/main.py
```

Select the Concur files, click **Run Audit**, and check the output folders.

To change the policy, edit `config/policies/policy_rules.txt`. The model ID is set in `services/auditor.py`, and the policy length limit is in `config/settings.py`.

## Team

Davit Hakobyan, Isabela Fernandez, Wenfan Wei, Ellie Romero, and Ashton Liu

## License

MIT
