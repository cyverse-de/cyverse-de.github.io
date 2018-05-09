---
layout: page
title: Discovery Environment (DE) Docs
---

# Table of Contents

* [Short-Term OSG Integration](#short-term-osg-integration)
    * [Image Overview](#image-overview)
        * [Requirements](#requirements)
    * [Ticket List Files](#ticket-list-files)
    * [Job Config Files](#job-config-files)
        * [arguments](#arguments)
        * [irods_host](#irods_host)
        * [irods_port](#irods_port)
        * [irods_job_user](#irods_job_user)
        * [irods_user](#irods_user)
        * [input_ticket_list](#input_ticket_list)
        * [output_ticket_list](#output_ticket_list)
        * [status_update_url](#status_update_url)
        * [stdout](#stdout)
        * [stderr](#stderr)
    * [Initializing the iRODS Connection](#initializing-the-irods-connection)
    * [Sending Job Status Updates](#sending-job-status-updates)
        * [Status Codes](#status-codes)
    * [Retrieving Input Files](#retrieving-input-files)
    * [Running the Job](#running-the-job)
    * [Uploading Output Files](#uploading-output-files)
    * [Caveats](#caveats)


# Short-Term OSG Integration

The DE can submit jobs to OSG, but the current integration mechanism is very limited in scope and places much of the
burden on the app integrator. For OSG jobs, the DE performs the following tasks:

- obtains iRODS read tickets for each of the job input files and creates a file listing the tickets and paths
- obtains an iRODS write ticket for the job output folder and creates a file listing the tickets and paths
- creates a JSON configuration file containing information needed to run the job
- submits the job to OSG

The app integrator is expected to provide a wrapper script that performs the following tasks:

- reads and parses the JSON configuration file
- initializes the iRODS connection based on information available in the configuration file
- sends job status updates back to the DE using a URL provided in the configuration file
- parses the input ticket list file and retrieves all of the input files from iRODS
- runs the job using the arguments provided in the configuration file
- redirects job output to the file names provided in the configuration file
- parses the output ticket list file and uploads all job outputs to iRODS

## Image Overview

OSG creates the Singularity images in its CVM-FS repository by pulling Docker images from Dockerhub and converting them
to Singularity images. This means that the first step to create a Singularity image for OSG is to create a Docker image
and push it to Dockerhub. Instructions for getting Docker images into OSG's CVM-FS repository can be found [here](1).

### Requirements

The primary requirements are that the image must contain an executable file called `/usr/bin/wrapper`, and this program
must read its configuration settings from a file called `config.json` in the current working directory. The
implemenation language is unimportant as long as all of the tasks required of the wrapper can be performed. CyVerse has
a reference implementation of such a wrapper script implemented in Python 2.7 available [here](2).

Each image must also have a way to communicate with iRODS. Usually, this means going with one of two options. If you're
using a JVM language such as Java, Scala or Clojure for the wrapper script then you can communicate with iRODS using
Jargon. If you're not using a JVM language then the easiest option is to use the iRODS icommands. The wrapper script
reference implementation provided by CyVerse uses the icommands which are installed as a step in the [corresponding
Dockerfile](3).

This image should also contain an empty directory called `/cvmfs`. When the job is executed, OSG's CVM-FS repository
will be mounted to this path.

The location of the default working directory in the image is unimportant.

## Ticket List Files

The input and output ticket list files are comma-delimited files with each significant line containing an iRODS ticket
string and a file path. Blank lines and lines beginning with a hash mark (`#`) are ignored. The first line in the file
is a comment line containing the MIME type, `application/vnd.de.tickets-path-list+csv; version=1`.

As mentioned above, each significant line in the file contains the iRODS ticket string followed by a comma and the path
corresponding to the ticket. The ticket string is guaranteed not to contain commas, but the path may. None of the fields
in the ticket list file are ever quoted. Instead, parsers must rely on the fact that tickets will never contain commas
to parse the field. For example, suppose the following line is found in a ticket list file:

```
521CDB78-8EA4-4F14-94FF-D506DB0D45D7,/iplant/home/nobody/the foo, the bar, and the baz
```

In this example the ticket string is `521CDB78-8EA4-4F14-94FF-D506DB0D45D7` and the path is `/iplant/home/nobody/the
foo, the bar, and the baz`.

The [wrapper script reference implementation](2) contains a convenient context manager class that can be used to
correctly parse a ticket list file:

``` python
# This is a simple context manager class designed to make it easier to read and write iRODS ticket list files.
class TicketListReader:
    def __init__(self, path):
        self.path = path

    def __enter__(self):
        self.fd = open(self.path, "r")
        self.r = itertools.imap(lambda l: l.strip().split(",", 1), itertools.ifilter(lambda l: l[0] != '#', self.fd))
        return self.r

    def __exit__(self, type, value, traceback):
        self.fd.close()
```

## Job Config Files

The job configuration file is a JSON file named `config.json` containing configuration settings for the job. The
following configuration settings are provided:

### arguments

This setting is an array of strings containing the command-line arguments generated by the DE based on the app
configuration and the parameter values selected by the user. In cases where the wrapper script runs the tool itself as a
subprocess, this argument list can generally be passed to the subprocess as-is. In other cases, the wrapper script must
interpret the argument array directly.

### irods_host

This setting contains the host name to use when connecting to iRODS.

### irods_port

This setting contains the port number to use when connecting to iRODS.

### irods_job_user

This setting contains the username of the person who submitted the job. This user should have ownership permissions on
the output files after the job completes.

### irods_user

This setting contains the username to use when authenticating to iRODS. This setting can be ignored if you're using
custom iRODS credentials.

### input_ticket_list

This setting contains the name of the input ticket list file.

### output_ticket_list

This setting contains the name of the output ticket list file.

### status_update_url

This setting contains a URL that should be used to send job status updates back to the DE. This will be described in
more detail later.

### stdout

This setting contains the name of the file that should contain the redirected standard output produced by the tool.

### stderr

This setting contains the name of the file that should contain the redirected standard error output produced by the
tool.

## Initializing the iRODS Connection

Initializing the iRODS connection is actually fairly simple if you're using version 4.x of the icommands, which is
highly recommended. The username in the `irods_user` configuration setting is guaranteed to have an empty password, so
iRODS can be initialized by creating a file called `$HOME/.irods/irods_environment.json`. This file should be in the
following format:

```
{
    "irods_user_name": "<username>",
    "irods_host": "<host-name>",
    "irods_port": <port-number>,
    "irods_zone_name": ""
}
```

For the time being, the zone name should always be equal to the empty string. The rest of the settings are extracted
from the job configuration file. After creating this file, `iget` and `iput` can be used in conjunction with the iRODS
tickets to retrieve files from or push files to iRODS.

If you're using custom iRODS credentials with a password then (assuming you're not using Jargon) `iinit` must be called
before using any of the iRODS icommands. `iinit` only runs in interactive mode, so it will be necessary to send the
password to `iinit` using redirection. Using custom iRODS credentials is expected to be rare, so precise instructions
for running `iinit` are beyond the scope of this documentation. Similarly, using Jargon is beyond the scope of this
documentation.

## Sending Job Status Updates

Job status updates can and should be sent to the DE using HTTP POST requests. The complete URL used to send job status
updates for the current job is provided in the job configuration file. The request body is a JSON object consisting of
three fields. The `state` field contains the current job status. The `message` field contains a brief description of
what the job is currently doing. The `hostname` field contains the host name or IP address tht the request is being sent
from.

### Status Codes

There are three job status codes that can be used in the wrapper script: `running`, `completed` and `failed`. A status
update with the status code `running` should be sent immediately after the job configuration file is parsed. This
informs the user that the job has started. The wrapper script may send as many `running` job status updates as it
wants. It is often helpful to send job status updates when the wrapper script is about to start downloading files, about
to kick off the analysis itself, and about to start uploading output files.

The other two status codes, `completed` and `failed`, are intended to be sent only once just before the wrapper script
exits. If the job completed successfully then the status code should be `completed`. Otherwise, it should be `failed`.

## Retrieving Input Files

In most cases, input files are retrieved using `iget`, which is the method that will be described here. Other file
retrieval methods are available, but they are beyond the scope of this documentation.

The first step in retrieving a file from iRODS is to extract the ticket string and path from the input ticket list
file. Once you have the ticket and path, `iget` can be executed as follows:

```
iget -rt <ticket> "<path>"
```

Note: if you're using a command shell to execute `iget` then the quotes around the path are important if the path
contains whitespace. In cases where the wrapper script is not a UNIX shell, the arguments can usually explicitly be
passed in as a list. For example, this function can be used in Python:

``` python
# Download a file or directory from iRODS.
def download_file(ticket, src):
    rc = subprocess.call(["iget", "-rt", ticket, src])
    if rc != 0:
        raise Exception("could not download {0}".format(src))
```

## Running the Job

Once the input files have been downloaded, the job itself can be executed. In many cases, the job is executed as a
subprocess of the wrapper script. This is a useful practice because it provides a clear boundary between the code that
interacts with the DE and the code that performs the analysis itself. This Python example simply calles `wc` passing in
the arguments retrieved from the configuration file:

``` python
# Run the word count job.
def run_job(arguments, output_filename, error_filename):
    with open(output_filename, "w") as out, open(error_filename, "w") as err:
        rc = subprocess.call(["wc"] + arguments, stdout=out, stderr=err)
        if rc != 0:
            raise Exception("wc returned exit code {0}".format(rc))
```

## Uploading Output Files

Uploading output files is a little bit more complicated than retrieving input files because additional steps have to be
taken to ensure that the user who ran the job can view the output files. The steps are:

- upload the files
- give the user ownership of the file if the iRODS user is different from the user who submitted the job
- remove the iRODS user's privileges on the file if the iRODS user is different from the user who submitted the job

The following example is from the [wrapper script reference implementation](2):

``` python
# Upload a file or directory to iRODS.
def upload_file(ticket, irods_user, owner, src, dest):
    rc = subprocess.call(["iput", "-rt", ticket, src, dest])
    if rc != 0:
        raise Exception("could not upload {0} to {1}".format(src, dest))

    # Don't bother modifying the file permissions if the iRODS user and file owner are the same.
    if irods_user == owner:
        return

    # Update the file permissions so that the person who submitted the job can see the output files.
    fullpath = os.path.join(dest, src)
    rc = subprocess.call(["ichmod", "own", owner, fullpath])
    if rc != 0:
        raise Exception("could not change the owner of {0} to {1}".format(fullpath, owner))
    rc = subprocess.call(["ichmod", "null", irods_user, fullpath])
    if rc != 0:
        raise Exception("could not remove {0} permissions on {1}".format(irods_user, fullpath))

# Upload a set of files to the directories referenced in a ticket list file to iRODS.
def upload_files(ticket_list_path, irods_user, owner, files):
    with TicketListReader(ticket_list_path) as tlr:
        for ticket, dest in tlr:
            for src in files:
                upload_file(ticket, irods_user, owner, src, dest)
```

## Caveats

TODO: add a description of the file creator problem.

[1]: https://support.opensciencegrid.org/support/solutions/articles/12000024676-docker-and-singularity-containers
[2]: https://github.com/iPlantCollaborativeOpenSource/docker-builds/blob/master/osg-word-count/wrapper
[3]: https://github.com/iPlantCollaborativeOpenSource/docker-builds/blob/master/osg-word-count/Dockerfile
