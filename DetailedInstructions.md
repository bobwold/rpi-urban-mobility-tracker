# Raspberry Pi Urban Mobility Tracker

## Hardware

### Primary Components
1) [Raspberry Pi 4b](https://www.raspberrypi.org/products/raspberry-pi-4-model-b)
2) [Raspberry Pi Camera V2](https://www.raspberrypi.org/products/camera-module-v2)
 
## Install (from macOS)
1) Insert memory card
2) Install [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
	- Select `Raspberry Pi OS (other)`
	- Select `Raspberry Pi OS Lite (32-bit)`
	- Make note of hostname - Bikestreet is using the format `umt-00N`
	- Enable SSH
	- Set username and passwod
	- Configure wireless LAN
	- Burn the image to the memory card
3. Insert memory card in to PRi and turn on
4. `ssh pi@raspberrypi.local`
5. `sudo raspi-config`
	- Enable legacy camera support
	- Add any additional WiFi credentials you may need
	- Check for updates
6. `sudo apt-get update && sudo apt-get upgrade`
7. Install Docker
	- `curl -fsSL https://get.docker.com -o get-docker.sh`
	- `sudo sh get-docker.sh`
	- `rm get-docker.sh`
8. Add non-root user to Docker user group `sudo usermod -aG docker pi`
9. Create UMT output directory and move into it `mkdir umt_output && cd umt_output`
10. From your developer machine, copy your current source to the pi using scp. `scp -r ~/dev/bike/rpi-urban-mobility-tracker/* pi@umt-001.local:/home/pi/umt_output/`
11. Download a replacement Python file `wget https://raw.githubusercontent.com/TravelByRocket/bikestreets-umt/master/generate_detections.py`
12. Build the Docker container `sudo docker build . -t umt` 
	- or `sh build_container.sh` if script downloaded
14. For Development, it's best to start the docker container in 'one-time' mode.
`docker run -it --privileged --device /dev/apex_0:/dev/apex_0 --mount  type=bind,src=/home/pi/umt_output,dst=/root umt`
For Field testing, you'll want your docker image to restart when the device boots: 
`docker run -it --privileged --device /dev/apex_0:/dev/apex_0 --mount type=bind,src=/home/pi/umt_output,dst=/root -d --restart unless-stopped umt`
	- or `sh run_container.sh` if script downloaded
	- `-d` start the container detached
	- attach container with `docker attach CONTAINER_NAME`
	- get container name from `docker ps` or simply type `docker attach ` and press `Tab` button for auto-completion
	- detach container with `Ctrl-P Ctrl-Q`

## Object Paths File (from macOS)

1. Connect to the same network as the Raspberry Pi
2. Open terminal and enter `cd Downloads`
3. Get all the paths with `scp -r pi@raspberrypi.local:/home/pi/umt_output/object_paths .` to put it in the current directory (Downloads)
3. Get all the images with `scp -r pi@raspberrypi.local:/home/pi/umt_output/output .` to put it in the current directory (Downloads)

## Notes
- Change `umt` to use a new filename every time it starts
- Use `scp` to move over a directory of CSVs or somehow specify a range
- Can execute commands after/within `ssh` as shown [here](https://stackoverflow.com/questions/26434604/bash-script-execute-commands-after-ssh)
- Not sure of behavior of `umt` when `object_paths.csv` is deleted while it is running
- Might need to make sure entire card is being utilized
- `sudo rm -rf output` to delete all images in the `output` directory if you are already in `umt_output` directory
- `sudo rm -rf object_paths` to delete all CSVs in the `object_paths` directory if you are already in `umt_output` directory
- `object_paths` directory must already exist