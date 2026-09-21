# TIL: Running Databricks Notebooks from VS Code

**Date:** September 21, 2026

**Purpose:** A guide on setting up VS Code so you can connect to Databricks, run a notebook, and see the output directly in VS Code.

This is based on what I actually did and the issues I encountered while setting it up.

---

## What are we trying to do?

The goal is to use **VS Code as the interface for working with a Databricks notebook**.

Instead of opening the notebook only in the Databricks workspace, we want to be able to:

1. Connect VS Code to our Databricks workspace.
2. Open a Databricks notebook from our repository.
3. Connect the notebook to Databricks.
4. Run a cell from VS Code.
5. See the output directly in VS Code.

<img width="1917" height="1078" alt="image" src="https://github.com/user-attachments/assets/6a7d1522-17ed-4f7a-b79f-2defad6e15f6" />


There are a few different things involved, so the setup can feel confusing at first.

---

# Step 1: Install the required VS Code extensions

Open VS Code and go to the **Extensions** panel.

Install these three extensions:

### 1. Databricks: IDE support for Databricks

<img width="1245" height="437" alt="image" src="https://github.com/user-attachments/assets/cf1d1d46-18d1-402a-a869-60aae6875d27" />

This is what allows VS Code to connect to and work with your Databricks workspace.

### 2. Databricks Driver for SQL Tools

<img width="1251" height="552" alt="image" src="https://github.com/user-attachments/assets/7d6de054-1d7c-42d1-b632-433673d33b60" />

We will use this later to create a SQL connection to Databricks.

### 3. GitHub Actions

<img width="1255" height="405" alt="image" src="https://github.com/user-attachments/assets/3fe92abe-fd1a-426b-a132-34a762fe3e2a" />

This is useful if your project uses GitHub Actions for CI/CD.

---

# Step 2: Open the Databricks extension

After installing the extensions, look at the icons on the left side of VS Code.

Click the **Databricks extension icon**.

From here, we can create a Databricks configuration.

Click:

> **Configuration → Create New Configuration**

---

# Step 3: Connect VS Code to your Databricks workspace

When creating the configuration, enter your Databricks workspace information.

Paste your **Databricks workspace URL** when prompted.

Then authenticate with your Databricks account and authorize the connection.

After authentication, VS Code should be able to communicate with your Databricks workspace.

---

# Step 4: Select the Databricks cluster and environment

Once the workspace is connected, select the appropriate:

* Cluster
* Environment

For our project, the environment is related to the `dev` target.

At this point, I encountered an error.

## Problem: `dev` target shows an error

The Databricks extension showed an error for the `dev` target.

The problem was not with the authentication itself.

The issue was in my `databricks.yml` file.

My `dev` target did not have the Databricks workspace host configured.

---

# Step 5: Fix the `databricks.yml`

Open the project's:

```text
databricks.yml
```

Look for the `targets` section.

For example:

```yaml
targets:
  dev:
    mode: development
    workspace:
      root_path: /Workspace/Users/...
```

The `host` needs to be specified under `workspace`.

It should look like:

```yaml
targets:
  dev:
    mode: development
    workspace:
      host: https://<your-databricks-workspace-url>
      root_path: /Workspace/Users/...
```

The important part is:

```yaml
workspace:
  host: https://<your-databricks-workspace-url>
```

So the structure is:

```text
targets
└── dev
    └── workspace
        ├── host
        └── root_path
```

### Why did this matter?

The Databricks extension needs to know **which Databricks workspace the `dev` target belongs to**.

Without the `host`, the `dev` target could not be properly resolved.

After adding the host, the target configuration worked.

---

# Step 6: Access the Databricks workspace from VS Code

Go back to the **Databricks extension**.

You should now be able to access the Databricks workspace through VS Code.

At this point, however, there is an important distinction:

> Being connected to the Databricks workspace does not automatically mean that the notebook is ready to execute SQL.

I could access Databricks, but I still needed to configure a SQL connection.

---

# Step 7: Open SQL Tools

Go to the **SQL Tools** extension.

Create a new connection.

Choose the Databricks driver:

> **Databricks Driver for SQL Tools**

Create a connection and give it a name.

For example:

```text
Databricks Dev
```

You can use any name that makes sense to you.

---

# Step 8: Get the Databricks Server Hostname and HTTP Path

Go to your Databricks workspace.

Open your SQL environment and use the:

> **Serverless Starter Warehouse**

Click **Connection**.

You will need two pieces of information:

### Server Hostname

Copy the server hostname and put it into the **Host** field in VS Code.

### HTTP Path

Copy the HTTP path and put it into the **HTTP Path** field.

The important distinction is:

```text
Host
HTTP Path
```

These are two separate pieces of connection information.

---

# Step 9: Create an access token

For the authentication, create a Databricks personal access token.

In Databricks:

1. Click your profile.
2. Go to **Developer**.
3. Open **Manage Access Tokens**.
4. Click **Generate New Token**.

You will be asked for information such as:

### Comment / Token description

Enter something that tells you what the token is for.

For example:

```text
VS Code SQL Tools
```

### Lifetime

Choose how many days the token should remain valid.

Use a reasonable expiration period based on your project's requirements.

### Scope

For my setup, I selected:

```text
Other APIs
```

Then I selected:

```text
All APIs
```

Generate the token.

**Important:** Copy the token immediately and store it securely. Do not commit it to GitHub or put it directly inside your repository files.

---

# Step 10: Put the credentials into the SQL Tools connection

Return to the SQL Tools connection configuration.

You should now have the information needed:

```text
Connection Name
Host
HTTP Path
Token
Catalog
Schema
```

Fill them in.

For example:

```text
Host: <server hostname>
HTTP Path: <HTTP path>
Token: <your access token>
Catalog: <your catalog>
Schema: leave blank
```

For my setup, I entered the name of the catalog but did **not** specify a schema.

---

# Step 11: Test the connection

Before trying to run the notebook, test the SQL connection.

Click:

> **Test Connection**

If everything is configured correctly, you should get a successful connection.

Then save the connection.

---

# Step 12: Run the notebook

Now that the Databricks extension and SQL Tools connection are configured, open the notebook in VS Code.

The notebook should now be able to connect to Databricks.

You can run a cell from VS Code and the execution happens through the Databricks environment.

The result is then displayed directly in VS Code.

Conceptually:

```text
VS Code
   │
   │ Databricks Extension
   ▼
Databricks Workspace
   │
   │ SQL Tools Connection
   ▼
Databricks SQL Warehouse
   │
   ▼
Notebook Cell
   │
   ▼
Output shown in VS Code
```

---

# Troubleshooting: What confused me

## "I can access Databricks, but I can't run my notebook."

There are multiple connections involved.

The Databricks extension lets VS Code communicate with your Databricks workspace.

The SQL Tools connection provides the SQL connection information needed to execute SQL against Databricks.

So make sure both are configured.

---

## "`dev` target has an error."

Check your `databricks.yml`.

Make sure the `dev` target has a workspace host:

```yaml
targets:
  dev:
    workspace:
      host: https://<your-workspace-url>
      root_path: /Workspace/Users/...
```

---

## "Where do I get the Host and HTTP Path?"

In Databricks, open the SQL warehouse:

> **Serverless Starter Warehouse → Connection**

Copy the corresponding values.

---

## "Where do I get the token?"

Go to:

> **Profile → Developer → Manage Access Tokens → Generate New Token**

Do not put the token in your Git repository.

---

# What I learned

At first, I thought that connecting the Databricks extension to my workspace would be enough to run everything from VS Code.

It wasn't.

There are different pieces involved:

```text
Databricks Extension
        │
        ├── Workspace connection
        │
        └── Databricks project / target configuration

SQL Tools
        │
        ├── Host
        ├── HTTP Path
        ├── Token
        └── Catalog / Schema
```

The biggest issue I encountered was the missing `host` in the `dev` target of `databricks.yml`.

Once that was fixed and the SQL Tools connection was configured, I was able to run the notebook and see the output in VS Code.

---

# Quick Setup Checklist

Before troubleshooting, check these one by one:

```text
[ ] Databricks: IDE support for Databricks installed
[ ] Databricks Driver for SQL Tools installed
[ ] GitHub Actions extension installed

[ ] Databricks workspace authenticated
[ ] Databricks configuration created
[ ] Cluster/environment selected

[ ] databricks.yml has a dev target
[ ] dev.workspace.host is configured

[ ] SQL Tools connection created
[ ] Correct Databricks host entered
[ ] Correct HTTP path entered
[ ] Access token generated
[ ] Token entered securely
[ ] Catalog entered
[ ] Schema configured if needed

[ ] Connection test successful
[ ] Connection saved

[ ] Notebook opened in VS Code
[ ] Notebook connected to Databricks
[ ] Cell runs successfully
[ ] Output appears in VS Code
```

## The main idea

You are essentially setting up two things:

**1. Databricks Extension → connects VS Code to the Databricks workspace and project.**

**2. SQL Tools → provides the SQL connection to Databricks so the notebook can execute and return results.**

Once both are properly configured, VS Code can become your working interface while Databricks handles the actual execution.
