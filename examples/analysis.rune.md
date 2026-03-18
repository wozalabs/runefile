---
name: competitor_analysis
version: 1.0.0
description: Analyzes a company's competitive landscape
author: woza
tags: [strategy, analysis, business]

model: claude-sonnet-4-20250514
temperature: 0.5
max_tokens: 4000

inputs:
  - name: company_name
    type: string
    required: true
    description: Name of the company to analyze

  - name: industry
    type: enum
    required: true
    options: [saas, fintech, ecommerce, healthtech, climatetech, other]

  - name: focus_areas
    type: array
    items_type: string
    required: false
    default: [pricing, positioning, product]
    description: Specific areas to focus the analysis on

  - name: depth
    type: enum
    required: false
    default: standard
    options: [quick, standard, deep]

output:
  format: json
  schema:
    type: object
    properties:
      company:
        type: string
      competitors:
        type: array
        items:
          type: object
          properties:
            name: { type: string }
            threat_level: { type: string, enum: [low, medium, high] }
            strengths: { type: array, items: { type: string } }
            weaknesses: { type: array, items: { type: string } }
      recommendations:
        type: array
        items: { type: string }
    required: [company, competitors, recommendations]
---

## System

You are a senior strategy consultant with 20 years of experience in competitive analysis. You provide structured, actionable insights.

## Examples

**User:** Analyze Notion in the saas industry.
**Assistant:** {"company":"Notion","competitors":[{"name":"Confluence","threat_level":"medium","strengths":["Enterprise adoption","Atlassian ecosystem"],"weaknesses":["Complex UX","Slow innovation"]}],"recommendations":["Focus on AI-native features","Expand template marketplace"]}

## Context

Analysis depth: {{depth}}
Focus areas: {{focus_areas}}

## Prompt

Perform a competitive analysis for **{{company_name}}** in the **{{industry}}** industry.

Identify the top 3-5 competitors and for each provide:
- Threat level (low/medium/high)
- Key strengths
- Key weaknesses

Then provide 3-5 strategic recommendations for {{company_name}}.

Respond in the specified JSON format.
