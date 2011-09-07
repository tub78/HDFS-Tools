

If you're like me, you get frustrated by the amount of typing that is required to copy a file from your Hadoop filesystem to your local filesystem, e.g.:

``` bash
hdfs dfs -get hdfs://xxx/very/long/path/to/a/file \
    /yyy/very/long/path/to/a/file
```

Also, if you are like me, you want the directory structures of the two filesystems to be mirror-images.  This means you typically have to type a common path component twice, which is redundant, time consuming, and error prone. 

To address this issue (and to exercise my Bash scripting skills), I hacked together a collection of shell scripts that automate this process, together called *HDFS-Tools*.  The *HDFS-Tools* simplify the management of files in your Hadoop Filesystem by helping to synchronize a _local_ copy of the filesystem with *HDFS*.

## How Does It Work?

To enable *HDFS-Tools*, one must first designate a directory to hold the _root_ of the _local_ copy; this is done by setting the `HDFS_PREFIX` environment variable.  Paths relative to `HDFS_PREFIX` in the local copy are the same as in *HDFS*.

Once this is done, copying data between *HDFS* and your _local_ copy is simply a matter of getting or putting a file; e.g.:

``` bash
hget <path> 
```

*HDFS-Tools* deals with the task of expanding the `path` arguments to create the conventional command format, using the `HDFS_PREFIX` and your *HDFS*'s configuration.  Furthermore, with some code from [rapleaf's dev blog](http://blog.rapleaf.com/dev/2009/11/17/command-line-auto-completion-for-hadoop-dfs-commands/), these commands have been augmented with filename auto-completion. Together, these features make `hget`, `hput`, etc., more convenient than using:

``` bash
hdfs dfs -get <hdfs_path> <local_path>
```

Say goodbye to the frustration of typing long paths in *HDFS*.  Indeed, you rarely need to type more than the commands themselves.

## Filename Auto-Completion

Auto-completion is available for `hls`, `hget`, and `hput`, by pressing `<TAB>`.  There may be a delay before results are displayed, as the query to the remote *HDFS* is issued.  When the `CWD` is below `HDFS_PREFIX`, filename auto-completion displays paths relative to `CWD`; otherwise, they are relative to `HDFS_PREFIX`.  In the later case, the paths are displayed with a `/` prefix.

Auto-completion for directories is a little clunky because a space character is appended to the result.  In order to extend the path further, you must type `<backspace><TAB>`.

## Details

*HDFS-Tools* consists of the following:

[hpwd](file:///Users/stu/Research/HDFS-Tools/hpwd)
: List corresponding path in *HDFS*.  When the current working directory resides under `HDFS_PREFIX`, the `hpwd` command lists the corresponding location in *HDFS*.  The result has the form: `hdfs://host/path`.  The command `hpwd -r` lists only the `path` component, while `hpwd -p` lists only the `hdfs://host/` component.

[hls](file:///Users/stu/Research/HDFS-Tools/hls)
: List files from *HDFS*.  `hls [path ..]` lists files from *HDFS* that correspond to `path`; e.g. `hdfs://host/[path ..]`.  When the current working directory resides under `HDFS_PREFIX`, the path is relative to it; e.g. `hdfs://host/CWD/[path ..]`.  A recursive directory listing is produced with a `-r` flag.

[hget](file:///Users/stu/Research/HDFS-Tools/hget)
: Retrieve files from *HDFS*.  `hget [path ..]` copies the corresponding files from *HDFS* to the local filesystem.  Directories will not be created unless the `-p` flag is present.  Local files will not be overwritten, unless the `-f` flag is included.

[hput](file:///Users/stu/Research/HDFS-Tools/hput)
: Copy files to *HDFS*.  `hput [path ..]` copies local files to the corresponding locations in *HDFS*.  *HDFS* files will not be overwritten, unless the `-f` flag is included.

[hconnect](file:///Users/stu/Research/HDFS-Tools/hconnect)
: Connect to a remote *HDFS*.  `hconnect` opens or closes an ssh tunnel for communication with remote *HDFS*.

[henv](file:///Users/stu/Research/HDFS-Tools/henv)
: This is a configuration script for *HDFS-Tools* auto-completion.

### Notes

 * Use option `-h` to display help for a command, and `-v` for extra debugging information.
 * When the current working directory is outside of `HDFS_PREFIX`, *HDFS-Tools* behave as though they have been invoked with the current working directory set to `HDFS_PREFIX`.
 * One drawback of *HDFS-Tools* is that filename globbing is not supported, so you can not do things like `hget '[io]*'`.

## Installation & Setup

*HDFS-Tools* are available on [GitHub](https://github.com/tub78).

Note: *HDFS-Tools* are configured for use with Hadoop 0.21.0.

### Bare Minimum

 1. Install these scripts somewhere on your path
 1. `HDFS_PREFIX` - Select the _local_ directory where you wish to mirror *HDFS*
 1. `HADOOP_CONF_DIR` - Select the directory containing the active configuration, in order to lookup information on *HDFS*
 1. Add the following line to your `.bash_profile`
  ``` bash
  source <HDFS-TOOLS>/henv
  ```

### For Remote Connections

 1. `HDFS_USER` - Set the user name used to connect to the remote hadoop filesystem
 1. `HDFS_HOST` - Set the host
 1. `HDFS_PORT` - Set the port

`hconnect` opens an ssh tunnel to the remote host using `ssh -ND $HDFS_PORT $HDFS_USER@$HDFS_HOST`



## Examples Part 1

The first set of examples demonstrate the behavior of *HDFS-Tools* with `CWD=HDFS_PREFIX`, where `HDFS_PREFIX=~/Data/Hdfs-2011-08-28`.

### List Files

 1. -> `hls`

``` bash
  Found 3 items
  drwxr-xr-x   - stu supergroup          0 2011-09-03 21:50 /Users
  drwxr-xr-x   - stu supergroup          0 2011-09-03 21:51 /jobtracker
  drwxr-xr-x   - stu supergroup          0 2011-09-03 21:51 /user
```


 1. -> `hls -v user/stu`

``` bash
  HDFS_PREFIX=/Users/stu/Data/Hdfs-2011-08-28
  HDFS_PWD=
  HDFS_URL=/user/stu/input/hdfs-site.xml
  Found 2 items
  drwxr-xr-x   - stu supergroup          0 2011-09-03 21:45 /user/stu/input
  drwxr-xr-x   - stu supergroup          0 2011-09-03 21:51 /user/stu/output
```


 1. -> `hls -v not/a/valid/file`
  ``` bash
  HDFS_PREFIX=/Users/stu/Data/Hdfs-2011-08-28
  HDFS_PWD=
  HDFS_URL=not/a/valid/file
  ls: Cannot access hdfs://localhost:9000//not/a/valid/file: No such file or directory.
  ```


### Get Files

 1. -> `hget /user/stu/output`
  ``` bash
  hget > Local path already exists /Users/stu/Data/Hdfs-2011-08-28/user/stu/output/
  ```

 1. -> `hget -vf /user/stu/output`
  ``` bash
  hget > Local path already exists /Users/stu/Data/Hdfs-2011-08-28/user/stu/output/
  HDFS_PREFIX=/Users/stu/Data/Hdfs-2011-08-28
  HDFS_PWD=
  HDFS_URL=user/stu/output/
  LOCAL_URL=/Users/stu/Data/Hdfs-2011-08-28/user/stu/output/
  LOCAL_DIR=/Users/stu/Data/Hdfs-2011-08-28/user/stu
  ```


### Put Files

 1. -> `hput /user/stu/output`
  ``` bash
  put: Target hdfs://localhost:9000/user/stu/output is a directory
  ```


 1. -> `hput -vf /user/stu/output`
  ``` bash
  HDFS_PREFIX=/Users/stu/Data/Hdfs-2011-08-28
  HDFS_PWD=
  HDFS_URL=user/stu/output
  LOCAL_URL=/Users/stu/Data/Hdfs-2011-08-28/user/stu/output
  HDFS_DIR=user/stu
  ```


### Tab Completion

 1. -> `hls <TAB>`
  ``` bash
  Users       jobtracker  user
  -> hls *
  ```

 1. -> `hget u<TAB>`
  ``` bash
  -> hget user/stu *
  ```

 1. -> `hput user/stu<TAB>`
  ``` bash
  /user/stu/input   /user/stu/output
  -> hput /user/stu/ *
  ```

 1. -> `hput user/stu/<TAB>`
  ``` bash
  /user/stu/input   /user/stu/output
  -> hput /user/stu/*
  ```


## Examples Part 2

When the `CWD` is located below `HDFS_PREFIX`, *HDFS-Tools* use relative paths.

 1. -> `hget <TAB>` 
  ``` bash
  input   output
  -> hget *
  ```



## Examples Part 3

When the `CWD` is not below `HDFS_PREFIX`, *HDFS-Tools* behave as though they were involked from `HDFS_PREFIX`.  The only difference is that paths on the command line are prefixed with `/`.

 1. -> `hls` 
  ``` bash
  Found 3 items
  drwxr-xr-x   - stu supergroup          0 2011-09-03 21:50 /Users
  drwxr-xr-x   - stu supergroup          0 2011-09-03 21:51 /jobtracker
  drwxr-xr-x   - stu supergroup          0 2011-09-03 21:51 /user
  ```


 1. -> `hls <TAB>` 
  ``` bash
  /Users       /jobtracker  /user
  -> hls /*
  ```


 1. -> `hput /use<TAB>` 
  ``` bash
  -> hput /user/ *
  ```


 1. -> `hget /user/stu/input` 
  ``` bash
  hget > Local path already exists /Users/stu/Data/Hdfs-2011-08-28/user/stu/input
  ```


## Examples Part 4

 1. -> `hconnect -c`
  ``` bash
  ENABLED:  0
  RUNNING PROCESS: 
  ```

 1. -> `hconnect -t`
  ``` bash
  ENABLED:  0
  PID:
    ssh -ND 2600 sta2013@rodin.med.cornell.edu
  Started HDFS tunnel with PID: '7647'
  ```

 1. -> `hconnect -c`
  ``` bash
  ENABLED:  1
  RUNNING PROCESS:  7647 ssh -ND 2600 sta2013@rodin.med.cornell.edu
  ```

 1. -> `hconnect`
  ``` bash
  ENABLED:  1
  PID:  7647
  Stopping HDFS tunnel with PID: '7647'
  + kill -9 7647
  ```


(_A duplicate post appears on my blog [stuartjandrews.blogspot.com](http://stuartjandrews.blogspot.com/2011/09/hadoop-filesystem-tools.html)_)



<!-- vim: set ft=vimwiki: -->
