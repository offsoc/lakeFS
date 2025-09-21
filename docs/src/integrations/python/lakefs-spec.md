---
title: lakefs-spec
description: Use lakefs-spec for higher-level file operations on lakeFS objects
---

# Using lakefs-spec for higher-level file operations

The [lakefs-spec](https://lakefs-spec.org/latest/) project
provides higher-level file operations on lakeFS objects with a filesystem API,
built on the [fsspec](https://github.com/fsspec/filesystem_spec) project.

!!! note
    This library is a third-party package and not maintained by the lakeFS developers; please file issues and bug reports directly
    in the [lakefs-spec](https://github.com/aai-institute/lakefs-spec) repository.

## Installation

Install `lakefs-spec` directly with `pip`:

```shell
python -m pip install --upgrade lakefs-spec
```

## Interacting with lakeFS through a file system

To write a file directly to a branch in a lakeFS repository, consider the following example:

```python
from pathlib import Path

from lakefs_spec import LakeFSFileSystem

REPO, BRANCH = "example-repo", "main"

# Prepare a local example file.
lpath = Path("demo.txt")
lpath.write_text("Hello, lakeFS!")

fs = LakeFSFileSystem()  # will auto-discover credentials from ~/.lakectl.yaml
rpath = f"{REPO}/{BRANCH}/{lpath.name}"
fs.put(lpath, rpath)
```

Reading it again from remote is as easy as the following:

```python
f = fs.open(rpath, "rt")
print(f.readline())  # prints "Hello, lakeFS!"
```

Many more operations like retrieving an object's metadata or checking an
object's existence on the lakeFS server are also supported. For a full list,
see the [API reference](https://lakefs-spec.org/latest/reference/lakefs_spec/).

## Integrations with popular data science packages

A number of Python data science projects support fsspec, with [pandas](https://pandas.pydata.org/) being a prominent example.
Reading a Parquet file from a lakeFS repository into a Pandas data frame for analysis is very easy, demonstrated on the quickstart repository sample data:

```python
import pandas as pd

# Read into pandas directly by supplying the lakeFS URI...
lakes = pd.read_parquet(f"lakefs://quickstart/main/lakes.parquet")
german_lakes = lakes.query('Country == "Germany"')
# ... and store directly, again with a raw lakeFS URI.
german_lakes.to_csv(f"lakefs://quickstart/main/german_lakes.csv")
```

A list of integrations with popular data science libraries can be found in the [lakefs-spec documentation](https://lakefs-spec.org/latest/guides/integrations/).

## Using transactions for atomic versioning operations

As with the high-level SDK (see above), lakefs-spec also supports transactions
for conducting versioning operations on newly modified files. The following is an example of creating a commit on the repository's main branch directly after a file upload:

```python
from lakefs_spec import LakeFSFileSystem

fs = LakeFSFileSystem()

# assumes you have a local train-test split as two text files:
# train-data.txt, and test-data.txt.
with fs.transaction("example-repo", "main") as tx:
    fs.put_file("train-data.txt", f"example-repo/{tx.branch.id}/train-data.txt")
    tx.commit(message="Add training data")
    fs.put_file("test-data.txt", f"example-repo/{tx.branch.id}/test-data.txt")
    sha = tx.commit(message="Add test data")
    tx.tag(sha, name="My train-test split")
```

Transactions are atomic - if an exception happens at any point of the transaction, the repository remains unchanged.

## Further information

For more user guides, tutorials on integrations with data science tools like pandas, and more, check out the [lakefs-spec documentation](https://lakefs-spec.org/latest/).
