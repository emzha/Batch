# Azure Batch Node Agent Change Log
### About the Batch Node Agent
The Batch Node Agent executes customer-defined tasks on compute nodes 
within a pool. This includes any actions that must be performed on the 
compute nodes themselves including (but not limited to) preparation,
reporting, maintenance, execution, and removal of said tasks.

### A Note on Dates
Next to each version below, a date is given. Due to the global nature of the
Azure Batch service, new versions of the agent are not available in every 
region simultaneously upon release. The date given next to each version is
the date the new version begins deployment to all supported regions. It is 
the _earliest possible_ time you could see the new agent version. There is
no guarantee that a pool has a particular version of the agent even if 
it was created after the release date of that version.

### Getting the Latest Version of the Batch Node Agent
All new Azure Batch pools are automatically provisioned with the latest 
Node Agent, unless the Azure Batch team deems it necessary to limit or 
rollback a certain version due to possible customer impacting concerns.
In order to get the latest version of the Batch Node Agent, a new pool should
be created.

### Determining what version of the Batch Node Agent is on a Node
Using a client targeting REST API version `2018-08-01.7.0` or
newer call the [get compute node](
https://docs.microsoft.com/en-us/rest/api/batchservice/computenode/get)
or [list compute nodes](
https://docs.microsoft.com/en-us/rest/api/batchservice/computenode/list)
APIs and examine the `version` property of [nodeAgentInfo](
https://docs.microsoft.com/en-us/rest/api/batchservice/computenode/list#nodeagentinformation).

## Change Log

### 1.5.5 - 2019-04-03
- Fix bug where application package environment variables were sometimes
  not correctly updated when the default version of the application was 
  changed and the node was rebooted.
- Fix bug where after a node reboot in Linux, sometimes the `List Task Files`
  or `Get Task File` APIs would fail to find a file that was actually on disk.
- Fix bug where `ExitConditions` were ignored and overwritten to be empty.

### 1.5.4 - 2019-03-25
- Internal telemetry improvements to help the product team better diagnose 
  issues after VMs have been removed from a pool.
- Fix issue where a concurrent termination or deletion operation in 
  conjunction with command process exit would prevent new tasks from 
  being run.

### 1.5.3 - 2019-03-12
- Add support for `--mount` option on container tasks.
- Add support for an upcoming Batch Service feature allowing container
  tasks to use the working directory of the container rather than 
  the default Batch working directory for the task.
  - This feature will be enabled in a future Batch REST API version.
- Fix bug which could cause a task to get stuck if deleted immediately
  after it started running.
- Fix bug where the multi-instance master subtask could 
  mistakenly overwrite details about the execution of the other subtasks. This 
  caused the subtasks to not show up in the API until they reached 
  `completed` state.
- Fix bug where `docker login` was not retried when throttled.
- Improve handling of Windows user profile creation getting stuck.

### 1.4.12 - 2019-02-15
- Fix bug which could corrupt the download of the final portion of a 
  `ResourceFile` in rare cases.

### 1.4.11 - 2019-02-05
- Fix bug where the OS drive is replaced (but the ephemeral drive is retained)
  which caused nodes to get stuck in `Unusable`.

### 1.4.10 - 2019-02-03
- Fix bug that caused the node to get stuck in `Unusable` 
  due to rare cases of unavailable networking at the time of agent boot.
- Fix bug where frequent reboots could cause the agent to be unable
  to join a pool, even if the reboots stopped happening.
- Fix bug where job preparation tasks could get stuck on VMs with 
  `MaxTaskPerVMs` greater than 1 if multiple tasks were scheduled to
  the node at the same time.
- Fix bug where files downloaded by the node could get corrupted.
- Fix bug where the agent could not download `ResourceFile`s which specified
  `AutoStorageContainerName`.
- Fix bug where node preparation tasks, job preparation tasks, and job release
  tasks were put into an incorrect state if the agent had an internal error.

### 1.4.9 - 2019-01-14
- Fix bug where connection/HTTP failures happening persistently for a long 
  period of time could cause the agent to enter an unrecoverable state.
- Retry longer on connection errors.
- Retry on intermittent DNS failures.
- Move Windows user profiles to the ephemeral drive for 
  `CloudServiceConfiguration` based pools.

### 1.4.7 - 2018-11-29
- Implement a timeout on Windows user profile creation. If the user profile
  creation takes too long, the task is retried (possibly on another 
  compute node).
- Fix bug on Windows where some tasks which create child processes could 
  prevent task termination even if the parent process exited, causing 
  the task to get stuck.
- Fix application package root directory to be `applications`
  and not `apppackages`.
- Fix bug where tasks which were retried on the node did 
  not properly clean up their working directory prior to retry.
- Add support for specifying Windows user login mode (`batch` or `interactive`).
- Add support for Windows Server 2019 in `VirtualMachineConfiguration` pools.

### 1.4.5 - 2018-11-13
- Fix bug where certain connection errors were not properly 
  retried when performing resource file downloads on Windows.

### 1.4.4 - 2018-11-06
- Fix bug where multi-instance tasks with common resource files and normal 
  resource files would fail to download the common resource files in 
  some cases.
- Fix bug where the agent could leak memory when preparing the task
  command line for execution in Windows.
- Fix bug which prevented Batch service logs from being written to
  disk on Windows.
- Improve error handling for all resource file downloads.

### 1.4.3 - 2018-10-29
- Fix bug where files which were marked read-only on Windows could not
  be deleted in some cases.
- Fix bug where file deletion failures during node boot could delay
  initialization for a long time, causing the node get marked `Unusable`.
- Fix bug where the tasks spawned by the NodeAgent on Windows would incorrectly 
  have the win32 `DllDirectory` path set. This allows native applications 
  using the `SetDllDirectory` API to behave as intended.
- Use a more generous connection/transmission timeout when uploading files via
  `OutputFiles`.
- Fix bug where unversioned environment variables for applications 
  (e.g. `AZ_BATCH_APP_PACKAGE_AppName`) could incorrectly point at older 
  package versions in some cases if both old and new packages were
  installed onto the same node.
- Reduce log file disk usage for VMs with limited ephemeral disk space.

### 1.4.2 - 2018-10-16
- Fix bug introduced in `1.3.0.12` where nodes with application packages
  could go to `Unusable` after being rebooted.
- Improve the logs generated when uploading `OutputFiles` in 
  `fileuploadout.txt`.

### 1.3.0.12 - 2018-09-28
- Fix bug where once a node had installed a task level application package
  it would include that application package environment variable on other 
  tasks even if they didn't specify a dependency on that application package.
- Fix bug where job preparation tasks which failed due to a certificate 
  installation error could never be re-run, which blocked the job on that node
  forever.
- Reduce the time to install and prepare the Batch components on Windows 
  compute nodes. In ideal cases this reduces the time taken for a node to
  reach `idle` state when initially joining a pool.

### 1.3.0.11 - 2018-09-13
- Signed binaries are now provided for Windows builds of the Node Agent.
- Fix bug where some Node Agent files had a corrupt digital signature.
- Reduce how aggressively the Node Agent retries on directory delete failures.
- Fix bug where sometimes autoscale metrics could be invalid.
- Fix bug where Windows tasks running in parallel (on pools with 
  `MaxTasksPerNode` > 1) could inherit each others stdout/stderr handles.
  This could cause failures when trying to delete the task's files because
  running processes would still have locks on stdout/stderr or lead 
  to restarts.

### 1.3.0.10 - 2018-08-31
- Fix regression where rebooting a node with certificates could cause it to 
  become unresponsive.

### 1.3.0.9 - 2018-08-21
- Reduce agent disk usage by removing extraneous logging and reducing
  footprint of stored certificate data.
- Fix bug where the agent could get stuck attempting to perform environment
  variable expansion on an infinite loop of environment variables.
- Improve error returned to user if an environment block has a loop in it.
- Fix bug when using `ContainerSettings` with `--hostname`.

### 1.3.0.8 - 2018-08-13
- Add support required for task `result` OData filter for all task types.
- Improve detection of Blob Not Found errors when downloading 
  resource files or application packages.
- Improve directory deletion logic:
  - Files or directories locked by running processes.
  - Files or directories have unexpected ownership/permissions.
- Added support for the following new images:
  - `Canonical:UbuntuServer:18.04-LTS`
  - `microsoft-azure-batch:centos-container:7-5`
- Fix bug and added retries where where docker login could fail due to 
  a timeout error.
- Fix bug where node couldn't run any tasks after a reboot, due to getting
  stuck trying to garbage collect task files.

### 1.3.0.6 - 2018-07-10
- Add support for software entitlements given to the Node Agent when it joins
  the pool (for use in the StartTask).
- Add support for the Node Agent exposing its version and
  last update time in the public Azure Batch REST API.
  - This will be utilized in an upcoming REST API release.
- Improve telemetry emitted by the Node Agent if it fails to properly
  bootstrap.
- Fix bug where using 
  `microsoft-azure-batch:ubuntu-server-container:16-04-lts` or
  `microsoft-azure-batch:ubuntu-server-container-rdma:16-04-lts` marketplace
  images on a GPU SKU would cause kernel updates to be perpetually held.
- Fix bug where in rare cases tasks could get stuck in `running` state if
  the Node Agent process was terminated at a certain point.
- Fix bug where task application packages could sometimes fail to
  download if the task was rescheduled after previously running elsewhere.

### 1.3.0.2 - 2018-05-25
- Add Batch icon and file version info to Windows executables.
- Cap tasks to 95% CPU on single core Windows VMs to prevent Azure
  infrastructure agents from starvation.
- Reduce Node Agent CPU/disk usage when spawning new tasks and running tasks.
- Update agent to support new `microsoft-azure-batch` published marketplace
  images with Docker container support.
  - Includes using nvidia-docker2 in pools supporting containers.
- Fix bug where node state transition times were not updated in certain cases.
- Fix bug where very old nodes which have run hundreds of thousands of tasks
  could have a significant fraction of their overall disk space used by the
  Node Agent. In extreme cases their disk could even be filled up.
- Fix bug where `AZ_BATCH_JOB_PREP_DIR` was set to the wrong value.
  - This bug was a regression which caught early on in the rollout of
    1.3.0.1.
- Fix bug where resource file download could corrupt a file.
- Fix bug on Windows when running a multi-instance task with scope == 'task'
  which caused all subsequent multi-instance tasks in that scope
  (or any task with scope == 'pool') to fail, because we inadvertently
  deleted the pool scope user when the first task completed.

### 1.2.0.14 - 2018-04-09
- Fix bug where deleting a job in 'active' state would incorrectly skip
  running the job release task.
- Fix bug where the agent encounters an internal error but fails to
  yield the task back to Batch for scheduling on a different node,
  instead the agent just completed the task with a
  'schedulingError'/'failureInfo'.
- Fix bug where a task with 'maxRetries' == -1 didn't retry at all.
  The expected behavior if 'maxRetries' == -1 is for the task to retry
  forever.

### 1.2.0.12 - 2018-04-04
- Reduce Node Agent disk/CPU usage while the agent is idle or running a task
  and waiting for that task to exit.

### 1.2.0.11 - 2018-03-18
- Improve 'docker pull' error handling and add retries for Node Agents running
  in container pools.
- Fix bug where container pools didn't do 'image get' before 'docker pull'
  in some cases.

### 1.2.0.9 - 2018-03-02
- Fix bug bootstrapping nvidia driver version.
- Fix bug where container pools did an unnecessary extra "GET" operation when
  pulling the container image.
- Fix bug which caused tasks to get stuck in "running" forever caused by
  incorrect read logic.

### 1.2.0.6 - 2018-02-15
- For container pools, attempt Docker service start if the Docker service is
  not already started on Windows.
- Improve error handling/logging for CreateProfile during task process
  execution on Windows.
- Add Debian 9 support.

### 1.2.0.4 - 2018-02-01
- Add support for Node Agent log file upload feature, which allows customers
  to upload the Azure Batch Node Agent internal log files to an Azure Storage
  container for later investigation by the product team.
- Fix bug where the Node Agent would fail to start properly in some
  sovereign clouds.
