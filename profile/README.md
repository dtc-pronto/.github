# DTC PRONTO :mask:

This contains all the necessary code and configs for the Darpa Triage Challenge PRONTO Team. 

Below are some usefule tidbits on the Jackal. Feel free to add more descriptions here.

### Powering On and Off
To power on the jackals short the second and third pins from the left. The jackals should be powered off from the command line. Run `sudo shutdown now`. This will power off the entire jackal.

### Logging In
There are thre main ways to log into the jackal. If you are indoors the jackal will automatically connect to the `mrsl_perch` network with the following static ips:
 - Phobos: `192.168.129.111`
 - Deimos: `192.168.129.112`

If you go outdoors you can use the `jackalnet2` hotspot, the robots will automatically connect to it only if they are out of range with `mrsl_perch` with the following static ips:
 - Phobos: `192.168.50.111`
 - Deimos: `192.168.50.112`

Finally, you may connect over the rajant network. The jackals will automatically connect to the rajant no matter what but note that the rajants take a few minutes to boot. To connect via rajant, use a rajant brick and connect to via ethernet. Assign yourself and a static ip address in the network configuration window on linux. Give yourself an ip address of `10.10.10.X` where X is in the range of `0 - 100` (over 100 is reserved for the robots), and a netmask of `255.0.0.0`. I strongly recommoned not using a windows computer for this. On the rajant network the jackals will have the following static ips:
 - Phobos: `10.10.10.111`
 - Deimos: `10.10.10.112`

Once you have network configurations correct on your ground station you can ssh into the jackal with: `ssh dtc@<jackal_ip>`, the password is a space.

### Running the Jackal
Once you have logged into the jackal, navigate to the `Docker` directory with `cd Docker`. Here you can configure what components you want to run in the `docker-compose.yml`. In the `jackal-base` image you can configure what sensors you want to run, by setting the environment variable for the respective sensor to `true/false`. Setting the value to true will start the ros device driver, otherwise it will the driver will not be running and you will not see the rostopics. Additionally, in `docker-compose.yml` you can configure what inference components you want to run in the respective image by setting the `RUN` variable to `true/false`. Once you have your `docker-compose.yml` configured you can start the jackal stack with the following steps.
 - `tmux`
 - `docker compose up`
The system is now running and the jackal will be driveable.

### Bagging Data
If you want to bag data, follow the start up instruction above. Once your started the jackal with the desired sensors and inference componenets, enter the `jackal-base` image. If you started in `tmux` split your window with `ctrl-b + %` or `ctrl-b + "`, then `cd jackal`. Enter the `jackal-base` image with `./join.bash`. You can also open a new terminal on your groundstation, ssh into the jackal navigate to the jackal directory and run `./join.bash` there. Once in `jackal-base` docker image run `cd data`. This is the persistent data directory and is where all rosbags should be saved. Now you can run `rosbag record <list_of_topics>` to start bagging. If you are bagging camera data bag the `<topic_name>/compressed`, additionally, do not bag event camera data for too long. If you want to bag lidar data, bag the `lidar_packets` and `imu_packets` rather than the point clouds.

Once you have collected your bag file, you can copy it over to your machine. If the file is not too large you can copy it over wifi with:
``` 
scp /path/to/<date_time>.bag <your_username>@<your_ip_address>:/path/to/<where_you_want_to_save_file>
```
If you files are large it will be faster to use one of the portable SSDs. __Note__: you can only use one of the dtc encrypted drives if you have a linux machine to copy it to. Plug the drive into the robot via usb. Run:
```
lsblk # to see the usb device name, usually sda1 or sdb1
sudo cryptsetup luksOpen /dev/sda1 samsung_t7 # unencrypt
sudo mount /dev/mapper/samsung_t7 /media/dtc # mount
sudo cp /path/to/<bag_file>.bag /media/dtc/ #copies the files
sudo umount /media/dtc # unmount when your done copying data
sudo cryptsetup luksClose samsung_t7
```
Now the portable ssd can be safely unplugged.
