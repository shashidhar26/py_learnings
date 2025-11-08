Here we collect some general notes on python learnings. These are specific use cases for specific tasks. However, they are not big enough to be grouped into a folder of their own. Hence, they will be noted down here. If at a later point of time, they get big enough to be made into a specific folder, it will be moved. 

# Connections to Databases
## Snowflake connection

There are two ways to connect to snowflake db using python:
- using ```snowflake-connector-python``` package and directly making the connection
- using ```snowflake-sqlalchemy``` package to make the connection using sqlalchemy

### Using snowflake connector: 
In most cases, the purpose of connecting to the database would be to execute a query on the database and load the results on to a pandas dataframe. For such cases, the recommended approach is to use the ```snowflake-connector``` :

Make sure you install the snowflake-connector package with the extras for pandas: 
```
pip install "snowflake-connector-python[pandas]"
```

Make the connection: 
```
import snowflake.connector
con = snowflake.connector.connect(
    user='XXXX',
    password='XXXX',
    account='XXXX',
    session_parameters={
        'QUERY_TAG': 'EndOfMonthFinancials',
    }
)
```
The advantage with this is that it lets you set session_parameters which is not available with the sql-alchemy connection. Also, as long as the use-case is to retrieve data from the db into a pandas df, this can  be the preferred approach. 

Run the query and retrive into pandas: 
```
# Create a cursor object.
cur = con.cursor()

# Execute a statement that will generate a result set.
sql = "select * from t"
cur.execute(sql)

# Fetch the result set from the cursor and deliver it as the Pandas DataFrame.
df = cur.fetch_pandas_all()
```

### Using sql-alchemy: 
If you are using sql-alchemy, then this is the way to make the connection: 
```
from snowflake.sqlalchemy import URL
from sqlalchemy import create_engine

engine = create_engine(URL(
    account = 'myorganization-myaccount',
    user = 'testuser1',
    password = '0123456',
    database = 'testdb',
    schema = 'public',
    warehouse = 'testwh',
    role='myrole',
))

connection = engine.connect()
try:
    connection.execute(<SQL>)
finally:
    connection.close()
    engine.dispose()
```

In our case, we did not choose to go ahead with this approach as the first approach seemed very much sufficient. All this is clearly documented in the below link: <br>
[snowflake documentation](https://docs.snowflake.com/en/developer-guide/python-connector/python-connector-install)


# Modules and script execution
To execute a python file, the straightforward command is :
```
python path/to/file.py
```

This seems simple, but can easily get confusing when there is a project and code is organized into folders. Let us consider a project structure like this: 

```
proj_root
├── myfolder1
│   ├── __init__.py
│   └── module1.py
├── src
│   ├── __init__.py
│   └── run.py
└── __init__.py
```

Assume that `run.py` is the main script and it contains a import from `myfolder1` something like this:
```
from myfolder1.module1 import module1
```

If we run the `run.py` from the `proj_root`, the details will be:
```
# command to execute:
python src\run.py

# sys.path will contain: 
proj_root\src

```
this command will fail because the `proj_root` would not be in the sys.path and there will be an 'Module Not Found' error. This has been explained in the SO issue [here](https://stackoverflow.com/questions/49095869/how-can-i-run-a-python-3-script-with-imports-from-a-subfolder)

The solution which works better instead of the `sys.path.append` is the following:

```
# Run the script as: 
python -m src.run 

# Note the use of the '-m' switch and dots to refer to subfolders
```

# Jupyter notebooks with conda

To run jupyter notebooks with conda, you need to install the jupyter notebook extension (or `jupyterlab` extension). It is recommended to install these extensions in the `base` environment in conda. 

Once this is done, you can add as many conda environments to this. 

```
# first create the conda envs: 
(base)$ conda create env --name myenv

# then activate the new env:
(base)$ conda activate myenv
```

To display the new env `myenv` in the jupyterlab notebooks, you need to install ipykernel in the env and add this kernel to the list of jupyter's known kernels. Like this: 


```
(myenv)$ pip install ipykernel
(myenv)$ python -m ipykernel install --user --name myenv --display-name "Python (myenv)"
```
Once this is done, you will be able to see the new env in the jupyterlab notebooks. You can check this on command line by swtiching back to the base env. 

```
(myenv)$ conda deactivate
(base)$ jupyter kernelspec list 
# Should display the following: 
Available kernels:
  myenv    C:\Users\un\AppData\Roaming\jupyter\kernels\myenv
  python3     C:\Users\un\AppData\Roaming\jupyter\kernels\python3

```

# Environment management using uv (instead of conda and pip)
The package manager `uv` is used to setup the `.venv` directly in the repo root. If you dont have `uv` installed, you can get it by using: 
```
pip install uv
```
uv depends on having a `pyproject.toml` file on the project root with all dependencies. Hence, there is no need to have a separate `requirements.txt` file anymore. Unless you choose to move back to conda/pip, you can choose only keep `pyproject.toml` file in the root, eliminating the need for separate `environment.yml` (for conda) and `requirements.txt` (pip). 

The initial template for the `pyproject.toml` might look like this: 
```
[project]
name = "my_project"
version = "0.1.0"
description = "my_project to do my work"
readme = "README.md"
requires-python = "==3.13"
dependencies = [
    "jinja2",
    "snowflake-connector-python"
]
```

Once done, you can simply setup the environment by using the following: <br>
`uv sync`

And then activate the environment(which gets installed in the `.venv` folder of the project root) like this:<br>
`.venv\scripts\activate`
