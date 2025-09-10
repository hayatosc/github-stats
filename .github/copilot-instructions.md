# GitHub Stats Visualization

GitHub Stats Visualization is a Python application that generates SVG badges showing GitHub user and repository statistics. It fetches data from GitHub's GraphQL and REST APIs and creates visual representations that can be embedded in README files or GitHub profile pages.

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Bootstrap and Setup
- Install Python 3.8 or higher: `python3 --version` (Python 3.8+ required)
- Install core dependencies: `python3 -m pip install --upgrade pip setuptools wheel` -- may fail due to network timeouts. NEVER CANCEL. Set timeout to 300+ seconds.
- Install project requirements: `python3 -m pip install -r requirements.txt` -- takes 30-60 seconds normally, may fail due to network timeouts. NEVER CANCEL. Set timeout to 300+ seconds.
- **NOTE**: pip install commands may fail with "Read timed out" errors due to PyPI network issues. This is a known limitation in sandboxed environments.
- Validate syntax: `python3 -m py_compile github_stats.py generate_images.py` -- takes under 5 seconds

### Running the Application
- **CRITICAL**: This application requires valid GitHub credentials to run fully
- Set environment variables:
  - `ACCESS_TOKEN`: Personal access token with `read:user` and `repo` permissions
  - `GITHUB_ACTOR`: GitHub username to analyze
- Run statistics generation: `python3 generate_images.py` -- takes 2-10 minutes depending on repository count and GitHub API response times. NEVER CANCEL. Set timeout to 15+ minutes.
- **Without valid credentials**: The application will fail with "A personal access token is required to proceed!" - this is expected behavior

### GitHub Actions Execution
- Primary execution method is via GitHub Actions workflow (`.github/workflows/main.yml`)
- Workflow runs on Ubuntu with Python 3.8
- Executes automatically on push to master and weekly on schedule
- Workflow steps:
  1. Checkout repository
  2. Setup Python 3.8
  3. Install dependencies -- takes 1-2 minutes. NEVER CANCEL. Set timeout to 5+ minutes.
  4. Generate images -- takes 2-10 minutes depending on user's repository count. NEVER CANCEL. Set timeout to 15+ minutes.
  5. Commit generated files

## Validation

### Manual Validation Requirements
- **ALWAYS** validate syntax after making code changes: `python3 -m py_compile github_stats.py generate_images.py`
- **ALWAYS** test imports after changes: `python3 -c "import github_stats, generate_images; print('Import successful')"`
- Test error handling with missing credentials: `python3 generate_images.py` (should show "A personal access token is required to proceed!")
- Test error handling with invalid credentials: `ACCESS_TOKEN=invalid_token GITHUB_ACTOR=test python3 generate_images.py` (should show authentication errors)

### End-to-End Validation Scenarios
**Note**: Full validation requires valid GitHub personal access token
- **Complete workflow test**: Set valid `ACCESS_TOKEN` and `GITHUB_ACTOR`, run `python3 generate_images.py`, verify `generated/overview.svg` and `generated/languages.svg` are created
- **GitHub Actions test**: Push changes to master branch, monitor Actions tab for successful workflow completion
- **SVG validation**: Generated files should be valid SVG format and display correctly in browsers
- **Complete walkthrough**: Follow all bootstrap commands in sequence, verify each step produces expected output

### No Testing Infrastructure
- This repository has no unit tests, integration tests, or test configuration
- No linting tools (flake8, pylint, black) are configured
- Manual validation is the primary testing method

## Common Tasks

### Repository Structure
```
.
├── .github/
│   └── workflows/
│       └── main.yml          # GitHub Actions workflow
├── generated/                # Output directory for SVG files
│   ├── overview.svg         # Generated overview statistics
│   └── languages.svg       # Generated language statistics
├── templates/               # SVG templates
│   ├── overview.svg         # Template for overview statistics
│   └── languages.svg       # Template for language statistics
├── github_stats.py         # Main Stats class and GitHub API interactions
├── generate_images.py      # Main script that generates SVG files
├── requirements.txt        # Python dependencies (requests, aiohttp)
└── README.md              # Installation and usage instructions
```

### Key Dependencies
```
cat requirements.txt
requests
aiohttp
```

### Environment Variables
- `ACCESS_TOKEN` (required): GitHub personal access token with `read:user` and `repo` permissions
- `GITHUB_ACTOR` (required): GitHub username to analyze statistics for
- `EXCLUDED` (optional): Comma-separated list of repositories to exclude (format: `owner/repo`)
- `EXCLUDED_LANGS` (optional): Comma-separated list of programming languages to exclude
- `EXCLUDE_FORKED_REPOS` (optional): Set to `true` to exclude forked repositories from analysis

### Common Command Outputs

#### Python Version Check
```bash
python3 --version
# Expected: Python 3.8.x or higher
```

#### Dependencies Installation
```bash
python3 -m pip install -r requirements.txt
# Expected output: Successfully installed aiohttp-x.x.x requests-x.x.x [and dependencies]
# Time: 30-60 seconds normally, up to 2 minutes with network issues
```

#### Syntax Validation
```bash
python3 -m py_compile github_stats.py generate_images.py
# Expected: No output (success), completes in under 5 seconds
```

#### Import Testing
```bash
python3 -c "import github_stats, generate_images; print('Import successful')"
# Expected output: "Import successful"
# Time: Under 1 second
```

### Error Scenarios and Expected Behavior
- **Missing ACCESS_TOKEN**: `Exception: A personal access token is required to proceed!`
- **Missing GITHUB_ACTOR**: `RuntimeError: Environment variable GITHUB_ACTOR must be set.`
- **Invalid ACCESS_TOKEN**: `aiohttp failed for GraphQL query` followed by JSON decode errors
- **Network issues**: `aiohttp` connection errors or timeouts

## Development Notes

### Code Architecture
- `github_stats.py`: Contains `Stats` and `Queries` classes for GitHub API interactions
- `generate_images.py`: Main entry point, orchestrates statistics gathering and SVG generation
- Uses `asyncio` and `aiohttp` for concurrent API requests to improve performance
- Templates use string substitution to insert statistics into SVG templates

### API Rate Limits
- Application uses GitHub GraphQL API (v4) and REST API (v3)
- Personal access tokens have higher rate limits than unauthenticated requests
- Execution time varies based on:
  - Number of repositories to analyze
  - GitHub API response times
  - Network latency
  - Rate limiting delays

### Modifications Guidance
- **Always** test syntax and imports after making changes
- **Always** validate with both valid and invalid credentials
- Consider GitHub API rate limits when modifying API query logic
- SVG template modifications require understanding of SVG format and CSS styling
- Environment variable changes should be tested in GitHub Actions context

### Deployment Process
- Changes are deployed via GitHub Actions automatically
- Generated SVG files are committed back to the repository
- Users embed generated SVGs in their GitHub profiles or READMEs
- No traditional "deployment" process - it's a content generation tool