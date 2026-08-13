# Serverless & Event-Driven Architecture Cheatsheet

Complete quick reference guide for AWS Lambda, API Gateway, DynamoDB, SQS/SNS, EventBridge, Step Functions, security, observability, cost optimization, and deployment patterns.

## ⚡ AWS Lambda Quick Reference

### Lambda Basics

- Timeout: Default 3s, max 900s (15 min)
- Memory: 128–10,240 MB (1 MB increments)
- Concurrent Executions: Default 1000 per region
- Cold Start Time: Python 50-200ms, Node 30-150ms, Java 500-2000ms
- Provisioned Concurrency: $0.015/concurrency-hour

### Lambda Pricing

- Invocation: $0.0000002 per request (1M free/month)
- Duration: $0.0000166667 per GB-second
- Provisioned Concurrency: $0.015 per concurrency-hour
- Data Transfer: $0.09 per GB out (1GB free/month)

### Environment Variables

- Max 4 KB per function
- Encrypted at rest with KMS
- Access via: `os.environ` (Python), `process.env` (Node)

## 🌐 AWS API Gateway Quick Reference

### API Gateway Types

- REST API: HTTP/HTTPS endpoints; AWS-native
- HTTP API: Faster, cheaper; minimum features
- WebSocket API: Persistent connections; real-time

### Common Integrations

- Lambda (`AWS_PROXY`), HTTP (`HTTP_PROXY`), AWS Service (`AWS`)
- Mock responses for testing
- VPC Link for private integrations

### Authentication Options

- API Keys (simple, limited security)
- AWS IAM (AWS service-to-service)
- Lambda Authorizers (custom logic)
- Cognito (user authentication)
- OAuth 2.0 / OpenID Connect

## 🗄️ DynamoDB Quick Reference

### DynamoDB Basics

- Partition Key: Required; determines shard
- Sort Key: Optional; sorts within partition
- GSI (Global Secondary Index): Alternative partition + sort
- LSI (Local Secondary Index): Same partition, different sort
- Max Item Size: 400 KB

### Pricing Modes

- On-Demand: $1.25/M writes, $0.25/M reads (bursty traffic)
- Provisioned: Fixed capacity; autoscaling available

### Common Operations

- `GetItem`, `PutItem`, `UpdateItem`, `DeleteItem` (single item)
- `Query` (partition key required), `Scan` (expensive)
- `BatchGetItem`, `BatchWriteItem` (25 items max)
- `TransactGetItems`, `TransactWriteItems` (ACID transactions)

## 📬 AWS SQS and SNS Quick Reference

### SQS (Simple Queue Service)

- Standard Queue: Best effort ordering, at-least-once delivery
- FIFO Queue: Exactly-once processing, strict ordering
- Visibility Timeout: Default 30s; message invisible while processing
- Dead-Letter Queue (DLQ): Failed messages after max retries
- Message Retention: Default 4 days; max 14 days

### SNS (Simple Notification Service)

- Pub/Sub Model: Publishers don't know subscribers
- Delivery Methods: HTTP, Email, SQS, Lambda, SMS
- Message Filtering: Subscribers only get matching messages
- FIFO Topics: Ordered, exactly-once delivery (with SQS FIFO)

## 🔀 AWS EventBridge Quick Reference

### EventBridge Basics

- Event bus (default or custom) receives events
- Rules route events to targets
- Targets: Lambda, SNS, SQS, Kinesis, Step Functions, etc.

### Event Patterns

- Service event patterns: match source and detail-type
- Custom events: define your own JSON structure
- Scheduled events: cron expressions for scheduled functions

Example rule pattern matching a source and detail-type:
```json
{
  "source": ["myapp.orders"],
  "detail-type": ["OrderCreated"],
  "detail": {
    "status": ["confirmed"]
  }
}
```

Example scheduled rule (cron expression):
```json
{
  "ScheduleExpression": "cron(0 12 * * ? *)"
}
```

### Advantages

- Event routing logic centralized (vs Lambda routing)
- Supports 90+ AWS services as sources
- Cross-account event routing
- Dead-letter queues for failed targets

## 🪜 AWS Step Functions Quick Reference

### State Types

- Task: Perform work (Lambda, API call, etc.)
- Choice: Branch logic (if/then/else)
- Wait: Delay execution
- Parallel: Execute multiple branches simultaneously
- Map: Iterate over array items
- Pass: Transform data without work
- Succeed/Fail: Terminal states

### Pricing

- $0.000025 per state transition (includes retries)
- Standard: long-running workflows (up to 1 year); billed per state transition
- Express: short-duration workflows (<5 minutes); billed by invocation, duration, and memory

## 🔐 Security Quick Reference

### IAM Best Practices

- Least privilege: only required permissions
- Service-specific policies (`AWSLambdaBasicExecutionRole`)
- Resource-specific policies (ARN, not wildcards)
- Condition keys for additional context

### Secrets Management

- Secrets Manager: Rotating secrets, audit, encryption
- Parameter Store: Config values, no rotation
- Never: Hardcode secrets in code or IaC

### Encryption

- At rest: KMS encryption for sensitive data
- In transit: TLS 1.2+ for all connections
- VPC Endpoints: Private connections to AWS services

## 📊 Observability Quick Reference

### Logging

- Format: JSON structured logging recommended
- Fields: `timestamp`, `level`, `requestId`, `message`, `context`
- CloudWatch Logs Insights: Query with filter and stats

### Metrics

- Standard: Duration, Errors, Invocations, Throttles, Concurrent Executions
- Custom: Business KPIs (orders, revenue, users)
- Dimensions: Filter and aggregate (environment, function, user)

### Distributed Tracing

- X-Ray: AWS service map, request journey visualization
- Correlation ID: Track request through services
- Sampling: 100% errors, 10% successes for cost management

## 💰 Cost Optimization Quick Reference

### Cost Formula

- Lambda: `(Invocations x $0.0000002) + (GB-seconds x $0.0000166667)`
- Cost per invocation ≈ `(Memory/1024) x (Duration/1000) x $0.0000166667`

### Optimization Strategies

- Memory tuning: Test multiple levels; find optimal
- Caching: CloudFront (50-80% reduction), ElastiCache, in-memory
- Batching: 10-100x invocation reduction
- Event-driven: 50-80% cheaper than polling
- DynamoDB: Right pricing model (on-demand vs provisioned)

### Monitoring Costs

- Cost Explorer: Track spending trends
- Budgets: Set limits; alert on overages
- Anomaly Detection: Alert on unusual spending
- Tag resources: Cost allocation by team/project

## 🛠️ SAM CLI Commands

SAM template validation:
```bash
sam validate --template template.yaml
```

Local testing:
```bash
sam local start-api                              # Start API locally
sam local invoke Function -e event.json          # Invoke function with event
sam local generate-event sns publish --help      # Generate sample events
```

Building and deploying:
```bash
sam build                 # Build application
sam deploy --guided       # Deploy with prompts
sam delete                # Delete stack
```

Testing:
```bash
sam local start-lambda    # Start Lambda server
sam local start-db        # Start local DynamoDB
```

## 🏗️ Terraform Essentials

```bash
terraform init            # Initialize Terraform
terraform validate        # Validate configuration
terraform fmt              # Format code
terraform plan             # Preview changes
terraform apply             # Apply changes
terraform destroy           # Destroy resources
terraform state list         # List resources
terraform output             # Show outputs
terraform workspace list      # List workspaces
```

## 📈 CloudWatch Logs Insights Queries

Find errors in last hour:
```
fields @timestamp, @message
| filter level = "ERROR" AND @timestamp > ago(1h)
```

P95 latency:
```
fields duration
| stats pct(duration, 95) as p95_latency
```

Error rate by endpoint:
```
fields endpoint, level
| stats count(level = "ERROR") / count(*) as error_rate by endpoint
```

Top 10 slowest requests:
```
fields duration, requestId
| sort duration desc
| limit 10
```

## 💵 Common Patterns and Their Costs

| Pattern | Cost/Month (1M Requests) | Use Case |
|---|---|---|
| API → Lambda → DynamoDB | ~$2-5 | REST API, read-heavy |
| Event-driven with SQS → Lambda | ~$1-2 | Async processing, cost-optimized |
| Polling SQS every 10s | ~$50+ | Wasteful; use event-driven instead |
| S3 → Lambda → DynamoDB | ~$3-6 | File processing, analytics |
| CloudFront Cache Hit (80%) | ~$0.5 + CF cost | Recommended for API responses |

## 🌳 Decision Trees

### Choosing Message Service

- SQS? Decoupled processing, retries, DLQ
- SNS? Fan-out to multiple subscribers
- EventBridge? Event routing, 90+ sources, cross-account
- Kinesis? Real-time streaming, multiple consumers

### Choosing Workflow Orchestration

- Step Functions? Complex workflows, human approval, retries
- EventBridge? Event routing, simple rules
- SQS/SNS? Simple queuing, fan-out
- Code? Simple logic, <1 minute execution

### Choosing Database

- DynamoDB? Key-value, unpredictable traffic (on-demand)
- RDS? Relational, complex queries, ACID
- ElastiCache? Caching, sessions, real-time leaderboards
- S3? Large files, archives, static content

## 🎯 Interview Preparation Checklist

### Core Concepts You Should Know

- What is serverless? Advantages vs disadvantages?
- Lambda cold starts: causes, impact, solutions?
- Event-driven vs API-driven architecture?
- DynamoDB partition key design?
- Idempotency: why important, how to implement?
- SLA vs SLO: what's the difference?
- Three pillars of observability?
- Cost drivers in serverless?

### Common Follow-up Questions

- "How would you handle failures in this design?"
- "How would you scale this to handle 1M requests/second?"
- "How would you monitor and debug this in production?"
- "What's the estimated cost? How would you optimize?"
- "How would you deploy this safely?"

## 🚨 Emergency Troubleshooting Checklist

### Lambda Not Invoking?

- Check IAM role permissions (Lambda execution role)
- Check event source configuration (SNS, SQS, API Gateway)
- Check Lambda timeout (default 3s)
- Check CloudWatch Logs for error message
- Check concurrency limits (may be throttled)

### API Gateway Returns 500?

- Check Lambda execution role has API Gateway invoke permission
- Check Lambda function response format (proxy integration)
- Check Lambda CloudWatch Logs for exception
- Check request body matches expected format

### DynamoDB Throttled?

- Check provisioned capacity (if provisioned mode)
- Check partition key hotspots (uneven distribution)
- Check for Scans (expensive; use Query instead)
- Enable autoscaling or switch to on-demand

## 📚 Resources and References

### Official Documentation

- AWS Lambda: docs.aws.amazon.com/lambda
- AWS SAM: docs.aws.amazon.com/serverless-application-model
- Terraform AWS Provider: registry.terraform.io/providers/hashicorp/aws
- Azure Functions: docs.microsoft.com/en-us/azure/azure-functions

### Best Practices and Guides

- OWASP API Security: owasp.org/www-project-api-security
- AWS Well-Architected Framework: aws.amazon.com/architecture/well-architected
- DORA DevOps Metrics: getdora.dev

### Community Tools

- Serverless Framework: serverless.com
- AWS Lambda Powertools: docs.powertools.aws.dev
- LocalStack: localstack.cloud (local AWS emulation)

## 💲 AWS Services Pricing Summary

| Service | Primary Cost Metric | Typical Cost (1M requests) |
|---|---|---|
| Lambda | Invocations + Duration x Memory | $0.50-5 |
| API Gateway | Per request (REST), per call (HTTP) | $3.50 REST, $0.50 HTTP |
| DynamoDB (on-demand) | Per read/write unit | $0.25 reads + $1.25 writes |
| SQS | Per request (64 KB chunks) | $0.50 |
| SNS | Per request | $0.50 |
| EventBridge | Per event published | $1.00 |
| CloudWatch Logs | Per GB ingested | $2-5 (varies by logging) |
| CloudFront | Per GB transferred | $0.085 (varies by region) |

## 🩹 Common Failure Patterns and Solutions

### Lambda Timeouts

- Cause: Long-running operation (DB query, API call)
- Solution: Async pattern, increase timeout, optimize code
- Max Timeout: 900 seconds (15 minutes)

### DynamoDB Throttling

- Cause: Exceeding provisioned capacity or hot partition
- Solution: Increase capacity, improve partition key, use on-demand
- Exponential Backoff: Retry with delays

### Lambda Concurrency Exceeded

- Cause: Too many simultaneous invocations (1000 default limit)
- Solution: Request limit increase, implement queue, scale async

### API Gateway 503

- Cause: Lambda or downstream service overloaded
- Solution: Add error handling, implement circuit breaker, scale

## ⏱️ Performance Tuning Checklist

### Lambda Performance

- Memory: Test multiple levels (128, 256, 512, 1024, 3008 MB)
- Initialization: Move outside handler; cache connections
- Dependencies: Minimize package size; lazy load
- Cold Start: Acceptable? Consider provisioned concurrency
- Duration: Profile with X-Ray; identify bottlenecks

### API Performance

- Caching: CloudFront for static; TTL for dynamic
- Compression: gzip responses; reduce payload
- Pagination: Large result sets should paginate
- Filtering: Server-side filtering more efficient than client

### Database Performance

- DynamoDB: Query > Scan; use indexes; batch operations
- RDS: Connection pooling; prepared statements; read replicas
- Caching: ElastiCache for hot data; TTL for freshness

## 🛡️ Security Checklist

### API Security

- API Keys for non-critical endpoints only
- IAM for service-to-service authentication
- Lambda authorizers for custom logic
- CORS: Only allow required origins
- Throttling: Rate limit to prevent abuse
- Input validation: Validate all inputs

### Data Security

- Encryption at rest: KMS for DynamoDB, RDS
- Encryption in transit: TLS 1.2+; HTTPS everywhere
- VPC Endpoints: Private connections to AWS services
- Secrets Manager: Rotate credentials, audit access

### IAM Security

- Least privilege: Only required permissions
- Resource-specific policies: ARN specific, not wildcards
- Condition keys: Time-based, IP-based restrictions
- Regular audits: Review who has what access

## 📟 Monitoring and Alerting Quick Reference

### Key Metrics to Monitor

- Availability: Error rate (target: <0.1%)
- Performance: P50, P95, P99 latency (target: <500ms p95)
- Capacity: Concurrent executions, queue depth
- Cost: Cost per invocation, cost trend
- Deployment: Deployment frequency, MTTR

### Alert Thresholds

- P95 latency > 500ms: Investigate bottleneck
- Error rate > 1%: Page on-call engineer
- Cost spike > 20%: Investigate root cause
- Deployment failure > 10%: Review deployment process

### Dashboard Essential Panels

- Invocation count (trend)
- Error rate (with error breakdown)
- P95 latency (by endpoint/service)
- Cost (daily trend, cost per invocation)
- Active alarms (on-call status)

## 🚀 Deployment Patterns Quick Reference

| Strategy | Downtime | Resource Cost | Rollback |
|---|---|---|---|
| All-at-Once | Minutes | 1x | Slow (redeploy) |
| Rolling | None | 1.5x during | Complex |
| Blue-Green | None | 2x during | Instant (alias switch) |
| Canary | None | 1.1x during | Automatic (if monitoring) |

## 🧩 Common AWS SDK Patterns

Python Boto3: DynamoDB Query
```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Orders')

response = table.query(
    KeyConditionExpression='pk = :pk AND sk > :sk',
    ExpressionAttributeValues={
        ':pk': 'user123',
        ':sk': '2025-Q1-01'
    }
)
```

Python Boto3: SQS Send Message
```python
sqs = boto3.client('sqs')

sqs.send_message(
    QueueUrl='https://queue.amazonaws.com/queue-name',
    MessageBody='{"order_id": "123"}'
)
```

Python Boto3: Lambda Invoke
```python
lambda_client = boto3.client('lambda')

lambda_client.invoke(
    FunctionName='process-order',
    InvocationType='Event',  # Async
    Payload=json.dumps({'order_id': '123'})
)
```

## ✅ Lambda Environment Setup Checklist

### Development Environment

- Python/Node/Java SDK installed
- AWS CLI configured (`aws configure`)
- SAM CLI installed (`sam --version`)
- Docker installed (for local testing)
- IDE with linting (PyCharm, VSCode)

### Testing Setup

- Unit test framework (pytest, jest)
- Mocking library (moto for AWS mocks)
- Coverage tools (pytest-cov)
- Integration test environment (local/staging)

### Production Readiness

- Structured logging configured
- Monitoring/alarms set up
- Secrets management in place
- Disaster recovery plan documented
- Runbooks for common issues

## ☁️ AWS vs Azure Quick Comparison

| Aspect | AWS | Azure |
|---|---|---|
| Function Service | AWS Lambda | Azure Functions |
| Queue Service | SQS / Kinesis | Service Bus / Queue Storage |
| Pub/Sub | SNS / EventBridge | Event Grid / Service Bus |
| Workflow Orchestration | Step Functions | Logic Apps / Durable Functions |
| NoSQL Database | DynamoDB | Cosmos DB |
| Relational Database | RDS | SQL Database / PostgreSQL |
| Monitoring | CloudWatch / X-Ray | Monitor / Application Insights |
| IaC | CloudFormation / SAM | ARM Templates / Bicep |

## 🖊️ Lambda Handler Signatures by Runtime

Python 3.11 handler:
```python
def lambda_handler(event, context):
    return {'statusCode': 200, 'body': 'success'}
```

Node.js 18.x handler:
```javascript
exports.handler = async (event, context) => {
    return { statusCode: 200, body: 'success' };
};
```

Java 11 handler:
```java
public class Handler implements RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {
    public APIGatewayProxyResponseEvent handleRequest(
            APIGatewayProxyRequestEvent event, Context context) {
        return new APIGatewayProxyResponseEvent();
    }
}
```

## ❌ Common Misconceptions Clarified

**"Serverless means no servers"**
Servers exist; AWS manages scaling, patching, and availability.

**"Serverless has no cold starts"**
Cold starts occur (50-2000ms); only on first invocation or scale-up.

**"Serverless can't handle long-running tasks"**
Lambda max is 15 minutes; use Step Functions for longer workflows.

**"Serverless always costs less"**
Costs scale with usage; must optimize architecture.

**"IaC is optional"**
Essential for reproducibility, disaster recovery, collaboration.

**"Monitoring adds expensive overhead"**
Monitoring costs ~5% of compute; prevents outage costs.

## 🏁 Your Complete Serverless Reference

This cheat sheet contains essential quick-reference information for building production serverless systems:

- **Development**: Lambda handlers, API responses, SDK examples
- **Architecture**: Service comparisons, AWS vs Azure, decisions
- **Optimization**: Cost formulas, pricing, performance tuning
- **Security**: Security checklist, IAM patterns
- **Troubleshooting**: Common failures, disaster recovery
- **Deployment**: Strategies, CI/CD patterns
- **Emergency**: Quick reference during incidents

Use it to:

- Design serverless systems efficiently
- Make informed architectural decisions
- Implement security best practices
- Optimize for cost and performance
- Monitor and observe production systems
- Deploy safely with CI/CD and IaC
- Troubleshoot issues quickly

Continue learning, stay current with AWS/Azure releases, and build amazing serverless applications.

---
*Source: adapted from the Serverless & Event-Driven Architecture cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
