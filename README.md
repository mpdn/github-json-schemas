# Github JSON Schema Dataset

This is a dataset of all JSON schemas on found in the
[GitHub BigQuery Dataset](https://console.cloud.google.com/marketplace/product/github/github-repos).

The schema files are stored as gzipped newline-delimited JSON. Each line is a unique schema file,
with an array of repositories referencing the file. See the [schema](schema.json) for more details.

See [Releases](https://github.com/mpdn/github-json-schemas/releases/latest) for downloading the
dataset.

## Query

```sql
select
  id,
  content,
  usages
from `bigquery-public-data.github_repos.contents`
join (
  select
    id,
    array_agg(struct(repo_name, ref, path, license)) as usages
  from `bigquery-public-data.github_repos.files`
  join `bigquery-public-data.github_repos.licenses` using (repo_name)
  where ends_with(path, '.json')
  group by id
) using (id)
where
  not binary
  and contains_substr(content, '$schema')
  and contains_substr(content, 'json-schema.org')
  and contains_substr(json_query(content, '$."$schema"'), 'json-schema.org')
```