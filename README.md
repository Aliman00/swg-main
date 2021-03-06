# SWGSource V1.2 Build Instructions

## Credit
Credit the StellaBellum team (DarthArgus, Cekis, SilentGuardian, Ness, SwgNoobs, DevCodex) for making their repositories open.  All source is forked from those repositories and progressed from that point.

## What Do You Need To Do To Get A Server Running?

Most of what is contained in this section assumes you're running a Linux base VM - specifically Debian.  It is assumed your home directory is /home/swg.

Download the main repo:

		cd /home/swg
		git clone https://github.com/SWG-Source/swg-main.git

When the GIT repository has been cloned successfully, open the swg-main directory:

		cd /home/swg/swg-main

Create a .setup file such that the build script knows where to start:

		touch .setup

## Building
The process is nearly fully automated by executing the build_linux.sh script located in swg-main.  The script will ask you a series of questions with each
question corresponding to a different step in the build process.  Each step is defined below.  To complete building, kick off the build script and follow
these instructions as detailed here to understand each step.

### Starting the Build Process
Follow the instructions of the build script. The binary building phase will take roughly 1 hour for each CPU core assigned to your VM.  Do not skip any steps.
Execute the following script to start building (see detail on each step below this):

		./build_linux.sh

#### Cloning the Repositories
The script will first ask if you would like to retrieve any GIT folders necessary (it knows which ones it needs to get).  It will attempt to get the pre-configured
repositories that are outlined in the build script by cloning them.  It will simply do a pull if the repositories already exist to bring them up to date.
This currently includes "src", "dsrc", and others:

		Do you want to pull/update git? (y/n)

If you answer "y" to the question, it will attempt to retrieve the latest version set of the code.  If you hit enter or choose "n" it will skip to the next step.

### Compile Phase
After getting the latest version of the code, the build script will then attempt to build the server.

#### Compile Mode
The system first needs to know how you will be compiling the code.  You have a choice of either DEBUG (d) mode or RELEASE (r) mode.  If you do not enter "d" or "debug" for the mode then it is assumed you wish to build a RELEASE build. (release is recommended unless you have a specific reason to want debug.)

		Is this for DEBUG mode or RELEASE mode? (d/r):

#### Compiling SRC
The first compile step is to build the server C++ code.  This is the source found in the "src" folder.  Once compiled, you should have compiled code in the "build" directory of swg-main:

		Do you want to recompile the server code (C++) now? (y/n)

#### Building the Server Configuration Files
The next step will attempt to build the configuration files.  This step captures data needed for the server to execute properly.  The following information is captured in this step:

		Do you want to build the config environment now? (y/n)
		Enter your IP address (LAN for port forwarding or internal, outside IP for DMZ)

Enter the IP address of the host machine that will be the server.  This is typically your VM IP Address.

		Enter the DSN for the database connection

Enter the name of the database.  This is typically just "//127.0.0.1/swg" without quotes.  It uses the 127.0.0.1 localhost loopback instead of the VM IP on purpose.

		Enter the database username

Enter the user name for the database connection.  This is also typically just "swg" without quotes.

		Enter the database password

Enter the password for the database connection.  This is also typically just "swg" without quotes.

		Enter a name for the galaxy cluster

This will be the cluster name of the galaxy.  Use something creative (or not), this is the name of the galaxy you're going to choose when creating a new character. NOTE: Remember your cluster name because you will need it again in a later step.

#### Compiling DSRC
The next step will be to build the Java code that resides in the dsrc folder.  This code is where 99% of the game content and AI resides.  This step takes a long time so go watch a movie or something while it builds.  Note: It will give you the option of using "multi" or "safe" building.  Multi takes advantage of multi-threaded compiling (typically faster). However, it was found that using it SOMETIMES caused issues, but these issues are likely no around any longer.  Needless to say, if you encounter issues using MULTI, try using Safe instead which does not use multi-threaded compiling:

		Do you want to recompile the scripts (.java)? (y/n)
		
The Script will currently build the ENTIRE dsrc at this step. If you want to build just an individual part, please see `./build_dsrc.sh` information at the bottom of this readme.

#### Build your chat server

This step will (re)build the stationapi chat server.

		Do you want to build stationapi chat server now? (y/n)

#### Importing the Database SQL
This step will attempt to create the database schema from the SQL files that are in the src directory structure.  It will create all database tables, functions, views and stored procedures the game needs to use.  You can run this step without fear of ruining anything that is currently in place.  Keep in mind that if you do attempt to import the database over the top of an existing swg database, it will simply throw errors after a very short attempt, but will not cause issue with your current data.  Basically, you can't mess this part up:

		Import database? (y/n)

If you haven't previously entered the necessary database information above, it will ask you for that information here.

#### Clientdata symlink
The Clientdata Repo is in /home/swg/swg-main/clientdata.  You MUST have this in place and have the symbolic link created before the server can execute successfully:

		Do you want to create the symlink for the clientdata folder? (y/n)

#### Final configuration:

After build_linux.sh is finished, you may want to edit your configuration to turn off/on planets and other zones. The configuration files will be located at `/home/swg/swg-main/exe/linux/` NOT at `/home/swg/swg-main/configs`. (Those are just the templates for the build script.)

## First Start
To start the server after building, execute the following script:

		./startServer.sh

Point your login.cfg in your game folder (on your client machine, NOT the VM) to the IP address of the Virtual Machine such that you can connect successfully.

AND YOU'RE DONE!


## build_dsrc.sh

You may want to build certain parts of the dsrc individually instead of all at once as done in build_linux.sh. Here is some information about the specific steps in build_dsrc.sh:

#### Compiling the mIFF files
This step will compile all *.mif files into *.iff binary files.

		Do you want to build the mIFF files (.mif)? (y/n)

#### Compiling the Datatable files
This step will compile all the *.tab files into *.iff binary files.  Again, speed constraints aside, you have the choice of using MULTI or SAFE for compiling (see DSRC Compiling above):

		Do you want to build the datatables (.tab)? (y/n)

#### Compiling Template Files
This step will compile all the *.tpf files into *.iff binary files.  You have a choice of Multi or Safe here as well:

		Do you want to build the template files (.tpf)? (y/n)

If you have built the TPF files in this step (i.e. you didn't skip this step) then the build script will also attempt to recreate the Object Template and Quest CRC tables and subsequently will attempt to push those changes to the database since this will also be required.  A GREAT feature to have when creating new template files or changing existing ones.

### Database Phase
#### Building Object Template and Quest CRC Files
This step will compile the object template and quest CRC files.  These files translate the long name of these files (including file path) into a very short code that allows the server to identify them without the danger of long text being transferred over the internet in packets.  Basically a optimization that SOE implemented:

		Do you want to build the Object Template or Quest CRC files? (y/n)

Building these files will also trigger the build script to then populate the database with the CRC's that were generated.  If you are doing this build script in pieces (i.e. you're selectively building), this is a GREAT way to re-import new or changed TPF file changes.  In order to re-import CRC's into the database, if you haven't already entered it above, it will ask you for the database information here.
