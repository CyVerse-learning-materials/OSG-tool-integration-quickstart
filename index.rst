.. include:: cyverse_rst_defined_substitutions.txt

Making OSG-RMTA app in CyVerse's DE
===================================

This repo contains the complete notes for building OSG-RMTA app in DE which is mainly intended for high-throughput processing of RNA-Seq data using `RMTA <https://github.com/Evolinc/RMTA>`_ which runs on OSG through DE. 

Basic steps for building OSG-RMTA app in DE are as follows:
-----------------------------------------------------------


#. 
   Create a Dockerfile for OSG-RMTA

#. 
   Build and push the OSG-RMTA Docker image to DockerHub

#. 
   Test OSG-RMTA Docker image

#. 
   Submit a pull request to OSG github repo

#. 
   Integrate DE tool using “Add Tools” option in DE

#. 
   Make DE app for OSG tool using App Builder in DE

#. 
   Create a Docker image for OSG-RMTA

This is the first step in the process of making a OSG-RMTA app in DE. The minimum requirements for creating a Docker image include

1.1 Ubuntu Operating system (preferred 16.04 and beyond)

1.2 Directories named ``/cvmfs`` ``/work``

1.3 iRODS icommands version 4.0 or above.

1.4 An executable wrapper (\ ``wrapper``\ ) script

1.5 An upload file (\ ``upload-files``\ ) at /usr/bin in the container

Here is the OSG-RMTA `Dockerfile <https://github.com/upendrak/osg-rmta/blob/master/Dockerfile>`_ that satisfies the above requirements. 


#. Build and push the OSG-RMTA Docker image to Dockerhub

Once you create the Dockerfile, make sure you build the Docker image and push it to Dockerhub

.. code-block::

   $ docker build -t upendradevisetty/osg-rmta:1.0 .
   $ docker push upendradevisetty/osg-rmta:1.0


#. Test OSG-RMTA Docker image

Before making a OSG-RMTA DE app, make sure you test your Docker image thoroughly. It would be very hard to troubleshoot the app once it is made. Testing of OSG-RMTA docker image can be done in two ways: Locally using Singularity and on Open Science Grid (OSG)

3.1. Local: Creating an environment for testing OSG-RMTA docker image locally

Since singularity will be used for testing local images, make sure you install Singularity on your computer/server

The following files in this repo need not have to be changed. They can be used without further modification.


#. 
   `create-tickets.sh <https://github.com/upendrak/osg-rmta/blob/master/create-tickets.sh>`_

#. 
   `job <https://github.com/upendrak/osg-rmta/blob/master/job>`_

#. 
   `upload-files <https://github.com/upendrak/osg-rmta/blob/master/upload-files>`_

The following files need to be created before using them.


#. Create input and output files of sample data

1.1 Create an input file containing the paths to the input files on the datastore. Here is an example file (\ ``input-paths2.txt``\ ) 

.. code-block::

   $ cat input-paths2.txt
   /iplant/home/upendra_35/osg-rmta/sample_1_R1.fq.gz
   /iplant/home/upendra_35/osg-rmta/sample_1_R2.fq.gz
   /iplant/home/upendra_35/osg-rmta/Sorghum_bicolor.Sorbi1.20_chr8.gtf
   /iplant/home/upendra_35/osg-rmta/Sorghum_bicolor.Sorbi1.20.dna.toplevel_chr8.fa

1.2 Create an output file containing the path to the output folder on the data store. Here is an example file (\ ``output-paths2.txt``\ )

.. code-block::

   $ cat output-paths2.txt
   /iplant/home/upendra_35/osg-rmta/output

..

   Make sure that the output folder is empty, otherwise you will get errors



#. Make a ``test_run`` folder

.. code-block::

   $ mkdir test_run && cd test_run


#. Create tickets using ``create-tickets.sh`` script

3.1 Create input tickets for ``input-paths2.txt`` file. Here is an example file ``input_ticket.list``

.. code-block::

   $ ../create-tickets.sh -r ../input-paths2.txt > input_ticket.list

Let's look at the contents of the ``input_ticket.list`` file:

.. code-block::

   $ cat input_ticket.list
   # application/vnd.de.path-list+csv; version=1
   51505b93cba04b399d7d3b2696069d,/iplant/home/upendra_35/osg-rmta/sample_1_R1.fq.gz
   d6092bbc07b048ecbd5392454f98f5,/iplant/home/upendra_35/osg-rmta/sample_1_R2.fq.gz
   d822768972cb4b0b898ee18545cf3f,/iplant/home/upendra_35/osg-rmta/Sorghum_bicolor.Sorbi1.20_chr8.gtf
   caa59de25b9c415d964b7d85474d3a,/iplant/home/upendra_35/osg-rmta/Sorghum_bicolor.Sorbi1.20.dna.toplevel_chr8.fa

3.2 Create output tickets for ``output-paths2.txt`` file

.. code-block::

   $ ../create-tickets.sh -w ../output-paths2.txt > output_ticket.list

Let's look at the contents of the ``output_ticket.list`` file:

.. code-block::

   $ cat test_out/output_ticket.list
   # application/vnd.de.path-list+csv; version=1
   955e2224f8d2493b8194378322eb1d,/iplant/home/upendra_35/osg-rmta/output3


#. Create a ``config.json`` file. Here is an example of ``config.json`` for the OSG-RMTA tool

.. code-block::

   $ cat test_out/config.json
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

This is similar to running on the commandline like this..

.. code-block::

   $ ./Hisat2-Cuffcompare-Cuffmerge.sh -g Sorghum_bicolor.Sorbi1.20.dna.toplevel_chr8.fa -A Sorghum_bicolor.Sorbi1.20_chr8.gtf -l "FR" -1 sample_1_R1.fq.gz -2 sample_1_R2.fq.gz -O final_out -p 6 -5 0 -3 0 -m 20 -M 50000 -q -t -f 2 -k 2


#. Pull the Docker image as singularity file (.sif). You need to have Singularity installed first inorder to run this..

.. code-block::

   $ singularity pull docker://upendradevisetty/osg-rmta:1.0

..

   Before you run this, make sure that you remove the irods password onto your system by running ``rm ~/.irods/.irodsA``



#. Test the singularity file now

..

   It will prompt you to enter your irods passwords several times, if so, then keep pressing the ENTER until the job is successfully finished. The output files will be uploaded to your output folder in datastore.


.. code-block::

   $ singularity exec osg-rmta_1.0.sif ../wrapper

   running: configuration successfully loaded
   running: initializing the iRODS connection
   running: downloading the input files
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   running: processing the input files
   running: uploading the output files
   running: excluded files: [u'.job.ad', u'.machine.ad', u'_condor_stderr', u'_condor_stdout', u'condor_exec.exe', u'.chirp_config', u'.chirp.config', u'logs/logs-stderr-output', u'logs/logs-stdout-output', u'config', u'job', u'iplant.cmd', 'job', 'config.json', u'input_ticket.list', u'output_ticket.list', 'sample_1_R1.fq.gz', 'sample_1_R2.fq.gz', 'Sorghum_bicolor.Sorbi1.20_chr8.gtf', 'Sorghum_bicolor.Sorbi1.20.dna.toplevel_chr8.fa']
   running: paths ['out.txt', 'err.txt', 'final_out', 'index']
   Enter your current iRODS password:Enter your current iRODS password:
   Enter your current iRODS password:Enter your current iRODS password:Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:Enter your current iRODS password:
   Enter your current iRODS password:Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:
   Enter your current iRODS password:completed: job completed successfully


#. Job outputs

Once your job has finished, you should expect to see the input and output files in the current working directory and also in the in the output directory (\ ``/iplant/home/upendra_35/osg-rmta/output``\ ). If everything was successful, it should have returned:


* ``final_out`` which contains bam, gtf and other files
* ``index`` which contains the indices of the reference genome
* ``err.txt`` and ``out.txt``

.. code-block::

   $ ils /iplant/home/upendra_35/osg-rmta/output
   /iplant/home/upendra_35/osg-rmta/output:
     err.txt
     out.txt
     C- /iplant/home/upendra_35/osg-rmta/output/final_out
     C- /iplant/home/upendra_35/osg-rmta/output/index


#. Creating an environment for testing OSG-RMTA docker image on OSG

In order to run OSG-RMTA on OSG, you first need to have account with OSG. Register for an account at `osg-connect <http://osgconnect.net/>`_. After you register, you need to add your public keys because OSG no longer allow password access. You can find more information `here <https://support.opensciencegrid.org/support/solutions/articles/12000027675-generate-ssh-key-pair-and-add-the-public-key-to-your-account>`_ 


#. Login to Submit Host

.. code-block::

   $ ssh <username>@login.osgconnect.net # username is your username


#. Download the sample data into a directory

.. code-block::

   $ mkdir sample_data_osg
   $ wget https://raw.githubusercontent.com/upendrak/osg-rmta/master/test_run/sample_data/Sorghum_bicolor.Sorbi1.20.dna.toplevel_chr8.fa
   $ wget https://raw.githubusercontent.com/upendrak/osg-rmta/master/test_run/sample_data/Sorghum_bicolor.Sorbi1.20_chr8.gtf
   $ wget https://github.com/upendrak/osg-rmta/raw/master/test_run/sample_data/sample_1_R1.fq.gz
   $ wget https://github.com/upendrak/osg-rmta/raw/master/test_run/sample_data/sample_1_R2.fq.gz


#. Create an executable script. Here is an example of executable script

.. code-block::

   $ cat osg-rmta.sh

   #!/bin/bash
   Hisat2-Cuffcompare-Cuffmerge.sh -g Sorghum_bicolor.Sorbi1.20.dna.toplevel_chr8.fa -A Sorghum_bicolor.Sorbi1.20_chr8.gtf -l "FR" -1 sample_1_R1.fq.gz -2 sample_1_R2.fq.gz -O final_out -p 6 -5 0 -3 0 -m 20 -M 50000 -q -t -f 2 -k 2


#. Create a wrapper script. Here is an example of wrapper script

.. code-block::

   $ cat osg-rmta-wrapper.sh

   #!/bin/bash
   bash osg-rmta.sh > osg-rmta.out


#. Create a job description file (submit script). Here is an example of job description file

.. code-block::

   $ cat osg-rmta.submit

   # The UNIVERSE defines an execution environment. You will almost always use VANILLA.
   Universe = vanilla

   # These are good base requirements for your jobs on OSG. It is specific on OS and
   # OS version, core cound and memory, and wants to use the software modules. 
   Requirements = HAS_SINGULARITY == True
   request_cpus = 1
   request_memory = 2 GB
   request_disk = 4 GB

   # Singularity settings
   +SingularityImage = "docker://upendradevisetty/osg-rmta:1.0"
   +SingularityBindCVMFS = false

   # EXECUTABLE is the program your job will run It's often useful
   # to create a shell script to "wrap" your actual work.
   Executable = osg-rmta-wrapper.sh
   Arguments =

   # inputs/outputs
   transfer_input_files = osg-rmta.sh, Sorghum_bicolor.Sorbi1.20.dna.toplevel_chr8.fa, Sorghum_bicolor.Sorbi1.20_chr8.gtf, sample_1_R1.fq.gz, sample_1_R2.fq.gz
   transfer_output_files = final_out, index

   # ERROR and OUTPUT are the error and output channels from your job
   # that HTCondor returns from the remote host.
   Error = $(Cluster).$(Process).error
   Output = $(Cluster).$(Process).output

   # The LOG file is where HTCondor places information about your
   # job's status, success, and resource consumption.
   Log = $(Cluster).$(Process).log

   # Send the job to Held state on failure. 
   on_exit_hold = (ExitBySignal == True) || (ExitCode != 0)

   # Periodically retry the jobs every 1 hour, up to a maximum of 5 retries.
   periodic_release =  (NumJobStarts < 5) && ((CurrentTime - EnteredCurrentStatus) > 60*60)

   # QUEUE is the "start button" - it launches any jobs that have been
   # specified thus far.
   Queue 1


#. Submit the job to OSG

.. code-block::

   $ condor_submit osg-rmta.submit


#. Check the job status

The ``condor_q`` command tells the status of currently running jobs. Generally you will want to limit it to your own jobs by adding your own username to the command.

.. code-block::

   $ condor_q <username>


#. Job outputs

Once your job has finished, you can look at the files that HTCondor has returned to the working directory. If everything was successful, it should have returned:


* 
  ``final_out`` which contains bam, gtf and other files

* 
  ``index`` which contains the indices of the reference genome

After the Docker image worked, I created a new github repo under https://github.com/Evolinc and moved the Dockerfile and other files that are needed for building the Docker image usign automated build. the final image after automated building is - ``evolinc/osg-rmta:2.1``. 


#. Submit a pull request to OSG github repo for ``evolinc/osg-rmta:2.1`` image

Once the Docker image works, open a `PR <https://github.com/opensciencegrid/cvmfs-singularity-sync/pull/101>`_ at OSG github repo. Ping Mats if you think it takes some time.

After the PR is merged, it takes few hours for the image to be available on CVMFS. You can check that under ``"/cvmfs/singularity.opensciencegrid.org/``. OSG-RMTA docker image is here ``/cvmfs/singularity.opensciencegrid.org/evolinc/``. Next, test this new image using the same procedure as above except this in the job descriptor file.

.. code-block::

   # Singularity settings
   +SingularityImage = "/cvmfs/singularity.opensciencegrid.org/evolinc/osg-rmta:2.1"

Once it works, then procede to the next step.


#. Integrate DE tool using “Add Tools” option in DE

After integrating the DE, ask one of the DET team member to change the tool type from DE to OSG (Hopefully this will eventually change)


#. Make DE app for OSG tool using App Builder in DE

This is the last step. After this, go ahead and test it out..
