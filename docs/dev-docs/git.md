# Git Connection

Integrating Git with DataTrovo allows you to execute your scripts directly from your Git repository. This is useful for both managing your scripts through version control and to integrate DataTrovo with your teams existing workflows.

Utilize [**synctatic sugar**](#Synctatic Sugar) to define your queries and let DataTrovo index the metadata allowing your team complete flexibility to make their scripts searchable, executable, and shareable with the rest of the team.

Utilize [**Docker**](#Docker) to create a custom execution environment for your scripts. This allows your team to define and manage dependencies and configuration for your scripts.

DataTrovo currently supports the following Git repositories:

1. GitHub

## Set up Database Connections
Steps to connect to your Git repository:

1. Navigate to `Settings` > `Secrets` > `+` to add all the required environment variables required by your scripts. **A symetric key is used for environment variable encryption.**
2. Navigate to `Settings` > `Connections` > `+` to add a new connection.
3. Select one of the connections from the list above.
4. Fill in the required connection fields. **A symetric key is used for PAT Key encryption.**
5. DataTrovo will test the connection and save the connection details.

## Synctatic Sugar

Synctatic sugar allows DataTrovo to index the metadata of your scripts. You can use this to populate the `Title`, `Description` and arbitrary `Tags` for your scripts.

The following comment structure is used for the respective languages:

- Python: `#>`
- R: `#>`
- SQL: `-->`

**Required** Metadata:

- `title` - The title of the script to be displayed in DataTrovo.
- `description` - A brief description of the script. Also to be displayed in DataTrovo.
- `run` - The command to run the script. A `{file_path}` variable is available to reference the script.
- `output` - The output file name of the script in the project's root directory.

**Optional** Metadata:

If your script requires any environment variables, you can define them in the script using the following syntax:

- `environment` - A list of environment variables required by the script.

**Other** Metadata:

Any other metadata provided other than the supported and reserved names above will be indexed as Tags. This is other metadata that is searchable and visible on the DataTrovo UI. This can be used to record authors, dates, columns, or any other metadata associated with the script.

**Examples**:

`.SQL` Example:

```
--> title: Top 100 facilities
--> description: Download the top 100 facilities
--> environment: [PGHOST, PGPORT, PGDATABASE, PGUSER, PGPASSWORD]
--> columns: [facility_name, state, county]
--> run: "sh -c 'psql -f {file_path} --csv > output.csv'"
--> output: "output.csv"
--> author: [John Doe]
```

`.R` Example:

```
#> title: R Top 100 facilities
#> description: Download the top 100 facilities using R
#> author: [R Master]
#> environment: [PGHOST, PGPORT, PGDATABASE, PGUSER, PGPASSWORD]
#> columns: [facility_name, state, county]
#> run: "Rscript {file_path}"
#> output: "facilities_data.xlsx"
```

## Docker

Every repository needs **one** Dockerfile in the root directory. This Dockerfile will be used to build the execution environment for your scripts. The Dockerfile should contain all the dependencies the the script `Run` command is executed within this context.

The only **requirement** for the dockerfile is to set the working directory to `/app`. This is because the script is volume mapped into the `/app` directory on container startup.

The dockerfile is rebuilt within DataTrovo whenever the dockerfile is updated. Allowing scripts to be changed without incuring the exepense of rebuilding the docker image. This is achieved through volume mapping the script into the container. However, this introduces a few limitations you need to be aware of when writing the dockerfile.

If package dependencies are installed within the root directory of the project, they will be lost. One example of this is the `renv` package manager for `R`. This is because the volume mapping will overwrite the dependencies within the script.

This limitation is resolved by building the dependencies within a subdirectory of the project. Finally copy the script into the project directly on container startup.

**Example Operations in the Dockerfile**:

```
# Set the working directory to /opt/renv, a temporary working directory
WORKDIR /opt/app

# Install packages from a lockfile
RUN Rscript -e "renv::restore()"

# Reset the working directory to /app
WORKDIR /app

# On container startup, copy the script into the working directory
ENTRYPOINT ["sh", "-c", "cp -r /opt/app/renv/library/ /app/renv/ && exec \"$@\"", "--"]
```
