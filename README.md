# Tester Agent

A comprehensive VS Code AI agent customization for QA engineers and testers. Provides specialized skills for test planning, API test automation, and UI test automation with a file-based workflow.

## Features

### 🎯 Tester Agent Mode
Specialized agent persona with QA expertise for:
- Test plan creation from specification files
- QA-focused code reviews
- Test automation implementation (API & UI)
- Edge case identification
- Test coverage assessment

### 📋 Test Plan Creator Skill
Create comprehensive test plans from plain-text or Markdown specification files.

**4-Phase Workflow:**
1. **Requirements Analysis** - Gap detection with severity classification (🔴 Critical | ⚠️ Medium | ⚠️ Low)
2. **Test Case Design** - Risk-based prioritization (P1-P4) with detailed steps
3. **Quality Evaluation** - Weighted scoring system (100-point scale, 75+ to pass)
4. **Automated Output** - Generated summary file with complete test plan

**Key Features:**
- File-based input from `specs/` folder (.txt or .md)
- Risk matrix evaluation (Probability × Severity)
- Coverage matrix tracking (🖐 manual | 🤖 api-automation | 🌐 ui-automation)
- Human approval gates after each phase
- Support for positive, negative, and edge-case tests
- Automated duplicate detection

**Output Structure:**
- Platform identification (web/mobile/both)
- Complete acceptance criteria list
- Detailed test cases with pre-conditions, actions, and expected behaviors
- Coverage matrix showing AC-to-TC mappings
- Risk-based prioritization
- Quality evaluation score

### 🔧 API Test Implementor Skill
Generate Python pytest automation tests for REST or GraphQL endpoints.

**Capabilities:**
- Runnable test files with proper structure
- Fixtures and test data management
- Pydantic schema validators
- httpx/requests integration
- conftest configuration

### 🌐 UI Test Implementor Skill
Generate Playwright TypeScript automation tests for web applications.

**Capabilities:**
- Spec files with best practices
- Page Object Model implementation
- Fixtures and test utilities
- Visual regression support
- E2E test scenarios

## Project Structure

```
.
├── .github/
│   ├── agents/
│   │   └── tester.agent.md         # Symlink to Tester agent definition
│   ├── copilot-instructions.md      # Repository-level AI instructions
│   └── skills/
│       ├── test-plan-creator/       # Symlink to test planning skill
│       ├── api-test-implementor/    # Symlink to API test skill
│       └── ui-test-implementor/     # Symlink to UI test skill
├── agents/
│   └── vscode/
│       └── tester.agent.md          # Tester agent mode definition
├── skills/
│   ├── test-plan-creator/
│   │   └── SKILL.md                 # Test plan creation workflow
│   ├── api-test-implementor/
│   │   └── SKILL.md                 # API test implementation
│   └── ui-test-implementor/
│       └── SKILL.md                 # UI test implementation
├── setup/
│   └── docker/
│       └── jira-mcp/
│           └── docker-compose.yml   # Jira MCP server setup
├── specs/                            # Specification files (gitignored)
│   └── .gitkeep
├── .vscode/
│   └── mcp.json                     # MCP server configuration
├── .env.example                      # Environment variables template
└── .gitignore
```

## Setup

### Prerequisites
- VS Code with GitHub Copilot
- Docker (optional, for Jira MCP integration)
- Git

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/olgasobolevsky/tester-agent.git
   cd tester-agent
   ```

2. **Open in VS Code:**
   ```bash
   code .
   ```

3. **The skills and agent are automatically loaded** by VS Code Copilot from the `.github/` directory.

### Optional: Jira MCP Integration

If you want to integrate with Jira for requirements fetching:

1. **Create `.env` file** from the example:
   ```bash
   cp .env.example .env
   ```

2. **Configure your Jira credentials** in `.env`:
   ```
   JIRA_HOST=your-company.atlassian.net
   JIRA_USER=your-email@company.com
   JIRA_API_TOKEN=your-api-token
   ```

3. **Start the Jira MCP server:**
   ```bash
   cd setup/docker/jira-mcp
   docker-compose up -d
   ```

## Usage

### Creating a Test Plan

1. **Create a specification file** in the `specs/` folder:
   ```bash
   # Create your spec as .txt or .md
   echo "Your requirements here" > specs/my-feature.md
   ```

2. **Activate the Tester agent** in VS Code:
   - Press `Cmd+Shift+P` (Mac) or `Ctrl+Shift+P` (Windows/Linux)
   - Type "Copilot: Select Agent"
   - Choose "Tester"

3. **Request a test plan:**
   ```
   Create a test plan for specs/my-feature.md
   ```

4. **Review and approve each phase:**
   - Phase 1: Requirements analysis and gap detection
   - Phase 2: Test case design with risk prioritization
   - Phase 3: Quality evaluation
   - Phase 4: Automated summary generation

5. **Find your test plan** in `specs/my-feature-summary.md`

### Example Workflow

```
User: @Tester create a test plan for specs/wiki.md

Agent: [Reads spec, performs Phase 1 analysis]
       Platform: web
       Acceptance Criteria: 12 identified
       Gaps Found: 6 (2 🔴 Critical, 3 ⚠️ Medium, 1 ⚠️ Low)
       
       Type "proceed" to continue to Phase 2, or "correct" to revise.

User: proceed

Agent: [Performs risk evaluation, creates test cases]
       16 test cases created:
       - 9 positive scenarios
       - 5 negative scenarios
       - 2 edge cases
       
       Type "proceed" to evaluate, or "correct" to revise.

User: proceed

Agent: [Evaluates test plan]
       Score: 97/100 ✅ PASS
       
       Type "proceed" to generate summary file.

User: proceed

Agent: Summary saved to specs/wiki-summary.md
```

### Implementing Tests

After creating a test plan:

**For API tests:**
```
@Tester implement API tests for TC-001 through TC-005 from specs/wiki-summary.md
```

**For UI tests:**
```
@Tester implement Playwright tests for TC-006 through TC-010 from specs/wiki-summary.md
```

## Skills Reference

### test-plan-creator

**Trigger phrases:** test plan, test cases, requirements analysis, coverage mapping, test strategy

**Input:** Specification file in `specs/` folder (.txt or .md)

**Output:** Comprehensive test plan with:
- Requirements digest with gap analysis
- Detailed test cases with risk prioritization
- Coverage matrix
- Quality evaluation score
- Generated summary file

### api-test-implementor

**Trigger phrases:** implement API tests, write pytest, api automation, REST test, GraphQL test

**Output:** Python pytest files with:
- Test functions
- Fixtures
- Schema validators (Pydantic)
- Configuration

### ui-test-implementor

**Trigger phrases:** implement UI tests, write Playwright, e2e automation, browser test

**Output:** TypeScript Playwright files with:
- Test specs
- Page objects
- Fixtures
- Selectors

## Configuration

### Agent Configuration
Edit [`agents/vscode/tester.agent.md`](agents/vscode/tester.agent.md) to customize:
- Agent description and personality
- Responsibilities
- Default approach
- Tool restrictions

### Skill Configuration
Each skill's `SKILL.md` file contains:
- YAML frontmatter with metadata
- Detailed workflow instructions
- Output format specifications
- Quality criteria

## Test Plan Quality Criteria

Test plans are evaluated on a 100-point scale:

| Category | Points | Criteria |
|----------|--------|----------|
| Requirements Coverage | 25 | All acceptance criteria have test coverage |
| Scenario Completeness | 20 | Positive, negative, and edge cases included |
| Test Case Quality | 20 | Clear steps with pre-conditions and expected results |
| Risk-Based Prioritisation | 15 | P1/P2 cases have appropriate coverage |
| Clarity & Traceability | 10 | AC-to-TC mapping clear, descriptions complete |
| No Duplications | 10 | Test cases are unique and non-redundant |

**Passing threshold:** 75/100

## Risk Priority Levels

| Priority | Risk Score | Criteria | Test Requirements |
|----------|------------|----------|-------------------|
| P1 | 6-9 | High probability × High severity | Must have negative + edge-case tests |
| P2 | 4-5 | Medium risk | Must have negative + edge-case tests where applicable |
| P3 | 2-3 | Low-medium risk | Positive scenarios |
| P4 | 1 | Low risk | Positive scenarios |

**Risk Calculation:** Probability (1-3) × Severity (1-3)

## Contributing

This is a personal tester agent configuration. Feel free to fork and customize for your needs.

## License

MIT

## Author

Olga Sobolevsky

---

**Note:** Generated test plans and spec files in the `specs/` folder are ignored by git. Only commit your source specification files if needed, keeping generated summaries local.