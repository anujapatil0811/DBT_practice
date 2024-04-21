# DBT_practice

Fundamentals of DBT 
<br>
By Anuja 
dbt Cloud IDE
The dbt Cloud integrated development environment (IDE) is a single interface for building, testing, running, and version-controlling dbt projects from your browser. With the Cloud IDE, you can compile dbt code into SQL and run it against your database directly
 
Basic layout 
The IDE streamlines your workflow, and features a popular user interface layout with files and folders on the left, editor on the right, and command and console information at the bottom.
 

1. Git repository link — Clicking the Git repository link, located on the upper left of the IDE, takes you to your repository on the same active branch.
2. Documentation site button — Clicking the Documentation site book icon, located next to the Git repository link, leads to the dbt Documentation site. The site is powered by the latest dbt artifacts generated in the IDE using the dbt docs generate command from the Command bar. 
3. Version Control — The IDE's powerful Version Control section contains all git-related elements, including the Git actions button and the Changes section.
4. File Explorer — The File Explorer shows the filetree of your repository. You can: 
•	Click on any file in the filetree to open the file in the File Editor. 
•	Click and drag files between directories to move files. 
•	Right click a file to access the sub-menu options like duplicate file, copy file name, copy as ref, rename, delete. 
o	Note: To perform these actions, the user must not be in read-only mode, which generally happens when the user is viewing the default branch. 
•	Use file indicators, located to the right of your files or folder name, to see when changes or actions were made: 
o	Unsaved (•) — The IDE detects unsaved changes to your file/folder 
o	Modification (M) — The IDE detects a modification of existing files/folders 
o	Added (A) — The IDE detects added files 
o	Deleted (D) — The IDE detects deleted files

 

5.Command bar — The Command bar, located in the lower left of the IDE, is used to invoke dbt commands. When a command is invoked, the associated logs are shown in the Invocation History Drawer. 
6. IDE Status button — The IDE Status button, located on the lower right of the IDE, displays the current IDE status. If there is an error in the status or in the dbt code that stops the project from parsing, the button will turn red and display "Error". If there aren't any errors, the button will display a green "Ready" status. To access the IDE Status modal, simply click on this button.
Models
•	Models are .sql files that live in the models folder.
•	Models are simply written as select statements - there is no DDL/DML that needs to be written around this. This allows the developer to focus on the logic.
•	In the Cloud IDE, the Preview button will run this select statement against your data warehouse. The results shown here are equivalent to what this model will return once it is materialized.
•	After constructing a model, dbt run in the command line will actually materialize the models into the data warehouse. The default materialization is a view.
•	The materialization can be configured as a table with the following configuration block at the top of the model file:
{{ config(
materialized='table'
) }}
•	The same applies for configuring a model as a view:
{{ config(
materialized='view'
) }}
•	When dbt run is executing, dbt is wrapping the select statement in the correct DDL/DML to build that model as a table/view. If that model already exists in the data warehouse, dbt will automatically drop that table or view before building the new database object. *Note: If you are on BigQuery, you may need to run dbt run --full-refresh for this to take effect.
•	The DDL/DML that is being run to build each model can be viewed in the logs through the cloud interface or the target folder.
•	View is the default materialization of a model if you don't proactively configure the materialization for a model.
•	A given model called `events` is configured to be materialized as a view in dbt_project.yml and configured as a table in a config block at the top of the model. When you execute dbt run, dbt will build the `events` model as a table.
•	dbt run --select dim_customers+ attempt to only materialize dim_customers and its downstream models.
 
Modularity
•	We could build each of our final models in a single model as we did with dim_customers, however with dbt we can create our final data products using modularity.
•	Modularity is the degree to which a system's components may be separated and recombined, often with the benefit of flexibility and variety in use.
•	This allows us to build data artifacts in logical steps.
•	For example, we can stage the raw customers and orders data to shape it into what we want it to look like. Then we can build a model that references both of these to build the final dim_customers model.
•	Thinking modularly is how software engineers build applications. Models can be leveraged to apply this modular thinking to analytics engineering.
ref Macro
•	Models can be written to reference the underlying tables and views that were building the data warehouse (e.g. analytics.dbt_jsmith.stg_customers). This hard codes the table names and makes it difficult to share code between developers.
•	The ref function allows us to build dependencies between models in a flexible way that can be shared in a common code base. The ref function compiles to the name of the database object as it has been created on the most recent execution of dbt run in the particular development environment. This is determined by the environment configuration that was set up when the project was created.
•	Example: {{ ref('stg_customers') }} compiles to analytics.dbt_jsmith.stg_customers.
•	The ref function also builds a lineage graph like the one shown below. dbt is able to determine dependencies between models and takes those into account to build models in the correct order.
 
Modeling History
•	There have been multiple modeling paradigms since the advent of database technology. Many of these are classified as normalized modeling.
•	Normalized modeling techniques were designed when storage was expensive and computational power was not as affordable as it is today.
•	With a modern cloud-based data warehouse, we can approach analytics differently in an agile or ad hoc modeling technique. This is often referred to as denormalized modeling.
•	dbt can build your data warehouse into any of these schemas. dbt is a tool for how to build these rather than enforcing what to build.
Naming Conventions 
In working on this project, we established some conventions for naming our models.
•	Sources (src) refer to the raw table data that have been built in the warehouse through a loading process. (We will cover configuring Sources in the Sources module)
•	Staging (stg) refers to models that are built directly on top of sources. These have a one-to-one relationship with sources tables. These are used for very light transformations that shape the data into what you want it to be. These models are used to clean and standardize the data before transforming data downstream. Note: These are typically materialized as views.
•	Intermediate (int) refers to any models that exist between final fact and dimension tables. These should be built on staging models rather than directly on sources to leverage the data cleaning that was done in staging.
•	Fact (fct) refers to any data that represents something that occurred or is occurring. Examples include sessions, transactions, orders, stories, votes. These are typically skinny, long tables.
•	Dimension (dim) refers to data that represents a person, place or thing. Examples include customers, products, candidates, buildings, employees.
•	Note: The Fact and Dimension convention is based on previous normalized modeling techniques.
Reorganize Project
•	When dbt run is executed, dbt will automatically run every model in the models directory.
•	The subfolder structure within the models directory can be leveraged for organizing the project as the data team sees fit.
•	Subdirectories allow you to configure materializations at the folder level for a collection of models.
•	This can then be leveraged to select certain folders with dbt run and the model selector.
•	Example: If dbt run -s staging will run all models that exist in models/staging. (Note: This can also be applied for dbt test as well which will be covered later.)
•	The following framework can be a starting part for designing your own model organization:
•	Marts folder: All intermediate, fact, and dimension models can be stored here. Further subfolders can be used to separate data by business function (e.g. marketing, finance)
•	Staging folder: All staging models and source configurations can be stored here. Further subfolders can be used to separate data by data source (e.g. Stripe, Segment, Salesforce). (We will cover configuring Sources in the Sources module)
Sources
•	Sources represent the raw data that is loaded into the data warehouse.
•	We can reference tables in our models with an explicit table name ().raw.jaffle_shop.customers
•	However, setting up Sources in dbt and referring to them with the function enables a few important tools.source
o	Multiple tables from a single source can be configured in one place.
o	Sources are easily identified as green nodes in the Lineage Graph.
o	You can use to check the freshness of raw tables.dbt source freshness
Configuring sources
•	Sources are configured in YML files in the models directory.
•	The following code block configures the table and :raw.jaffle_shop.customersraw.jaffle_shop.orders
version: 2

sources:
  - name: jaffle_shop
    database: raw
    schema: jaffle_shop
    tables:
      - name: customers
      - name: orders
•	View the full documentation for configuring sources on the source properties page of the docs.
Source function
•	The function is used to build dependencies between models.ref
•	Similarly, the function is used to build the dependency of one model to a source.source
•	Given the source configuration above, the snippet in a model file will compile to .{{ source('jaffle_shop','customers') }}raw.jaffle_shop.customers
•	The Lineage Graph will represent the sources in green.
 
Source freshness
•	Freshness thresholds can be set in the YML file where sources are configured. For each table, the keys and must be configured.loaded_at_fieldfreshness
version: 2

sources:
  - name: jaffle_shop
    database: raw
    schema: jaffle_shop
    tables:
      - name: orders
        loaded_at_field: _etl_loaded_at
        freshness:
          warn_after: {count: 12, period: hour}
          error_after: {count: 24, period: hour}
•	A threshold can be configured for giving a warning and an error with the keys and .warn_aftererror_after
•	The freshness of sources can then be determined with the command .dbt source freshness

 
Sources
•	Sources represent the raw data that is loaded into the data warehouse.
•	We can reference tables in our models with an explicit table name ().raw.jaffle_shop.customers
•	By default, in .yml file in the models folder, will dbt look for source configurations
•	However, setting up Sources in dbt and referring to them with the function enables a few important tools.source
o	Multiple tables from a single source can be configured in one place.
o	You can visualize raw tables in your DAG. Sources are easily identified as green nodes in the Lineage Graph.
o	You can use to check the freshness of raw tables.dbt source freshness
o	You can document raw tables in your data warehouse
Configuring sources
•	Sources are configured in YML files in the models directory.
•	The following code block configures the table and :raw.jaffle_shop.customersraw.jaffle_shop.orders
version: 2

sources:
  - name: jaffle_shop
    database: raw
    schema: jaffle_shop
    tables:
      - name: customers
      - name: orders
•	View the full documentation for configuring sources on the source properties page of the docs.
Source function
•	The function is used to build dependencies between models.ref
•	Similarly, the function is used to build the dependency of one model to a source.source
•	Given the source configuration above, the snippet in a model file will compile to .{{ source('jaffle_shop','customers') }}raw.jaffle_shop.customers
•	The Lineage Graph will represent the sources in green.
 
Source freshness
•	Freshness thresholds can be set in the YML file where sources are configured. For each table, the keys and must be configured.loaded_at_fieldfreshness
version: 2

sources:
  - name: jaffle_shop
    database: raw
    schema: jaffle_shop
    tables:
      - name: orders
        loaded_at_field: _etl_loaded_at
        freshness:
          warn_after: {count: 12, period: hour}
          error_after: {count: 24, period: hour}
•	A threshold can be configured for giving a warning and an error with the keys and .warn_aftererror_after
•	The freshness of sources can then be determined with the command .dbt source freshness
Testing
•	Testing is used in software engineering to make sure that the code does what we expect it to.
•	In Analytics Engineering, testing allows us to make sure that the SQL transformations we write produce a model that meets our assertions.
•	In dbt, tests are written as select statements. These select statements are run against your materialized models to ensure they meet your assertions.
Tests in dbt
•	In dbt, there are two types of tests - generic tests and singular tests:
o	Generic tests are written in YAML and return the number of records that do not meet your assertions. These are run on specific columns in a model.
o	Singular tests are specific queries that you run against your models. These are run on the entire model.
•	dbt ships with four built in tests: unique, not null, accepted values, relationships.
o	Unique tests to see if every value in a column is unique
o	Not_null tests to see if every value in a column is not null
o	Accepted_values tests to make sure every value in a column is equal to a value in a provided list
o	Relationships tests to ensure that every value in a column exists in a column in another model (see: referential integrity)
•	Generic tests are configured in a YAML file, whereas singular tests are stored as select statements in the tests folder.
•	Tests can be run against your current project using a range of commands:
o	dbt test runs all tests in the dbt project
o	dbt test --select test_type:generic
o	dbt test --select test_type:singular
o	dbt test --select one_specific_model
•	Read more here in testing documentation.
•	In development, dbt Cloud will provide a visual for your test results. Each test produces a log that you can view to investigate the test results further.
 
In production, dbt Cloud can be scheduled to run dbt test. The ‘Run History’ tab provides a similar interface for viewing the test results.
 

command use to only run tests configured on a source named my_raw_data
dbt test --select source:my_raw_data

File type is used for specifying which generic tests to run by model and column: .yaml
In Tests folder should singular tests be saved in your dbt project
How do you associate a singular test with a particular dbt model or source
Using the ref or source macro in the SQL query in the singular test file
Query would use to create a singular test to assert that no record in cool_model has a value in Column A that is less than the value in Column B? 
select * from {{ ref( ‘cool_model’) }} where Column A < Column B
What is most likely true if a generic test on your sources fails? 
An assumption about your raw data is no longer true


Documentation
•	Documentation is essential for an analytics team to work effectively and efficiently. Strong documentation empowers users to self-service questions about data and enables new team members to on-board quickly.
•	Documentation often lags behind the code it is meant to describe. This can happen because documentation is a separate process from the coding itself that lives in another tool.
•	Therefore, documentation should be as automated as possible and happen as close as possible to the coding.
•	In dbt, models are built in SQL files. These models are documented in YML files that live in the same folder as the models.
•	dbt will automatically pull descriptions from your dbt project and metadata about your models and sources into the documentation site
Writing documentation and doc blocks
•	Documentation of models occurs in the YML files (where generic tests also live) inside the models directory. It is helpful to store the YML file in the same subfolder as the models you are documenting.
•	For models, descriptions can happen at the model, source, or column level.
•	If a longer form, more styled version of text would provide a strong description, doc blocks can be used to render markdown in the generated documentation.
Generating and viewing documentation
•	In the command line section, an updated version of documentation can be generated through the command dbt docs generate. This will refresh the `view docs` link in the top left corner of the Cloud IDE.
•	The generated documentation includes the following:
o	Lineage Graph
o	Model, source, and column descriptions
o	Generic tests added to a column
o	The underlying SQL code for each model
o	and more..
command will generate documentation for your project in development?
dbt docs generate
the correct way to reference the doc block called ‘orders’, contained in a file called docs_jaffle_shop.md
description: ‘{{ doc(“orders”) }}’


What aspects of the generated DAG can help you understand your data flow? 
Dependencies between models can help identify how code is used and possible redundancies
The selector can help narrow the scope of the shown DAG, which allows you to see how your specific model is used upstream and/or downstream.

Development vs. Deployment
•	Development in dbt is the process of building, refactoring, and organizing different files in your dbt project. This is done in a development environment using a development schema (dbt_jsmith) and typically on a non-default branch (i.e. feature/customers-model, fix/date-spine-issue). After making the appropriate changes, the development branch is merged to main/master so that those changes can be used in deployment.
•	Deployment in dbt (or running dbt in production) is the process of running dbt on a schedule in a deployment environment. The deployment environment will typically run from the default branch (i.e., main, master) and use a dedicated deployment schema (e.g., dbt_prod). The models built in deployment are then used to power dashboards, reporting, and other key business decision-making processes.
•	The use of development environments and branches makes it possible to continue to build your dbt project without affecting the models, tests, and documentation that are running in production.
Creating your Deployment Environment
•	A deployment environment can be configured in dbt Cloud on the Environments page.
•	General Settings: You can configure which dbt version you want to use and you have the option to specify a branch other than the default branch.
•	Data Warehouse Connection: You can set data warehouse specific configurations here. For example, you may choose to use a dedicated warehouse for your production runs in Snowflake.
•	Deployment Credentials: Here is where you enter the credentials dbt will use to access your data warehouse:
o	IMPORTANT: When deploying a real dbt Project, you should set up a separate data warehouse account for this run. This should not be the same account that you personally use in development.
o	IMPORTANT: The schema used in production should be different from anyone's development schema.
Scheduling a job in dbt Cloud
•	Scheduling of future jobs can be configured in dbt Cloud on the Jobs page.
•	You can select the deployment environment that you created before or a different environment if needed.
•	Commands: A single job can run multiple dbt commands. For example, you can run dbt run and dbt test back to back on a schedule. You don't need to configure these as separate jobs.
•	Triggers: This section is where the schedule can be set for the particular job.
•	After a job has been created, you can manually start the job by selecting Run Now
Reviewing Cloud Jobs
•	The results of a particular job run can be reviewed as the job completes and over time.
•	The logs for each command can be reviewed.
•	If documentation was generated, this can be viewed.
•	If dbt source freshness was run, the results can also be viewed at the end of a job

Resource	Description
models
Each model lives in a single file and contains logic that either transforms raw data into a dataset that is ready for analytics or, more often, is an intermediate step in such a transformation.
snapshots
A way to capture the state of your mutable tables so you can refer to it later.
seeds
CSV files with static data that you can load into your data platform with dbt.
data tests
SQL queries that you can write to test the models and resources in your project.
macros
Blocks of code that you can reuse multiple times.
docs
Docs for your project that you can build.
sources
A way to name and describe the data loaded into your warehouse by your Extract and Load tools.
exposures
A way to define and describe a downstream use of your project.
metrics
A way for you to define metrics for your project.
groups
Groups enable collaborative node organization in restricted collections.
analysis
A way to organize analytical SQL queries in your project such as the general ledger from your QuickBooks.
Feature	Description
Handle boilerplate code to materialize queries as relations	For each model you create, you can easily configure a materialization. A materialization represents a build strategy for your select query – the code behind a materialization is robust, boilerplate SQL that wraps your select query in a statement to create a new, or update an existing, relation. Read more about Materializations.

Use a code compiler	SQL files can contain Jinja, a lightweight templating language. Using Jinja in SQL provides a way to use control structures in your queries. For example, if statements and for loops. It also enables repeated SQL to be shared through macros. Read more about Macros.

Determine the order of model execution	Often, when transforming data, it makes sense to do so in a staged approach. dbt provides a mechanism to implement transformations in stages through the ref function. Rather than selecting from existing tables and views in your warehouse, you can select from another model.
Document your dbt project	dbt provides a mechanism to write, version-control, and share documentation for your dbt models. You can write descriptions (in plain text or markdown) for each model and field. In dbt Cloud, you can auto-generate the documentation when your dbt project runs. Read more about the Documentation.

Test your models	Tests provide a way to improve the integrity of the SQL in each model by making assertions about the results generated by a model. Read more about writing tests for your models Testing

Manage packages	dbt ships with a package manager, which allows analysts to use and publish both public and private repositories of dbt code which can then be referenced by others. Read more about Package Management.

Load seed files	Often in analytics, raw values need to be mapped to a more readable value (for example, converting a country-code to a country name) or enriched with static or infrequently changing data. These data sources, known as seed files, can be saved as a CSV file in your project and loaded into your data warehouse using the seed command. Read more about Seeds.

Snapshot data	Often, records in a data source are mutable, in that they change over time. This can be difficult to handle in analytics if you want to reconstruct historic values. dbt provides a mechanism to snapshot raw data for a point in time, through use of snapshots.


