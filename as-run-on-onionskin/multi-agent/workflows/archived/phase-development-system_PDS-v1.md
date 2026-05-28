# Phase Development System (PDS)
**Last updated:** 2026-04-21

## What PDS is

This is the multi-agent system by which Phases are brainstormed, planned / spec'd, audited, implemented, and finalized.


## How the Phase Development System Works

### Three important files
- `multi-agent/plans/PHASE<N>_BRAINSTORM.md` : this file is where goals, aims, and ideas for the Phase are collected and generated, not necessarily in a sensible order.
- `multi-agent/plans/PHASE<N>_FEEDBACK.md` : where human and agent develops leave feedback on the phase's BRAINSTORM file as it develops, and later feedback on the SPEC file as it develops.
- `multi-agent/plans/PHASE<N>_SPEC.md` : The SPEC file is the culmination of BRAINSTORM and FEEDBACK where the ideas are mapped to the most sensible ordering and atomized into sensible "Priorities".


### Three main stages
**1. Brainstorm Stage:**
- In this stage, ideas are collected and generated. In short, in this stage:
	- Start BRAINSTORM file
	- Have other agents give feedback
	- Update BRAINSTORM
	- Iterate over feedback and updates until it feels done
- For a given phase a `multi-agent/plans/PHASE<N>_BRAINSTORM.md` is created.
	- That file is  where the aims and goals of the phase are discussed, where related ideas from KNOWN_ISSUES.md and BRAINSTORM.md are collected, and where new ideas are formed.
	- That file is not necessarily in most sensible order of development.
	- The focus is on completeness of ideas surrounding the aims and goals of the phase. 
	- This file gets updated based on feedback added to `multi-agent/plans/PHASE<N>_FEEDBACK.md`.


**2. SPEC engineering stage:**
- When the brainstorm stage is done, an agent makes an attempt to convert it into a SPEC in the most sensible order accounting for dependencies, atomized into Priorities, etc.
- That first attempt is put in `multi-agent/plans/PHASE<N>_SPEC.md`
- Another agent gives feedback, added to the bottom `multi-agent/plans/PHASE<N>_FEEDBACK.md` with appropriate new section title.
	- Feedback is on whether the SPEC includes everything from the brainstorm and in the most sensible order.
- Agents iterate over FEEDBACK and SPEC updates until the SPEC is considered done
- When the SPEC file is finished:
	- the BRAINSTORM file is moved to `multi-agent/plans/archived/YYYYDDMM-PHASE<N>_BRAINSTORM.md` 
	- the FEEDBACK file is moved to `multi-agent/plans/archived/YYYYDDMM-PHASE<N>_FEEDBACK.md`
	- for both, the YYYYMMDD prefix is the date of the day the SPEC is finished

**3. SPEC Audit / Implement stage:**
- agents then carry out the spec using the system defined in `multi-agent/workflows/spec_plan_three_role_audit_loop.md`
- When the Phase is closed out, the SPEC file is moved to `multi-agent/plans/archived/YYYYDDMM-PHASE<N>_SPEC.md`


## Phase Development Rules

You are in a repo under active development by multiple agents. Assume that in the time between each of our interactions, other agents might have already made changes to the codebase, making your context window stale. To combat stale knowledge, do all of the following:

**Before making any decisions:**
- re-read all files you will modify
- do not rely on prior context or memory
- assume the codebase may have changed since your last interaction

**For each file you modify:**
- show the current relevant snippet
- confirm it matches your assumptions
- then apply changes

**Do not rely on:**
- previous audits
- earlier session context
- remembered file structure

**Only trust the current file contents.**

**Before inserting into CHANGELOG.md:**
- locate the most recent entry
- confirm insertion point explicitly
- do not assume location from memory

**After making changes:**
- re-read the modified files
- confirm they follow conventions exactly
- check against AGENT_CONVENTIONS.md

**On cold starts** 
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.


**After finishing up, before writing your CHANGELOG entry:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**Before giving your wrap-up summary comments to the user:**
- ensure that you have completed all that was asked of you from the prompt and from this file.

**Be thorough during brainstorming and SPEC engineering phases:**
- Identify all code that touches or is touched by changes we are proposing, and that will need to be updated in parallel.
- Identify CLI flags that need updates:
	- If a CLI flag was previously restricted to one pipeline, but now applies to more, it needs to have a CLI flag in the universal section as well as pipeline-specific flags that can override the universal.
	- help strings may need to be updated
- If a new CLI flag is needed:
	- If it will eventually apply to multiple pipelines, then there needs to be one in the universal section, and pipeline-specific versions in the pipeline sections that can override the universal.
	- try to put the CLI flag in the organization category that makes most sense
		- Applies to all pipelines -> Universal, with possible pipeline-specific overrides
		- APS-related -> APS section
		- Timing-related -> Timing section
		- HMM-related -> HMM section
		- Etc.
- Search through markdown files as well as code.
	- Look for overlapping ideas in KNOWN_ISSUES.md and BRAIN_STORM.md
	- Include updates needed for README.md and similar documentation files



## Typical form of the Opening brainstorm prompt to Agent 1 from Orchestrator
```text
Below "N" and "<N>" = N, for Phase N.

If this is a cold start, orient youself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are embarking on development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v1.md

You are acting as Agent 1. This is the first pass in the brainstorming stage, and will be helping to construct the initial versions of:
- multi-agent/plans/PHASE<N>_BRAINSTORM.md 
- multi-agent/plans/PHASE<N>_FEEDBACK.md

Phase <N> will be focused on the following:
1. 
2.
...
N.

My motivation for this phase is:
- ...
- ...

First I want to brainstorm for Phase <N>. 
- To do so, I would like for you to do a deep dive audit of the code base to come up with ideas for the Priorities in this Phase. 
- Let's also make sure to audit scripts in tests/ and scripts/ for filepaths and other things that would need to change along with the Phase <N> changes.
- In addition to auditing the code base, search BRAINSTORM.md and KNOWN_ISSUES.md for ideas, goals, and projects we have previously thought up that are in the same spirit of how we defined the overarching goals for Phase <N>.  
- Can you find ideas, concepts, designs or other things that harmonize with what Phase <N> is shaping up to be? 
- Do you have any new ideas to add to this BRAINSTORM?

When questions arise, you should ask me for input, but also recognize the following aims:
- **Atomize steps:** If it is a new process to update the output, it is a new step. 
- **No backwards cramming:** The pipeline is a chain of steps and a chain of outputs. The output from a step further down the chain goes into the output directory for that step, not to a directory upstream in the chain. If it is a question of merging outputs, merge them, but put the updated file in the later step. We can potentially have a final step directory that contains all the final results.
- **Do it the same:** If the other pipelines do it one way, the per-stage pipeline should do it the same way or follow the same logic.

To start: 
- We can first collect those ideas and create a relatively unordered / arbitrarily ordered way in multi-agent/plans/PHASE<N>_BRAINSTORM.md. 
- You can add open questions for me in multi-agent/plans/PHASE<N>_FEEDBACK.md
- I will update multi-agent/plans/PHASE<N>_FEEDBACK.md with my answers
- When we are done wiht open questions, I will ask other agents for feedback and updates, which they will place in multi-agent/plans/PHASE<N>_FEEDBACK.md
- Then you will read their new FEEDBACK in multi-agent/plans/PHASE<N>_FEEDBACK.md and update multi-agent/plans/PHASE<N>_BRAINSTORM.md accordingly
- After we are satisfied with the BRAINSTORM for this phase, you will then take the contents of multi-agent/plans/PHASE<N>_BRAINSTORM.md and make a proper SPEC in multi-agent/plans/PHASE<N>_SPEC.md.
- Once the SPEC for this phase is made, we can begin the audit+implement volleying between you and another agent as described in multi-agent/workflows/spec_plan_three_role_audit_loop.md

Examples of how to structure files:
- examples from previous phases can be found in the archive here: multi-agent/plans/archived/

Does that sound good?

**After finishing up, before writing your CHANGELOG entry:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v1.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```


## Typical form of the follow-up opening brainstorm prompts to Agents 2+ from Orchestrator
```text
Below "N" and "<N>" = N, for Phase N.

If this is a cold start, orient youself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are continuing development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v1.md

You are acting as Agent 2. This is an audit and feedback session in the brainstorming stage, and you will be providing feedback in:
- multi-agent/plans/PHASE<N>_FEEDBACK.md


Phase <N> will be focused on the following:
1. 
2.
...
N.

My motivation for this phase is:
- ...
- ...

First I want to brainstorm for Phase <N>. 
- I have already started this process with Agent 1 (and Agent 2, ..., ). 
- Agent 1 did a deep dive audit of the codebase, and through a few iterations of back and forth, we constructed multi-agent/plans/PHASE<N>_BRAINSTORM.md. 
- Other agents may have already given feedback as well. If so, it will be in multi-agent/plans/PHASE<N>_FEEDBACK.md.  
- The brainstorm started out arbitrarily ordered, but it is shaping up.
- I would like to continue brainstorming Phase <N> with you to get more perspective. 
- To do so, I would like for you to do a deep dive audit of the code base to come up with ideas for the Priorities in this Phase. 
- Let's also make sure to audit scripts in tests/ and scripts/ for filepaths and other things that would need to change along with the changes made during this Phase.  
- Give feedback on what we do already have in multi-agent/plans/PHASE<N>_BRAINSTORM.md. 
- Importantly, focus on things we missed or do not yet discuss in multi-agent/plans/PHASE<N>_BRAINSTORM.md. 
- In addition to auditing the code base, search BRAINSTORM.md and KNOWN_ISSUES.md for ideas, goals, and projects we have previously thought up that are in the same spirit of how we defined the overarching goals for this Phase. 
- Can you find ideas, concepts, designs or other things that harmonize with what Phase <N> is shaping up to be? 
- Do you have any new ideas to add to this BRAINSTORM?
- Add your findings and ideas to multi-agent/plans/next/PHASE<N>_FEEDBACK.md
- Never delete anything from multi-agent/plans/next/PHASE<N>_FEEDBACK.md, only append your report to the end of it.

When questions arise, you should ask me for input, but also recognize the following aims:
**Atomize steps:** If it is a new process to update the output, it is a new step. 
**No backwards cramming:** The pipeline is a chain of steps and a chain of outputs. The output from a step further down the chain goes into the output directory for that step, not to a directory upstream in the chain. If it is a question of merging outputs, merge them, but put the updated file in the later step. We can potentially have a final step directory that contains all the final results.
**Do it the same:** If the other pipelines do it one way, the per-stage pipeline should do it the same way or follow the same logic.


Whatever your findings are, add them near the bottom of multi-agent/plans/PHASE<N>_FEEDBACK.md in a section named by you, and sign off on your audit. Do not edit anything in multi-agent/plans/PHASE<N>_BRAINSTORM.md directly unless given express permission by me. Do not delete anything in any file unless given permission by me. The contributions you make will be reviewed by myself and other agents for further integration into the plans. Your contribution should be recorded as a CHANGELOG entry as well, and you will possibly want to update TASK.md and/or HANDOFF.md when you are done.

Does that sound good?

**After finishing up, before writing your CHANGELOG entry:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v1.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```

## Typical form of the returning BRAINSTORM prompts to Agent 1 from Orchestrator
```text
Below "N" and "<N>" = N, for Phase N.

If this is a cold start, orient youself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are continuing the development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v1.md



You are acting as Agent 1, and we are still in the brainstorming stage. In this session, you will be responding to feedback found in: 
- multi-agent/plans/PHASE<N>_FEEDBACK.md

And you will be incorporating feedback into:
- multi-agent/plans/PHASE<N>_BRAINSTORM.md

Other agents have reviewed multi-agent/plans/PHASE<N>_BRAINSTORM.md and left their feedback and ideas in multi-agent/plans/PHASE<N>_FEEDBACK.md. Can you review the FEEDBACK file and update the BRAINSTORM file accordingly? Please also update the FEEDBACK file with your Triage decisions based on feedback. What feedback did you accept and incoporate? What feedback did you reject and why? Add your triage section to the bottom of the FEEDBACK file. 
- Never delete anything from multi-agent/plans/next/PHASE<N>_FEEDBACK.md, only append your report to the end of it.

**After finishing up, before writing your CHANGELOG entry:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v1.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```

## Typical form of the returning BRAINSTORM prompts to Agent 2+ from Orchestrator
```text
Below "N" and "<N>" = 14, for Phase 14.

If this is a cold start, orient youself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are continuing development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v1.md

You are acting as Agent 2. This is an audit and feedback session in the brainstorming stage, and you will be providing feedback in:
- multi-agent/plans/PHASE<N>_FEEDBACK.md


Continued Brainstorming:
- I would like to continue brainstorming Phase <N> with you to get more perspective. 
- To do so, I would like for you to do a deep dive audit of the code base to come up with ideas for the Priorities in this Phase. 
- Let's also make sure to audit scripts in tests/ and scripts/ for filepaths and other things that would need to change along with the changes made during this Phase.  
- Give feedback on what we do already have in multi-agent/plans/PHASE<N>_BRAINSTORM.md. 
- Importantly, focus on things we missed or do not yet discuss in multi-agent/plans/PHASE<N>_BRAINSTORM.md. 
- In addition to auditing the code base, search BRAINSTORM.md and KNOWN_ISSUES.md for ideas, goals, and projects we have previously thought up that are in the same spirit of how we defined the overarching goals for this Phase. 
- Can you find ideas, concepts, designs or other things that harmonize with what Phase <N> is shaping up to be? 
- Do you have any new ideas to add to this BRAINSTORM?
- Add your findings and ideas to multi-agent/plans/next/PHASE<N>_FEEDBACK.md. 
- Never delete anything from multi-agent/plans/next/PHASE<N>_FEEDBACK.md, only append your report to the end of it.

When questions arise, you should ask me for input, but also recognize the following aims:
**Atomize steps:** If it is a new process to update the output, it is a new step. 
**No backwards cramming:** The pipeline is a chain of steps and a chain of outputs. The output from a step further down the chain goes into the output directory for that step, not to a directory upstream in the chain. If it is a question of merging outputs, merge them, but put the updated file in the later step. We can potentially have a final step directory that contains all the final results.
**Do it the same:** If the other pipelines do it one way, the per-stage pipeline should do it the same way or follow the same logic.

Whatever your findings are, add them near the bottom of multi-agent/plans/PHASE<N>_FEEDBACK.md in a section named by you, and sign off on your audit. Do not edit anything in multi-agent/plans/PHASE<N>_BRAINSTORM.md directly unless given express permission by me. Do not delete anything in any file unless given permission by me. Do not delete anything in any file unless given permission by me. The contributions you make will be reviewed by myself and other agents for further integration into the plans. Your contribution should be recorded as a CHANGELOG entry as well, and you will possibly want to update TASK.md and/or HANDOFF.md when you are done.

Does that sound good?

**After finishing up, before writing your CHANGELOG entry:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v1.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```

## Typical form of opening SPEC ENGINEERING stage prompt to Agent 1 from Orchestrator
```text
Below "N" and "<N>" = N, for Phase N.

If this is a cold start, orient youself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are continuing the development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v1.md

You are acting as Agent 1, and we are entering the phase SPEC engineering stage. In this session, you will be: responding to feedback found in: 
- constructing multi-agent/plans/PHASE<N>_SPEC.md
- using the ideas in multi-agent/plans/PHASE<N>_BRAINSTORM.md
- and providing any questions or notes in multi-agent/plans/PHASE<N>_FEEDBACK.md

It is time for the first pass of spec engineering. Go ahead and create multi-agent/plans/PHASE<N>_SPEC.md from multi-agent/plans/PHASE<N>_BRAINSTORM.md. Atomize it into Priorities in the most sensible order possible. Any dependencies for a given Priority need to be in an earlier Priority. When you are finished making the Spec, audit it again against multi-agent/plans/PHASE<N>_BRAINSTORM.md to ensure it is complete and there are no gaps. Your audit is a time to challenge and pressure test the weaknesses present. Update the spec file with the results of your own audit, if any changes are still needed. When you are done, another agent will attempt to find things to add or improve, and will place that feedback at the bottom of multi-agent/plans/PHASE<N>_FEEDBACK.md. After their feedback, you will be given an opportunity to triage it and implement the ideas you agree with into multi-agent/plans/PHASE<N>_SPEC.md. Other agents will have a chance to make final judgements on multi-agent/plans/PHASE<N>_SPEC.md, and to give the final go-ahead into the subsequent "audit / implement" cycle. We are not done with SPEC engineering until I say so. We are not ready to move on to implementation until I say so.

**After finishing up, before writing your CHANGELOG entry:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v1.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```

## Typical form of subsequent SPEC ENGINEERING stage prompt to Agent 2 from Orchestrator
```text
Below "N" and "<N>" = N, for Phase N.

If this is a cold start, orient youself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are continuing the development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v1.md

You are acting as Agent 2, and we have begun the phase SPEC engineering stage. In this session, you will be: 
- evaluating multi-agent/plans/PHASE<N>_SPEC.md
- against multi-agent/plans/PHASE<N>_BRAINSTORM<>.md
- and providing any feedback, questions, or notes in multi-agent/plans/PHASE<N>_FEEDBACK.md
- Never delete anything from multi-agent/plans/next/PHASE<N>_FEEDBACK.md, only append your report to the end of it.

We are done with the phase <N> brainstorm stage. We have moved on the spec engineering stage and are using the brainstorm file as guide for engineering the spec file here: multi-agent/plans/PHASE<N>_SPEC.md

Agent 1 has gone through multi-agent/plans/PHASE<N>_BRAINSTORM.md and atomized it into Priorities with the goal of the most sensible order possible. Any dependencies for a given Priority should be in an earlier Priority, and part of your job is to make sure that is true.

You will now audit multi-agent/plans/PHASE<N>_SPEC.md against multi-agent/plans/PHASE<N>_BRAINSTORM.md. Look for gaps, errors, inconsistencies, and anything that can be improved. Is anything missing? Are the priorities ordered correctly? Is it complete and ready to go? Report your findings of this audit at the bottom of multi-agent/plans/PHASE<N>_FEEDBACK.md. Unless given express permission by me, do not directly edit anything in multi-agent/plans/PHASE<N>_BRAINSTORM.md or multi-agent/plans/PHASE<N>_SPEC.md, and never delete anything from multi-agent/plans/PHASE<N>_FEEDBACK.md (only add to the end of it). If the SPEC is ready to go, I will move the multi-agent/plans/PHASE<N>_BRAINSTORM.md file to the archive multi-agent/plans/archived. If there were issues, we can discuss implementing changes to the SPEC based on your FEEDBACK, or I will have Agent 1 review your feedback. We are not done with SPEC engineering until I say so. We are not ready to move on to implementation until I say so.

**After finishing up, before writing your CHANGELOG entry:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v1.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```


## Typical form of subsequent SPEC ENGINEERING stage prompt to Agent 1 from Orchestrator
```
Below "N" and "<N>" = N, for Phase N.

If this is a cold start, orient youself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are continuing the development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v1.md

You are acting as Agent 1, and we are still in the phase SPEC engineering stage. In this session, you will be: 
- evaluating the feedback, questions, or notes placed in multi-agent/plans/PHASE<N>_FEEDBACK.md by other agent(s) (e.g. Agent 2, Agent 3, etc)

Other agent(s) (e.g. Agent 2, Agent 3, etc) have audited multi-agent/plans/PHASE<N>_SPEC.md against multi-agent/plans/PHASE<N>_BRAINSTORM.md, and made some suggestions for improvements that are appended to the bottom of multi-agent/plans/PHASE<N>_FEEDBACK.md.

They thought these issues were most important:
1. ..
2. ...
X. ...

Please read the audits and evaluate their findings and feedback, think of how to improve them further, and implement your plan for updating `multi-agent/plans/PHASE<N>_SPEC.md`. 

Audit the updated SPEC against the conventions used in the codebase and by the pipelines in general. Ensure that the SPEC conforms to expectations set by other pipelines. Ensure that everything across the entire repo that is affected by this work is anticipated in our SPEC. Is there anything across the codebase that touches or is touched by the work we are about to do that we did not include as part of the SPEC? If so, we need to update the SPEC to include instructions on how to handle each. Ask yourself, what else do I need to check for to ensure the SPEC is complete and valid and ready to go? And when you answer your own question, then check for that stuff as well. Update the spec file with the results of your own audit, if any changes are still needed. It will then might go to a follow-up audit round with another agent. 

Ultimately, we are trying to move on to the "audit / implement" cycles between 2 agents, following instructions here: multi-agent/workflows/spec_plan_three_role_audit_loop.md. So, those rounds will catch stuff as well. Please let us know if we are ready to move on and can close out the SPEC engineering stage, and move on to the implementation-oriented stage of Phase <N>. Nebertheless, we are not done with SPEC engineering until I say so. We are not ready to move on to implementation until I say so. If the SPEC is ready to go, I will let you know, and I will move the multi-agent/plans/PHASE<N>_BRAINSTORM.md file to the archive in multi-agent/plans/archived.

**After finishing up, before writing your CHANGELOG entry:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v1.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```

## Typical form of additional rounds of SPEC ENGINEERING stage prompts to Agent 2 from Orchestrator
```
Below "N" and "<N>" = N, for Phase N. X = X for Round X.

If this is a cold start, orient youself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are continuing the development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v1.md

You are acting as Agent 2, and we are still in the phase SPEC engineering stage. In this session, you will continue to: 
- evaluate multi-agent/plans/PHASE<N>_SPEC.md
- against multi-agent/plans/PHASE<N>_BRAINSTORM<>.md
- and provide any feedback, questions, or notes in multi-agent/plans/PHASE<N>_FEEDBACK.md
- Never delete anything from multi-agent/plans/next/PHASE<N>_FEEDBACK.md, only append your report to the end of it.

It is time for round <X>. Agent 1 implemented fixes based on the previous audit. 

You will now audit multi-agent/plans/PHASE<N>_SPEC.md against multi-agent/plans/PHASE<N>_BRAINSTORM.md. Look for gaps, errors, inconsistencies, and anything that can be improved. Is anything missing? Are the priorities ordered correctly? Is it complete and ready to go? 

Also audit the updated SPEC against the conventions used in the codebase and by the pipelines in general. Ensure that the SPEC conforms to expectations set by other pipelines. Ensure that everything across the entire repo that is affected by this work is anticipated in our SPEC. Is there anything across the codebase that touches or is touched by the work we are about to do that we did not include as part of the SPEC? If so, we need to update the SPEC to include instructions on how to handle each. Ask yourself, what else do I need to check for to ensure the SPEC is complete and valid and ready to go? And when you answer your own question, then check for that stuff as well. Update the spec file with the results of your own audit, if any changes are still needed. 

Report your findings of this audit at the bottom of multi-agent/plans/PHASE<N>_FEEDBACK.md. 

Unless given express permission by me, do not directly edit anything in multi-agent/plans/PHASE<N>_BRAINSTORM.md or multi-agent/plans/PHASE<N>_SPEC.md, and never delete anything from multi-agent/plans/PHASE<N>_FEEDBACK.md (only add to the end of it).

Ultimately, we are trying to move on to the "audit / implement" cycles between 2 agents, following instructions here: multi-agent/workflows/spec_plan_three_role_audit_loop.md. So, those rounds will catch stuff as well. Please let us know if we are ready to move on and can close out the SPEC engineering stage, and move on to the implementation-oriented stage of Phase <N>. Nebertheless, we are not done with SPEC engineering until I say so. We are not ready to move on to implementation until I say so.

If the SPEC is ready to go, I will move the multi-agent/plans/PHASE<N>_BRAINSTORM.md file to the archive in multi-agent/plans/archived. If there were issues, we can discuss implementing changes to the SPEC based on your FEEDBACK, or I will have Agent 1 review your feedback.

**After finishing up, before writing your CHANGELOG entry:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v1.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```

## Typical form of Opening Prompt for "AUDIT / IMPLEMENT" cycles to Agent 1 from Orchestrator

See the following files for more on how to enter this phase:
- `multi-agent/workflows/orchestrator.spec_plan_three_role_audit_loop.md`
- `multi-agent/workflows/spec_plan_three_role_audit_loop.md`

Below is from those files, and is here for convenience:
```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template A — Role 1 Initial Audit.

Target file: multi-agent/plans/PHASE<N>_SPEC.md
Target section: ## Priority <N>>.1 ...
Current round: initial audit (audit round 1)

Audit this section deeply against the live codebase. Determine whether it is closed or open.
If it is open, append audit findings and exact implementation instructions into the same section,
including file references, function references, and concrete repair notes. Update CHANGELOG.md for
the audit. Read actual code rather than trusting prior reports. Include scripts/ and tests/ when
relevant.

After finishing up, before writing your CHANGELOG entry:
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop.md
- ensure that you have completed all that is expected of you from your role and template.
```