# **Setting Up a Private Ollama Environment in Barkla**

The provided script sets up a private Python environment that automatically starts an **Ollama server** whenever you activate it.
No root is required.

## **What it does**

When you run the script:

1. It creates a **Python virtual environment**
2. It downloads the **Ollama version specified in the script**.
3. It sets things up so that:

   * **Ollama server automatically starts** when you activate the environment.
   * **Ollama server automatically stops** when you deactivate the environment.
4. Models are stored in a user settable directory.

## **Before you start**

Avoid accidentally serving models in the login nodes:

* The `makevenv.sh` script only creates the venv, it is safe to run in login nodes.
* The venv location matters, preffer the `fastscratch` partition, it will be available to all compute nodes, `localscratch` may give issues.

No administrator/root permissions are needed.

## **Step 1: Run the script**

For reasonable defaults, move the script to the desired location and run:

```bash
bash makevenv.sh
```

This will create a default `ollama_env` directory.

To specify the models directory for ollama to use, or the version of ollama to use:

```bash
export OLLAMA_MODELS=/path/to/models/dir
export OLLAMA_VERSION=0.12.7
bash makevenv.sh
```

The script will:

* Tell you where it will create the environment
* Tell you which Ollama version it will install
* Ask if you want to continue

## **Step 2: Activating the venv**

**(Ideally near the start of your slurm script)**

To use the ollama in the venv, source the activation script (you may need to specify the full path to the ollama env):

```bash
source ollama_env/bin/activate
```

**On success**:

* Ollama server starts in the background using a random free port
* Ollama is added to PATH
* Confirmation should look like:
```
{"status": "ok", "host": "127.0.0.1:54321", "pid": "123456"}
```

## **Step 3: Use Ollama normally**

While the environment is active, you can run commands such as:

```bash
ollama pull llama3
ollama run llama3
```

Models you download will be stored inside the set directory.

## **Step 4: Deactivating the venv**

**(Ideally near the end of your slurm script)**

To stop the server and leave the environment:

```bash
deactivate
```

This automatically:

* Stops the Ollama server
* Restores your normal shell environment

## **Cleanup: Deleting everything**
If you no longer need the environment/ollama version, deletion can be as simple as:

```bash
rm -rf ollama_env
```

If custom paths were used for `$OLLAMA_PREFIX` or `$OLLAMA_MODELS`, these may be kept or deleted separately.

## **If something goes wrong**

Probable causes:

### **Running the script more than once**

Running the script multiple times may break the created venv or activation script.
The simplest fix is to delete the venv and create a new one.

### **"Port not available" errors**

Just deactivate and try again:

```bash
deactivate
source ollama_env/bin/activate
```

### **Ollama will not start**

Try removing old process files:

```bash
rm ollama_env/ollama_*/ollama.pid.*
```

Then reactivate.

## **Settable env vars**

The following environment variables can be exported before using `makevenv.sh` to modify the venv creation:

* ENV_PREFIX (The path to the location were the prefix will be created, default: ./ollama_env)
* PYTHON (The python version that will be used by the prefix, default: python3)
* OLLAMA_VERSION (The version of ollama to be used, default: 0.12.11)
* OLLAMA_PREFIX (The path to the location were ollama will be installed, default: ./ollama_env/ollama_0.12.11)
* OLLAMA_MODELS (The path were ollama will download/check for models, default: ./ollama_env/models)

The following environment variables can be exported beforing sourcing the activate script:

* OLLAMA_PREFIX (The path to the location were ollama will be installed, default: ./ollama_env/ollama_0.12.11)
* OLLAMA_MODELS (The path were ollama will download/check for models, default: ./ollama_env/models)

