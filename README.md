# AWS Resources MCP Server

## Overview

A Model Context Protocol (MCP) server implementation that provides integration with AWS resources through boto3. This server enables AI models to execute AWS queries through a standardized interface.

<img width="1619" alt="image" src="https://github.com/user-attachments/assets/2fe266ca-e641-4ab6-8407-630d221f1402" />



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

### Tools

The server offers a tool for executing AWS queries:

* `query_aws_resources`
  * Execute a boto3 code snippet to query AWS resources
  * Input:
    * `code_snippet` (string): Python code using boto3 to query AWS resources
    * The code must set a `result` variable with the query output
  * Allowed imports:
    * boto3
    * operator
    * json
    * datetime
    * pytz
  * Available built-in functions:
    * Basic types: dict, list, tuple, set, str, int, float, bool
    * Operations: len, max, min, sorted, filter, map, sum, any, all
    * Object handling: hasattr, getattr, isinstance
    * Other: print, __import__

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

Note: Keep your AWS credentials secure and never commit them to version control.

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

### Docker Usage
```
