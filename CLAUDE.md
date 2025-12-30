# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI agent system for frontend developers (3 years experience) preparing for job interviews. Uses Claude CLI custom commands and skills to provide resume review, interview questionnaire generation, and 3-agent evaluation.

## Custom Commands

```bash
# Analyze and score resume (out of 100)
/review-resume resumes/[name].md

# Generate customized interview questionnaire based on resume
/create-questionnaire resumes/[name].md

# Comprehensive evaluation via 3-agent discussion system
/evaluate resumes/[name]_questionnaire.md
```

## Architecture

### Workflow
```
Resume → /review-resume → /create-questionnaire → Write Answers → /evaluate → Final Report
```

### 3-Agent Evaluation System
- **Technical Evaluator** (evaluator-technical.md): Strict and critical, 45% weight on technical skills
- **Communication Evaluator** (evaluator-communication.md): Flexible and positive, 30% weight on soft skills
- **Final Arbiter** (evaluator-final.md): Neutral, reconciles opinions and makes final judgment

### File Structure
- `.claude/commands/`: User-executable command definitions
- `.claude/skills/`: Evaluation criteria and agent persona definitions
- `resumes/`: Resume storage and output files (review, questionnaire, evaluation)

## Output File Naming Convention

| Command | Output File |
|---------|-------------|
| `/review-resume` | `resumes/[name]_review.md` |
| `/create-questionnaire` | `resumes/[name]_questionnaire.md` |
| `/evaluate` | `resumes/[name]_evaluation.md` |

## Evaluation Criteria Summary

### Resume Review (100 points)
- Basic Components: 20 pts
- Technical Skills Expression: 30 pts
- Achievement-focused Description: 25 pts
- Project Experience: 15 pts
- Readability & Format: 10 pts

### 3-Year Experience Weight Distribution
- Technical Skills: 45%
- Soft Skills: 30%
- Growth Potential: 25%

### Final Judgment Criteria
- 85-100: Strong Pass
- 70-84: Pass
- 60-69: Hold
- 50-59: Conditional
- 0-49: Fail
