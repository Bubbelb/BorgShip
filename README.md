# BorgShip
Try and create a Borg Backup environment, utilising file system features, like snapshotting .

# Introduction
When I stumbled on Borg Backup, a whole new world of backup options, features, problems and desires arose. Later on Docker came into play and one of the biggest gripes to date is that there is not yet a simple, robust way to utilise things like snapshotting on the local filesystem, that is lying under the docker volumes to make backups.

Why do you wnat to create snapshots, you ask? Well, to make consistent backups of systems, wile the downtime, or interruption is reduces to an absolute minimum.

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


