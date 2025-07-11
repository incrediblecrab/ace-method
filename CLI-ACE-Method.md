# CLI ACE Method: Command Line Interface Development

Specialized implementation of the ACE Method for building world-class command line interfaces.

## Overview

Command line interfaces require unique considerations in user experience, performance, and system integration. This guide adapts the ACE Method specifically for CLI applications, from simple scripts to complex developer tools.

### CLI-Specific Principles
- **Keyboard-first interaction**: No mouse, pure efficiency
- **Composability**: Unix philosophy of doing one thing well
- **Scriptability**: Automation-friendly by design
- **Cross-platform**: Consistent behavior across operating systems

## CLI Development Considerations

### Application Types
1. **Simple Scripts**: Single-purpose tools (ls, grep, curl)
2. **Interactive CLIs**: REPL environments (node, python, psql)
3. **Complex Tools**: Multi-command applications (git, docker, kubectl)
4. **TUI Applications**: Terminal user interfaces (htop, vim, tmux)

### Key Differences from GUI Applications
- No visual design system - typography and color are your only tools
- Immediate feedback required - no loading spinners
- Stdin/stdout/stderr as primary interfaces
- Exit codes matter for scripting
- Argument parsing is the primary API

## ANALYZE Phase for CLI

### 1. CLI Problem Definition

```markdown
## CLI Analysis Framework

### User Context
- **Primary Users**: [Developers | DevOps | System Admins | End Users]
- **Skill Level**: [Beginner | Intermediate | Expert]
- **Usage Pattern**: [Daily | Occasional | Automation-only]

### CLI Requirements
- **Interface Type**: [Simple | Interactive | Complex | TUI]
- **Integration Needs**: [Pipes | Scripts | CI/CD | Other Tools]
- **Platform Support**: [POSIX | Windows | Cross-platform]

### Command Structure Analysis

Command structure follows this pattern:
- Basic: app [global-flags] command [command-flags] [arguments]

Examples:
- Simple: grep with recursive flag and pattern in directory
- Complex: docker run with interactive terminal and volume mount
- Interactive: redis-cli with host and port specification

### Success Metrics
- **Performance**: Fast startup time
- **Usability**: Intuitive command structure
- **Reliability**: Proper exit codes and error messages
- **Integration**: Works in pipes and scripts
```

### 2. CLI User Research

```python
# CLI Usage Analysis Script

import subprocess
import json
from collections import Counter

def analyze_shell_history():
    """Analyze common CLI patterns from shell history"""
    
    # Get shell history
    history = subprocess.run(['fc', '-l', '1'], 
                           capture_output=True, 
                           text=True).stdout
    
    # Parse commands
    commands = []
    for line in history.split('\n'):
        if line.strip():
            parts = line.split(maxsplit=1)
            if len(parts) > 1:
                cmd = parts[1].split()[0]
                commands.append(cmd)
    
    # Analyze patterns
    command_freq = Counter(commands)
    
    # Identify patterns
    patterns = {
        'pipes': len([c for c in history.split('\n') if '|' in c]),
        'redirects': len([c for c in history.split('\n') if '>' in c]),
        'background': len([c for c in history.split('\n') if '&' in c]),
        'multi_command': len([c for c in history.split('\n') if ';' in c or '&&' in c])
    }
    
    return {
        'most_common': command_freq.most_common(20),
        'patterns': patterns,
        'total_commands': len(commands)
    }

# Analyze common CLI workflows
def identify_workflows():
    """Identify common command sequences"""
    
    workflows = {
        'git_workflow': [
            'git status',
            'git add .',
            'git commit -m "message"',
            'git push'
        ],
        'docker_workflow': [
            'docker build -t app .',
            'docker run -it app',
            'docker ps',
            'docker logs <container>'
        ],
        'debug_workflow': [
            'ps aux | grep process',
            'lsof -i :port',
            'tail -f logfile',
            'htop'
        ]
    }
    
    return workflows
```

### 3. CLI Architecture Patterns

```markdown
## CLI Architecture Decision

### Pattern Selection
Based on complexity and requirements:

1. **Simple Script Pattern**
   - When: Single command, minimal code
   - Structure: Single file, minimal dependencies
   - Example: `backup.sh`, `clean-logs.py`

2. **Modular CLI Pattern**
   - When: Multiple subcommands, complex logic
   - Structure: Plugin architecture, command modules
   - Example: `git`, `npm`, `cargo`

3. **Interactive REPL Pattern**
   - When: Stateful interaction required
   - Structure: Read-eval-print loop, session management
   - Example: `node`, `python`, `mysql`

4. **TUI Application Pattern**
   - When: Rich terminal interface needed
   - Structure: Event-driven, screen management
   - Example: `vim`, `htop`, `tig`

### Technology Stack for CLI

| Language | Best For | Pros | Cons |
|----------|----------|------|------|
| Bash/Shell | System scripts | Universal, no deps | Limited features |
| Python | General CLI tools | Rich ecosystem | Slower startup |
| Go | Performance tools | Fast, single binary | Larger binaries |
| Rust | System utilities | Safe, fast | Compile times |
| Node.js | Developer tools | NPM ecosystem | Runtime dependency |
```

## CREATE Phase for CLI

### 1. CLI Framework Setup

```python
# Modern Python CLI with Click

import click
import sys
import os
from pathlib import Path

# CLI application structure
@click.group()
@click.version_option(version='1.0.0')
@click.pass_context
def cli(ctx):
    """Your awesome CLI tool description."""
    ctx.ensure_object(dict)
    
    # Global configuration
    ctx.obj['config'] = load_config()
    
    # Setup logging
    setup_logging(ctx.obj['config'].get('log_level', 'INFO'))

@cli.command()
@click.argument('input', type=click.Path(exists=True))
@click.option('--output', '-o', type=click.Path(), 
              help='Output file path')
@click.option('--format', '-f', 
              type=click.Choice(['json', 'yaml', 'csv']),
              default='json',
              help='Output format')
@click.option('--verbose', '-v', is_flag=True,
              help='Enable verbose output')
@click.pass_context
def process(ctx, input, output, format, verbose):
    """Process input file and generate output."""
    
    # Show progress for long operations
    with click.progressbar(items, label='Processing') as bar:
        for item in bar:
            process_item(item)
    
    # Colored output
    click.secho('Success!', fg='green', bold=True)
    
    # Handle pipes
    if not sys.stdout.isatty():
        # Output is being piped
        write_raw_output(result)
    else:
        # Terminal output
        write_formatted_output(result)

# Error handling
class CLIError(click.ClickException):
    """Custom CLI exception with proper exit codes"""
    
    def __init__(self, message, exit_code=1):
        super().__init__(message)
        self.exit_code = exit_code
    
    def show(self):
        click.secho(f'Error: {self.message}', fg='red', err=True)
        sys.exit(self.exit_code)

# Configuration management
def load_config():
    """Load configuration from multiple sources"""
    config = {}
    
    # 1. Default configuration
    config.update(DEFAULT_CONFIG)
    
    # 2. System-wide configuration
    if Path('/etc/myapp/config.yaml').exists():
        config.update(load_yaml('/etc/myapp/config.yaml'))
    
    # 3. User configuration
    user_config = Path.home() / '.myapp' / 'config.yaml'
    if user_config.exists():
        config.update(load_yaml(user_config))
    
    # 4. Environment variables
    for key, value in os.environ.items():
        if key.startswith('MYAPP_'):
            config[key[6:].lower()] = value
    
    return config
```

### 2. Go CLI Implementation

```go
// High-performance CLI with Cobra

package cmd

import (
    "fmt"
    "os"
    "time"
    
    "github.com/spf13/cobra"
    "github.com/spf13/viper"
    "github.com/fatih/color"
    "github.com/schollz/progressbar/v3"
)

var (
    cfgFile string
    verbose bool
)

// Root command
var rootCmd = &cobra.Command{
    Use:   "myapp",
    Short: "A powerful CLI tool",
    Long: `MyApp is a CLI tool that demonstrates best practices
for building command line applications.`,
    
    PersistentPreRun: func(cmd *cobra.Command, args []string) {
        // Setup configuration
        initConfig()
        
        // Setup logging
        if verbose {
            log.SetLevel(log.DebugLevel)
        }
    },
}

// Subcommand with advanced features
var processCmd = &cobra.Command{
    Use:   "process [file]",
    Short: "Process a file",
    Args:  cobra.ExactArgs(1),
    RunE: func(cmd *cobra.Command, args []string) error {
        filename := args[0]
        
        // Validate input
        if err := validateFile(filename); err != nil {
            return fmt.Errorf("invalid file: %w", err)
        }
        
        // Progress bar for long operations
        bar := progressbar.Default(100)
        for i := 0; i < 100; i++ {
            bar.Add(1)
            time.Sleep(shortDuration)
        }
        
        // Colored output
        color.Green(" Processing complete!")
        
        // Table output
        table := tablewriter.NewWriter(os.Stdout)
        table.SetHeader([]string{"Name", "Status", "Time"})
        table.Append([]string{filename, "Processed", "duration"})
        table.Render()
        
        return nil
    },
}

// Interactive mode
var interactiveCmd = &cobra.Command{
    Use:   "interactive",
    Short: "Start interactive mode",
    RunE: func(cmd *cobra.Command, args []string) error {
        // Create readline instance
        rl, err := readline.New("> ")
        if err != nil {
            return err
        }
        defer rl.Close()
        
        // REPL loop
        for {
            line, err := rl.Readline()
            if err != nil { // io.EOF
                break
            }
            
            // Process command
            result, err := processInteractiveCommand(line)
            if err != nil {
                color.Red("Error: %v", err)
                continue
            }
            
            fmt.Println(result)
        }
        
        return nil
    },
}

// Configuration management
func initConfig() {
    if cfgFile != "" {
        viper.SetConfigFile(cfgFile)
    } else {
        // Search for config in standard locations
        viper.SetConfigName("config")
        viper.SetConfigType("yaml")
        viper.AddConfigPath(".")
        viper.AddConfigPath("$HOME/.myapp")
        viper.AddConfigPath("/etc/myapp/")
    }
    
    // Environment variables
    viper.SetEnvPrefix("MYAPP")
    viper.AutomaticEnv()
    
    // Read config
    if err := viper.ReadInConfig(); err == nil {
        fmt.Println("Using config file:", viper.ConfigFileUsed())
    }
}

// Completion generation
func init() {
    rootCmd.AddCommand(completionCmd)
}

var completionCmd = &cobra.Command{
    Use:   "completion [bash|zsh|fish|powershell]",
    Short: "Generate completion script",
    Long: `To load completions:

Bash:
  $ source <(myapp completion bash)

Zsh:
  $ source <(myapp completion zsh)

Fish:
  $ myapp completion fish | source

PowerShell:
  $ myapp completion powershell | Out-String | Invoke-Expression
`,
    DisableFlagsInUseLine: true,
    ValidArgs:             []string{"bash", "zsh", "fish", "powershell"},
    Args:                  cobra.ExactValidArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
        switch args[0] {
        case "bash":
            cmd.Root().GenBashCompletion(os.Stdout)
        case "zsh":
            cmd.Root().GenZshCompletion(os.Stdout)
        case "fish":
            cmd.Root().GenFishCompletion(os.Stdout, true)
        case "powershell":
            cmd.Root().GenPowerShellCompletionWithDesc(os.Stdout)
        }
    },
}
```

### 3. Terminal UI Implementation

```python
# Rich TUI with Python and Textual

from textual.app import App, ComposeResult
from textual.widgets import Header, Footer, Input, Button, DataTable
from textual.containers import Container, Horizontal, Vertical
from textual.reactive import reactive
import asyncio

class CLIApp(App):
    """Modern TUI application"""
    
    CSS = """
    Screen {
        layout: vertical;
    }
    
    #input-container {
        height: 3;
        margin: 1;
    }
    
    #results {
        border: solid green;
    }
    
    Button {
        margin: 1;
    }
    """
    
    BINDINGS = [
        ("q", "quit", "Quit"),
        ("d", "toggle_dark", "Toggle dark mode"),
        ("ctrl+c", "copy", "Copy"),
        ("ctrl+v", "paste", "Paste"),
    ]
    
    def compose(self) -> ComposeResult:
        """Create child widgets"""
        yield Header()
        yield Container(
            Input(placeholder="Enter command...", id="command-input"),
            id="input-container"
        )
        yield DataTable(id="results")
        yield Footer()
    
    def on_mount(self) -> None:
        """Initialize the application"""
        table = self.query_one(DataTable)
        table.add_columns("Time", "Command", "Status", "Output")
        
        # Focus on input
        self.query_one("#command-input").focus()
    
    async def on_input_submitted(self, event) -> None:
        """Handle command submission"""
        command = event.value
        input_widget = self.query_one("#command-input")
        input_widget.clear()
        
        # Execute command asynchronously
        result = await self.execute_command(command)
        
        # Update table
        table = self.query_one(DataTable)
        table.add_row(
            datetime.now().strftime("time format"),
            command,
            "" if result.success else "",
            result.output[:50]
        )
    
    async def execute_command(self, command: str):
        """Execute command asynchronously"""
        try:
            proc = await asyncio.create_subprocess_shell(
                command,
                stdout=asyncio.subprocess.PIPE,
                stderr=asyncio.subprocess.PIPE
            )
            
            stdout, stderr = await proc.communicate()
            
            return CommandResult(
                success=proc.returncode == 0,
                output=stdout.decode() if stdout else stderr.decode()
            )
        except Exception as e:
            return CommandResult(success=False, output=str(e))

# Terminal capabilities detection
def detect_terminal_features():
    """Detect terminal capabilities"""
    import os
    import shutil
    
    features = {
        'color': os.isatty(sys.stdout.fileno()),
        'unicode': sys.stdout.encoding.lower() in ['utf-8', 'utf8'],
        'width': shutil.get_terminal_size().columns,
        'height': shutil.get_terminal_size().lines,
        'ansi': os.environ.get('TERM', '') != 'dumb',
        'hyperlinks': 'kitty' in os.environ.get('TERM', ''),
    }
    
    return features

# Cross-platform considerations
def get_platform_specific_paths():
    """Get platform-specific configuration paths"""
    import platform
    from pathlib import Path
    
    system = platform.system()
    
    if system == 'Windows':
        return {
            'config': Path(os.environ['APPDATA']) / 'MyApp',
            'cache': Path(os.environ['LOCALAPPDATA']) / 'MyApp' / 'Cache',
            'data': Path(os.environ['APPDATA']) / 'MyApp' / 'Data'
        }
    elif system == 'Darwin':  # macOS
        home = Path.home()
        return {
            'config': home / 'Library' / 'Preferences' / 'MyApp',
            'cache': home / 'Library' / 'Caches' / 'MyApp',
            'data': home / 'Library' / 'Application Support' / 'MyApp'
        }
    else:  # Linux/Unix
        home = Path.home()
        return {
            'config': Path(os.environ.get('XDG_CONFIG_HOME', home / '.config')) / 'myapp',
            'cache': Path(os.environ.get('XDG_CACHE_HOME', home / '.cache')) / 'myapp',
            'data': Path(os.environ.get('XDG_DATA_HOME', home / '.local' / 'share')) / 'myapp'
        }
```

### 4. CLI Testing Strategies

```python
# Comprehensive CLI Testing

import subprocess
import pytest
from click.testing import CliRunner
import pexpect
import tempfile
import os

class TestCLI:
    """Test suite for CLI applications"""
    
    @pytest.fixture
    def runner(self):
        """Click test runner"""
        return CliRunner()
    
    def test_basic_command(self, runner):
        """Test basic command execution"""
        result = runner.invoke(cli, ['process', 'test.txt'])
        assert result.exit_code == 0
        assert 'Success!' in result.output
    
    def test_pipe_support(self):
        """Test stdin/stdout pipe support"""
        # Test reading from stdin
        process = subprocess.Popen(
            ['echo', 'test input'],
            stdout=subprocess.PIPE
        )
        
        result = subprocess.run(
            ['myapp', 'process', '-'],
            stdin=process.stdout,
            capture_output=True,
            text=True
        )
        
        assert result.returncode == 0
        assert 'processed' in result.stdout.lower()
    
    def test_exit_codes(self):
        """Test proper exit codes"""
        # Success case
        result = subprocess.run(['myapp', 'version'], capture_output=True)
        assert result.returncode == 0
        
        # Error case
        result = subprocess.run(['myapp', 'invalid'], capture_output=True)
        assert result.returncode == 1
        
        # Specific error codes
        result = subprocess.run(['myapp', 'process', 'nonexistent'], capture_output=True)
        assert result.returncode == 2  # File not found
    
    def test_interactive_mode(self):
        """Test interactive CLI behavior"""
        child = pexpect.spawn('myapp interactive')
        
        # Wait for prompt
        child.expect('> ')
        
        # Send command
        child.sendline('help')
        child.expect('Available commands:')
        
        # Test auto-completion
        child.send('proc\t')
        child.expect('process')
        
        # Exit
        child.sendline('exit')
        child.expect(pexpect.EOF)
    
    def test_configuration_loading(self, tmp_path):
        """Test configuration file handling"""
        # Create test config
        config_file = tmp_path / 'config.yaml'
        config_file.write_text('''
        verbose: true
        output_format: json
        ''')
        
        # Test with config file
        result = subprocess.run(
            ['myapp', '--config', str(config_file), 'process', 'test.txt'],
            capture_output=True,
            text=True
        )
        
        assert '"format": "json"' in result.stdout
    
    def test_signal_handling(self):
        """Test graceful shutdown on signals"""
        import signal
        import time
        
        # Start long-running process
        proc = subprocess.Popen(['myapp', 'serve'])
        time.sleep(shortPause)
        
        # Send SIGTERM
        proc.send_signal(signal.SIGTERM)
        
        # Wait for graceful shutdown
        returncode = proc.wait(timeout=reasonable)
        assert returncode == 0
    
    def test_performance(self, benchmark):
        """Benchmark CLI startup time"""
        def run_cli():
            subprocess.run(['myapp', '--version'], capture_output=True)
        
        # Benchmark should be fast response
        result = benchmark(run_cli)
        assert result < 0.1
```

### 5. CLI Distribution

```yaml
# GitHub Actions for CLI distribution

name: Release CLI

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        include:
          - os: ubuntu-latest
            asset_name: myapp-linux-amd64
          - os: macos-latest
            asset_name: myapp-darwin-amd64
          - os: windows-latest
            asset_name: myapp-windows-amd64.exe
    
    runs-on: ${{ matrix.os }}
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Go
      uses: actions/setup-go@v4
      with:
        go-version: '1.21'
    
    - name: Build
      run: |
        go build -ldflags="-s -w" -o ${{ matrix.asset_name }} ./cmd/myapp
    
    - name: Compress
      if: matrix.os != 'windows-latest'
      run: |
        tar czf ${{ matrix.asset_name }}.tar.gz ${{ matrix.asset_name }}
    
    - name: Upload Release Asset
      uses: actions/upload-release-asset@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        upload_url: ${{ github.event.release.upload_url }}
        asset_path: ./${{ matrix.asset_name }}.tar.gz
        asset_name: ${{ matrix.asset_name }}.tar.gz
        asset_content_type: application/gzip

  homebrew:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - name: Update Homebrew Formula
      run: |
        # Update homebrew formula with new version and checksums
        # This would typically update a tap repository
```

```bash
# Installation script for various platforms

#!/bin/bash
# Universal installation script

set -e

BINARY_NAME="myapp"
GITHUB_REPO="username/myapp"

# Detect OS and architecture
OS=$(uname -s | tr '[:upper:]' '[:lower:]')
ARCH=$(uname -m)

case "$ARCH" in
    x86_64) ARCH="amd64" ;;
    arm64) ARCH="arm64" ;;
    aarch64) ARCH="arm64" ;;
    *) echo "Unsupported architecture: $ARCH"; exit 1 ;;
esac

# Construct download URL
VERSION=$(curl -s https://api.github.com/repos/$GITHUB_REPO/releases/latest | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/')
URL="https://github.com/$GITHUB_REPO/releases/download/$VERSION/${BINARY_NAME}-${OS}-${ARCH}.tar.gz"

# Download and install
echo "Downloading $BINARY_NAME $VERSION for $OS/$ARCH..."
curl -L "$URL" | tar xz

# Install to appropriate location
if [ "$OS" = "darwin" ]; then
    INSTALL_DIR="/usr/local/bin"
else
    INSTALL_DIR="$HOME/.local/bin"
    mkdir -p "$INSTALL_DIR"
fi

mv "$BINARY_NAME" "$INSTALL_DIR/"
chmod +x "$INSTALL_DIR/$BINARY_NAME"

echo "$BINARY_NAME installed successfully to $INSTALL_DIR"
echo "Make sure $INSTALL_DIR is in your PATH"

# Install completions
"$INSTALL_DIR/$BINARY_NAME" completion bash > /etc/bash_completion.d/$BINARY_NAME 2>/dev/null || true
"$INSTALL_DIR/$BINARY_NAME" completion zsh > /usr/local/share/zsh/site-functions/_$BINARY_NAME 2>/dev/null || true
```

## EVALUATE Phase for CLI

### 1. CLI-Specific Testing

```python
# CLI Usability Testing Framework

import time
import statistics
from typing import List, Dict

class CLIUsabilityTester:
    """Test CLI usability metrics"""
    
    def __init__(self, cli_command: str):
        self.cli = cli_command
        self.results = {}
    
    def test_response_time(self, commands: List[str]) -> Dict:
        """Measure command response times"""
        timings = {}
        
        for cmd in commands:
            times = []
            for _ in range(10):
                start = time.time()
                result = subprocess.run(
                    f"{self.cli} {cmd}",
                    shell=True,
                    capture_output=True
                )
                elapsed = time.time() - start
                times.append(elapsed)
            
            timings[cmd] = {
                'mean': statistics.mean(times),
                'median': statistics.median(times),
                'p95': statistics.quantiles(times, n=20)[18],
                'acceptable': statistics.mean(times) < 0.2  # 200ms threshold
            }
        
        return timings
    
    def test_error_messages(self) -> Dict:
        """Evaluate error message quality"""
        test_cases = [
            {
                'command': f'{self.cli} nonexistent',
                'expected_elements': ['error', 'command not found', 'help'],
                'bad_patterns': ['traceback', 'exception', 'null pointer']
            },
            {
                'command': f'{self.cli} process /nonexistent/file',
                'expected_elements': ['error', 'file not found', 'path'],
                'bad_patterns': ['errno', 'stack trace']
            }
        ]
        
        results = []
        for test in test_cases:
            result = subprocess.run(
                test['command'],
                shell=True,
                capture_output=True,
                text=True
            )
            
            stderr = result.stderr.lower()
            
            # Check for good patterns
            has_expected = all(
                elem in stderr for elem in test['expected_elements']
            )
            
            # Check for bad patterns
            has_bad = any(
                pattern in stderr for pattern in test['bad_patterns']
            )
            
            results.append({
                'command': test['command'],
                'quality': 'good' if has_expected and not has_bad else 'poor',
                'stderr': result.stderr
            })
        
        return results
    
    def test_help_system(self) -> Dict:
        """Evaluate help documentation"""
        checks = {
            'main_help': f'{self.cli} --help',
            'command_help': f'{self.cli} help process',
            'version': f'{self.cli} --version',
            'man_page': f'man {self.cli.split()[0]}'
        }
        
        results = {}
        for check_name, command in checks.items():
            result = subprocess.run(
                command,
                shell=True,
                capture_output=True,
                text=True
            )
            
            if result.returncode == 0:
                output = result.stdout
                results[check_name] = {
                    'exists': True,
                    'length': len(output),
                    'has_examples': 'example' in output.lower(),
                    'has_options': '--' in output,
                    'quality_score': self._score_help_quality(output)
                }
            else:
                results[check_name] = {'exists': False}
        
        return results
    
    def test_scriptability(self) -> Dict:
        """Test CLI scriptability features"""
        script_tests = {
            'exit_codes': self._test_exit_codes(),
            'quiet_mode': self._test_quiet_mode(),
            'machine_output': self._test_machine_readable_output(),
            'stdin_support': self._test_stdin_support(),
            'batch_mode': self._test_batch_processing()
        }
        
        return script_tests
    
    def _test_exit_codes(self):
        """Verify consistent exit codes"""
        tests = [
            ('success', f'{self.cli} version', 0),
            ('error', f'{self.cli} invalid-command', 1),
            ('file_not_found', f'{self.cli} process /nonexistent', 2),
        ]
        
        results = []
        for name, cmd, expected in tests:
            result = subprocess.run(cmd, shell=True, capture_output=True)
            results.append({
                'test': name,
                'expected': expected,
                'actual': result.returncode,
                'passed': result.returncode == expected
            })
        
        return results
```

### 2. CLI Performance Benchmarks

```python
# CLI Performance Benchmarking

import os
import psutil
import resource
import tempfile

class CLIPerformanceBenchmark:
    """Comprehensive CLI performance testing"""
    
    def benchmark_startup_time(self):
        """Measure cold and warm startup times"""
        
        # Cold start (clear caches)
        os.system('sync && echo 3 > /proc/sys/vm/drop_caches 2>/dev/null')
        
        cold_times = []
        for _ in range(5):
            start = time.perf_counter()
            subprocess.run([self.cli, '--version'], capture_output=True)
            cold_times.append(time.perf_counter() - start)
        
        # Warm start
        warm_times = []
        for _ in range(50):
            start = time.perf_counter()
            subprocess.run([self.cli, '--version'], capture_output=True)
            warm_times.append(time.perf_counter() - start)
        
        return {
            'cold_start': {
                'mean': statistics.mean(cold_times),
                'p95': statistics.quantiles(cold_times, n=20)[18]
            },
            'warm_start': {
                'mean': statistics.mean(warm_times),
                'p95': statistics.quantiles(warm_times, n=20)[18]
            },
            'acceptable': statistics.mean(warm_times) < 0.05  # 50ms
        }
    
    def benchmark_resource_usage(self):
        """Measure memory and CPU usage"""
        
        def measure_process():
            proc = subprocess.Popen(
                [self.cli, 'process', 'large-file.txt'],
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE
            )
            
            process = psutil.Process(proc.pid)
            
            # Monitor resource usage
            cpu_percents = []
            memory_usage = []
            
            while proc.poll() is None:
                try:
                    cpu_percents.append(process.cpu_percent(interval=0.1))
                    memory_usage.append(process.memory_info().rss / 1024 / 1024)  # MB
                except psutil.NoSuchProcess:
                    break
            
            return {
                'cpu': {
                    'max': max(cpu_percents) if cpu_percents else 0,
                    'avg': statistics.mean(cpu_percents) if cpu_percents else 0
                },
                'memory': {
                    'max_mb': max(memory_usage) if memory_usage else 0,
                    'avg_mb': statistics.mean(memory_usage) if memory_usage else 0
                }
            }
        
        return measure_process()
    
    def benchmark_large_input_handling(self):
        """Test performance with large inputs"""
        
        test_sizes = [1, 10, 100, 1000]  # MB
        results = {}
        
        for size_mb in test_sizes:
            # Create test file
            with tempfile.NamedTemporaryFile(mode='w', delete=False) as f:
                f.write('x' * (size_mb * 1024 * 1024))
                temp_file = f.name
            
            try:
                start = time.time()
                result = subprocess.run(
                    [self.cli, 'process', temp_file],
                    capture_output=True,
                    timeout=standard
                )
                elapsed = time.time() - start
                
                results[f'{size_mb}MB'] = {
                    'time': elapsed,
                    'throughput_mbps': size_mb / elapsed,
                    'success': result.returncode == 0
                }
            except subprocess.TimeoutExpired:
                results[f'{size_mb}MB'] = {
                    'time': None,
                    'error': 'timeout',
                    'success': False
                }
            finally:
                os.unlink(temp_file)
        
        return results
```

### 3. CLI Accessibility Testing

```python
# CLI Accessibility Validation

class CLIAccessibilityTester:
    """Test CLI accessibility features"""
    
    def test_screen_reader_compatibility(self):
        """Verify screen reader friendly output"""
        
        tests = {
            'no_unicode_required': self._test_ascii_fallback(),
            'clear_prompts': self._test_prompt_clarity(),
            'no_color_only_info': self._test_color_independence(),
            'descriptive_progress': self._test_progress_indicators()
        }
        
        return tests
    
    def _test_ascii_fallback(self):
        """Test ASCII-only mode"""
        env = os.environ.copy()
        env['LANG'] = 'C'
        env['LC_ALL'] = 'C'
        
        result = subprocess.run(
            [self.cli, 'status'],
            env=env,
            capture_output=True,
            text=True
        )
        
        # Check for non-ASCII characters
        try:
            result.stdout.encode('ascii')
            return {'passed': True, 'message': 'ASCII-safe output'}
        except UnicodeEncodeError:
            return {
                'passed': False, 
                'message': 'Contains non-ASCII characters without fallback'
            }
    
    def _test_color_independence(self):
        """Ensure information isn't conveyed by color alone"""
        
        # Run with no color
        env = os.environ.copy()
        env['NO_COLOR'] = '1'
        
        no_color_result = subprocess.run(
            [self.cli, 'status'],
            env=env,
            capture_output=True,
            text=True
        )
        
        # Run with color
        color_result = subprocess.run(
            [self.cli, 'status'],
            capture_output=True,
            text=True
        )
        
        # Compare information content
        no_color_info = self._extract_information(no_color_result.stdout)
        color_info = self._extract_information(color_result.stdout)
        
        return {
            'passed': no_color_info == color_info,
            'message': 'All information available without color'
        }
    
    def test_keyboard_efficiency(self):
        """Test keyboard-only operation efficiency"""
        
        metrics = {
            'tab_completion': self._test_tab_completion(),
            'shortcut_availability': self._test_shortcuts(),
            'navigation_efficiency': self._test_navigation_keystrokes()
        }
        
        return metrics
```

## Best Practices Summary

### CLI Design Principles

1. **Fast Startup**: fast warm start, acceptable cold start
2. **Intuitive Commands**: Follow established patterns (git, docker)
3. **Helpful Errors**: Clear, actionable error messages
4. **Scriptable**: Proper exit codes, quiet mode, machine-readable output
5. **Composable**: Works with pipes, supports stdin/stdout
6. **Cross-platform**: Consistent behavior across OS
7. **Accessible**: Screen reader friendly, no color-only information
8. **Documented**: Man pages, help text, examples

### Common Pitfalls to Avoid

1. **Slow Startup**: Lazy load modules, minimize dependencies
2. **Poor Error Messages**: Always explain what went wrong and how to fix
3. **Breaking Changes**: Version your CLI API, maintain compatibility
4. **Hidden Functionality**: Document all features in help text
5. **Platform Assumptions**: Test on Windows, macOS, and Linux
6. **Hardcoded Paths**: Use platform-specific path resolution
7. **No Config Management**: Support config files and environment variables
8. **Missing Completions**: Provide shell completion scripts

### Testing Checklist

Before release, ensure:

- [ ] Fast startup time
- [ ] All commands have help text
- [ ] Exit codes are consistent
- [ ] Works in pipes and scripts
- [ ] Handles Ctrl+C gracefully
- [ ] No terminal corruption
- [ ] Cross-platform tested
- [ ] Shell completions work
- [ ] Man page exists
- [ ] Examples in documentation

---

**Remember**: A great CLI feels invisible. Users should focus on their task, not on learning your tool.