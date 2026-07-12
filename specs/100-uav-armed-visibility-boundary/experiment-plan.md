# Code Experiment Plan

## Material Passport
- Origin Skill: experiment-agent
- Origin Mode: plan
- Origin Date: 2026-07-12T00:35:00-05:00
- Verification Status: UNVERIFIED
- Version Label: code_plan_v1

## Experiment Overview
- **Objective**: validate cross-log visibility attribution and final cached read
- **Hypothesis**: final-observation-missed disappears; network visibility failures may remain
- **Primary metric**: 5/5 classified runs
- **Secondary**: completion, lifecycle abort, duplicate dispatch
- **Stop rule**: one frozen cell, no tuning or rerun

## Setup
Use the exact MiniNDN command in `quickstart.md`; retain raw logs under the named result path.
