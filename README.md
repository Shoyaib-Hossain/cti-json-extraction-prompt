# cti-json-extraction-prompt

A collection of LLM prompts designed to extract structured JSON data from Cyber Threat Intelligence (CTI) reports, advisories, and unstructured text.

## Overview

Cyber Threat Intelligence reports are often published as unstructured prose — blog posts, PDFs, security advisories, or analyst write-ups. This project provides ready-to-use prompts that you can pass to a Large Language Model (LLM) to automatically parse those reports and produce machine-readable JSON output, making CTI data easier to ingest into SIEM platforms, threat-intelligence platforms (TIPs), or STIX/TAXII pipelines.

## Features

- **Structured extraction** – Prompts that reliably produce valid JSON objects covering indicators of compromise (IoCs), threat actors, TTPs (MITRE ATT&CK techniques), malware families, and more.
- **Schema-driven output** – Each prompt targets a well-defined JSON schema so downstream consumers know exactly what fields to expect.
- **Model-agnostic** – Prompts are written in plain English and work with any capable instruction-following LLM (GPT-4, Claude, Gemini, Mistral, Llama 3, etc.).
- **Easy to extend** – Add your own prompts or tweak existing ones to match your organisation's data model.

## Example

### Input (excerpt from a threat report)

```
APT29, also known as Cozy Bear, has been observed targeting government entities
in Western Europe using a new spear-phishing campaign. The emails contain a
malicious PDF that drops a PowerShell loader (SHA-256:
d3adb33fd3adb33fd3adb33fd3adb33fd3adb33fd3adb33fd3adb33fd3adb33f).
The loader communicates with the C2 server at 192.0.2.45 on port 443.
MITRE ATT&CK techniques observed: T1566.001 (Spear-phishing Attachment),
T1059.001 (PowerShell).
```

### Prompt

```
You are a Cyber Threat Intelligence analyst. Extract all threat intelligence
entities from the text below and return ONLY a valid JSON object that conforms
to the following schema. Do not include any explanation outside the JSON.

Schema:
{
  "threat_actors": [{"name": "", "aliases": []}],
  "malware": [{"name": "", "type": ""}],
  "iocs": [{"type": "", "value": "", "context": ""}],
  "ttps": [{"technique_id": "", "technique_name": ""}],
  "targeted_sectors": [],
  "targeted_regions": []
}

Text:
<paste report text here>
```

### Output

```json
{
  "threat_actors": [
    { "name": "APT29", "aliases": ["Cozy Bear"] }
  ],
  "malware": [],
  "iocs": [
    {
      "type": "sha256",
      "value": "d3adb33fd3adb33fd3adb33fd3adb33fd3adb33fd3adb33fd3adb33fd3adb33f",
      "context": "Malicious PDF payload / PowerShell loader"
    },
    {
      "type": "ipv4",
      "value": "192.0.2.45",
      "context": "C2 server, port 443"
    }
  ],
  "ttps": [
    { "technique_id": "T1566.001", "technique_name": "Spear-phishing Attachment" },
    { "technique_id": "T1059.001", "technique_name": "PowerShell" }
  ],
  "targeted_sectors": ["Government"],
  "targeted_regions": ["Western Europe"]
}
```

## Usage

1. Choose a prompt from the `prompts/` directory that matches your extraction goal.
2. Replace the `<paste report text here>` placeholder with your CTI report text.
3. Send the prompt to your preferred LLM API or chat interface.
4. Parse the returned JSON in your application.

### Tips for best results

- Provide the full report text rather than excerpts when possible.
- Use a model with a large context window for lengthy reports.
- If the model returns prose instead of JSON, add the instruction: *"Return only the JSON object with no additional text."*
- For very long documents, split them into sections and merge the resulting JSON arrays.

## Prompts

| File | Description |
|------|-------------|
| `prompts/ioc-extraction.md` | Extract IP addresses, domains, URLs, file hashes, and email addresses |
| `prompts/ttp-extraction.md` | Map observed behaviours to MITRE ATT&CK technique IDs |
| `prompts/threat-actor-profile.md` | Build a structured threat-actor profile (aliases, motivation, targets) |
| `prompts/full-report.md` | Comprehensive extraction combining IoCs, TTPs, actors, and context |

> **Note:** The `prompts/` directory will be populated in upcoming commits. The table above lists the planned prompts.

## Contributing

Contributions are welcome! If you have a prompt that reliably extracts CTI data in a useful structure, please open a pull request:

1. Fork the repository.
2. Create a new branch: `git checkout -b feature/my-prompt`.
3. Add your prompt file under `prompts/` with a brief description at the top.
4. Open a pull request describing what the prompt extracts and which models you tested it with.

## License

This project is released under the [MIT License](LICENSE).
