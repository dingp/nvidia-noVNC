# Nvidia Image with noVNC for Perlmutter

This is mostly copied from the repository [twobombs/deploy-nvidia-docker](https://github.com/twobombs/deploy-nvidia-docker). The image contains the following elemenets:
1. base image is nvidia's cuda image `nvidia/cuda:12.3.2-base-ubuntu22.04`;
2. desktop environment and noVNC;
3. start up scripts to start noVNC session on container startup.

## Running noVNC on login node for GUI apps

### Start the container on one login node

Note:
- if someone else is using port `6080` on the login node, you may need to change the first `6080` in `-p 6080:6080` to another free port.

```bash=
podman-hpc run \
	--gpu \
	--rm -d -p 6080:6080 \
	--name ubuntu2204-novnc \
	-v <software_dirs_on_global_common>:<path_in_container> \
	dingpf/novnc-nvidia:latest
```

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

You should now be able to go to `localhost:6080` in a browser window. The default noVNC password is `00000000`.





