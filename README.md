# dockersap
Let's test a minisap installation.

This is a direct and short guide to install SAP NW ABAP 7.52 at home, if you need further details please refer to the [Acknowledgements](#acknowledgements) section.


## Installation
Tested with NW ABAP 7.52 SP04 on different Windows and Linux Ubuntu machines.

1. Register at SAP.com, you will need it to download the installation files and two test licenses (database and application), at this moment the registration, download and the test licenses are free of charge from SAP.
1. Install Docker, you may need to increase the RAM size (6GB) or Disk limits (100GB).

1. Clone this repository
	```sh
	git clone https://github.com/pmoronm/dockersap
	cd dockersap
	```

1. Download [SAP NetWeaver ABAP 7.52 from SAP](https://developers.sap.com/trials-downloads.html) (search for **7.52**, download Part1 to Part11 and don't forget to download the License file as well):
	- Create the extraction folder:
    
	```sh
	mkdir sapdownloads
	```
	
	- Extract the download contents into it (use your favorite decompressor tool if you prefer):

	```sh
	unrar x [replace-with-download-path]/TD752SP04part01.rar ./sapdownloads/
	```
	
	- Extract the `.lic` file from `License.rar` and copy it to the  `sapdownloads/server/TAR/x86_64` folder.

1. Before going any further, you need to tune the installation script to avoid errors with newer opensuse distributions. Create a backup copy of `sapdownloads/install.sh` and then edit the file, replacing this code fragment:

	```sh
		./saphostexec -install || do_exit $ERR_install_saphost

	# TODO: is it ok to remove /tmp/hostctrl?
		cd /
		rm -rf /tmp/hostctrl || log_echo "Failed to clean up temporary directory"
	```

	with this chunk:

	```sh
	#Replace this line with one which tries to continue (this) main script using ‘&’:
		#./saphostexec -install || do_exit $ERR_install_saphost
		./saphostexec -install &

	#Wait for a while so that hopefully the asynchronous call ends:
		log_echo "Waiting 30 seconds for asynchronous call to /tmp/hostctrl/saphostexec -install to complete..."
		sleep 30
		log_echo "30 seconds are up, continuing the main script."

		# TODO: is it ok to remove /tmp/hostctrl?
		cd /
	#Let's not remove the temporary directory, in case saphostexec command
	#is still executing. So commenting out:
		# rm -rf /tmp/hostctrl || log_echo "Failed to clean up temporary directory"
	```

1. Build the docker image

	```sh
	docker build -t nwabap:7.52 .
	```

1. Create the container from the image you just built

	```sh
	docker run -p 8000:8000 -p 44300:44300 -p 3300:3300 -p 3200:3200 -h vhcalnplci --name nwabap752 -it nwabap:7.52 /bin/bash
	```

1. Now you are inside the container, set **vm.max_map_count** to avoid an installation error

    ```sh
    sudo sysctl -w vm.max_map_count=1000000
    ```

1. Run the installation script, should no prompt to accept the disclaimer text appears, hit Ctrl-C once and you'll see it, then accept with 'yes'
	```sh
	/usr/sbin/uuidd
	./install.sh
	```

	After 20 to 30 minutes the installation should success displaying this message:
	`**Installation of NPL successful**`

1. Stop and exit the container, be sure to learn how to [start](#starting-the-sap-container) and [stop](#stopping-the-sap-container) your container and then perform the [post-install](#post-installation-steps).


## Starting the SAP container
- For normal operation, you have to start the container before login to your SAP installation: 
    ```sh
    docker start -i nwabap752
    /usr/sbin/uuidd
    su npladm
    startsap ALL
    ```

## Stopping the SAP container 
- To stop and exit, just issue the following commands to your container console:
    ```sh
    su npladm
    stopsap
    exit
    exit
    ```

## Post Installation Steps
1. Install the SAP client.

1. Update the License:
	- Open the client (SAP GUI)
	- Login **User** SAP*, **Password** Down1oad, **Client** 000
	- Open transaction `SLICENSE` and copy the key shown at `Active Hardware Key`
	- Head your browser to [SAP License Keys for Preview, Evaluation, and Developer Versions](https://go.support.sap.com/minisap/#/minisap)
    - Choose `NPL - SAP NetWeaver 7.x (Sybase ASE)`
    - Fill out the fields. Use the `Hardware Key` you copied from `SLICENSE`
    - Keep the downloaded file `NPL.txt` and go back to the `SLICENSE`
    - Delete the `Installed License` from the table
    - Press the button `Install` below the table
    - Choose the downloaded file `NPL.txt`
    - Done - happy learning. Now logon with the dev user.

    You can now logon to `client 001` with any of the following users (all share the same password `Down1oad`, typically you would work with `DEVELOPER`):

      - **User:** DEVELOPER (Developer User)
      - **User:** BWDEVELOPER (Developer User)
      - **User:** DDIC (Data Dictionary User)
      - **User:** SAP* (SAP Administrator)

1. In case you need it, you may generate some test data
      - **Report:** SAPBC_DATA_GENERATOR
      - **Transaction Code:** SEPM_DG

1. Suggestion: Activate the good old ping service
    - Go to Transaction `SICF`
    - Activate the node `/sap/public/ping` (default_host)
    - Test the HTTP and HTTPS connection with your browser

        - **HTTP:**  [http://localhost:8000/sap/public/ping](http://localhost:8000/sap/public/ping)
        - **HTTPS:** [https://localhost:44300/sap/public/ping](https://localhost:44300/sap/public/ping)



## Acknowledgements
This project wouldn't have been possible without previous work from some great people, it is built upon such a wonderful help:

[Nabi Zamani](https://blogs.sap.com/2018/05/30/installing-sap-nw-abap-into-docker/),  7.51/7.52 repositories. 

[Julie Plummer](https://blogs.sap.com/2019/07/01/as-abap-752-sp04-developer-edition-to-download/), always helping.

[Gregor Wolf](https://bitbucket.org/gregorwolf/dockernwabap750/src/25ca7d78266bef8ed41f1373801fd5e63e0b9552/Dockerfile?at=master&fileviewer=file-view-default), 7.50 repository.

[Tobias Hofman](https://github.com/tobiashofmann/sap-nw-abap-docker/blob/master/Dockerfile), 7.5x running on SuSE.

[Dylan Drummond](https://blogs.sap.com/2021/06/07/adjusting-installer-script-for-sap-netweaver-dev-edition-for-distros-with-kernel-version-5.4-or-higher/), key update for newer opensuse distributions.


Take into account that none of the above will currently lead you to an installed system, but this repository comes to group them together into a working installation.

