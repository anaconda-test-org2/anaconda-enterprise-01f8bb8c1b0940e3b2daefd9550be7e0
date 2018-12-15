# Hadoop, Spark, Hive and Impala project template

This project includes the libraries and example notebooks necessary to connect to multiple services in the Hadoop ecosystem including Spark, Impala, Hive and HDFS.

## Authenticating with Kerberos

Many *Hadoop* installations are secured using Kerberos. In order to authenticate with
Kerberos, your system administrator must at least have provided a configuration file, normally located at `/etc/krb5.conf`.

To perform the authentication, open an environment-based terminal in the interface; this
is normally in the launchers panel, bottom row of icons, the right-most icon. When the interface
appears, execute
```
kinit myname@DOMAIN.COM
```
where the email-like argument is your Kerberos *principal*, i.e., combination of your username
and security domain. The correct value should be provided to you by your administrator.

Executing the command requires you to enter a password. If there is no error, authentication
has succeeded. You can check by issuing the `klist` command and ensuring it provides some
entries.

Once that you have been authenticated you can access the *Hadoop* services as shown in the respective examples.

NOTE: Kerberos authentication will lapse after some time, requiring you to repeat the above
process. It is common to have credentials last for 24 hours, but the actual length of time
is determined by your cluster security administration.

## Spark

This project includes Sparkmagic, so that you can connect to a Spark cluster with a running
Livy server.

You can use Spark in two ways:
- Starting a notebook with one of the Spark kernels, in which
case all code will be executed on the cluster except for cells explicitly marked with
`%%local`. Note that a connection and all cluster resources will be assigned as soon
as you execute any ordinary code cell (i.e., not a cell marked as local).
- Starting a normal notebook with a Python kernel, and using `%load_ext sparkmagic.magics`.
That command will enable a set of functions to run code on the cluster, see
[examples](https://github.com/jupyter-incubator/sparkmagic/blob/master/examples/Magics%20in%20IPython%20Kernel.ipynb)
(external link).

To display graphical output directly from the cluster, you must use SQL commands. This
is also the only way to have results passed back to your local Python kernel, so that you
can do further manipulation on it with `pandas` or other packages.

You may also run non-Spark notebooks in this project, where you can obtain the outputs
of Spark calculations using interface libraries like `hdfs3` (for data on HDFS) and `s3fs`
(for data in an S3-like storage system). Both of these will require some configuration and/or
credentials.

In the common case, the configuration provided for you in the Session will be correct and
not require modification. However, there are instructions for modifications that you may need
for sandbox or ad-hoc environments, see below.

### Using Custom Python Paths

If a CDH Parcel, HDP MPack, or other Custom Python has been installed on the target cluster,
you can configure Sparkmagic to use this custom Python path by setting the following in
first cell in a Sparkmagic-based Jupyter Notebook

```
%%configure -f
{"conf": {"spark.yarn.appMasterEnv.PYSPARK_PYTHON": "/opt/anaconda/bin/python",
         "spark.yarn.appMasterEnv.PYSPARK_DRIVER_PYTHON": "/opt/anaconda/bin/python",
         "spark.yarn.executorEnv.PYSPARK_PYTHON": "/opt/anaconda/bin/python",
         "spark.pyspark.python": "/opt/anaconda/bin/python",
         "spark.pyspark.driver.python": "/opt/anaconda/bin/python"
        }
}
```

*See Notebook: pyspark-custom-python.ipynb*

### Overriding session settings

Certain jobs may require more cores or memory, or custom environment variables, such as
Python worker settings. The configuration passed to Livy is generally defined in a file
`~/.sparkmagic/conf.json`, which has been provided to you. You may inspect this file,
particularly the section `"session_configs"`, or you may refer to the example file in this
directory, `sparkmagic_conf.example.json` (which has not been tailored to your specific
cluster).

In a Sparkmagic kernel (PySpark, SparkR, etc.), you can change the configuration with the
magic `%%configure` to change options. The syntax of this is pure JSON, and the values are
passed directly to the driver application.

EXAMPLE:

```
%%configure -f
{"executorMemory": "4G", "executorCores":4}
```

If you are using a Python kernel and have done `%load_ext sparkmagic.magics`, you can use
the `%manage_spark` command to set configuration options; the session options are in the
"Create Session" pane, under "Properties".

### Overriding basic settings

In some more experimental situations, it may be desirable to change the Kerberos of Livy
connection settings, perhaps when first configuring the platform for a cluster. This is more
likely to be done by an administrator with intimate knowledge of the cluster's
security model, than by a regular data scientist.

In these cases, we recommend creating a `krb5.conf` file and a sparkmagic_conf.json file in the
project directory (so they will be saved along with the project itself). An example Sparkmagic
configuration is provided, `sparkmagic_conf.example.json`, listing the fields that are
typically set. Of particular importance are the `"url"` and `"auth"` keys in each of the
kernel sections.

The `krb5.conf` file is normally taken from the Hadoop cluster, rather than written by hand,
and may make reference to additional config or certificate files. These must all be uploaded
via the interface.

In order to make use of these alternate configuration files, set the `KRB5_CONFIG` variable
default to point to the full path of `krb5.conf` and set the values of `SPARKMAGIC_CONF_DIR`
and `SPARKMAGIC_CONF_FILE` to point to the sparkmagic config file. You can set these either
using the Project pane on the left of the interface, or by directly editing the
`anaconda-project.yml` file.

For example, the file may end up with the variables section
looking like:
```
variables:
  KRB5_CONFIG:
    description: Location of config file for kerberos authentication
    default: /opt/continuum/project/krb5.conf
  SPARKMAGIC_CONF_DIR:
    description: Location of sparkmagic configuration file
    default: /opt/continuum/project
  SPARKMAGIC_CONF_FILE:
    description: Name of sparkmagic configuration file
    default: sparkmagic_conf.json
```

NOTE: You must perform these actions **before** running `kinit` or starting any
notebook/kernel.
