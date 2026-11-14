# NSSM

## Introduction

NSSM (Non-Sucking Service Manager) is a Windows utility that runs ordinary executables and command-line applications as Windows services. It provides a service wrapper for programs that do not implement the Windows Service Control Manager (SCM) interface. Administrators use NSSM to keep background applications running without an interactive user session, start them automatically with Windows, and manage them through standard service controls.

Typical workloads include Python scripts, Node.js servers, Java applications, monitoring agents, and internal utilities. NSSM launches the configured executable with specified arguments and a working directory, then monitors the resulting process. If the application exits unexpectedly, the wrapper can restart it according to configured exit actions and restart delays. This is useful for long-running processes that require supervision but have no native Windows service implementation.

Configuration is available through a graphical interface and command-line commands. Key options include application path, startup directory, command-line parameters, service account, startup mode, environment variables, and process priority. NSSM also supports redirecting standard output and standard error to log files, with rotation options to help control disk usage. Shutdown settings determine how the wrapper attempts to stop an application and its child processes.

NSSM does not convert application code into a native service. Instead, it registers the wrapper with Windows and manages the target program on its behalf. That distinction matters when troubleshooting startup failures, permissions, application dependencies, or unexpected exits in production environments.

## **How to Install and Configure NSSM as a Windows Service**

To install NSSM as a Windows service wrapper, place the appropriate executable in a permanent directory and open an elevated Command Prompt or PowerShell session. The executable must remain at its installed location because Windows registers its path as the service binary.

For example, create a Windows service for a Python worker:

```
nssm install DataWorker "C:\Python311\python.exe" "C:\Apps\worker.py"
nssm set DataWorker AppDirectory "C:\Apps"
nssm set DataWorker DisplayName "Data Worker"
nssm set DataWorker Start SERVICE_AUTO_START
nssm start DataWorker
```

The first command registers the service and identifies the Python interpreter as the application. `AppDirectory` specifies the working directory, which is important when scripts use relative paths. `Start` selects automatic startup at system boot. Adjust the example paths to match the local installation.

For interactive configuration, run `nssm install DataWorker` without an application argument. Enter the executable path, startup directory, and arguments in the application settings, then install the service. Use `nssm edit DataWorker` to change settings later.

Check service state with `nssm status DataWorker`. You can also use `nssm stop DataWorker` and `nssm restart DataWorker` for routine administration. These commands interact with the registered service without requiring administrators to open the Windows Services console.

When retiring the service, stop it first and execute `nssm remove DataWorker confirm`. Removal unregisters the service; it does not delete the Python script or its files. If startup fails, verify executable paths, working-directory permissions, and the account running the service.

## **NSSM Service Management: Automatic Restart, Logging, and Troubleshooting**

For reliable NSSM service management, configure restart behavior and capture application logs before deploying a background process. By default, NSSM can restart an application that exits unexpectedly. Its restart throttling prevents rapid failures from producing a tight relaunch loop, while an additional restart delay can provide time for external dependencies to recover.

For the `DataWorker` service, configure a five-second restart delay and separate output files:

```
nssm set DataWorker AppExit Default Restart
nssm set DataWorker AppRestartDelay 5000
nssm set DataWorker AppStdout "C:\Logs\worker-out.log"
nssm set DataWorker AppStderr "C:\Logs\worker-error.log"
nssm set DataWorker AppRotateFiles 1
```

Create the log directory beforehand and grant the service account write access. Standard output and standard error redirection only capture information emitted to those streams; applications using their own logging systems may require separate configuration. File rotation settings can help manage existing logs, but retention and available disk space still need monitoring.

To inspect a setting, run `nssm get DataWorker AppRestartDelay`. Use `nssm edit DataWorker` to review shutdown timeouts, exit actions, and the account used to launch the process. A service running as LocalSystem may lack access to network shares or user-profile resources; assign a suitable account with only the required permissions.

When troubleshooting a Windows service that repeatedly stops, inspect redirected errors, validate command-line arguments, and confirm that required files and dependencies are accessible without an interactive desktop. NSSM supervises process execution, but it cannot repair failures inside the application.
