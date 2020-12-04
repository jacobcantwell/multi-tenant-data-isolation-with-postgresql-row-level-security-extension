# Multi-tenant data isolation with PostgreSQL Row Level Security - Extension and Code Examples

Customers using Aurora Postgres Serverless are interested in Row Level Security (RLS) as an isolation strategy for the multi-tenant Postgres data source.

This post add some code examples and shows integration with the Data API to the AWS blog post [Multi-tenant data isolation with PostgreSQL Row Level Security](https://aws.amazon.com/blogs/database/multi-tenant-data-isolation-with-postgresql-row-level-security/)

More information about multi-tenant data partitioning, see the following AWS SaaS Factory whitepaper. (page 22)

## Issues

When you run ExecuteStatement alone without a transaction block, that works like a transaction in itself. For reference you can read about Postgresql autocommit behavior here: https://dzone.com/articles/autocommit-in-postgresqls-psql .

So when you issue the next statement, DATA API initiates a new connection/transaction to PostgreSQL. That's why the session variables are lost and RLS doesn't work.

## Data API

If using the Aurora Postgres Serverless Data API, then the process is:

1. Start the transaction
2. Send the RLS SET statement
3. Send the actual query(-ies)
4. Close the transaction


