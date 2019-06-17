# How-to-use-docker-in-a-window-vm-
How to make a window vm running docker without hyperv


0. install Visual C++ Redistributable 2012 2013 2015 
https://www.microsoft.com/en-us/download/details.aspx?id=30679
https://www.microsoft.com/en-us/download/details.aspx?id=40784
https://www.microsoft.com/en-us/download/details.aspx?id=52685
1. download vmware workstation player 12.5 
https://my.vmware.com/web/vmware/free#desktop_end_user_computing/vmware_workstation_player/12_0
2. download vmware vix 1.15.8
https://my.vmware.com/web/vmware/free#desktop_end_user_computing/vmware_workstation_player/12_0|PLAYER-1259|drivers_tools
3. download vddk
https://code.vmware.com/web/sdk/6.7/vddk

4. install 1 & 2 
5. extract put 3 into c:/Program file (x86)\vmware\
6  set path of 2 and 3 for vmrun and vmware-vdiskmanager.exe

7. install Docker Toolbox without virtualbox 
   DockerToolbox-.exe /COMPONENTS="Docker,DockerMachine"
   https://github.com/pecigonzalo/docker-machine-vmwareworkstation
8.Replace contents of C:\Program Files\Docker Toolbox\start.sh with this script.
```
--------------------------------------------
#!/bin/bash

export PATH="$PATH:/mnt/c/Program Files (x86)/VMware/VMware Workstation"

trap '[ "$?" -eq 0 ] || read -p "Looks like something went wrong in step ´$STEP´... Press any key to continue..."' EXIT

VM=${DOCKER_MACHINE_NAME-default}
DOCKER_MACHINE=./docker-machine.exe

BLUE='\033[1;34m'
GREEN='\033[0;32m'
NC='\033[0m'


if [ ! -f "${DOCKER_MACHINE}" ]; then
  echo "Docker Machine is not installed. Please re-run the Toolbox Installer and try again."
  exit 1
fi

vmrun.exe list | grep \""${VM}"\" &> /dev/null
VM_EXISTS_CODE=$?

set -e

STEP="Checking if machine $VM exists"
if [ $VM_EXISTS_CODE -eq 1 ]; then
  "${DOCKER_MACHINE}" rm -f "${VM}" &> /dev/null || :
  rm -rf ~/.docker/machine/machines/"${VM}"
  #set proxy variables if they exists
  if [ -n ${HTTP_PROXY+x} ]; then
    PROXY_ENV="$PROXY_ENV --engine-env HTTP_PROXY=$HTTP_PROXY"
  fi
  if [ -n ${HTTPS_PROXY+x} ]; then
    PROXY_ENV="$PROXY_ENV --engine-env HTTPS_PROXY=$HTTPS_PROXY"
  fi
  if [ -n ${NO_PROXY+x} ]; then
    PROXY_ENV="$PROXY_ENV --engine-env NO_PROXY=$NO_PROXY"
  fi  
  "${DOCKER_MACHINE}" create -d vmwareworkstation $PROXY_ENV "${VM}"
fi

STEP="Checking status on $VM"
VM_STATUS="$(${DOCKER_MACHINE} status ${VM} 2>&1)"
if [ "${VM_STATUS}" != "Running" ]; then
  "${DOCKER_MACHINE}" start "${VM}"
  yes | "${DOCKER_MACHINE}" regenerate-certs "${VM}"
fi

STEP="Setting env"
eval "$(${DOCKER_MACHINE} env --shell=bash ${VM})"

STEP="Finalize"
clear
cat << EOF


                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/

EOF
echo -e "${BLUE}docker${NC} is configured to use the ${GREEN}${VM}${NC} machine with IP ${GREEN}$(${DOCKER_MACHINE} ip ${VM})${NC}"
echo "For help getting started, check out the docs at https://docs.docker.com"
echo
cd

docker () {
  MSYS_NO_PATHCONV=1 docker.exe "$@"
}
export -f docker

if [ $# -eq 0 ]; then
  echo "Start interactive shell"
  exec "$BASH" --login -i
else
  echo "Start shell with command"
  exec "$BASH" -c "$*"
fi

```
-----------------------------------------------------------------------
9. download and install git bash 
https://git-scm.com/downloads

10. download docker machine by git bash

$ if [[ ! -d "$HOME/bin" ]]; then mkdir -p "$HOME/bin"; fi && \
curl -L https://github.com/docker/machine/releases/download/v0.16.1/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe" && \
chmod +x "$HOME/bin/docker-machine.exe"


11.download vmwareworkstation plugin
https://github.com/pecigonzalo/docker-machine-vmwareworkstation
https://github.com/pecigonzalo/docker-machine-vmwareworkstation/releases/download/v1.1.0/docker-machine-driver-vmwareworkstation.exe

12. copy vmwareworkstation plugin to a place like c:\Docker
13. run following command
docker-machine -D create --driver=vmwareworkstation --vmwareworkstation-boot2docker-url=https://github.com/StefanScherer/boot2docker/releases/download/18.09.0-vmware/boot2docker.iso default

14. docker-machine env default > dockerdev.bat

15. run dockerdev

16.test by docker version

17. after restart  a. run docker-machine start default , b. run dokerdev.bat 
