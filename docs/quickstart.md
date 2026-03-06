# Quick Start Guide

## Prerequisites

1. A Snowflake Workspace Notebook
2. An External Access Integration (EAI) configured for outbound network
   access. The toolkit can generate the required SQL for you.

## Step 1: Install the Toolkit

In a notebook cell:

```python
!pip install sfnb-multilang
```

## Step 2: Configure Network Access

The toolkit can attempt to create the EAI automatically. If your role has
`CREATE INTEGRATION` privilege:

```python
from sfnb_multilang import apply_eai
apply_eai(session, languages=["r", "scala"])
```

If you lack privileges, the installer will print the SQL and save it to
`eai_setup.sql` -- share this with your Snowflake administrator.

After the EAI is created, enable it in Snowsight:
**Notebook settings > External access > toggle on multilang_notebook_eai**

## Step 3: Install Languages

```python
from sfnb_multilang import install

# Install R and Scala
install(languages=["r", "scala"])

# Or install from a config file
install(config="config.yaml")
```

**Typical install times (fresh container):**

| Configuration | Time |
|---|---|
| R base + tidyverse | ~20s |
| R + ADBC Snowflake driver | ~2 min (Go compilation) |
| R + ADBC + DuckDB Snowflake extension | ~2.5 min |
| Subsequent runs (cached) | ~2s |

ADBC is the slowest step because it compiles the Go-based Snowflake
driver from source. DuckDB adds ~10s for extension downloads.

## Step 4: Use the Languages

### R

```python
from r_helpers import setup_r_environment
setup_r_environment()
```

Then in subsequent cells:

```r
%%R
library(dplyr)
mtcars %>% group_by(cyl) %>% summarise(mean_mpg = mean(mpg))
```

### Scala

```python
from scala_helpers import setup_scala_environment
setup_scala_environment()
```

```scala
%%scala
val x = 42
println(s"The answer is $x")
```

### Julia

```python
from julia_helpers import setup_julia_environment
setup_julia_environment()
```

```julia
%%julia
using DataFrames
df = DataFrame(x=1:10, y=rand(10))
describe(df)
```

## Using a YAML Config

Create a `config.yaml` (or use one of the presets in `configs/`):

```yaml
languages:
  r:
    enabled: true
    conda_packages:
      - r-tidyverse
      - r-dbplyr
    addons:
      adbc: true
  scala:
    enabled: true
    spark_connect:
      enabled: true
```

Then:

```python
from sfnb_multilang import install
install(config="config.yaml")
```

## CLI Usage

From a `!` shell cell or terminal:

```bash
sfnb-setup install --r --scala --verbose
sfnb-setup generate-eai --r --scala --account myaccount
sfnb-setup validate --config config.yaml
```
