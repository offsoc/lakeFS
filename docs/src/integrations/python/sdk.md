---
title: High-Level Python SDK
description: Use the lakeFS Python SDK to interact with your objects on lakeFS.
---

# High-Level Python SDK

The lakeFS high-level Python SDK provides a convenient and Pythonic way to interact with your lakeFS repositories. It simplifies common data versioning operations and integrates seamlessly with your existing Python data workflows.

## Key Features

*   **Simplified Interface:** The SDK offers a more straightforward and intuitive API compared to the raw lakeFS API, making it easier to perform common tasks.
*   **Automatic Authentication:** It can automatically infer credentials from your environment (e.g., `lakectl.yaml` file or environment variables), reducing boilerplate code.
*   **Powerful Abstractions:** The SDK provides high-level abstractions for complex operations like I/O, transactions, and data import, allowing you to write cleaner and more concise code.

## Getting Started

This guide will walk you through the basics of using the lakeFS Python SDK.

### Installation

Install the Python client using pip:

```shell
pip install lakefs
```

### A Quick Example

Here's a quick example of how to use the SDK to create a repository, commit a file, and read it back:

```python
import lakefs

# 1. Configure your lakeFS credentials.
# The SDK will automatically look for a .lakectl.yaml file in your home directory
# or you can configure credentials using environment variables.
# Alternatively, you can explicitly create a client:
# from lakefs.client import Client
# client = Client(host="http://localhost:8000", username="<YOUR_ACCESS_KEY>", password="<YOUR_SECRET_KEY>")

# 2. Create a new repository.
repo = lakefs.repository("my-new-repo").create(storage_namespace="s3://my-storage-bucket/repos")

# 3. Get the main branch.
main = repo.branch("main")

# 4. Write a new object to the branch.
with main.object("my-file.txt").writer() as w:
    w.write(b"Hello, lakeFS!")

# 5. Commit the changes.
main.commit("Add my-file.txt")

# 6. Read the object back.
with main.object("my-file.txt").reader() as r:
    content = r.read()
    print(content.decode("utf-8"))  # Output: Hello, lakeFS!
```

## Core Concepts

The lakeFS Python SDK is built around a few core concepts that map directly to the lakeFS data model:

*   **Client:** The entry point for all interactions with the lakeFS server. It handles authentication and configuration.
*   **Repository:** Represents a lakeFS repository and is used to perform repository-level operations.
*   **Branch:** Represents a branch within a repository, allowing you to work with an isolated version of your data.
*   **Reference:** A pointer to a specific commit in the repository's history. Branches and tags are types of references.
*   **Object:** Represents a file or object within a repository. The SDK provides methods for reading, writing, and managing objects.



## The Client

The `Client` object is the main entry point for interacting with the lakeFS server. It handles authentication and configuration.

### Initialization

The SDK will automatically attempt to configure the client from your environment. It looks for a `.lakectl.yaml` file in your home directory or for the following environment variables:

*   `LAKECTL_SERVER_ENDPOINT_URL`
*   `LAKECTL_CREDENTIALS_ACCESS_KEY_ID`
*   `LAKECTL_CREDENTIALS_SECRET_ACCESS_KEY`

You can also initialize the client explicitly with your credentials:

```python
from lakefs.client import Client

client = Client(
    host="http://localhost:8000",
    username="<YOUR_ACCESS_KEY>",
    password="<YOUR_SECRET_KEY>",
)
```

### Authentication with AWS IAM Roles (for lakeFS Enterprise)

If you are using lakeFS Enterprise, you can authenticate using your AWS IAM role:

```python
import boto3
from lakefs.client import from_aws_role

# Create a boto3 session
session = boto3.Session()

# Create a lakeFS client from the session
client = from_aws_role(session)
```

## Repositories

The `Repository` object represents a lakeFS repository and is used to perform repository-level operations.

### Getting a Repository

To get a `Repository` object, use the `lakefs.repository()` function:

```python
import lakefs

repo = lakefs.repository("my-repo")
```

### Creating a Repository

You can create a new repository using the `create()` method:

```python
repo = lakefs.repository("my-new-repo").create(storage_namespace="s3://my-storage-bucket/repos")
```

### Listing Repositories

To list all repositories, use the `lakefs.repositories()` function:

```python
for repo in lakefs.repositories():
    print(repo.id)
```



## Branches

The `Branch` object allows you to work with an isolated version of your data.

### Getting a Branch

To get a `Branch` object, use the `branch()` method on a `Repository` object:

```python
main = repo.branch("main")
```

### Creating a Branch

You can create a new branch from an existing reference (another branch or a commit) using the `create()` method:

```python
new_branch = repo.branch("my-experiment").create("main")
```

### Listing Branches

To list all branches in a repository, use the `branches()` method on a `Repository` object:

```python
for branch in repo.branches():
    print(branch.id)
```

### Committing Changes

To commit changes to a branch, use the `commit()` method:

```python
new_branch.commit("Add new data for my experiment")
```

### Merging Changes

You can merge one branch into another using the `merge_into()` method:

```python
new_branch.merge_into("main")
```

### Cherry-Picking Commits

You can use the `cherry_pick()` method to apply the changes from a specific commit to the current branch.

```python
# Cherry-pick a commit by its ID
main.cherry_pick("a1b2c3d4")
```

### Reverting Commits

The `revert()` method allows you to create a new commit that reverses the changes made in a specific commit.

```python
# Revert the last commit on the main branch
main.revert(main.head)
```

### Managing Uncommitted Changes

You can view the uncommitted changes on a branch using the `uncommitted()` method.

```python
for change in new_branch.uncommitted():
    print(change.path, change.type)
```

To discard uncommitted changes, use the `reset_changes()` method.

```python
# Discard all uncommitted changes
new_branch.reset_changes()

# Discard changes to a specific object
new_branch.reset_changes(path="my-file.txt", path_type="object")
```

## References

A `Reference` is a pointer to a specific commit in the repository's history. Branches and tags are types of references.

### Getting a Reference

To get a `Reference` object, use the `ref()` method on a `Repository` object:

```python
# Get a reference to a specific commit
commit_ref = repo.ref("a1b2c3d4")

# Get a reference to the head of a branch
branch_ref = repo.ref("main")
```

### Getting the Commit

To get the `Commit` object that a reference points to, use the `get_commit()` method:

```python
commit = commit_ref.get_commit()
print(commit.message)
```

### Listing Commits (Log)

To get the commit history of a reference, use the `log()` method:

```python
for commit in branch_ref.log():
    print(commit.id, commit.message)
```

### Comparing References (Diff)

You can compare two references to see the differences between them using the `diff()` method.

```python
# Compare two branches
diff = main.diff("my-experiment")

for change in diff:
    print(change.path, change.type)
```

## Tags

A `Tag` is a named pointer to a specific commit, typically used to mark a release or a significant version of your data.

### Creating a Tag

To create a new tag, use the `tag()` method on a `Repository` object and then the `create()` method on the `Tag` object:

```python
repo.tag("v1.0").create("main")
```

### Getting a Tag

To get a `Tag` object, use the `tag()` method on a `Repository` object:

```python
tag = repo.tag("v1.0")
```



## Objects

The `StoredObject` and `WriteableObject` classes represent objects in your lakeFS repository. They provide methods for reading, writing, and managing your data.

### Getting an Object

To get an object, use the `object()` method on a `Branch` or `Reference` object:

```python
obj = repo.branch("main").object("my-file.txt")
```

### Reading Objects

To read an object's content, use the `reader()` method, which returns a file-like object:

```python
with obj.reader() as r:
    content = r.read()
```

### Writing Objects

To write to an object, use the `writer()` method, which returns a file-like object:

```python
with obj.writer() as w:
    w.write(b"New content")
```

You can also upload data directly using the `upload()` method:

```python
obj.upload(data=b"Even newer content")
```

### Deleting Objects

To delete an object, use the `delete()` method:

```python
obj.delete()
```

To delete multiple objects at once, use the `delete_objects()` method on a `Branch` object:

```python
branch = repo.branch("main")
branch.delete_objects(["file1.txt", "file2.txt"])
```

### Listing Objects

To list objects in a repository, use the `objects()` method on a `Branch` or `Reference` object:

```python
for obj in repo.branch("main").objects():
    print(obj.path)
```

### Copying Objects

You can copy an object to a different location within the repository using the `copy()` method.

```python
obj = repo.branch("main").object("my-file.txt")
obj.copy(destination_branch_id="main", destination_path="my-file-copy.txt")
```

## Import Manager

The `ImportManager` provides a convenient way to import data from your object store into your lakeFS repository.

### Creating an Import Manager

To start an import, first create an `ImportManager` object from a `Branch` object:

```python
mgr = repo.branch("main").import_data(commit_message="Import my data")
```

### Adding Import Sources

You can add objects and prefixes to be imported using the `object()` and `prefix()` methods:

```python
# Import a single object
mgr.object("s3://my-bucket/my-file.txt", destination="my-file.txt")

# Import all objects with a given prefix
mgr.prefix("s3://my-bucket/my-data/", destination="my-data/")
```

### Running the Import

Once you have added all the sources, you can run the import using the `run()` method. This will block until the import is complete.

```python
status = mgr.run()

if status.error:
    print(f"Import failed: {status.error.message}")
else:
    print(f"Import successful! Ingested {status.ingested_objects} objects.")
```

### Asynchronous Import

For large imports, you can run the import asynchronously using the `start()` and `wait()` methods:

```python
# Start the import
import_id = mgr.start()

# ... do other work ...

# Wait for the import to complete
status = mgr.wait()
```

You can also check the status of an import at any time using the `status()` method:

```python
status = mgr.status()
```

And cancel an ongoing import using the `cancel()` method:

```python
mgr.cancel()
```

## Transactions

Transactions allow you to perform a sequence of operations on a branch as an atomic unit. The changes are only applied to the branch if all operations in the transaction succeed.

```python
with repo.branch("main").transact("My atomic transaction") as tx:
    tx.object("file-to-delete.txt").delete()
    tx.object("new-file.txt").upload(data=b"This is a new file")
```

If any operation within the `with` block fails, all changes will be rolled back.

## Error Handling

The SDK raises custom exceptions for different types of errors. You can handle these exceptions using a `try...except` block:

```python
from lakefs.exceptions import NotFoundException

try:
    repo.ref("non-existent-branch").get_commit()
except NotFoundException as e:
    print(f"Error: {e}")
```

Common exceptions include:

*   `LakeFSException`: The base exception for all SDK exceptions.
*   `ServerException`: A generic exception for other server-side errors.
*   `NotFoundException`: Raised when a resource (e.g., repository, branch, object) is not found.
*   `ObjectNotFoundException`: A subclass of `NotFoundException` raised specifically for objects.
*   `ConflictException`: Raised when an operation conflicts with the current state of the repository (e.g., creating a branch that already exists).
*   `NotAuthorizedException`: Raised when you are not authorized to perform an operation.
*   `PermissionException`: A subclass of `NotAuthorizedException` raised for file-system like permission errors.
*   `ImportManagerException`: Raised for errors related to the `ImportManager`.
*   `TransactionException`: Raised for errors that occur during a transaction.