
Overview
This workflow is triggered on pull request events (opened, reopened, synchronize). Its goal is to:

Gather all relevant context about the PR (title, body, diffs, commit history, project structure, previous AI reviews, etc.)
Filter out already-reviewed changes if the PR was previously reviewed
Build a detailed prompt with project rules and context
Send the diff and context to OpenAI’s API for a code review
Post the AI-generated review as a comment in the PR
Step-by-step Breakdown
1. Checkout code
Uses actions/checkout@v3 to fetch the repository code, with a fetch depth of 100 to ensure enough git history for diffs and logs.
2. Get PR Details
Uses GitHub CLI (gh pr view) to extract the PR’s title and body.
Saves the title and (cleaned) body to files for later use.
3. Fetch Previous Bot Comments with Timestamps
Fetches all PR comments using gh pr view ... --json comments.
Filters for previous AI review comments (those starting with CODE REVIEW -), extracts their bodies and timestamps, sorts them by creation date (newest first), and saves to a file.
Outputs whether previous reviews exist and the timestamp of the latest one, formatted for later git operations.
Builds a human-readable summary of previous reviews.
4. Get Filtered PR Diff
If a previous AI review exists, computes a diff only for changes made since the last review (using the timestamp).
Finds the base commit of the PR.
Gets commits since the last review.
Produces a diff of only those changes.
If no previous review exists, gets the full PR diff.
Saves the result to pr_diff.txt.
5. Get Diff Stats
Uses gh pr diff ... --patch | git apply --stat to generate a summary of files changed, with additions and deletions.
Saves to diff_stats.txt.
6. Extract Recent Commit History
Gets the last 50 commits in the repo (hash, date, message) for context.
Saves to commit_history.txt.
7. Extract README.md
Copies the root README.md (if present) for project context.
8. Generate Project Structure
Uses find to list files and directories up to 3 levels deep, including type, size, and child count.
Saves to project_structure.txt.
9. Send PR Diff to OpenAI for Review
Gathers all collected context (diff, rules, title, body, structure, stats, commit history, previous reviews, etc.).
Constructs a detailed system prompt for OpenAI, emphasizing:
Only provide new feedback (don’t repeat previous reviews)
Focus on best practices, security, performance, and coding standards
Only comment on the diff content
Builds a JSON request file (openai_request.json) for the OpenAI API.
Sends the request to OpenAI’s chat completion endpoint using curl.
Checks for errors and extracts the AI’s review.
Posts the review as a comment on the PR, prefixed with CODE REVIEW - Commit ID: ....
Key Features and Best Practices
Diff Filtering: Only reviews new changes since the last AI review, avoiding redundant feedback.
Context Awareness: Supplies the AI with project rules, structure, commit history, and previous reviews for high-quality, relevant feedback.
Security: Uses GitHub secrets for API tokens.
Error Handling: Checks for missing files, empty responses, and HTTP errors.
Modularity: Breaks down the workflow into clear, logical steps, each saving outputs for later use.
Custom Prompt Engineering: The system prompt guides the AI to avoid repetition, focus on actionable feedback, and adhere to project-specific rules.
Example Use Case
When a developer opens or updates a PR:

The workflow gathers all necessary context and determines what’s new since the last AI review.
It sends only the new code changes (or the full diff if first review) to OpenAI, along with all relevant context.
The AI’s review is posted as a PR comment, helping developers improve code quality and adhere to best practices.
If you need clarification on a specific step or want to know how to customize this workflow, let me know!

