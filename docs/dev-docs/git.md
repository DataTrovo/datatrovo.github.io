# Git Connection

Integrating Git with DataTrovo allows you to execute your scripts directly from a Git repository. This is beneficial for managing scripts via version control and aligning DataTrovo with your team’s existing workflows.

Use **syntactic sugar** to define your queries and let DataTrovo index the metadata, giving your team the flexibility to make scripts searchable, executable, and shareable across the organization.

Leverage **Docker** to create a custom execution environment for your scripts. This allows your team to define and manage dependencies and configurations for any project.

Currently, DataTrovo supports the following Git repositories:

1. GitHub

## Set Up Git Connections

To connect to your Git repository in DataTrovo:

1. Go to **Settings** > **Secrets** > **+** to add all required environment variables for your scripts.  
   *A symmetric key is used to encrypt environment variables.*  
2. Go to **Settings** > **Connections** > **+** to create a new connection.  
3. Select one of the supported repository providers.  
4. Fill in the required connection details.  
   *A symmetric key is used to encrypt PAT keys.*  
5. DataTrovo will test and save the connection.

## Syntactic Sugar

“Syntactic sugar” in DataTrovo allows you to embed metadata in your scripts so the platform can automatically index it. This metadata can include:

The following comment structure is used for the respective languages:

- Python: `#>`
- R: `#>`
- SQL: `-->`

### Required Metadata
- **title** — The script’s title as displayed in DataTrovo.  
- **description** — A brief script description displayed in DataTrovo.  
- **run** — The command used to run the script. You can use `{file_path}` to reference the script.  
- **output** — The name of the output file generated in the project’s root directory.

### Optional Metadata
- **environment** — A list of environment variables that the script requires.

### Additional Metadata as Tags
Any additional metadata beyond the above required fields is indexed as tags. This can include authors, dates, columns, or any other information. These tags are searchable and visible in the DataTrovo UI.

### Examples

**.SQL Example**:

```sql
--> title: Top 100 facilities
--> description: Download the top 100 facilities
--> environment: [PGHOST, PGPORT, PGDATABASE, PGUSER, PGPASSWORD]
--> columns: [facility_name, state, county]
--> run: "sh -c 'psql -f {file_path} --csv > output.csv'"
--> output: "output.csv"
--> author: [John Doe]
```

**.R Example**:

```r
#> title: R Top 100 facilities
#> description: Download the top 100 facilities using R
#> author: [R Master]
#> environment: [PGHOST, PGPORT, PGDATABASE, PGUSER, PGPASSWORD]
#> columns: [facility_name, state, county]
#> run: "Rscript {file_path}"
#> output: "facilities_data.xlsx"
```

## Docker

Example Files:

- [SQL Example](https://github.com/DataTrovo/postgres-example/blob/main/Dockerfile)
- [R Example](https://github.com/DataTrovo/r-example/blob/main/Dockerfile)

Each repository should include **one** Dockerfile in the root directory. This Dockerfile is used to build the execution environment for your scripts. The primary requirement is that the working directory be set to `/app`, as DataTrovo mounts your scripts into `/app` when the container starts.

DataTrovo automatically rebuilds the Docker image whenever the Dockerfile changes, allowing you to modify scripts without incurring the overhead of a full image rebuild every time. This is achieved by volume-mounting the scripts into the container. However, this approach can cause certain limitations. For example, any dependencies installed in the project’s root directory will be overwritten when the scripts are volume-mounted.

This limitation can be avoided by installing dependencies in a temporary directory and copying the dependencies into the working directory at container startup.

**Example Operations in the Dockerfile**:

```
# Use a temporary directory for installing dependencies
WORKDIR /opt/app

# Install packages (e.g., using renv for R)
RUN Rscript -e "renv::restore()"

# Reset the working directory to /app
WORKDIR /app

# On container startup, copy the script into /app
ENTRYPOINT ["sh", "-c", "cp -r /opt/app/renv/library/ /app/renv/ && exec \"$@\"", "--"]
```

In this example, dependencies are installed in a subdirectory `(/opt/app)`, then the script is copied over at container startup, avoiding conflicts due to volume mounting.
