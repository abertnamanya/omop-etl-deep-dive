
## Prerequisites

### Installing Docker

<details>
<summary>mac OS</summary>


1. **Manual Installation:**
    - Download Docker Desktop from [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
    - Install and launch Docker Desktop
    - Ensure Docker is running (you should see the Docker icon in your menu bar)
2. Or ** Using Homebrew:**
   ```bash
   brew install --cask docker
   ```
   Then launch Docker Desktop from Applications.
</details>

<details>
<summary>Windows</summary>

1. Download Docker Desktop from [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Install and launch Docker Desktop
3. Ensure WSL 2 is enabled if prompted
</details>

<details>
<summary>Linux (Ubuntu/Debian)</summary>


```bash
# Update package index
sudo apt-get update

# Install prerequisites
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to docker group (optional, to avoid sudo)
sudo usermod -aG docker $USER
```

</details>

## Setting up the project

1. Download the repository (or clone it with git):
   https://github.com/jayasanka-sack/omop-etl-deep-dive/archive/refs/heads/main.zip
2. Build images:
    ```bash
    docker compose --profile manual build
   ```
3. Start the services
```bash
docker compose up
```

## Development


### Usagi Mappings (optional)

For this exercise you have provided a mapped concept file, so this step is optional. But in production, you should do the following.

1. Open **Usagi** and import the file:

   ```
   concepts/selected_concepts_1to1_updated.csv
   ```

2. Perform your mappings inside Usagi.
   When finished, go to **File > Save** (⚠️ do not use *Export*).

3. Save the file inside the `concepts/` folder and rename it to:

   ```
   mapping.csv
   ```

4. This file will be automatically imported into the MySQL database `raw` under the view `CONCEPT_MAPPING`.
   You can then reference it directly when writing SQL queries.

### Accessing the Database with cloudBeaver
If you want to explore the OMOP and openmrs databases using a graphical interface, you can use CloudBeaver. It is included in the current setup so you don't need to install it separately.

Make sure you run: `docker compose up`.

#### 1. Create an Admin Account

1. Open CloudBeaver in your browser:
   [http://localhost:8978](http://localhost:8978)

2. The **Setup Wizard** will appear the first time you run CloudBeaver.

3. Enter your desired **Admin username** (e.g., `cbadmin`).

4. Enter and confirm your **Admin password** (e.g., `Admin@123`).

5. Complete the wizard to create the admin account.

6. Use these credentials to log in to CloudBeaver.


#### 2. Create a PostgreSQL Connection


2. Open CloudBeaver in your browser:
   [http://localhost:8978](http://localhost:8978)

3. Log in with your **CloudBeaver admin credentials**.

4. From the top menu, click **New Connection**.
   <img src="/docs/img/initiate-new-connection.png">

5. In the connection type list, select **PostgreSQL**.

6. Fill in the connection details:

    * **Host**: `omop-db` 
    * **Port**: `5432`
    * **Database**: `omop`
    * **User name**: `omop`
    * **Password** `omop`:
6. (Optional) Click **Test Connection** to verify the details.

7. Click **Create** to save the connection.

#### 3. Create a MySQL Connection

1. From the top menu, click **New Connection**.

2. In the connection type list, select **MySQL**.

3. Fill in the connection details:

    * **Host**: `sqlmesh-db`
    * **Port**: `3306`
    * **Database**: *(leave this field empty)*
    * **User name**: `root`
    * **Password**: `openmrs`
7. Click **Create** to save the connection.


### **Creating/Updating SQL Models**

Models are SQL files that define the transformations and logic for your ETL process. They are located in the `core/models` directory of the project.
For more details, see the [SQLMesh Models Overview documentation](https://sqlmesh.readthedocs.io/en/stable/concepts/models/overview/).


#### **Applying Changes**

Once you have created or updated your SQL models, you need to apply these changes to the database.

##### Option 1: Full Pipeline (PostgreSQL)

This runs the entire ETL pipeline and creates all final tables in PostgreSQL:

```bash
docker compose run --rm core run-pipeline
```

---

##### Option 2: Quick Preview (MySQL)

If you only want to quickly check how your final tables will look, you can apply the plan to MySQL instead.
This is much faster and useful for previewing results before running the full pipeline:

```bash
docker compose run --rm core apply-sqlmesh-plan
```

* Results will appear under the **MySQL connection** (host: `sqlmesh-db`)
* Database name: `omop_db`
* The resulting tables will show up as **views**.

Once you’re satisfied with the structure in MySQL, you can run the full pipeline (Option 1) to generate the actual tables in PostgreSQL.


Once the commands complete, your OMOP postgress dayabase will be updated with the latest transformations defined in your SQL models.
You can access the database at `localhost:5433` with following credentials:
- **Database:** omop
- **User:** omop
- **Password:** omop
- **host:** localhost
- **Port:** 5433

### 🎉 _That's it! You now have a local OMOP CDM database populated with data from OpenMRS!!_



Once your work is done, you can stop the services with:

```bash
docker compose stop
```

### Fancy a User Interface?

```bash
    docker compose run --rm --service-ports core sqlmesh-ui
```

Access the UI at [http://localhost:8000](http://localhost:8000)
   
<img src="/docs/img/sql_mesh.jpeg" alt="SQLMesh UI"> width="600"/>

-- 

<summary>Fancy a Data Quality Check? (This will cover on upcoming weeks)</summary>

<details>

### 1. **Run Achilles to generate data summaries** (Check What Achilles does below.)
   ```
   docker compose run achilles
   ``` 
### 2. **Run DQD to perform data quality checks**
This runs the [OHDSI Data Quality Dashboard (DQD)](https://github.com/OHDSI/DataQualityDashboard) on the OMOP database.
   ```bash
    docker compose run --rm dqd run 
   ```
### 3. **View the Data Quality Dashboard**
      This serves the DQD results on a local web server. Once it's running, open your browser and go to [http://localhost:3000](http://localhost:3000).
   ```
   docker compose run --rm --service-ports dqd view
   ``` 

## 🧪 What does Achilles do?
Achilles analyzes the OMOP CDM data and generates summary statistics, data quality metrics, and precomputed reports. These results are essential for visualizations in tools like Atlas.

When you run:

```
docker compose run achilles
```
- ✅ It connects to your omop-db
- ✅ Scans and summarizes data in the public schema
- ✅ Produces results in the Achilles_results and Achilles_analysis tables
- ✅ Prepares your OMOP CDM for use with the web-based Atlas UI

</details>




<summary>Setting Up Git LFS for This Repository</summary>

<details>

### Setting Up Git LFS for This Repository

This repository uses **Git Large File Storage (LFS)** to handle large files like `CONCEPT.csv`. If you're cloning or pulling the repository, make sure to set up Git LFS to download the actual files instead of pointers.

### Step 1: Install Git LFS
Before cloning, install Git LFS:

- **macOS (Homebrew)**  
  ```sh
  brew install git-lfs
  ```

- **Linux (Ubuntu/Debian)**
  ```sh
  sudo apt update && sudo apt install git-lfs
  ```

- **Windows**  
  Download and install Git LFS from [Git LFS official site](https://git-lfs.github.com/).

### Step 2: Clone the Repository
After installing Git LFS, clone the repository:

```sh
git clone https://github.com/jayasanka-sack/openmrs-to-omop.git
cd openmrs-to-omop
```

Git LFS will automatically download the large files.

### Step 3: Pulling Updates
If you have already cloned the repository before installing Git LFS, or if you are pulling new changes, run:

```sh
git lfs install
git lfs pull
```

This ensures all large files are properly downloaded.

### Troubleshooting
If you see pointer files instead of actual data when opening a large file (e.g., `CONCEPT.csv`), it means Git LFS is not set up correctly. Run:

```sh
git lfs pull
```

For more information, refer to the [Git LFS documentation](https://git-lfs.github.com/).
</details>
