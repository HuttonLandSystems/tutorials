# Connecting to the Crop Diversity HPC [gruffalo] for climate data 

Assuming you have an HPC login and have worked your way through the [help docs](https://help.cropdiversity.ac.uk) for gruffalo, here is a intro guide to accessing the MetOffice's UKCP18 data which has been downscaled and bias-corrected from 12km to 1km.


## first steps
The language used to interact with the HPC is called ```bash``` and for an introduction, look at the gruffalo help docs or see the many online resources. There is also a very helpful manual built into bash. If you type ```man bash```, for example, the manual for bash is printed in the console. ```man man``` shows the manual for the manual!

I recommend downloading [MobaXTerm SSH client](https://mobaxterm.mobatek.net/) to use as a GUI to interact with gruffalo. You can save a user session (as in PuTTY), but the difference is that you can see directory and file structures.


## directories 
There are two main directories which are user-accessible by default: ```scratch``` and ```projects```, both located in ```/home``` (or equally ```/mnt/shared/username```). Shortcuts for both directories are ```$SCRATCH``` and ```$PROJECTS```.

If you want to set up a new projects directory, you'll have to email one of the admins.


## data
The climate database is located at ```/mnt/shared/projects/jhi/misc/202107_ukcp18_downsample/ukcp18_1km_corrected.db``` on gruffalo.

The database uses SQLite, which is a self-contained, full-featured SQL database engine. You'll need to install it on your instance of the HPC by running ```conda install -c anaconda sqlite``` in the command line. It can then be accessed using the ```sqlite3``` command.

This is good for running basic ```SQL``` queries against the database to see the structure of the tables and data which you can do directly through the shell. For example, 

```bash 
# assign your working directory
cd $PROJECTS/jhi/misc/202107_ukcp18_downsample

# use sqlite3 to select from the db
sqlite3 ukcp18_1km_corrected.db "select * from ukcp18_1km_downscaleandcorrect_uk where id_1km = 1 and year = 1981;"
```
If you are running an interactive sqlite3 session, remember to end all your queries with ```;```


## extract data 
Say for example that you'd like to extract data or calculate new data using the database. 


### [Interactive Jobs](https://help.cropdiversity.ac.uk/slurm-overview.html#interactive-jobs)

For small extractive jobs or quick calculations, you can begin an interactive session using Slurm with the command ```srsh```. This starts a remote shell on a compute node with a default 1.5 GB and 1 CPU alocated to you (although you can alter these defaults if need be). You'll see that your home directory is no longer @gruffalo but at @n19-192-crossbones (e.g.). Keyword ```exit``` will stop your interactive session.


### [Batch Jobs](https://help.cropdiversity.ac.uk/slurm-overview.html#batch-jobs)

Because of the db's size (3TB), you will have to find a way to automate your query if you need a large extraction or analysis. Here is an example of a batch job:

```bash
#!/bin/bash 
#SBATCH --job-name="testjob"
#SBATCH --mem=2GB
#SBATCH --partition=long
#SBATCH --cpus-per-task=2
#SBATCH --mail-user=first.last@hutton.ac.uk
#SBATCH --mail-type=END,FAIL
#SBATCH --chdir=$SCRATCH/directoryname/

# this is a normal comment in your code

# this will give your script a 120 second delay before starting
sleep 120

variable="Hello World!"
echo $variable
```

Save this script as a ```test.sh``` and run it with ```sbatch test.sh```
The shell will return a job id number. 

You can check on the status of your job with ```squeue --me```. Omit the --me and you'll see all the jobs running on the cluster. 

Things that begin with a # are a comment. Those that begin with #SBATCH are a special type of comment that tell SLURM what the parameters of your job are. At the beginning of the script, the ```#!/bin/bash``` is called a shebang and specifies the interpreter to use. In this case, bash.


## quick hints 
* there is no such thing as Ctr+C and Ctrl+V to Copy and Paste into the shell. If you have something copied to your clickboard, click the middle mouse button to paste.
* if you ever find yourself stuck in the interface, use Ctrl+C or Ctrl+D. These are typical keyboard interrupts.
* If you find that you have a long running interactive script, you can send it to the background so you have access to the shell once more. There are two ways to do this. If you have already submitted the job, Ctrl+Z will pause the job and then ```bg``` will send it to the background. The second way is <i>before</i> you submit your job, you can end your command in a ```&``` and it will automatically send it to the background. You can check the status of any job running in the background with the ```jobs``` command.


## essential commands to know
A typical command is made up of: 
* its keyword or command word. 
* 'Options' and/or 'Flags' (begin with a ```-``` for one letter options or ```--``` for named options). 'Options' allow information to be passed to the command, and 'Flags' can be thought of as Boolean options, which invert default command behaviour.
* an argument. This is the input of the command. Best thought of as the thing on which the command is acting.

```pwd``` - where am I? (absolute path)

```ls -l -D``` - a list of the contents of your working directory, with a 'long' flag and a 'date' option

```cd directory/name``` - change directory 

```cat filename.ext``` - gives you a peak into the contents of the file. It comes from the concatonate command, but if you only list one argument, it shows its contents.

```variable='Hello World!'``` - assign a variable (very important that there are no spaces between the variable and the definition)

```echo $variable``` - same as print()

```clear``` cleans up console (keeps variables in memory)

<i>Remember if you're stuck or are unsure of which flags to use, look at the ```man``` pages</i>

