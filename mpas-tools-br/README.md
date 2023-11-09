# mpas-tools-br

This is a singularity definition file to build a Singularity Image File (SIF) to use along with the MPAS-BR distribution. You can use singularity to download a base Linux image that can be custom tailored to install the software you needed in the container ([see the SingularityRecipe repository with and example](https://github.com/cfbastarz/SingularityRecipe). The instructions used in this README file, uses the provided definition file `mpas-tools-br.def` to built the container image to be used with the MPAS-BR repository.

The key idea is to provide a base system as a commom ground, where all users will find the tools needed to run the MPAS-BR software. By doing so, the users avoid commom mistakes in the system configuration.

An introduction on Singularity, its purpose goals and characteristics, are reported in [Singularity: Scientific containers for mobility of compute](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5426675/pdf/pone.0177459.pdf).

## Software available

The `mpas-tools-br` comes with the `mpas-tools` pre-installed with software like the `jigsaw` and other tools needed to construct the grids for the MPAS model. Take a look at the contents of the `mpas-tools-br.def` to get a list of the software shipped within the container.

* Base system: Ubuntu Linux 20.04;
* Compilers: GNU Fortran, C and C++ 11;
* Miniconda Python distribution;
* mpas-tools python environment.

## Building the SIF image

You want to customize the `mpas-tools-br.def` boostrap file to build your own container, make the desired modifications and follow the provided instructions. You want just want to use the container, skip to the next section (Using the container to run the MPAS-BR).

To built the singularity container using the boostrap file, first you need to install singularity. If you have Anaconda/Miniconda Python installed on Linux or Windows WSL machine, create an environment and install singularity in it:

```
conda create -n singularity
conda activate singularity
conda install -c conda-forge singularity
```

Next, create a `cache` and `tmp` directories and instruct singularity to use them during the building process. This might avoid potential problems with the image size:

```
mkdir -p $HOME/cache
mkdir -p $HOME/tmp

export SINGULARITY_DISABLE_CACHE=false
export SINGULARITY_CACHEDIR=$HOME/cache
export SINGULARITY_TMPDIR=$HOME/tmp
```

Next, build the sif image as follows:

```
singularity build --fakeroot mpas-tools-br.sif mpas-tools-br.def
```

**Note:** The build process might take >30min depending on your machine.

## Using the container to run the MPAS-BR

### Pulling the Container

As shown in the previous section, in order to create and run the container, you need to install singularity. Again, if you have Anaconda/Miniconda Python installed on Linux or Windows WSL, create an environment and install singularity in it. If you want to run singularity images within Mac OS X, you will need VirtualBox and we will left the instructions to do so at the end of this sectio.

First, create a conda environment to install singularity:

```
conda create -n singularity
conda activate singularity
conda install -c conda-forge singularity
```

Next, pull a copy of the `mpas-tool-br.sif` container (~2.8GB) from the Sylabs Cloud (you don't need to create an account):

```
singularity pull library://cfbasz/default/mpas-tools-br
```

**Optional:** The image is signed by the author. You can check the container fingerprint with the command:

```
singularity verify mpas-tools-br_latest.sif
```

The output must be:

```
Verifying image: mpas-tools-br_latest.sif
[LOCAL]   Signing entity: Carlos Frederico Bastarz (development key) <carlos.bastarz@inpe.br>
[LOCAL]   Fingerprint: 0DF6CEC4DBF78AD54EF541EB9C45F085F2287079
Objects verified:
ID  |GROUP   |LINK    |TYPE
------------------------------------------------
1   |1       |NONE    |Def.FILE
2   |1       |NONE    |JSON.Generic
3   |1       |NONE    |JSON.Generic
4   |1       |NONE    |FS
Container verified: mpas-tools-br_latest.sif
```

#### Installing Singularity in Mac OS X

In a nutshell, you'll need to install the `brew` package manager:

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Then, install `virtualbox`, `vagrant` and `vagrant-manager`:

```
brew install --cask virtualbox
brew install --cask vagrant
brew install --cask vagrant-manager
```

At this point, you will have the software needed to run a virtualization of Linux. Bear in mind that the Windows Subsystem for Linux (WSL) is something similar, meaning that running Linux in Windows and Mac OS is just a virtualization. But in the case of Mac OS, singularity needs to run inside a virtualized Linux and, if you want to run the `mpas-tools-br` container, you will be adding an extra layer of virtualization. But it work.

Next, bootstrap a Linux base system where singularity will be installed:

```
vagrant init sylabs/singularity-3.7-ubuntu-bionic64
```

The previous command will create a `Vagrantfile` with the definitions of your virtual machine (you can further customize to use more cores and RAM from the host; by default, it will use just 1 core and 1GB of RAM).

To initilize the virtual machine use the command:

```
vagrant up
```

And to log-in, use:

```
vagrant ssh
```

**Note:** If you want to open another terminal session to use singularity (within the virtual machine, just issue the commando `vagrant ssh` at the same place the `Vagrantfile` is at. Once the all work is done, just shutdown the virtual machine with the command `vagrant halt`. If you want to start over, just do `vagrant up` and `vagrant ssh`. This is the way to use VirtualBox without the graphical interface.

### Running the MPAS-BR inside the container

In this section, its show how to use the `mpas-tools-br_latest.sif` container to run the MPAS-BR software.

First, open a shell environment from the container:

```
singularity shell -e --env DISPLAY=$DISPLAY --bind $HOME:$HOME mpas-tools-br.sif
```

**Notes:**

* The option `-e` will keep the previous exported shell variables;
* The option `--env DISPLAY=$DISPLAY` will allow to use the display;
* The option `--bind $HOME:$HOME` will bind your home directory to the container home directory. If you have data in other disks in you computer, you can bind it as well, just add another bind like so:
    ```
    --bind $HOME:$HOME --bind /extra:/data
    ```
    In this case, the `/extra` from the host machine will be mounted in the `/data` in the container.

You will notice that the prompt will change to `Singularity>`

Next, source the miniconda environment variables in order to activate the `mpas-tools` environment with `conda`:

```
source /opt/miniconda/etc/profile.d/conda.sh
conda activate mpas-tools
```

**Optional:** If you want, you can set some aliases to help you get around the container shell:

```
alias ls='ls --color'
alias la='ls -ltr --color'
```

In order to run the MPAS-BR software, choose a path and clone the repository. Here we'll use the $HOME directory:

```
cd $HOME
git clone https://github.com/pedrospeixoto/MPAS-BR.git
```

At this point, you already have all the software needed to run the MPAS-BR. By followind Pedro Peixoto's classes, procedure as follows to compile, run and plot the grid and some initial fields:

1. Compile the MPAS-BR code:

    ```
    cd MPAS-BR
    make gfortran CORE=init_atmosphere PNETCDF=/usr
    make gfortran CORE=atmosphere AUTOCLEAN=true PNETCDF=/usr
    ```

2. Prepare the MPAS grid:

    ```
    cd $HOME/MPAS-BR/grids/utilities/jigsaw
    python3 spherical_grid.py -g unif -o unif240km -r 240 -l 240
    ```

3. Plot the grid:

    ```
    cd $HOME/MPAS-BR/post_proc/py/grid_maps
    python3 mpas_plot_grid.py -g ../../../grids/utilities/jigsaw/unif240km/unif240km_mpas.nc -o ../../../grids/utilities/jigsaw/unif240km/unif240km_mpas.jpg

    display $HOME/MPAS-BR/grids/utilities/jigsaw/unif240km/unif240km_mpas.jpg
    ```

4. Prepare the initial fiels:

    ```
    cd $HOME/MPAS-BR/benchmarks/monan-class-example/jw_baroclinic_wave

    ln -s $HOME/MPAS-BR/init_atmosphere_model .
    ./init_atmosphere_model
    ```

5. Plot the initial fields and some grid properties:

    ```
    python3 ../../../post_proc/py/scalar_lat_lon_2d_plot/mpas_plot_scalar.py -v theta x1.40962.init.nc

    display figures/theta/theta_0_*

    python3 ../../../post_proc/py/plot_scalar_on_native_grid/mpas_plot.py -f x1.40962.init.nc -o x1.40962.theta.jpg -v theta -l 0 -t 0

    display x1.40962.theta.jpg

    python3 ../../../post_proc/py/plot_scalar_on_native_grid/mpas_plot.py -f x1.40962.init.nc -v areaCell -o areaCell.jpg

    display areaCell.jpg

    python3 ../../../post_proc/py/plot_scalar_on_native_grid/mpas_plot.py -f x1.40962.init.nc -v areaTriangle -o areaTriangle.jpg

    display areaTriangle.jpg

    cd $HOME/MPAS-BR/grids/utilities/jigsaw/unif240km

    python3 ../../../../post_proc/py/plot_scalar_on_native_grid/mpas_plot.py -f unif240km_mpas.nc -v triangleQuality -o triangleQuality.jpg

    display triangleQuality.jpg
    ```

carlos.bastarz@inpe.br (09/11/2023)
