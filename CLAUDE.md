# CLAUDE.md - aaPanel Development Guide for AI Assistants

## Project Overview

**aaPanel** is a simple but powerful web-based hosting control panel for Linux servers. It provides a GUI to manage web servers, databases, websites, FTP, SSL certificates, and other server components.

- **Primary Language**: Python 3.12
- **Web Framework**: Flask 2.2.5 with Gevent (WSGI server)
- **Frontend**: HTML/CSS/JavaScript (CKEditor, jQuery)
- **Database**: SQLite (via Peewee ORM), MySQL/MariaDB support
- **Target OS**: Ubuntu 22+/24+, Debian 11/12, CentOS 9, Rocky/AlmaLinux 8/9
- **License**: Custom (Copyright aaPanel 2015-2099)

### Key Features
- One-click LNMP/LAMP stack installation
- Website management with SSL support
- Database management (MySQL, PostgreSQL, MongoDB)
- Docker integration
- Firewall management
- Backup and restore functionality
- Plugin system
- Task scheduling (cron)

## Architecture Overview

### High-Level Structure

```
aaPanel/
├── BTPanel/              # Main Flask application
│   ├── __init__.py      # Flask app initialization
│   ├── app.py           # Application logic
│   ├── routes/          # API routes (v1.py, v2.py)
│   ├── templates/       # Jinja2 templates
│   ├── static/          # Static assets (CSS, JS, images)
│   └── languages/       # Internationalization files
├── class/               # Core business logic (v1)
│   ├── public/          # Shared utilities
│   ├── panelModel/      # Panel models
│   ├── databaseModel/   # Database management
│   ├── firewallModel/   # Firewall management
│   ├── sslModel/        # SSL certificate management
│   ├── safeModel/       # Security features
│   └── [other models]   # Various domain models
├── class_v2/            # Refactored business logic (v2)
│   └── [similar structure to class/]
├── mod/                 # Modular components
│   ├── base/            # Base modules
│   ├── common/          # Common utilities
│   ├── project/         # Project-specific logic
│   └── test/            # Unit tests
├── config/              # Configuration files
│   ├── config.json      # Main config
│   ├── menu.json        # Menu structure
│   ├── databases.json   # Database configs
│   └── [other configs]  # Various JSON configs
├── data/                # Runtime data
├── script/              # Shell scripts for automation
├── install/             # Installation scripts
├── rewrite/             # Web server rewrite rules
│   ├── nginx/           # Nginx rewrite rules
│   └── apache/          # Apache rewrite rules
├── vhost/               # Virtual host configurations
│   ├── nginx/           # Nginx vhost configs
│   └── apache/          # Apache vhost configs
├── webserver/           # Web server management
├── logs/                # Log files
├── ssl/                 # SSL certificates
├── BT-Panel             # Panel daemon (executable)
├── BT-Task              # Background task daemon
├── task.py              # Task scheduler
├── tools.py             # CLI tools
├── runserver.py         # Development server
├── requirements.txt     # Python dependencies
└── install.sh           # Main installation script
```

### Application Flow

1. **Entry Point**: `BT-Panel` (production) or `runserver.py` (development)
2. **Panel Initialization**: `BTPanel/__init__.py` initializes Flask app, session, routes
3. **Request Handling**: Routes defined in `BTPanel/routes/v1.py` and `v2.py`
4. **Business Logic**: Executed in `class/` or `class_v2/` modules
5. **Background Tasks**: Managed by `task.py` via `BT-Task` daemon

## Key Technologies & Dependencies

### Core Python Packages
- **Flask 2.2.5**: Web framework
- **Gevent 24.2.1**: Async WSGI server
- **Peewee 3.16.2**: ORM for SQLite
- **PyMySQL 1.0.3**: MySQL client
- **psutil 5.9.5**: System monitoring
- **Paramiko 3.4.0**: SSH client
- **Docker 6.0.1**: Docker integration
- **Requests 2.31.0**: HTTP client
- **cryptography 40.0.2**: Encryption/SSL
- **bcrypt 4.0.1**: Password hashing
- **Pillow 10.3.0**: Image processing
- **pandas 2.2.2**: Data analysis

### Cloud Storage Support
- Alibaba Cloud OSS (oss2, aliyun-python-sdk)
- Tencent Cloud COS (cos-python-sdk-v5)
- Google Cloud Storage (google-cloud-storage)
- AWS S3 (boto3)
- Qiniu Cloud (qiniu)
- Upyun (upyun)

### Frontend
- CKEditor (rich text editing)
- jQuery
- Bootstrap (implied from templates)

## Development Workflows

### Starting the Panel

**Production Mode:**
```bash
chmod +x /www/server/panel/BT-Panel
/www/server/panel/BT-Panel
```

**Development Mode (with auto-reload):**
```bash
# Create debug flag
touch /www/server/panel/data/debug.pl

# Run panel
python3 /www/server/panel/BT-Panel
```

The debug mode uses `pyinotify` to watch for file changes and auto-reloads the panel.

**Simple Development Server:**
```bash
python3 runserver.py
```

### Background Task System

The panel runs a separate daemon for background tasks:

```bash
chmod +x /www/server/panel/BT-Task
/www/server/panel/BT-Task
```

Tasks are defined in `task.py` and include:
- Scheduled backups
- Log rotation
- SSL certificate renewal
- System monitoring
- Database maintenance
- Email notifications

### Multi-Process Architecture

For high-performance deployments, the panel supports multi-process mode:
- Enabled via `/www/server/panel/data/is_process.pl`
- Dynamically spawns worker processes based on CPU load
- Process count configurable in `data/process_count.pl`
- Default calculation: Based on RAM and CPU cores

## Coding Conventions

### Python Style
- **Encoding**: UTF-8 (always include `#coding: utf-8` header)
- **Indentation**: Spaces (4 spaces per level)
- **Naming**:
  - Functions: `snake_case`
  - Classes: `PascalCase`
  - Constants: `UPPER_CASE`
- **Python Version**: Python 3.12+ (via `/www/server/panel/pyenv/bin/python`)

### File Headers
All Python files include this header:
```python
#coding: utf-8
# +-------------------------------------------------------------------
# | aaPanel
# +-------------------------------------------------------------------
# | Copyright (c) 2015-2099 aaPanel(www.aapanel.com) All rights reserved.
# +-------------------------------------------------------------------
# | Author: hwliang <hwl@aapanel.com>
# +-------------------------------------------------------------------
```

### Import Conventions
```python
# Hook import system first (for plugin loading)
from public.hook_import import hook_import
hook_import()

# Standard library imports
import sys
import os
import time

# Third-party imports
import public
from flask import Flask, request, session

# Local imports
from BTPanel import app
```

### Path Management
- **Panel Path**: Always use `/www/server/panel` as base
- **Change Directory**: `os.chdir('/www/server/panel')` in entry points
- **Sys Path**: Add `class/` and `class_v2/` to `sys.path`

### Configuration Files
- Stored in `config/` and `data/` directories
- Format: JSON (`.json`) or plain text (`.pl` extension)
- Read using `public.readFile()` and `public.writeFile()`

## Key Directories & Files Explained

### BTPanel/
Main Flask application directory.

- **`__init__.py`**: Flask app initialization, session config, SSL setup
- **`app.py`**: Application configuration and middleware
- **`routes/v1.py`**: Legacy API routes
- **`routes/v2.py`**: New API routes (refactored)
- **`routes/flask_hook.py`**: Request/response hooks (auth, CSRF protection)

### class/ vs class_v2/
- **`class/`**: Original business logic (still in use)
- **`class_v2/`**: Refactored/improved versions
- Both are maintained for backward compatibility

Common modules:
- **`public/`**: Shared utilities (`public.py` is the main utility module)
- **`panelModel/`**: Panel settings, user management
- **`databaseModel/`**: Database operations
- **`firewallModel/`**: Firewall rules, IP filtering
- **`sslModel/`**: SSL certificate management (Let's Encrypt, etc.)
- **`safeModel/`**: Security features, auditing
- **`projectModel/`**: Project/website management

### config/
Configuration files in JSON format:
- **`config.json`**: Main panel configuration
- **`menu.json`**: Admin menu structure
- **`databases.json`**: Supported database versions
- **`php_versions.json`**: Supported PHP versions
- **`crontab.json`**: Scheduled task definitions
- **`task.json`**: Task configuration

### data/
Runtime data and state files:
- **`port.pl`**: Panel listening port (default: 7800)
- **`ssl.pl`**: Flag for SSL mode
- **`debug.pl`**: Flag for debug mode
- **`is_process.pl`**: Flag for multi-process mode
- **`ipv6.pl`**: Flag for IPv6 support
- **`process_count.pl`**: Custom process count

### script/
Shell scripts for automation:
- **`backup`**: Backup scripts
- **`logsBackup`**: Log backup
- **`install.sh`**: Installation script
- **`upgrade_*.sh`**: Upgrade scripts for dependencies
- **`init_firewall.sh`**: Firewall initialization
- **Various `.py` scripts**: Python automation tasks

### Important Files

**`BT-Panel`** (Shebang: `#!/www/server/panel/pyenv/bin/python`)
- Main production entry point
- Forks daemon process
- Sets up SSL context
- Initializes multi-process workers
- Starts background task monitor

**`task.py`**
- Background task scheduler
- Handles cron jobs
- System monitoring
- Automatic maintenance tasks

**`tools.py`**
- CLI utility functions
- Database password reset
- System diagnostics
- Administrative tasks

**`public.py`** (in `class/public/`)
- Core utility library
- File operations: `readFile()`, `writeFile()`, `ExecShell()`
- System info: `get_glibc_version()`, `get_python_bin()`
- Security: `md5()`, encryption/decryption
- Web requests: HTTP client wrappers

## API Routes

### Route Structure
Routes are registered in `BTPanel/routes/v1.py` and `v2.py` using Flask decorators:

```python
@app.route('/api/endpoint', methods=['GET', 'POST'])
def handler():
    # Request handling
    return public.returnMsg(True, 'Success')
```

### API Response Format
Standard JSON response:
```json
{
    "status": true,
    "msg": "Success message",
    "data": { /* response data */ }
}
```

Use `public.returnMsg(status, message, data)` for consistent responses.

### Authentication
- Session-based authentication
- CSRF token validation (in `flask_hook.py`)
- Basic Auth support (configurable in `config/basic_auth.json`)
- API token authentication (for external integrations)

## Testing

### Test Location
Tests are in `mod/test/` directory.

### Running Tests
```bash
cd /www/server/panel
python3 -m pytest mod/test/
```

### Test Structure
- Unit tests for individual components
- Integration tests for API endpoints
- Test fixtures in `mod/test/` subdirectories

## Security Considerations

### Important Security Notes

1. **File Permissions**:
   - Panel files should be owned by root
   - Executable files: `chmod 700`
   - Config files: `chmod 600`

2. **Sensitive Data**:
   - Database passwords stored in encrypted form
   - SSL certificates in `ssl/` directory
   - API keys in `config/` (protected by file permissions)

3. **Path Traversal Protection**:
   - Always validate user-provided paths
   - Use `os.path.realpath()` to resolve paths
   - Check if resolved path is within allowed directories

4. **SQL Injection**:
   - Use Peewee ORM for queries (parameterized)
   - Never concatenate user input into raw SQL

5. **Command Injection**:
   - Be cautious with `public.ExecShell()` and `os.system()`
   - Always sanitize user input before shell execution
   - Use `subprocess` with argument lists instead of shell strings

6. **Authentication**:
   - Session timeout: 30 days (configurable)
   - Failed login attempts tracked
   - IP-based access control via firewall

### Security Reporting
Security issues should be reported to: `1249648969@qq.com`

## Plugin System

### Plugin Architecture
Plugins are loaded via `PluginLoader.so` (native extension):
- Platform-specific: `PluginLoader.{machine}.Python3.12.so`
- Supports x86_64, ARM architectures
- Special glibc 2.14 version for older systems

### Plugin Directory Structure
```
plugin/
├── plugin_name/
│   ├── __init__.py
│   ├── info.json        # Plugin metadata
│   ├── install.sh       # Installation script
│   └── [plugin files]
```

### Loading Plugins
Plugins are discovered and loaded automatically via the hook system:
```python
from public.hook_import import hook_import
hook_import()
```

## Common Tasks for AI Assistants

### 1. Adding a New API Endpoint

**Location**: `BTPanel/routes/v2.py` (or `v1.py` for legacy)

```python
@app.route('/api/new_feature', methods=['POST'])
def new_feature():
    try:
        # Get parameters
        param = request.form.get('param', '')

        # Validate input
        if not param:
            return public.returnMsg(False, 'Parameter required')

        # Business logic (delegate to class)
        import new_feature_module
        result = new_feature_module.main().do_something(param)

        # Return response
        return public.returnMsg(True, 'Success', result)
    except Exception as e:
        return public.returnMsg(False, str(e))
```

### 2. Creating a New Model Class

**Location**: `class_v2/new_model/` or `class/new_model/`

```python
#coding: utf-8
# +-------------------------------------------------------------------
# | aaPanel
# +-------------------------------------------------------------------
# | Copyright (c) 2015-2099 aaPanel(www.aapanel.com) All rights reserved.
# +-------------------------------------------------------------------
# | Author: Your Name <email@example.com>
# +-------------------------------------------------------------------

import public
import os
import sys

class main:
    def __init__(self):
        self.setup_path = '/www/server/panel'

    def your_method(self, param):
        """
        Method description
        @param param: Parameter description
        @return: Return value description
        """
        try:
            # Implementation
            result = self.do_work(param)
            return public.returnMsg(True, 'Success', result)
        except Exception as e:
            return public.returnMsg(False, str(e))
```

### 3. Adding a Scheduled Task

**Location**: `task.py`

Add your task to the appropriate section:
```python
def your_scheduled_task():
    """
    @name Task description
    @return None
    """
    try:
        # Task implementation
        public.print_log("Task executed successfully")
    except Exception as e:
        public.print_log("Task failed: {}".format(str(e)))

# Register in the task scheduler
# Add to the appropriate interval (hourly, daily, weekly, etc.)
```

### 4. Working with Databases

**Using Peewee ORM:**
```python
import db

# Get database instance
sql = db.Sql()

# Query
result = sql.table('websites').where('id=?', (site_id,)).find()

# Insert
sql.table('websites').add('name,path,status', ('example.com', '/www/wwwroot/example', 1))

# Update
sql.table('websites').where('id=?', (site_id,)).save('status', 1)

# Delete
sql.table('websites').where('id=?', (site_id,)).delete()
```

### 5. File Operations

**Always use public module functions:**
```python
import public

# Read file
content = public.readFile('/path/to/file')

# Write file
public.writeFile('/path/to/file', 'content')

# Execute shell command
result = public.ExecShell('ls -la')

# HTTP requests
response = public.httpGet('https://api.example.com/endpoint')
response = public.httpPost('https://api.example.com/endpoint', {'key': 'value'})
```

### 6. Logging

```python
import public

# Log to panel log
public.print_log("Log message")

# Log to specific file
public.writeFile('/www/server/panel/logs/custom.log',
                 "[{}] Log message\n".format(public.getDate()),
                 mode='a+')
```

## Environment Variables

- **`BT_TASK='1'`**: Set when running background tasks
- **`PATH`**: Should include `/www/server/panel/pyenv/bin`
- Panel path is hardcoded: `/www/server/panel`

## Debugging Tips

### Enable Debug Mode
```bash
# Create debug flag
touch /www/server/panel/data/debug.pl

# Restart panel
bash /www/server/panel/init.sh restart
```

### View Logs
```bash
# Panel errors
tail -f /www/server/panel/logs/error.log

# Task logs
tail -f /www/server/panel/logs/task.log

# Panel PID
cat /www/server/panel/logs/panel.pid

# Task PID
cat /www/server/panel/logs/task.pid
```

### Check Panel Status
```bash
# Panel service
bash /www/server/panel/init.sh status

# View port
cat /www/server/panel/data/port.pl

# Check if running
ps aux | grep BT-Panel
ps aux | grep BT-Task
```

## Common Pitfalls & Best Practices

### ✅ DO:
- Always use `public.readFile()` and `public.writeFile()` for file I/O
- Validate all user input before processing
- Use `public.returnMsg()` for consistent API responses
- Add proper error handling with try/except blocks
- Log errors using `public.print_log()`
- Use the ORM (Peewee) for database operations
- Test changes in debug mode first
- Follow the existing code structure and patterns

### ❌ DON'T:
- Hardcode paths (use variables or config)
- Use raw SQL queries with string concatenation
- Execute shell commands with unsanitized user input
- Modify files in production without backups
- Skip input validation
- Use global variables excessively
- Ignore existing error handling patterns
- Mix Python 2 and Python 3 syntax (this is Python 3.12+ only)

## Dependencies Management

### Installing New Dependencies
```bash
cd /www/server/panel
./pyenv/bin/pip3 install package_name

# Add to requirements.txt
echo "package_name==version" >> requirements.txt
```

### Updating Dependencies
```bash
cd /www/server/panel
./pyenv/bin/pip3 install --upgrade package_name
```

## Version Information

- Current task version: `1.0.1` (see `CURRENT_TASK_VERSION` in `task.py`)
- Panel versions tracked via git commits
- PHP versions: Configurable in `config/php_versions.json`
- Database versions: Defined in `config/databases.json`

## Additional Resources

- **Official Site**: https://www.aapanel.com
- **Documentation**: https://doc.aapanel.com
- **Demo**: https://demo.aapanel.com/fdgi87jbn/ (username: `aapanel`, password: `aapanel`)
- **Forum**: https://forum.aapanel.com
- **GitHub**: https://github.com/aaPanel/aaPanel

## Notes for AI Code Assistants

1. **Context Awareness**: This is a server control panel with root-level access. Changes can affect production systems.

2. **Path Conventions**: The panel expects to be installed at `/www/server/panel`. All paths should be relative to this.

3. **Multi-version Support**: Code must support multiple OS versions and configurations. Always check for version-specific logic.

4. **Backward Compatibility**: Both `class/` and `class_v2/` are active. Don't assume one is deprecated.

5. **Security First**: This panel manages web servers, databases, and has root access. Always validate inputs and sanitize commands.

6. **Internationalization**: The panel supports multiple languages (see `BTPanel/languages/`). Keep this in mind for user-facing strings.

7. **Plugin System**: Many features are implemented as plugins. Check `plugin/` directory for extensibility points.

8. **Database Schema**: Tables are not strictly versioned. Always check current schema before modifying database operations.

9. **Shell Scripts**: Many operations delegate to bash scripts in `script/`. Understand both Python and Bash components.

10. **Gevent Async**: The server uses Gevent for concurrency. Be aware of async/blocking operations.

---

**Last Updated**: 2025-12-05
**Panel Version**: 7.57.0
**Python Version**: 3.12+
