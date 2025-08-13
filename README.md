# AWS Resources MCP Server

[![Docker Hub](https://img.shields.io/docker/v/buryhuang/mcp-server-aws-resources?label=Docker%20Hub)](https://hub.docker.com/r/buryhuang/mcp-server-aws-resources)
[![Docker Hub](https://img.shields.io/docker/pulls/buryhuang/mcp-server-aws-resources?label=Docker%20Hub)](https://hub.docker.com/r/buryhuang/mcp-server-aws-resources)

<a href="https://glama.ai/mcp/servers/@baryhuang/mcp-server-aws-resources-python">
  <img width="380" height="200" src="https://glama.ai/mcp/servers/@baryhuang/mcp-server-aws-resources-python/badge" alt="AWS Resources Server MCP server" />
</a>

## Overview

A Model Context Protocol (MCP) server implementation that provides running generated python code to query any AWS resources through boto3.

**At your own risk**: 
I didn't limit the operations to ReadyOnly, so that cautious Ops people can be helped using this tool doing management operations. Your AWS user role will dictate the permissions for what you can do.

<img width="1619" alt="image" src="https://github.com/user-attachments/assets/2fe266ca-e641-4ab6-8407-630d221f1402" />

**Demo: Fix Dynamodb Permission Error**

https://github.com/user-attachments/assets/de88688d-d7a0-45e1-94eb-3f5d71e9a7c7

## Why Another AWS MCP Server?
I tried AWS Chatbot with Developer Access. Free Tier has a limit of 25 query/month for resources. Next tier is $19/month include 90% of the features I don't use. And the results are in a fashion of JSON and a lot of restrictions.

I tried using [aws-mcp](https://github.com/RafalWilinski/aws-mcp) but ran into a few issues:

1. **Setup Hassle**: Had to clone a git repo and deal with local setup
2. **Stability Issues**: Wasn't stable enough on my Mac
3. **Node.js Stack**: As a Python developer, I couldn't effectively contribute back to the Node.js codebase

So I created this new approach that:
- Runs directly from a Docker image - no git clone needed
- Uses Python and boto3 for better stability
- Makes it easy for Python folks to contribute
- Includes proper sandboxing for code execution
- Keeps everything containerized and clean

For more information about the Model Context Protocol and how it works, see [Anthropic's MCP documentation](https://www.anthropic.com/news/model-context-protocol).

## Components

### Resources

The server exposes the following resource:

* `aws://query_resources`: A dynamic resource that provides access to AWS resources through boto3 queries

### Example Queries

Here are some example queries you can execute:

1. List S3 buckets:
```python
s3 = session.client('s3')
result = s3.list_buckets()
```

2. Get latest CodePipeline deployment:
```python
def get_latest_deployment(pipeline_name):
    codepipeline = session.client('codepipeline')

    result = codepipeline.list_pipeline_executions(
        pipelineName=pipeline_name,
        maxResults=5
    )

    if result['pipelineExecutionSummaries']:
        latest_execution = max(
            [e for e in result['pipelineExecutionSummaries']
             if e['status'] == 'Succeeded'],
            key=itemgetter('startTime'),
            default=None
        )

        if latest_execution:
            result = codepipeline.get_pipeline_execution(
                pipelineName=pipeline_name,
                pipelineExecutionId=latest_execution['pipelineExecutionId']
            )
        else:
            result = None
    else:
        result = None

    return result

result = get_latest_deployment("your-pipeline-name")
```

**Note**: All code snippets must set a `result` variable that will be returned to the client. The `result` variable will be automatically converted to JSON format, with proper handling of AWS-specific objects and datetime values.

### Tools

The server offers a tool for executing AWS queries:

* `aws_resources_query_or_modify`
  * Execute a boto3 code snippet to query or modify AWS resources
  * Input:
    * `code_snippet` (string): Python code using boto3 to query AWS resources
    * The code must set a `result` variable with the query output
  * Allowed imports:
    * boto3
    * operator
    * json
    * datetime
    * pytz
    * dateutil
    * re
    * time
  * Available built-in functions:
    * Basic types: dict, list, tuple, set, str, int, float, bool
    * Operations: len, max, min, sorted, filter, map, sum, any, all
    * Object handling: hasattr, getattr, isinstance
    * Other: print, __import__

### Implementation Details

The server includes several safety features:
- AST-based code analysis to validate imports and code structure
- Restricted execution environment with limited built-in functions
- JSON serialization of results with proper handling of AWS-specific objects
- Proper error handling and reporting

## Setup

### Prerequisites

You'll need AWS credentials with appropriate permissions to query AWS resources. You can obtain these by:
1. Creating an IAM user in your AWS account
2. Generating access keys for programmatic access
3. Ensuring the IAM user has necessary permissions for the AWS services you want to query

The following environment variables are required:
- `AWS_ACCESS_KEY_ID`: Your AWS access key
- `AWS_SECRET_ACCESS_KEY`: Your AWS secret key
- `AWS_SESSION_TOKEN`: (Optional) AWS session token if using temporary credentials
- `AWS_DEFAULT_REGION`: AWS region (defaults to 'us-east-1' if not set)

You can also use a profile stored in the `~/.aws/credentials` file. To do this, set the `AWS_PROFILE` environment variable to the profile name.

Note: Keep your AWS credentials secure and never commit them to version control.

### Installing via Smithery

To install AWS Resources MCP Server for Claude Desktop automatically via [Smithery](https://smithery.ai/server/mcp-server-aws-resources-python):

```bash
npx -y @smithery/cli install mcp-server-aws-resources-python --client claude
```

### Docker Installation

You can either build the image locally or pull it from Docker Hub. The image is built for the Linux platform.

#### Supported Platforms
- Linux/amd64
- Linux/arm64
- Linux/arm/v7

#### Option 1: Pull from Docker Hub
```bash
docker pull buryhuang/mcp-server-aws-resources:latest
```

#### Option 2: Build Locally
```bash
docker build -t mcp-server-aws-resources .
```

Run the container:
```bash
docker run \
  -e AWS_ACCESS_KEY_ID=your_access_key_id_here \
  -e AWS_SECRET_ACCESS_KEY=your_secret_access_key_here \
  -e AWS_DEFAULT_REGION=your_AWS_DEFAULT_REGION \
  buryhuang/mcp-server-aws-resources:latest
```

Or using stored credentials and a profile:
```bash
docker run \
  -e AWS_PROFILE=[AWS_PROFILE_NAME] \
  -v ~/.aws:/root/.aws \
  buryhuang/mcp-server-aws-resources:latest
```

## Cross-Platform Publishing

To publish the Docker image for multiple platforms, you can use the `docker buildx` command. Follow these steps:

1. **Create a new builder instance** (if you haven't already):
   ```bash
   docker buildx create --use
   ```

2. **Build and push the image for multiple platforms**:
   ```bash
   docker buildx build --platform linux/amd64,linux/arm64,linux/arm/v7 -t buryhuang/mcp-server-aws-resources:latest --push .
   ```

3. **Verify the image is available for the specified platforms**:
   ```bash
   docker buildx imagetools inspect buryhuang/mcp-server-aws-resources:latest
   ```

## Usage with Claude Desktop

### Running with Docker
#### Example using ACCESS_KEY_ID and SECRET_ACCESS_KEY
```json
{
  "mcpServers": {
    "aws-resources": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "AWS_ACCESS_KEY_ID=your_access_key_id_here",
        "-e",
        "AWS_SECRET_ACCESS_KEY=your_secret_access_key_here",
        "-e",
        "AWS_DEFAULT_REGION=us-east-1",
        "buryhuang/mcp-server-aws-resources:latest"
      ]
    }
  }
}
```

#### Example using PROFILE and mounting local AWS credentials
```json
{
  "mcpServers": {
    "aws-resources": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "AWS_PROFILE=default",
        "-v",
        "~/.aws:/root/.aws",
        "buryhuang/mcp-server-aws-resources:latest"
      ]
    }
  }
}
```

### Running with Git clone
#### Example running with git clone and profile
```
{
  "mcpServers": {
    "aws": {
      "command": "/Users/gmr/.local/bin/uv",
      "args": [
        "--directory",
        "/<your-path>/mcp-server-aws-resources-python",
        "run",
        "src/mcp_server_aws_resources/server.py",
        "--profile",
        "testing"
      ]
    }
  }
}
```