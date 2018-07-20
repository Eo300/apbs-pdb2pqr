Note:  These instructions are meant to reproduce the steps necessary to build the apbs-pdb2pqr repository located at https://github.com/Electrostatics/apbs-pdb2pqr.  It is recommended you install all the necessary libraries within a virtualenv machine.

Setup Instructions for Amazon Linux EC2 Instance
The following steps assumes that the user is in the shell via SSH and possesses sudo permissions.  Permission to sudo should be on by default if singed in as ec2-user.

# First, a couple things...
	• Take note of the hostname/public IP address of the machine you'll be installing on
		○ This will be used to connect to its web server from your browser
		○ This information can be found within the EC2 section of the AWS console, under the description of a selected machine.
	• Be sure to configure the Security Group (more or less, the firewall settings) to allow inbound HTTP connections (port 80) from any destination.
		○ To accept from anywhere, set the Source to "0.0.0.0/0" (should be this by default)

# Installing Apache Web Server
	• Command line
		○ sudo yum install httpd
		○ 
# Installing APBS (Python libraries necessary for PDB2PQR)
	• make sure build directory is accessible with read and execute permission from public
	• Our ultimate goal here is to obtain two files necessary to build PDB2PQR
		○ apbslib.py
		○ _apbslib.so
	• Make sure you have access to the following packages, and if not, use "sudo yum install" to obtain
		○ gcc
		○ g++
		○ cmake
		○ swig
	• In the top directory of the repository, <ROOT DIRECTORY>/apbs-pdb2pqr/, import submodules
		○ $ git submodule init
		○ $ git submodule update
	• Navigate to the directory you wish to contain the build files for APBS
		○ Will refer to this as <APBS INSTALLATION DIR> herein
	• Use CMake to build files necessary to make APBS
		○ $ cmake <ROOT DIRECTORY>/apbs-pdb2pqr/apbs/ -DENABLE_PYTHON=ON -DBUILD_SHARED_LIBS=ON
	• Assuming all went well, there should be a <APBS INSTALLATION DIR>/tools/python/ should have been made
	• Lastly, make the APBS software
		○ From within the <APBS INSTALLATION DIR>
		○ $ make
	• You should now have access to the two target files we need to build PDB2PQR:
		○ <APBS INSTALLATION DIR>/tools/python/apbslib.py
		○ <APBS INSTALLATION DIR>/lib/_apbslib.so
	• Copy these files into your PDB2PQR source directory
		○ From within <APBS INSTALLATION DIR>
			§ $ mkdir -p <ROOT DIRECTORY>/apbs-pdb2pqr/pdb2pqr/abps_libs/linux/
			§ $ cp ./lib/_apbslib.so ./tools/python/apbslib.py <ROOT DIRECTORY>/apbs-pdb2pqr/pdb2pqr/apbs_libs/linux/
	
# Installing PDB2PQR
	• Make sure build directory is accessible with read and execute permission from public
		○ You may run into issues still, if the directories above it are not read/execute permissible.  If so, double-check these as well
	• With the apbslib files now in the pdb2pqr directory of your repository, there are two methods of deployment
		○ Directly using running the scons.py script in the scons directory
		○ Using fabric to perform a proper deployment
	• Configure the necessary build files, build_config.py and fabfile_settings.py
		○ Within both, you'll see variables referencing particular paths
			§ PREFIX: Desired path to the build directory for pdb2pqr
			§ URL:    Desired URL you wish to access the PDB2PQR server. Used to define paths within HTML
			§ APBS:   Path to your APBS build executable from the previous section
		○ Additional relevant variables within build_config.py
			§ MAX_ATOMS	Sets the maximum number of atoms in a protein. Only affects web tools. Default is 10000.
		○ Additional relevant variables within fabfile_settings.py
			§ linux_host	address for the host you're building on. Ex. "mysamplewebsite.com" if building for your own domain
			run_tests	Defaults to "false". Set to true if you want to run through test suite
# Building Via scons.py Script
	• Navigate to <ROOT DIRECTORY>/apbs-pdb2pqr/pdb2pqr/
		○ $ python scons/scons.py
		○ $ python scons/scons.py install
# Building Via Fabric (Recommended)
	• Will need the following packages within environment
		○ Openssh-server
	• Be sure to pip install the following libraries
		○ If running built-in tests
			§ numpy
			§ Networkx
		○ fabric (version 1.13.1)
	• Navigate to <ROOT DIRECTORY>/apbs-pdb2pqr/pdb2pqr/
	• Build
		○ If using password for SSH:	$ fab deploy_and_install
		○ If using SSH key for SSH: 	$ fab deploy_and_install -i <SSH key path>
	


