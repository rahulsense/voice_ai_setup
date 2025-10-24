# Repo Set-Up
## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [First Time Setup](#first-time-setup)
  - [PyCharm Configuration](#pycharm-configuration-recommended)
  - [Database Setup](#database-setup)
- [Running the Application](#running-the-application)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Configuration](#configuration)

---

## Prerequisites

### Required Software

- **Homebrew** (macOS package manager)
- **Python 3.12.8** (via pyenv)
- **Poetry** (Python dependency management)
- **Docker Desktop** (for MySQL containers)
- **PyCharm** (recommended IDE)

### Check Homebrew Installation

```bash
brew --version
```

If Homebrew is not installed:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

---

## Installation

### First Time Setup

#### 1. Install pyenv (if not already installed)

```bash
brew install pyenv
```

Add pyenv to your shell configuration:

```bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

Restart your terminal or run:

```bash
source ~/.zshrc
```

#### 2. Install Python 3.12.8

```bash
pyenv install 3.12.8
pyenv global 3.12.8
```

Verify installation:

```bash
python --version  # Should output: Python 3.12.8
```

#### 3. Install Poetry

```bash
brew install poetry
```

Verify installation:

```bash
poetry --version
```

#### 4. Clone the Repository

```bash
git clone git@github.com:Spaced-Out/chatbot-v2.git
cd chatbot-v2
```

#### 5. Install Project Dependencies

```bash
cd services/voice_ai
poetry install
```

#### 6. Install Docker Desktop

Download and install [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/).

---

### PyCharm Configuration (Recommended)

#### 1. Open Project

1. Download and install [PyCharm](https://www.jetbrains.com/pycharm/)
2. Open the `chatbot-v2` directory as your project root

#### 2. Configure Source Roots

Mark the following directories as source roots:

1. Right-click on `chatbot-v2/common` → **Mark Directory as** → **Sources Root**
2. Right-click on `chatbot-v2/services/voice_ai` → **Mark Directory as** → **Sources Root**

#### 3. Set Up Python Interpreter

1. Open **Settings/Preferences** (`Cmd + ,` on macOS)
2. Navigate to **Project: chatbot-v2** → **Python Interpreter**
3. Click **Add Interpreter** → **Add Local Interpreter**
4. Choose the interpreter at: `~/.pyenv/versions/3.12.8/bin/python3`
5. Click **OK** to save

---

### Database Setup

#### 1. Install Sequel Ace (Optional but Recommended)

```bash
brew install --cask sequel-ace
```

#### 2. Start MySQL Development Server

From the project root (`chatbot-v2`):

```bash
bash ./ops/docker_run.sh chatbot_mysql
```

#### 3. Configure Database Connection

Open Sequel Ace and create a new connection:

| Field    | Value                                                      |
|----------|------------------------------------------------------------|
| Name     | Voice AI Dev                                               |
| Username | `root`                                                     |
| Password | Check `chatbot-v2/ops/docker_run.sh` search for (`MYSQL_ROOT_PASSWORD`) copy paste the password|
| Port     | `3501`                                                     |

#### 4. Create Required Databases

Connect to MySQL and run:

```sql
-- Create unit test database
CREATE DATABASE IF NOT EXISTS sense_unittest;

-- Grant privileges
GRANT ALL PRIVILEGES ON sense_unittest.* TO 'root'@'%' IDENTIFIED BY 'my-secret-pw';
FLUSH PRIVILEGES;
```

---

## Running the Application

### 1. Start MySQL Development Server

```bash
# From chatbot-v2 directory
bash ./ops/docker_run.sh chatbot_mysql
```

### 2. Configure Local Database Connection (if needed)

Update database settings in:

- `chatbot-v2/common/common/soframework/config/dev.py`

Example configuration:

```python
MYSQL_LEGACY_HOST = "localhost"
MYSQL_MULTIENTITY_HOST = "localhost"
```
- `chatbot-v2/common/common/chatbot/config/dev.py`

Example configuration:

```python
MYSQL_CHATBOT_HOST = "localhost"
```

### 3. Run the Development Server

**Using PyCharm**

1. Open `services/voice_ai/runserver.py`
2. Right-click and select **Run 'runserver'**


The server should now be running on `http://localhost:8000` (or configured port).

---

## Testing

### 1. Start MySQL Test Server

```bash
bash ./ops/docker_run.sh chatbot_mysql_test
```

### 2. Configure Test Database Connection

Update the following files:

**File:** `common/common/soframework/config/testing.py`

```python
MYSQL_LEGACY_HOST = "localhost"
MYSQL_MULTIENTITY_HOST = "localhost"
```

**File:** `common/common/chatbot/config/testing.py`

```python
MYSQL_CHATBOT_HOST = "localhost"
```

### 3. Install pytest (if not already installed)

```bash
brew install pytest
```

Or within the Poetry environment:

```bash
cd services/voice_ai
poetry add --group dev pytest
```

### 4. Run Unit Tests

**Option A: Using PyCharm**

1. Navigate to `services/voice_ai/voice_ai/tests`
2. Right-click on any test file or directory
3. Select **Run 'pytest in tests'**

**Option B: Using Terminal**

```bash
cd services/voice_ai
poetry run pytest
```

Run specific test file:

```bash
poetry run pytest voice_ai/tests/test_example.py
```

> **💡 Pro Tip:** Test-driven development (TDD) is highly recommended for this project. The repository structure makes it easy to write and run unit tests.

---

## Troubleshooting

### Issue: Command Not Found After Installing Brew or Poetry

**Symptom:** Commands like `brew` or `poetry` work in terminal but not in PyCharm.

**Solution:**

1. Open PyCharm **Settings** → **Tools** → **Terminal**
2. Update the shell path to include Homebrew and Poetry:

```bash
export PATH="/opt/homebrew/bin:/usr/local/bin:$PATH"
```

3. Restart PyCharm

---

### Issue: pytest Module Not Found During Unit Tests

**Symptom:** `ModuleNotFoundError: No module named 'pytest'`

**Solution:**

```bash
cd services/voice_ai
poetry add --group dev pytest
```

Or install globally:

```bash
brew install pytest
```

---
### Issue: Cannot Connect to MySQL

**Solution:**

1. Verify Docker containers are running:

```bash
docker ps
```

2. Check if MySQL ports are exposed:
   
   **Check port exposure:**
   ```bash
   # Check if ports are listening
   lsof -i :3306  # For dev environment
   lsof -i :3501 # For test environment
   ```
   
   Or using netstat:
   ```bash
   netstat -an | grep 3306
   netstat -an | grep 3501
   ```
   
   **If ports are not exposed:**
   
   a. Verify Docker container port mapping:
   ```bash
   docker ps --format "table {{.Names}}\t{{.Ports}}"
   ```
   
   b. If port mapping is missing, stop and restart the container with correct ports:
   ```bash
   docker stop <container_name>
   docker rm <container_name>
   bash ./ops/docker_run.sh chatbot_mysql
   ```
   
   c. Check for port conflicts (another service using the same port):
   ```bash
   lsof -i :3501
   ```
   
   If another process is using the port, either:
   - Stop that process: `kill -9 <PID>`


---

### Issue: Missing Database

**Symptom:** Database does not exist errors.

**Solution:**

Create the database manually using Sequel Ace or MySQL CLI:

```sql
CREATE DATABASE IF NOT EXISTS <database_name>;
```

---

## Configuration

### Database Connection Details

#### Development Environment

| Parameter | Value          |
|-----------|----------------|
| Host      | `localhost`    |
| User      | `root`         |
| Password  | `my-secret-pw` |
| Port      | `3306`         |

#### Test Environment

| Parameter | Value          |
|-----------|----------------|
| Host      | `localhost`    |
| User      | `root`         |
| Password  | `my-secret-pw` |
| Port      | `3501`        |

---

## Project Structure

```
chatbot-v2/
├── common/                  # Shared utilities and configs
│   ├── common/
│   │   ├── soframework/
│   │   └── chatbot/
├── services/
│   └── voice_ai/           # Voice AI service
│       ├── voice_ai/
│       │   └── tests/      # Unit tests
│       ├── runserver.py    # Development server
│       └── pyproject.toml  # Poetry dependencies
└── ops/
    └── docker_run.sh       # Docker startup scripts
```

---

## Additional Resources

- **Project Repository:** [Spaced-Out/chatbot-v2](https://github.com/Spaced-Out/chatbot-v2)
- **Poetry Documentation:** [python-poetry.org](https://python-poetry.org/)
- **Docker Documentation:** [docs.docker.com](https://docs.docker.com/)

---

## Support

If you encounter issues not covered in this guide:

1. Check the project's internal documentation
2. Reach out to the development team
