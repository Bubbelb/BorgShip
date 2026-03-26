# BorgShip
Try and create a Borg Backup environment, utilising file system features, like snapshotting .

# Introduction
When I stumbled on Borg Backup, a whole new world of backup options, features, problems and desires arose. Later on Docker came into play and one of the biggest gripes to date is that there is not yet a simple, robust way to utilise things like snapshotting on the local filesystem, that is lying under the docker volumes to make backups.

Why do you what to create snapshots, you ask? Well, to make consistent backups of systems, wile the downtime, or interruption is reduces to an absolute minimum.

The way I see it, now is as follows:
```
+--------------+
| Start Backup |
+--------------+
       |
+-----------------------+
| Stop/Pause containers |
+-----------------------+
       |
+------------------+
| Snapshot volumes |
+------------------+
       |
+------------------+
| Start containers |
+------------------+
       |
+----------------------+
| Create (Borg) Backup |
+----------------------+
       |
+---------------------------+
| Remove obsolete Snapshots |
+---------------------------+
       |
+---------------+
| Finish Backup |
+---------------+
```

## Requirements
What does this project provide? What are the important parts that make it a success?

* __Consistent backups__: Make sure all the application data is in a consistent state during the actual backup.
* __Minimal disruptions__: Be as smart as possible to achieve consistency, without compromising consistency.
* __Plugins/Scripts__: Provide custom ways to help applications to achieve above two goals.

## Technical Constraints
Which technical limitations are there, that force a specific design.

* __Docker based__: The solution runs as a Docker container. Best to have a Docker Compose solution.
* __No Docker Plugins__: Create the solution without extra software installed on the host.
* __Snapshot capable FS/blockdev__: The Docker volumes/mounts should be on a snapshot capable block device, or filesystem.
* __Target applications are Docker Compose (compose.yaml)__: To make sure the volumes are mounted correctly, compose is the best way, for now.

# High Level design
```
+------------+
| Controller |
+------------+
       |
       +------------+-------------------+--------------------+------------------+
                    |                   |                    |                  |
             +------------+    +----------------+    +---------------+   +-------------+
             | App config |    | Backup Prepare |    | Actual Backup |   | Backup Post |
             +------------+    +----------------+    +---------------+   +-------------+

             +------------+
             | Converter  |
             +------------+

```
## Controller
* Checks if all the prerequisites are met.
* Owns the scheduler.
* Allows for instant backups.
* Monitors changes in apps their compose.yaml, and acts accordingly.
* Is the only long-lived container.
* Has access to the main directory containing the compose apps.
* Configures and starts all the underlying containers, except for the Converter
* Checks for a backup re-run when a backup has failed.
* Generates reports of backups. Per backup, or coalesced.

## App Config
* Started by the Controller
* Runs for every compose.yaml
* Injects the Controller as a dependency for all container in the compose.yaml

## Converter
* Is part of the target compose.yaml files
* Runs, and stops before any other container is started.
* For every volume:
  * Create a subvolume '@' in the root of the volume
  * Move all files/folders into '@'
  * Updates all volumes to use a SubDir (name @)
* FUTURE: Should have a undo/uninstall option
* FUTURE: Should provide exclusion for specific volumes
* FUTURE: Should have a way to process bind mounts

## Backup Prepare
* Started by the Controller
* Runs for every compose.yaml
* Do pre-backup Checks
* Prepare for backup, doing on or more of (Use plugins for common apps):
  * Put app into maintenance/backup mode
  * Put containers in suspend/pause
  * Stop containers
* Create snapshot of volumes
* Resume normal operations for apps (e.g. restart containers, exit maintenance mode, etc.)

## Actual Backup
* Started by the Controller
* Runs for every compose.yaml
* Do actual backup
* Do Backup maintenance tasks (Checks, pruning, etc.)
* Mark subvolumes as successfully, or unsuccessfully backed-up
* Report status to Controller.

## Backup Post
* Remove status files
* Remove old snapshots

