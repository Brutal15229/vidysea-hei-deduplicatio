# Architecture

## Deduplication Strategy

The pipeline treats every source record as evidence about a real-world institution. It normalizes names, domains, locations, country codes, and known external identifiers, then uses deterministic and fuzzy matching to build connected components. Each component becomes one canonical university.

High-precision signals, such as shared ROR/Wikidata/IPEDS identifiers or exact institutional domains, receive the highest confidence. Name-only matches are allowed only when supported by country, city/state, or strong token similarity.

## Matching Features

University matching uses:

- External identifiers: ROR, Wikidata, GRID, ISNI, IPEDS/unit IDs, WHED/IAU IDs, QS IDs where present.
- Website/domain equality after URL normalization.
- Cleaned institution names with stopwords removed.
- City and state/province agreement.
- Country agreement.
- Source-aware fields such as `portalUniId`, `universityNameClean`, `name`, `basic_info.name`, and source-specific IDs.

Program matching uses:

- Canonical university mapping.
- Normalized program name.
- Degree level / credential.
- Field of study when present.
- Duration, language, and delivery mode as supporting evidence.

## Blocking / Indexing

The pipeline builds candidate blocks before fuzzy comparison:

- `country + external_identifier`
- `country + domain`
- `country + name_fingerprint`
- `country + city + first_name_tokens`

This avoids all-pairs matching and allows the same approach to scale to more countries.

## Confidence Scoring

Scores are interpretable and additive:

- Identifier match: very high confidence.
- Domain match: high confidence.
- Strong name similarity plus location agreement: high confidence.
- Strong name similarity without location: medium confidence and often manual review.
- Conflicting domains or cities lower confidence.

Records with score above the merge threshold are automatically merged. Borderline pairs are emitted to manual review with the exact features that caused the decision.

## Manual Review Logic

Manual review files include:

- Borderline university matches.
- University clusters with conflicting names, domains, or cities.
- Programs that could not be mapped to a canonical university.
- Programs with ambiguous university matches.
- Program duplicates below the automatic threshold but above the review threshold.

## Provenance

Every canonical record keeps `source_records`, including:

- Source collection.
- Raw source `_id`.
- Source portal IDs when present.
- Original names and URLs.
- Fields contributed to the canonical record.

This makes every golden record auditable.

## Scaling

The pipeline streams BSON documents from gzip files, processes only configured countries/sources, and uses blocking indexes before fuzzy scoring. For larger batches, the same modules can be run country-by-country, or moved to Spark/DuckDB while keeping the same feature extraction and scoring logic.
