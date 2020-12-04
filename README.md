# Multi-tenant data isolation with PostgreSQL Row Level Security - Extension and Code Examples

This post add some code examples and shows integration with the Data API to the AWS blog post [Multi-tenant data isolation with PostgreSQL Row Level Security](https://aws.amazon.com/blogs/database/multi-tenant-data-isolation-with-postgresql-row-level-security/)

## Data API

If using the Aurora Postgres Serverless Data API, then the process is:

* Start the transaction
* Send the RLS SET statement
* Send the actual query(-ies)
* Close the transaction


