## Prerequisites
1. Be a student under School of Computing (SoC) at the National University of Singapore.
2. Have an SoC account. If you don't, you can [apply for one here.](https://mysoc.nus.edu.sg/~newacct/)
3. [Activate "SoC Compute Cluster"](https://mysoc.nus.edu.sg/~myacct/services.cgi "https://mysoc.nus.edu.sg/~myacct/services.cgi") in your SoC account.

## Steps to Take

### 1. SSH into SoC Compute Cluster
1. Identify which node you want to ssh into by looking at the [SoC Compute Cluster Hardware Configuration](https://dochub.comp.nus.edu.sg/cf/guides/compute-cluster/hardware). You can see the full list of available nodes together with the GPU specifications there.
2. SSH into preferred node. If you are on the School of Computing networks, you can SSH directly into any of the compute cluster nodes by doing:
	```shell
	ssh <soc_username>@<node>.comp.nus.edu.sg
	```
	If you are not on the SoC network, you can either access the compute cluster using the [SoC VPN](https://dochub.comp.nus.edu.sg/cf/guides/network/vpn) (recommended) or by [tunnelling through Sunfire.](https://nus-cs1010.github.io/2021-s1/environments.html#option-2-tunneling-through-sunfire)

3. Usually after successfully SSH-ing into a node, you will see the following messages:
	```
	Reserved till 30 Apr 2022 : xgpb0 (FYP)
	Reserved till 28 Jan 2022 : xgpd5-6
	Reserved till 31 Jan 2022 : cpga0
	Reserved 10 Jan - 7 May 2022 : xgpc5-9 (CS4347)
	Reserved 20 Jan - 20 Apr 2022 : xgpe2-6 (CS4246 / CS5446)
	...
	
	```
	This will tell you the nodes that are currently restricted for usage by other modules/research staff. Apart from these, the other nodes are free game, so pick the ones that best suit your needs.

### 2. Download virtual environment manager (miniconda)
Since you don't have sudo access, you'll need something that can create and manage virtual environments for you to install packages that are necessary for your development. In addition it's good to use virtual environments to keep package dependencies separate.

Personally I use [miniconda](https://docs.conda.io/en/latest/miniconda.html) since it's very easy to use and has access to most if not all important packages used for deep learning in python.
1. Download the latest miniconda installation script for linux on the node (verify link on the miniconda website!)
	```shell
	curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
	```
2. Run the script and follow the instructions. Type `yes` when they ask to prepend the conda install location to PATH in your `~/.bashrc`.
	```shell
	./Miniconda3-latest-Linux-x86_64.sh
	```
3. Source your `~/.bashrc` if the conda environment is not activated.
	```shell
	source ~/.bashrc
	```
4. Test the installation by trying the following command.
	```shell
	conda create -n myenv python=3.8 -y
	```

### 3. Install PyTorch according to nvidia driver version
You will be installing PyTorch and other necessary packages (such as CUDA toolkit) in a conda managed virtual environment. To do so, make sure you have already activated your virtual environment:
```shell
conda activate myenv
```

Then:
1. Run `nvidia-smi` to check the installed nvidia driver version.
	![[Nvidia driver version.png]]
2. Check the [compatibility of CUDA versions](https://docs.nvidia.com/deploy/cuda-compatibility/) based on the installed nvidia driver. Based on the driver version shown in the image above (460.91.03), it is compatible with CUDA versions 11.X.
3. Go to the [PyTorch website](https://pytorch.org/get-started/locally/) and install the correct version of PyTorch along with the CUDA toolkit using conda. Click on the "Previous PyTorch Versions" tab to install older PyTorch versions if the latest version does not suit your usage.
	```shell
	conda install pytorch==1.8.0 torchvision==0.9.0 torchaudio==0.8.0 cudatoolkit=11.1 -c pytorch -c conda-forge
	```
	Note that the CUDA version shown in the output of `nvidia-smi` shows the system installed CUDA (CUDA Version: 11.2). However, as explained [here](https://discuss.pytorch.org/t/pytorch-with-cuda-11-compatibility/89254/4), the system installed CUDA will not be used at all since conda installs its own CUDA toolkit in the above command.

### 4. Run python job to test installation
Run `test_pytorch.py` job in this repo:
```shell
python test_pytorch.py
```
If everything is correctly installed, you should see the following output messages:
```
Downloading https://www.cs.toronto.edu/~kriz/cifar-10-python.tar.gz to ./data/cifar-10-python.tar.gz
Extracting ./data/cifar-10-python.tar.gz to ./data
Files already downloaded and verified
cuda:0
[1,  2000] loss: 2.216
[1,  4000] loss: 1.839
[1,  6000] loss: 1.669
[1,  8000] loss: 1.574
[1, 10000] loss: 1.528
[1, 12000] loss: 1.469
Finished Training       
```
Note that you can also view the GPU usage of your current job by calling `nvidia-smi` as well:
![[gpu usage.png]]
## Other useful tips
### Use tmux to run multiple terminal sessions
[Tmux cheatsheet](https://tmuxcheatsheet.com/)
Since it is useful to monitor the GPU memory usage (and also check if other people are using the GPUs in the current node), I like to open a tmux session and run my training job in one pane while running `nvidia-smi` in another
1. Create a new tmux session called train
	```shell
	tmux new -s train
	
	```
2. Create a vertical pane in tmux by pressing Ctrl+b and %
3. Run training job in one pane and `nvidia-smi` in another
	```shell
	conda activate myenv  # remember to activate the environment
	python test_pytorch.py
	
	# switch to the other pane by pressing Ctrl+b and arrow key
	
	watch -n 0.5 nvidia-smi  # this loops nvidia-smi and refreshes it every 0.5 seconds
	```
	
Once done, it'll look something like this:
![[tmux.png]]

### Port-forwarding for tensorboard or jupyter notebook
[Running Tensorboard on remote server](https://stackoverflow.com/questions/37987839/how-can-i-run-tensorboard-on-a-remote-server)
[Running Jupyter Notebook on a remote server](https://docs.anaconda.com/anaconda/user-guide/tasks/remote-jupyter-notebook/)
