# Windows Batch Script Generator for GitHub Tool Installation

## Primary Objective
Generate a complete, production-ready Windows batch (.bat) file that automatically downloads, configures, and installs a tool from a GitHub repository, handling all dependencies, error scenarios, and system configurations with minimal user intervention.

## Context & Scope

### Target Environment
- **Operating System**: Windows 10/11 (x64)
- **User Level**: Both administrator and standard user scenarios
- **Prerequisites**: May require Git, Python, Node.js, or other runtimes
- **Network**: Internet connection for GitHub access

### Input Parameters
- **GitHub Repository URL**: `https://github.com/[owner]/[repo]`
- **Tool Type**: Specify the tool's technology stack (Python/Node.js/Go/etc.)
- **Installation Requirements**: Dependencies, build steps, configuration needs

## Detailed Requirements

### 1. Batch Script Structure

#### Header Section
```batch
@echo off
setlocal enabledelayedexpansion
title GitHub Tool Installer - [TOOL_NAME]
color 0A

:: Script metadata
set SCRIPT_VERSION=1.0.0
set TOOL_NAME=[TOOL_NAME]
set GITHUB_REPO=[REPO_URL]
set INSTALL_DATE=%date% %time%
```

#### Core Components Required
1. **System Compatibility Check**
   - Windows version detection
   - Architecture verification (x86/x64)
   - Administrative privilege check
   - Available disk space validation

2. **Prerequisite Detection**
   - Check for Git installation
   - Detect required runtimes (Python, Node.js, etc.)
   - Verify PowerShell version
   - Check for existing tool installation

3. **Download Management**
   - Use PowerShell, curl, or wget for downloading
   - Progress indication
   - Retry mechanism for failed downloads
   - Checksum verification

4. **Installation Process**
   - Extract archives (ZIP, TAR.GZ)
   - Run setup scripts
   - Configure environment variables
   - Create virtual environments if needed

5. **Error Handling**
   - Comprehensive error checking after each operation
   - Rollback capability
   - Detailed error logging
   - User-friendly error messages

### 2. Key Functionality Implementation

#### Environment Setup
```batch
:: Variables that must be defined
set DEFAULT_INSTALL_PATH=%LOCALAPPDATA%\[TOOL_NAME]
set TEMP_DOWNLOAD_DIR=%TEMP%\[TOOL_NAME]_installer
set LOG_FILE=%TEMP%\[TOOL_NAME]_install.log
set VENV_PATH=%DEFAULT_INSTALL_PATH%\venv
```

#### Download Methods
- **Option 1**: PowerShell (Windows 10+ native)
- **Option 2**: curl (if available)
- **Option 3**: Git clone (for full repository)
- **Option 4**: GitHub API for releases

#### Installation Workflows
1. **For Python Tools**:
   - Create virtual environment
   - Install pip dependencies
   - Configure entry points

2. **For Node.js Tools**:
   - Run npm/yarn install
   - Build if necessary
   - Create batch wrappers

3. **For Compiled Tools**:
   - Download appropriate binary
   - Extract to installation directory
   - Add to PATH

### 3. Required Script Sections

#### 1. Initialization and Validation
- Administrator privilege request (if needed)
- System requirements check
- Existing installation detection
- User confirmation prompt

#### 2. Prerequisite Installation
- Automatic download of missing dependencies
- Runtime installation (with user consent)
- PATH configuration
- Version compatibility checks

#### 3. Main Installation Flow
- Create directory structure
- Download tool from GitHub
- Extract/build/compile as needed
- Configure tool settings
- Create shortcuts

#### 4. Post-Installation
- PATH environment variable update
- Desktop/Start Menu shortcuts
- Configuration file creation
- First-run setup

#### 5. Cleanup and Finalization
- Remove temporary files
- Generate uninstall script
- Display success message
- Optional: Launch tool

### 4. Error Handling Requirements

#### Critical Error Scenarios
1. No internet connection
2. GitHub rate limiting
3. Insufficient permissions
4. Missing dependencies
5. Corrupted downloads
6. Build failures
7. PATH length exceeded

#### Error Response Strategy
- Clear error messages with solutions
- Automatic retry for transient failures
- Rollback on critical errors
- Detailed logging for debugging
- Option to continue with partial installation

### 5. User Interaction

#### Input Requirements
- Installation directory selection
- Component selection (if applicable)
- Virtual environment preference
- PATH modification consent
- Shortcut creation options

#### Progress Indication
- Step-by-step status updates
- Download progress percentage
- Estimated time remaining
- Current operation description

## Output Specifications

### Batch File Structure
```batch
:: The output should follow this structure:
1. Header and metadata
2. Function definitions
3. Initialization checks
4. Prerequisite handling
5. Main installation logic
6. Error handling routines
7. Cleanup procedures
8. Success/failure reporting
```

### Code Quality Requirements
- **Comments**: Detailed comments explaining each section
- **Variables**: Clear, descriptive variable names
- **Functions**: Modular design with reusable functions
- **Error Codes**: Consistent exit codes for different scenarios
- **Logging**: Comprehensive logging to file

### Feature Requirements
1. **Idempotency**: Safe to run multiple times
2. **Resumability**: Can continue from interruption
3. **Customization**: Easy to modify for different tools
4. **Portability**: No hard-coded paths
5. **Security**: No credentials in script

## Example Code Patterns

### Function Template
```batch
:FunctionName
:: Function description
:: Parameters: %1 = param1, %2 = param2
:: Returns: Sets ERRORLEVEL
setlocal
    :: Function body
    if errorlevel 1 (
        echo [ERROR] Function failed
        exit /b 1
    )
endlocal
exit /b 0
```

### Error Handling Pattern
```batch
command_to_execute
if !errorlevel! neq 0 (
    echo [ERROR] Command failed with error code !errorlevel!
    echo [ERROR] %date% %time% - Error details >> "%LOG_FILE%"
    call :Rollback
    goto :ErrorExit
)
```

### Download Pattern
```batch
:DownloadFile
:: Downloads file from URL to destination
:: %1 = URL, %2 = Destination path
powershell -Command "& {
    $ProgressPreference = 'SilentlyContinue'
    try {
        Invoke-WebRequest -Uri '%~1' -OutFile '%~2' -UseBasicParsing
        exit 0
    } catch {
        Write-Host 'Download failed:' $_.Exception.Message
        exit 1
    }
}"
```

## Success Criteria

### Functional Requirements
- ✓ Successfully installs tool on clean Windows 10/11 system
- ✓ Handles all common error scenarios gracefully
- ✓ Creates working installation with all features
- ✓ Generates uninstall capability
- ✓ Preserves user data on reinstall

### Performance Metrics
- Installation completes in < 5 minutes for typical tools
- Script size < 100KB
- Minimal memory footprint
- No system performance impact

### Compatibility
- Works on Windows 10 version 1809+
- Supports both PowerShell 5.1 and 7+
- Handles spaces in paths
- Unicode filename support
- Long path support (>260 characters)

### User Experience
- Clear, informative output
- No unnecessary prompts
- Graceful cancellation
- Professional appearance
- Helpful error messages

## Additional Specifications

### Security Considerations
```batch
:: Required security measures:
- Validate all downloaded files
- Use HTTPS for all downloads
- Check digital signatures where possible
- Sanitize user inputs
- Avoid privilege escalation unless necessary
```

### Logging Requirements
```batch
:: Log format:
[TIMESTAMP] [LEVEL] [COMPONENT] Message
Example: [2025-05-28 10:30:45] [INFO] [DOWNLOAD] Starting download from GitHub
```

### Customization Points
- Tool-specific configuration section
- Custom prerequisite checks
- Additional post-install steps
- Branding and messaging
- Optional component selection

### Testing Checklist
The generated batch file should be tested for:
- [ ] Fresh installation on clean system
- [ ] Upgrade over existing installation  
- [ ] Installation with missing prerequisites
- [ ] Network interruption recovery
- [ ] Cancellation and rollback
- [ ] PATH and environment variable handling
- [ ] Unicode and special character support
- [ ] Administrator vs. user installation
- [ ] Uninstallation completeness