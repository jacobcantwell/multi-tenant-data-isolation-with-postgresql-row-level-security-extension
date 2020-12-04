# Multi-tenant data isolation with PostgreSQL Row Level Security - Extension and Code Examples

[Status - Alpha Untested Code]

Customers using Aurora Postgres Serverless are interested in Row Level Security (RLS) as an isolation strategy for the multi-tenant Postgres data source.

This post add some code examples and shows integration with the Data API to the AWS blog post [Multi-tenant data isolation with PostgreSQL Row Level Security](https://aws.amazon.com/blogs/database/multi-tenant-data-isolation-with-postgresql-row-level-security/)

More information about multi-tenant data partitioning, see the following AWS SaaS Factory whitepaper. (page 22)

## Summary using Data API

When you run ExecuteStatement alone without a transaction block, that works like a transaction in itself. Read the [PostgreSQL autocommit behavior](https://dzone.com/articles/autocommit-in-postgresqls-psql). So when you issue the next statement, the DATA API initiates a new connection/transaction to PostgreSQL and the session variables are lost so RLS does not work.

If using the Aurora Postgres Serverless Data API, then the process is:

1. Start the transaction
2. Send the RLS SET statement
3. Send the actual query(-ies)
4. Close the transaction

## Code

Following is a sample code snippet when using the Data API.

### Enable Row Level Security (RLS) in Postgres

RLS has been enabled at Postgres as

```sql
ALTER TABLE product ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation_policy ON tenant USING (tenant_id = current_setting('app.current_tenant')::VARCHAR);
```

### Python Code

```python
rdsData = boto3.client('rds-data')

tr = rdsData.begin_transaction(
     resourceArn = cluster_arn, 
     secretArn = secret_arn, 
     database = 'onlineshop') 

# Setting the session variable with current tenant ID = tenant_3
rdsData.execute_statement(
     resourceArn = cluster_arn, 
     secretArn = secret_arn, 
     database = 'onlineshop', 
     sql = 'SET app.current_tenant = tenant_3', 
     transactionId = tr['transactionId']) 

# Execution of the CRUD operation
response = rdsData.execute_statement(
     resourceArn = cluster_arn, 
     secretArn = secret_arn, 
     database = 'onlineshop', 
     sql = 'select * from product', 
     transactionId = tr['transactionId']) 

rdsData.commit_transaction(
     resourceArn = cluster_arn, 
     secretArn = secret_arn, 
     transactionId = tr['transactionId']) 

# The records are scoped to tenant_3 only 
print(response['records']);
```

### TODO

* Worst case it is 4x the number of requests if this procedure is done for each individual query. If the queries can be batched, the negative impact of the extra calls can be reduced.

