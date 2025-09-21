---
title: Python
description: Use Python to interact with your objects on lakeFS
---

# Use Python to interact with your objects on lakeFS

!!! warning
    If your project is currently using the [legacy Python `lakefs-client`][legacy-pypi], please be aware that this version has been [deprecated][legacy-deprecated].
    As of release **v1.44.0**, it's no longer supported for new updates or features.

There are several ways to use Python with lakeFS. This page provides an overview of the available options.

!!! info "For previous Python SDKs follow these links"
    [legacy-sdk](https://pydocs.lakefs.io) (Deprecated)

## References & Resources

- **High Level Python SDK Documentation**: [https://pydocs-lakefs.lakefs.io](https://pydocs-lakefs.lakefs.io)
- **Generated Python SDK Documentation**: [https://pydocs-sdk.lakefs.io](https://pydocs-sdk.lakefs.io)
- **Boto S3 Router**: [https://github.com/treeverse/boto-s3-router](https://github.com/treeverse/boto-s3-router)
- **lakefs-spec API Reference**: [https://lakefs-spec.org/latest/reference/lakefs_spec/](https://lakefs-spec.org/latest/reference/lakefs_spec/)

## Python Integration Options

- [**lakeFS SDK**](./sdk.md): The recommended way to perform **object operations**, **versioning** and other **lakeFS-specific operations**.
- [**lakefs-spec**](./lakefs-spec.md): To perform high-level file operations through a file-system-like API, with integrations for pandas, Polars, DuckDB and other data science libraries.
- [**Boto3**](./boto.md): To perform **object operations** through the **lakeFS S3 gateway**.
- [**Generated lakefs-sdk**](https://pydocs-sdk.lakefs.io): For direct API access based on the OpenAPI specification of lakeFS.

[legacy-deprecated]:  ../../posts/deprecate-py-legacy.md
[legacy-pypi]:  https://pypi.org/project/lakefs-client/
