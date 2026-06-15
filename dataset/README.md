# Dataset Artifacts

This directory contains the dataset artifacts used by EvoHunt.

## Directory Layout

- `train/`: the training advisory split.
- `test/`: the held-out testing advisory split.
- `wild/`: open-source project lists used for prospective vulnerability discovery.

## Advisory Splits

The `train/` and `test/` splits are derived from the [GitHub Advisory Database](https://github.com/github/advisory-database). We use GitHub-reviewed advisories and organize them by advisory publication month.

The training split covers advisories published from January 2023 through December 2025. The held-out testing split covers advisories published from January 2026 through April 2026, after the training window.

The advisory selection follows the benchmark scope described in the paper. In brief, we retain high- and critical-severity advisories that are reachable and locally reproducible under the audit threat model. Cases requiring live third-party services, cloud infrastructure, physical access, or production-only conditions are excluded, as are duplicate or withdrawn advisories.

## Wild Targets

The `wild/` directory contains selected open-source projects used for discovering recent vulnerabilities outside the temporally split advisory benchmark. These projects are chosen as high-quality, security-relevant open-source targets.

We select candidate projects using the [OpenSSF Scorecard](https://github.com/ossf/scorecard) data available through the [BigQuery public datasets](https://cloud.google.com/bigquery/public-data). The selection favors projects with high criticality scores, substantial downstream dependency usage, and sufficient repository popularity. The lists are split by primary language:

- `js.json`
- `py.json`
- `go.json`
- `rust.json`

For example, the JavaScript/TypeScript query is:

```sql
SELECT
  repo.url,
  repo.language,
  repo.star_count,
  default_score,
  depsdev.dependent_count,
  legacy.commit_frequency,
  legacy.contributor_count,
  legacy.recent_release_count,
  repo.updated_at
FROM `openssf.criticality_score_cron.criticality-score-v0-latest`
WHERE
  repo.language IN ('JavaScript', 'TypeScript')
  AND default_score >= 0.50
  AND depsdev.dependent_count >= 100
  AND repo.star_count >= 100
ORDER BY
  default_score DESC,
  depsdev.dependent_count DESC,
  repo.star_count DESC
LIMIT 200;
```
