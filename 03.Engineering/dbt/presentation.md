
dbt (data build tool) is an open-source data transformation tool that helps data analysts and engineers build and maintain reliable and scalable data pipelines, without worrying about execution orchestration. it focus on transformation, left el to emf.  It is a command-line tool that uses SQL and Jinja2 a templating language to transform data in modern data warehouses, such as Amazon Redshift, Google BigQuery, and Snowflake. an important point, dbt run is working on an existing dataset.

\<switch to page to show dbt sql model and normal sql statement>
 
Using Jinja turns your dbt project into a programming environment for SQL, giving you the ability to do things that aren't normally possible in SQL. For example, with Jinja you can:

- Use control structures (e.g. `if` statements and `for` loops) in SQL
- Use [environment variables](https://docs.getdbt.com/reference/dbt-jinja-functions/env_var) in your dbt project for production deployments
- Change the way your project builds based on the current target.
- Operate on the results of one query to generate another query, for example:
    - Return a list of payment methods, in order to create a subtotal column per payment method (pivot)
    - Return a list of columns in two relations, and select them in the same order to make it easier to union them together
- Abstract snippets of SQL into reusable [**macros**](https://docs.getdbt.com/docs/build/jinja-macros#macros) — these are analogous to functions in most programming languages.

**Benefits of using dbt**

There are many benefits to using dbt, including:
- Improved data quality and reliability: dbt's built-in testing and documentation features help to ensure that data transformations are accurate and reliable.
- Increased productivity and efficiency: dbt's modular codebase and Jinja2 templating features make it easy to reuse code and automate tasks, which can save data teams a lot of time and effort.
- Collaboration and communication: dbt's support for version control and Git makes it easy for data teams to collaborate on data transformation projects and communicate changes to stakeholders.

**order**
- introduction
	- features
	- open confluence page, to show diff between normal sql and dbt sql model
	- benefits
		- reusable snippets - macros
- local development
	- \<open page> setup
	- docs after success dbt run
	- show dependencies and data linearage
	- show tests
	- show code base
		- modular
	- show dbt run 
		- full run
		- select run
		- less e2e batch run
	- show dbt test
- emf integration
	- architecture changes
	- deployed folder at cfs bucket
	- emf command
- next steps
1. migrate the rest of hkma modules to dbt sql models as much as possible
2. setup jenkins pipeline to support the nightly build
3. prepare more tests, and integrate into nightly build pipeline
4. work with gft for common deployment pattern
5. work with gft for best practices, advance usage of dbt

**Challenge facing**
1. There are still some Sql files Which is hard to convert into sql models or even not able to. will work with gft to get it resolved
2. 