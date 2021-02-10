Running Gunsch Lab RStudio Singularity container 
=============================

Below are instructions for running RStudio Server using one of the Gunsch Lab Singularity containers. These containers have all of the libraries need to run our analysis pipelines. These containers allow users to run our pipeline locally on a personal laptop or remotely on a server or cluster. The only requirement is that Singularity is installed on the host system. Singularity is pre-installed on most clusters so users should be able to run these containers with minimal effort. If installation is necessary, instructions can be found here: https://sylabs.io/guides/3.7/admin-guide/installation.html. Please contact Daniel Rodriguez @ daniel.rodriguez@duke.edu if you run into any errors or have any suggestions. 


## Available Containers
We maintain container images for transcriptomic, metagenomic, or amplicon sequencing analysis on the Sylabs Cloud. 
You can search for available Gunsch Lab containers by visiting https://cloud.sylabs.io/library/gunschlab or by using the following command
```
singularity search gunsch
```



## Running locally
Running locally is generally not ideal due to the limited resources on most laptops. With that said running a container locally is extremely simple. All you have to do download a container, start a singularity instance, and then run that instance. Here is an example:

1) Download
```
singularity pull [pull options...] [output file] <URI>
example: singularity pull microbiome_20200123.sif library://gunschlab/default/gunsch_microbiome:20201023 
```

2) Start instance
```
singularity instance start [start options...] <container path> <instance name> [startscript args...]
example: singularity instance start microbiome_20200123.sif microbiome # the instance name "microbiome" is arbitrary 
```

3a) Run instance
```
example: singularity run instance://microbiome
```
The runscript starts the interactive RSTudio server and prints the url, username, and a randomly generated password. Just past the url into your browser and enter your credentials and you're ready to go. As long as the terminal/command prompt stays open the RStudio session will be active. 

Singularity by default binds the host $HOME directory to the container, so you have access to any files in that directory. If you’d like to work in another directory you can bind that directory to your container by using the --bind option as shown below

3b) Run instance with binding directory
```
example: singularity run --bind /storage instance://microbiome
```

When you're done with your analysis you can stop the instance
4)Stopping the instance
```
example: singularity instance stop microbiome
```

## Running on a server or cluster 

Running remotely is very similar to running locally with two additional steps. 
-connecting to the server to start the container instance. 
-opening another connection to forward the port rstudio is broadcasting on the server to a port on your local machine. Below is the workflow.

1.) Connect to the server
```
ssh username@hostname
example on DCC: ssh dlr12@dcc-login-01.oit.duke.edu #where dlr12 in my netid
```

2.) Start a terminal multiplexer. This step simply ensures that your instance will run even if your connection drops. This example uses Tmux.
```
tmux new -s my_session
example: tmux new -s dan_microbiome
```

3.) Reserve cluster resources using Slurm (this step can be skipped if you connect directly to a server)
```
srun --pty -p <partition_names> --mem=<size[units]> -c<ncpus> bash -i
example using the gunschlab node: srun --pty -p gunschlab --mem=64G -c12 bash -i 
```
The amount of memory needed and cpus varies from job to job. The above example should be sufficient for most processes.

Once Slurm as allocated your resources you can start and run your instance

4.) Start instance
```
singularity instance start [start options...] <container path> <instance name> [startscript args...]
example using the gunschlab node: singularity instance start --bind /work/gunschlab/dlr12 /work/gunschlab/containers/microbiome_20200123.sif microbiome 
#'/work/gunschlab/dlr12' is the directory where my data is stored and '/work/gunschlab/containers/microbiome_20200123.sif' is the path to the container
```
Singularity by default binds the host $HOME directory to the container, so you have access to any files in that directory. However, on DCC the $HOME directory is limited to 5GB of space. You'll want to bind the /work directory to store your working files. This partition is not backed up and space is cleared out every couple of months. Be sure to move any files you want to keep to a permanent location when you're done with your analysis. 

5.) Run instance 
```
example: singularity run instance://microbiome
```
The runscript starts the interactive RSTudio server and prints a url, username, and a randomly generated password. 
```
RStudio URL:		http://dcc-gunschlab-01.rc.duke.edu:50000/
RStudio Username:	dlr12
RStudio Password:	gOB9HyOvj0B5GHl7iQK
```

6.) Open a new terminal and and start a new ssh connection to forward the broadcast port from the server to your local machine
```
ssh -L PORT:PARTITION_HOSTNAME:PORT username@LOGIN_NODE_HOSTNAME
example using gunschlab node: ssh -L 50000:dcc-gunschlab-01:50000 dlr12@dcc-login-01.oit.duke.edu
#where 'dcc-gunschlab-01' is the hostname bit from the url printed by the runscript and 'dcc-login-01.oit.duke.edu' is the original login node we connected to in step 1 
```

7.) open a browser and type in localhost:PORT
```
example from above connection: localhost:50000
#then enter your credentials printed from the runscript and your ready to start your analysis
```
As long as this connection from step 6 is active, you can interact with rstudio in the browser. If you lose the connection rstudio will still be running on the server, so just reconnect using step 6.


Tmux allows you to detach and reattach to a terminals without affecting their state. So you can go ahead and detach and close the terminal from stop 5. You'll only need to reconnect when your analysis is done so you can close out the Singularity instance and release the Slurm reservation.

8.) Detach and close Slurm terminal (the one with the url credentials printed)
click on the terminal
press Ctrl+b and d
the green bar on the bottom of the terminal should now be gone and you can close the terminal.

When you are done with your analysis you need to login to the node, reattach to the Tmux session, stop the Singularity instance, and finally exit out of the Slurm session. 

9.) reconnect to tmux session
```
example: tmux attach -t dan_microbiome
 
```
10.) stop instance 
```
example: singularity instance stop microbiome
```

11.) exit out of the Slurm reservation
```
exit
```

























