# Cyber Threat Intelligence Extraction Prompt

This repository contains a structured prompt for extracting verifiable Cyber Threat Intelligence (CTI) facts from security articles, crawled pages, and official vendor sources.

The prompt is designed for academic and reproducible security analysis, particularly where outputs must be consistent, evidence-based, and suitable for downstream processing.

## Purpose

The goal of this prompt is to extract only explicitly stated cybersecurity facts and return them in a fixed JSON schema.

## Initial Prompt

```text
You are a senior Cyber Threat Intelligence analyst.

Task: Fetch the article URL provided below.

Return only a JSON object in the following schema:

{
  "publication_date": null,
  "cve": null,
  "asset_type": null,
  "iocs": [],
  "Product_affected": null,
  "Product_affected_versions": [],
  "mitigation_recommend_version": null,
  "patching_link": null,
}

Rules:

1. Extract only facts explicitly stated in the article or in the relevant crawled pages.
2. Asset_type refers to the asset or physical device are affected.
3. For Product_affected_versions give the affected versions. If affected version indicates prior versions of a specific version then search for the prior versions from the official site of the product.
4. For "mitigation_recommend_version", include ONLY the official fixed version string, return null if official version not found.
5. For "patching_link", include the exact official vendor patch, update, or security notice URL that explicitly states the fixed version.
6. If no qualifying official link for "patching_link" is explicitly available in the article or crawled pages, return null.
7. Use null or empty array for any other scalar field not explicitly stated.

Article URL:
```
## Rationale for the Revised Prompt

The revised prompt was selected because it provides stricter and more reproducible extraction constraints than the initial version.

The initial prompt defined the target schema and general extraction requirements, but it did not fully constrain ambiguous version expressions. For example, affected versions expressed as `X and prior` could be returned directly as a range phrase instead of being resolved through official product version histories. Similarly, mitigation recommendations expressed as `X and above` or `X or later` could be returned as fixed versions, even though they are not exact vendor-confirmed remediation versions.

The revised prompt addresses these issues by adding explicit field-level rules.

For `Product_affected_versions`, the prompt requires official vendor or product sources to be checked when prior-version ranges are encountered. It also prohibits fabricated or mathematically expanded version lists.

For `mitigation_recommend_version`, the prompt requires only an exact official fixed version string. If the recommendation is expressed as `above X`, `newer than X`, or `X or later`, the exact fixed version must be verified from an official vendor or product source. If no exact official fixed version is available, the value must be `null`.

For `patching_link`, the prompt allows only official vendor URLs that explicitly state the fixed version. Generic vendor pages, third-party blogs, NVD, MITRE, exploit databases, and unrelated news articles are excluded.

Overall, the revised prompt improves precision, reduces unsupported inference, enforces source authority, and supports reproducible structured extraction for cyber threat intelligence analysis.

## Example Empty Output

```json
{
  "publication_date": null,
  "cve": null,
  "asset_type": null,
  "iocs": [],
  "Product_affected": null,
  "Product_affected_versions": [],
}
```
## Rationale for Omitting Remediation Fields

The fields mitigation_recommend_version and patching_link are omitted to maintain a strict separation between vulnerability intelligence extraction and remediation execution. The extractor is intended to report verifiable facts from source material, such as CVEs, affected products, affected versions, asset types, and IOCs, but not to prescribe operational changes.

We note that mitigation_recommend_version data is often context-dependent across distributions, release branches, package channels, and product editions. Encoding this information in a single field can obscure important constraints or produce misleading null values. Similarly, We also note that "patching_link" interprets by autonomous workflows as actionable update sources, increasing the risk of incorrect or unauthorised remediation.

This design requires remediation decisions to be made only after local asset validation confirms product presence, installed version, platform context, and exposure conditions. Thus, the schema remains factual, context-neutral, and suitable for CTI enrichment, while patch selection and deployment remain under controlled remediation workflows.

## Revised/Final Prompt

```text
You are a cyber threat intelligence extraction engine.

Your task is to extract only verifiable facts from the provided article and any relevant crawled pages.

Return valid JSON only, using the following schema:

{
  "publication_date": null,
  "cve": null,
  "asset_type": null,
  "iocs": [],
  "Product_affected": null,
  "Product_affected_versions": [],
}

Do not include markdown, commentary, explanations or extra fields in the final output.

FIELD RULES

publication_date:

1. Extract only the article publication date explicitly stated in the article.
2. If no publication date is explicitly stated, return null.

cve:

1. Extract only CVE identifiers explicitly stated in the article or relevant crawled pages.
2. If one CVE is stated, return it as a string.
3. If multiple CVEs are stated and the schema must remain exact, return an array of CVE strings in this field.
4. If no CVE is stated, return null.

asset_type:

1. Refers to the affected asset, system, product category, or physical device type.
2. Extract only if explicitly stated or directly clear from the affected product.
3. If not explicitly stated or directly identifiable from the product description, return null.

iocs:

1. Extract only indicators of compromise explicitly stated in the article or relevant crawled pages.
2. Include only values identified as indicators, observables, malicious infrastructure, exploit artifacts, or compromise evidence.
3. If no IOCs are explicitly stated, return [].

Product_affected:

1. Extract the affected product name exactly as stated.
2. If multiple affected products are stated and they share the same affected versions, fixed version, and patching link, return an array of product names in this field.
3. If multiple affected products have different affected versions, fixed versions, or patching links, return one JSON object per affected product using the exact same schema.
4. If the affected product is not explicitly stated, return null.

Product_affected_versions:

1. Return affected versions as an array of exact strings.
2. If no affected versions are explicitly stated, return [].
3. If the affected version is stated as “prior to X”, “earlier than X”, “X and earlier”, or equivalent, search only the official vendor/product source for officially listed prior versions.
4. Official sources may include vendor advisories, product security notices, release notes, changelogs, version history pages, official documentation, or official download pages.
5. Include only prior versions explicitly listed by the official vendor/product source.
6. Do not use GitHub tags, package registries, mirrors, third-party changelogs, vulnerability databases, or news articles unless they are the official vendor-maintained release source.
7. If official prior versions cannot be found, preserve the original affected range string exactly as stated.
8. Do not fabricate or mathematically expand version ranges.

mitigation_recommend_version:

1. Return only the official fixed version string.
2. The fixed version must be explicitly stated by an official vendor/product source.
3. Acceptable sources include official vendor advisories, official security notices, official release notes, official changelogs, official patch notes, or official update documentation.
4. If the mitigation recommendation version is stated as “above X”, “newer than X”, “X or later” or equivalent, verify the exact fixed version string from an official vendor/product source.
5. Do not treat “latest version”, “current version”, “upgrade immediately”, or “apply available updates” as a fixed version.
6. Do not use workaround-only mitigations as fixed versions.
7. If no official fixed version string is explicitly found, return null.

patching_link:

1. Return only the exact official vendor URL that explicitly states the fixed version.
2. The URL must be an official vendor advisory, security notice, release note, changelog, patch note, update page, or official documentation page.
3. The page must explicitly support the value used in mitigation_recommend_version.
4. Do not return generic vendor homepages, generic download portals, generic support pages, NVD pages, MITRE pages, third-party blogs, news articles, exploit databases, vulnerability databases, mirrors, or social media posts.
5. If no qualifying official URL is explicitly available in the article or crawled pages, return null.
6. If mitigation_recommend_version is null, patching_link must also be null unless an official patch page explicitly states a fix but no fixed version string is provided.

OUTPUT RULES

- Return JSON only.
- Do not wrap the JSON in markdown.
- Do not add comments.
- Do not add trailing commas.
- Use exactly the schema fields listed above.
- Use double quotes for all JSON keys and string values.
- Use null for unknown scalar values.
- Use [] for unknown array values.
- Do not include explanations, citations, source notes, or reasoning in the output.

GENERAL RULES

1. Extract only facts explicitly stated in the article, relevant crawled pages, or official vendor/product pages used for version verification.
2. Do not infer, guess or fill gaps using general knowledge.
3. Use null for missing scalar fields.
4. Use [] for missing array fields.
5. Prefer official vendor or product sources when extracting fixed versions, patch links, and version history.
6. Do not use third-party blogs, news articles, vulnerability databases, mirrors, social media posts, or non-vendor pages for fixed-version or patch-link fields unless they directly link to an official vendor source.
7. If sources conflict, prefer the official vendor source. If two official sources conflict, use the most recent official source. If the correct value cannot be determined, return null for the disputed scalar field or [] for the disputed array field.

DEFAULT EMPTY OUTPUT

If no relevant facts can be extracted, return exactly:

{
  "publication_date": null,
  "cve": null,
  "asset_type": null,
  "iocs": [],
  "Product_affected": null,
  "Product_affected_versions": [],
}
Article URL:
```
## Local Asset Validation

The extracted CTI output represents external vulnerability intelligence only. It does not confirm that the local environment is affected.

Before any remediation action is taken, the affected product, version, operating system, asset type, and exposure conditions must be validated against local asset inventory or endpoint telemetry.

External advisories answer: "What is vulnerable?"

Local asset validation answers: "Are we vulnerable?"

The extraction result must not be used as a direct patching instruction. It should be used as structured input for local validation, prioritisation, and approved remediation workflows.
