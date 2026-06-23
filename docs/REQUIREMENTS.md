# REQUIREMENTS.md

## Overview

The sql-query-automator is a software product designed to automate SQL query generation and management for data engineers. This document outlines the functional and non-functional requirements for the project.

## Functional Requirements

### FR-1: Query Generation

* The system shall generate SQL queries based on user input, including but not limited to:
	+ Table and column names
	+ Filter conditions
	+ Sorting and grouping requirements
	+ Aggregation functions
* The generated queries shall be syntactically correct and follow best practices for SQL query writing.

### FR-2: Query Management

* The system shall allow users to manage and track generated queries, including:
	+ Query history
	+ Query execution results
	+ Query modification and deletion

### FR-3: AI-Powered Query Suggestions

* The system shall provide AI-powered query suggestions based on user input and query history.
* The suggestions shall be ranked and filtered to provide the most relevant and useful options.

### FR-4: Integration with Data Sources

* The system shall integrate with various data sources, including but not limited to:
	+ Relational databases (e.g. MySQL, PostgreSQL)
	+ NoSQL databases (e.g. MongoDB)
	+ Cloud-based data warehouses (e.g. Amazon Redshift)

### FR-5: User Interface

* The system shall provide a user-friendly interface for users to input queries, view query history, and access query suggestions.
* The interface shall be responsive and accessible on various devices and browsers.

## Non-Functional Requirements

### Perf-1: Performance

* The system shall respond to user input within 2 seconds.
* The system shall generate queries and execute them within 10 seconds for queries with up to 10 joins and 5 filter conditions.

### Sec-1: Security

* The system shall store user input and query history securely using encryption.
* The system shall authenticate and authorize users to access query history and execute queries.

### Rel-1: Reliability

* The system shall have a uptime of 99.99% or higher.
* The system shall recover from failures and errors within 1 minute.

## Constraints

* The system shall be built using open-source technologies and frameworks.
* The system shall be deployable on cloud-based platforms (e.g. AWS, GCP).

## Assumptions

* The system shall assume that users have basic knowledge of SQL and data engineering concepts.
* The system shall assume that users have access to data sources and necessary permissions to execute queries.

## Dependencies

* The system shall depend on the following external libraries and services:
	+ AI-powered query suggestion library (e.g. vLLM)
	+ Data source integration libraries (e.g. MySQL Connector)
	+ Encryption and authentication libraries (e.g. OpenSSL, OAuth)

## Acceptance Criteria

* The system shall meet all functional and non-functional requirements outlined in this document.
* The system shall be tested and validated using automated testing frameworks and manual testing.
* The system shall be deployed and maintained using continuous integration and continuous deployment (CI/CD) pipelines.
