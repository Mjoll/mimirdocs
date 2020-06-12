# Deploying the Mimir processor

## Purpose

**Mimir processor** is an on-premise companion tool for Mimir that allows
processing of media files residing on-premise.

When running on a the **Mimir processor** on a local network, you can give it
access to the following local resources in order to enhance the capabilities of
your Mimir system:
 * Media files provided on local file systems, using using locally available
   file access speeds.
 * CPU processing resources for local processing of media files.
 * Adobe Media Encoder installations to delegate Premier conform tasks to.

For ease of deployment, the Mimir processor is provided as a Docker container.

## Pre-configuration

The processor requires several configuration settings to run successfully. Some
configuration settings must be set up before you run the Docker container for
the first time, and for some other configuration settings the processor will
guide you through the process of configuring them when the processor is run for
the first time.

Setting | Pre-Required | Guided | Required
------- | ------------ | ------ | --------
AWS_SERVICE_ROOT_API | Yes | No | Yes<sup>*1</sup>
MIMIR_EMAIL | No | Yes | Yes
MIMIR_PASSWORD | No | Yes | Yes
IMPORT_WORKFLOW | No | Yes | No
MONITOR_EDIT_PROXY_WATCH_FOLDER | No | No | Conditionally<sup>*2</sup>
MONITOR_DIRECT_UPLOAD_WATCH_FOLDER | No | No | Conditionally<sup>*2</sup>
MONITOR_PREPROCESSING_REQUESTS | No | No | Conditionally<sup>*2</sup>
MONITOR_INGEST_REQUESTS | No | No | Conditionally<sup>*2</sup>
MONITOR_CONFORM_REQUESTS | No | No | Conditionally<sup>*2</sup>
EDIT_PROXY_WATCH_FOLDER | No | No | Yes<sup>*3</sup>
DIRECT_UPLOAD_WATCH_FOLDER | No | No | Yes<sup>*3</sup>
SERVER_FOLDER | No | No | Yes<sup>*3</sup>
TEMP_FOLDER | No | No | Yes<sup>*3</sup>
P2_SINGLE_ITEM_WATCH_FOLDER | No | No | No
ADOBE_ENCODER_WATCH_FOLDER_PREFIX | No | No | No
CONFORM_SOURCE_FOLDER | No | No | No
LOG_FOLDER | No | No | No
STABILITY_THRESHOLD | No | No | No
POLLING_INTERVAL | No | No | No
FORCE_CONSOLE_LOG | No | No | No
ITEM_LINKS | No | No | No


<sup>*1</sup> Processor does not start up properly without it.

<sup>*2</sup> One of MONITOR_EDIT_PROXY_WATCH_FOLDER,
MONITOR_DIRECT_UPLOAD_WATCH_FOLDER, MONITOR_PREPROCESSING_REQUESTS,
MONITOR_INGEST_REQUESTS, or MONITOR_CONFORM_REQUESTS must be set to ```true```,
otherwise the processor terminates.

<sup>*3</sup> Processor starts up and runs mostly fine without it, but spits out
some errors.

### Creating a minimal configuration file

Create a file named ```/opt/mimir/config.json```:

```
sudo rmdir /opt/mimir/config.json
editor /opt/mimir/config.json
```

You can replace the ```editor``` command above with your favorite text editor.

Edit the contents of the ```config.json``` file to contain these settings:

```
{
    "AWS_SERVICE_ROOT_API": "https://prod.mimir.tv/api/v1",
    "EDIT_PROXY_WATCH_FOLDER": "/mnt/production/watchfolder",
    "DIRECT_UPLOAD_WATCH_FOLDER": "/mnt/production/minio_root/uploaded",
    "MONITOR_EDIT_PROXY_WATCH_FOLDER": true,
    "SERVER_FOLDER": "/opt/mimir/production/minio_root/minio_root",
    "TEMP_FOLDER": "/mnt/production/tmp"
}
```

Adjust the AWS_SERVICE_ROOT_API setting value to match the geographical region
of your Mimir system.

You can also customize the paths of EDIT_PROXY_WATCH_FOLDER,
DIRECT_UPLOAD_WATCH_FOLDER, MONITOR_EDIT_PROXY_WATCH_FOLDER, SERVER_FOLDER, and
TEMP_FOLDER, as well as the location of the config.json file. However you will
then need to change the ```docker``` command described below in a corresponding
fashion.

Note that while EDIT_PROXY_WATCH_FOLDER, DIRECT_UPLOAD_WATCH_FOLDER and
TEMP_FOLDER valuesrefer to file paths within the Docker container, the
SERVER_FOLDER value refers to a file system path outside the Docker container.

## Running the docker container interactively

Running the processor for the first time:

```
docker run --interactive --tty --rm \
  --volume /opt/mimir/config.json:/processor/config.json \
  --volume /opt/mimir/production:/mnt/production mjoll/processor
```

## The configuration file

The ```/opt/mimir/config.json``` in the first part of the first volume option,
refers to a configuration file on the local file-system which dictates the
behavior of the Mimir processor. You can place this configuration file on any
file system location accessible to the Docker host, as long as you specify the
path to the configuration file you actually use in the first part of the first
volume option.

## The files storage mount

The ```/opt/mimir/production``` in the first part of the second volume option
refers to a local directory which contains media files which Mimir should
preform processing operations on. You can specify any directory location on any
file system aceessible to the Docker host, as long as you specify the directory
path you actually want to expose to the Mimir processor in the first part of the
second volume option.

## Interactively specifying additional settings

This will run the container interactively, and it will prompt you to supply a
user login account name (MIMIR_MAIL) and the corresponding account password
(MIMIR_PASSWORD), and finally pick one of the *import workflows*
(IMPORT_WORKFLOW) that has been configured for your Mimir system.

> :warning: Configuration choices that you enter in interactively will not
> automatically be saved for future use.

## Stopping the interactive container

To stop the container, press Ctrl+C repeatedly until you're back to your regular
command prompt.

## Permanently storing required settings

To remove the need for manually entering settings during startup of the
container in the future, modify the ```config.json``` file to contain specify
setting values for MIMIR_MAIL, MIMIR_PASSWORD and IMPORT_WORKFLOW. The
IMPORT_WORKFLOW value is a globally unique identifier (GUID), which the
container will show you when running the container interactively without a
IMPORT_WORKFLOW setting. Here is an example of what the container would show:

```
Please select which import workflow to process files for:

1. (5d3ba6ee-f4c8-44e7-b00f-8615d5ce0383) my_workflow
```

To permanently select the workflow named "my_workflow", insert the following
setting into the config.json file:

```
"IMPORT_WORKFLOW": "5d3ba6ee-f4c8-44e7-b00f-8615d5ce0383"
```
## Running the container detached

Here is a ```config.json``` example that provides the minimal set of setting needed to run the container in a non-interactive fashion:

```
{
    "AWS_SERVICE_ROOT_API": "https://prod.mimir.tv/api/v1",
    "EDIT_PROXY_WATCH_FOLDER": "/mnt/production/watchfolder",
    "DIRECT_UPLOAD_WATCH_FOLDER": "/mnt/production/minio_root/uploaded",
    "MONITOR_EDIT_PROXY_WATCH_FOLDER": true,
    "SERVER_FOLDER": "/opt/mimir/production/minio_root/minio_root",
    "TEMP_FOLDER": "/mnt/production/tmp",
    "MIMIR_EMAIL": "myself@example.com",
    "MIMIR_PASSWORD": "<REDACTED>",
    "IMPORT_WORKFLOW": "5d3ba6ee-f4c8-44e7-b00f-8615d5ce0383"
}
```

With such a configuration file, you can run the Docker container like this:

```
docker run --detach --restart always --name mimir-processor \
  --volume /opt/mimir/config.json:/processor/config.json \
  --volume /opt/mimir/production:/mnt/production mjoll/processor
```

The this command will print out a container id such as
```8ceef449868264d535fd4a8a07e0a3cca54392bd3a4c69447826d04519a64a1c```which can
be used to inspect, kill, restart and remove the container in the future. The
example command-line also gives the container the name ```mimir-processor```
which can be used in place of the container id as a convenience.

Refer to the Docker documentation for more information about how to manage
Docker containers in general.

# Stopping and deleting the container permanently

To remove a deployment of the Mimir processor, run the following commands.

```
docker kill mimir-processor
docker rm mimir-processor
```

# Setting up directory roles

The file system mount that you expose to the Mimir processor can contain
multiple directories, which serve different *processing roles* which dictate
what processing operations the Mimir processor performs on these directories.

...TBD...

# Troubleshooting

## Cannot connect to the Docker daemon

The following error message indicates that the Docker client is installed on the local machine, but the service component required to run Docker containers is not available.

```
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.
See 'docker run --help'.
```

If you are testing out the Docker container in WSL 2 on Windows, be aware that the service component of Docker is not automatically started under WSL 2 on Windows. If you're running the Linux distribution Ubuntu 20.04 LTS or similar on under WSL 2, you can start the Docker daemon using this command:

```
sudo service docker start
```

But be aware that while running Docker containers under WSL 2 on Windows is
convenient for trying out the Mimir processor on a local Windows desktop, we do
not recommend this solution for production use.

## config.json is a directory instead of a file

The following error message indicate that the container was run without specifying a mount option that points to a pre-existing config.json file.

```
Error: config.json is a directory instead of a file !
```

As a side-effect of specifying a volume mount location that did not exist, Docker will have created a directory named config.json, rather than exposing a file named config.json to the container. To proceed you will need to delete the directory, and create a configuration file in its place:

```
sudo rmdir /opt/mimir/config.json
editor /opt/mimir/config.json
```

## Missing /tmp/import/Workflow.json

```
Error: ENOENT: no such file or directory, open '/tmp/importWorkflow.json'
```

...TBD...