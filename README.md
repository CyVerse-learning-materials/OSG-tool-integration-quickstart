## Creating an environment for testing OSG-RMTA app integration:

**Note:** The following commands worked on one of our server (Rogue). Here are the configurations for the same

```
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

The following files need not to be changed. They can be used without further modification.

1. create-tickets.sh

2. job

3. upload-files

The following files need to be created beforing using them..

1. Create input and output files 

1.1 Create an input file containing the paths to the input files on the datastore. Here is an example file (`input-paths2.txt`) 

```
$ cat input-paths2.txt
/iplant/home/upendra_35/osg-rmta/sample_1_R1.fq.gz
/iplant/home/upendra_35/osg-rmta/sample_1_R2.fq.gz
/iplant/home/upendra_35/osg-rmta/Sorghum_bicolor.Sorbi1.20_chr8.gtf
/iplant/home/upendra_35/osg-rmta/Sorghum_bicolor.Sorbi1.20.dna.toplevel_chr8.fa
```

1.2 Create an output file containing the path to the output folder on the data store. Here is an example file (`output-paths2.txt`)

```
$ cat output-paths2.txt
/iplant/home/upendra_35/osg-rmta/output2
```

2. Create tickets using `create-tickets.sh` script

2.1 Create input tickets for `input-paths2.txt` file. Here is an example file `input_ticket.list`

```
$ ./create-tickets.sh -r input-paths2.txt > test_out/input_ticket.list

$ cat input_ticket.list
# application/vnd.de.path-list+csv; version=1
51505b93cba04b399d7d3b2696069d,/iplant/home/upendra_35/osg-rmta/sample_1_R1.fq.gz
d6092bbc07b048ecbd5392454f98f5,/iplant/home/upendra_35/osg-rmta/sample_1_R2.fq.gz
d822768972cb4b0b898ee18545cf3f,/iplant/home/upendra_35/osg-rmta/Sorghum_bicolor.Sorbi1.20_chr8.gtf
caa59de25b9c415d964b7d85474d3a,/iplant/home/upendra_35/osg-rmta/Sorghum_bicolor.Sorbi1.20.dna.toplevel_chr8.fa
```

2.2 Create output tickets for `output-paths2.txt` file

```
$ ./create-tickets.sh -w output-paths2.txt > test_out/output_ticket.list

$ cat output_ticket.list# application/vnd.de.path-list+csv; version=1
955e2224f8d2493b8194378322eb1d,/iplant/home/upendra_35/osg-rmta/output2
```

3. Create a config file

```
$ cat config.json
{
    "arguments": [
        "-g",
        "Sorghum_bicolor.Sorbi1.20.dna.toplevel_chr8.fa",
        "-A",
        "Sorghum_bicolor.Sorbi1.20_chr8.gtf",
        "-l",
        "FR",
        "-1",
        "sample_1_R1.fq.gz",
        "-2",
        "sample_1_R2.fq.gz",
        "-O",
        "final_out",
        "-p",
        "6",
        "-5",
        "0",
        "-3",
        "0",
        "-m",
        "20",
        "-M",
        "50000",
        "-q",
        "-t",
        "-f",
        "2",
        "-k",
        "2"
    ],
    "irods_host": "davos.cyverse.org",
    "irods_port": 1247,
    "irods_job_user": "upendra_35",
    "irods_user_name": "job",
    "irods_zone_name": "",
    "input_ticket_list": "input_ticket.list",
    "output_ticket_list": "output_ticket.list",
    "status_update_url": "https://de.cyverse.org/job/bd1a1b53-9a7e-4031-bf0c-227a0c63f555/status",
    "stdout": "out.txt",
    "stderr": "err.txt"
}
```

5. Create your wrapper script for your application. Here is the wrapper script for RMTA (`Hisat2-Cuffcompare-Cuffmerge.sh`)

6. Modify the `wrapper` script to suit to your application. Here is an example for OSG-RMTA.

```
def run_job(arguments, output_filename, error_filename):
    with open(output_filename, "w") as out, open(error_filename, "w") as err:
        rc = subprocess.call(["Hisat2-Cuffcompare-Cuffmerge.sh"] + arguments, stdout=out, stderr=err)
        if rc != 0:
            raise Exception("Hisat2-Cuffcompare-Cuffmerge.sh returned exit code {0}".format(rc))
```

7. Finally create Dockerfile

8. Build the Dockerimage from the Dockerfile

```
docker build -t upendradevisetty/osg-rmta:1.0 .
```

9. Push it to the Dockerhub

```
docker login
docker push upendradevisetty/osg-rmta:1.0
```

10. Test Docker image now.

10.1 Pull the Docker image as singularity file (.sif)

```
singularity pull docker://upendradevisetty/osg-rmta:1.0
```

10.2 Run the wrapper script in the singularity image

```
singularity exec osg-rmta_1.0.sif wrapper
```

Note: Before testing, make sure to remove the `~/.irods/.irodsA` file if you have one in your environment

Note: Also it will prompt to enter several times your password, if so, then keep pressing the enter until the job is successfully finished. The output files will be uploaded to your output folder in datastore