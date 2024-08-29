# Nvidia Image with noVNC for Perlmutter

This is mostly copied from the repository [twobombs/deploy-nvidia-docker](https://github.com/twobombs/deploy-nvidia-docker). The image contains the following elemenets:
1. base image is nvidia's cuda image `nvidia/cuda:12.3.2-base-ubuntu22.04`;
2. desktop environment and noVNC;
3. start up scripts to start noVNC session on container startup.

## Running noVNC on login node for GUI apps

### Start the container on one login node

Generate a VNC password file. 

```bash
podman-hpc run --rm -it -v $HOME:/scratch --entrypoint=/bin/bash docker.io/dingpf/novnc-nvidia:latest
# inside the container, run 
vncpasswd
# once finished
cp ~/.vnc/passwd /scratch/.vnc_passwd
```

Run the container on the login node.

```bash=
podman-hpc run \
	--gpu \
	--rm -d -p 6080:6080 \
	--name ubuntu2204-novnc \
	-v <software_dirs_on_global_common>:<path_in_container> \
    -v ~/.vnc_passwd:/root/.vnc/passwd \
	dingpf/novnc-nvidia:latest
```

Note that if someone else is using port `6080` on the login node, you may need to change the first `6080` in `-p 6080:6080` to another free port.


### Setting up a SSH tunnel to the login node

Note: 
- you may need to change `~/.ssh/nersc` if the path to your nersc identity file is different;
- change `loginXX` to the hostname of the login node where you have your container running;

```bash=
ssh -o IdentitiesOnly=yes \
    -o IdentityFile=~/.ssh/nersc \
    -J perlmutter.nersc.gov \
    -L 6080:localhost:6080 loginXX -N -f
```

### Access noVNC in a browser

You should now be able to go to `localhost:6080` in a browser window. If you skipped setting the vnc password step above, the default noVNC password is `00000000`.


## Use JupyterLab's proxy server for the noVNC session

1. Go to https://jupyter.nersc.gov;
2. Start a JupyterLab server on Perlmutter login node;
3. Launch a terminal in the JupyterLab launcher, and run `podman-hpc run --gpu --rm -d -p 6080:6080 -v ~/.vnc_passwd:/root/.vnc/passwd dingpf/novnc-nvidia:latest`;
4. Go to https://jupyter.nersc.gov/user/<user-name>/perlmutter-login-node-base/proxy/6080/vnc.html (note: replace `<user-name>` with your username);
5. Click the gear button on the page, make sure the "Advanced" -> "WebSocket" section has the "encrypt" filed unchecked, "host" as "localhost", "Port" as 6080 and "Path" as "websockify", click "Connect"
6. Put in your VNC password, and you should get a desktop environment right after.




