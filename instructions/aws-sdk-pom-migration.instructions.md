---
description: "AWS SDK v1 to v2 migration rules for Maven POM files. Use when editing pom.xml that contains com.amazonaws dependencies or AWS SDK v1 artifacts."
applyTo: "**/pom.xml"
---

# AWS SDK v1 to v2 Migration — pom.xml

When modifying pom.xml files with AWS SDK v1 dependencies, apply these rules:

## Dependency Replacements
| v1 (com.amazonaws) | v2 (software.amazon.awssdk) |
|---|---|
| `aws-java-sdk` | Remove — use individual modules |
| `aws-java-sdk-s3` | `s3` |
| `aws-java-sdk-dynamodb` | `dynamodb` + `dynamodb-enhanced` |
| `aws-java-sdk-sqs` | `sqs` |
| `aws-java-sdk-simpleworkflow` | `swf` |
| `aws-java-sdk-swf-libraries` | `swf` |

## Keep vtw-aws-common
```xml
<dependency>
    <groupId>com.elsevier.vtw.core</groupId>
    <artifactId>vtw-aws-common</artifactId>
</dependency>
```
This dependency is retained — the jar itself has been upgraded to SDK v2 internally.

## Only add explicit SDK v2 dependencies when code uses AWS SDK classes directly
If a module only uses vtw-aws-common helpers (SQSHelper, S3Helper, DynamoDBDAOHelper, DynamoDBOMHelper), the transitive dependencies from vtw-aws-common should suffice.
