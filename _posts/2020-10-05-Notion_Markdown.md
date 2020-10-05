# Working with Linux

---

# Useful Linux commands

```bash
# Open a port
sudo ufw allow 4000

# Check open ports
sudo lsof -i -P -n | grep LISTEN
sudo netstat -tulpn | grep LISTEN

# Add the path to linux packages. 
# These are the directories stored as global variables where linux will search for the 
# programs, packages, and commands we use in the terminal.
export PATH="/root/poppler-0.52.0/poppler/.libs:$PATH"
export PKG_CONFIG_PATH="/root/poppler-0.52.0/poppler/.libs:$PKG_CONFIG_PATH"

# Check the paths or any environment variable
echo $PATH

# Check memory usage (task manager)
htop

# Copy a file to another directory
sudo cp /usr/local/lib/libpoppler.so.66 /usr/local/lib/pkgconfig

# Remove files and directories
rm my_file.txt
rm -r my_directory
rm -r *.png
```

---

# Bash shortcuts

We edit aliases as explained below, by adding them to the file `./bashrc` in the root directory.

```bash
nano ./bashrc
alias labb='docker exec -it lab /bin/bash'
alias checkports='sudo lsof -i -P -n | grep LISTEN'
alias dps='docker ps'
alias dimg='docker images'
```

In *nano*, we
* `ctrl+o` and `enter` to save.
* `ctrl+x` to exit

Then we reload the file, for the changes to take effect:
source `./bashrc`

---

# Updating the Web App

- **Pull the repository to work with if necesary**
    - Abrir `Anaconda` command promt
    - `cd` in la carpeta del proyecto where we want to save the code
    - `heroku login`
    - **`heroku git:clone -a paper-digest`**
- **Local machine**
    - Abrir Atom y el proyecto
    - Abrir `Anaconda` command promt
    - `cd` in la carpeta del proyecto y correr la app (`python run.py`)
    - Hacer cambios en `Atom` o otro editor.
- **Deploy to Heroku**
    - Abrir otra `Anaconda` command promt

    ```bash
    cd folder/de/la/webapp
    heroku login
    # Activate the virtual environment
    env\Scripts\activate
    git add .
    git commit -m "comments"
    git push heroku master
    ```

     

---

# Laboratory DEV environment

Sometimes I have to work with large data my pc can handle. In those cases is better to use cloud servers with custom ram and processing.  In those cases, I have 2 options ready to be used: 

- Use the container with all my code from the lab. This container is `cristianmejia00/lab` in my docker hub.
- Use the saved image in DigitalOcean. check for the `lab` image with the most recent date.

In any case, extra files needed can be added from my PC to the host and containers using WinSCP. The folder is `root/container`.

## With Docker container

```bash
# Access the host (droplet or server) from the Windows Terminal after changing 
# the proper URL of the server

docker login
docker run --rm -dp 8787:8787 -dp 8000:8000 -e PASSWORD=tokyo2015 -v /root/container:/var/container --name lab cristianmejia00/lab
docker exec -it lab /bin/bash
cd var/R-scripts
git pull
```

## With Digital Ocean image

Go to [digitalocean.com](http://digitalocean.com) and access with my Hotmail account. In the images tab, find the `lab` image and spin a powerful droplet based on that. 

Once done go to `url:8787` to open RStudio Server. Login with `rstudio` and `tokyo2015`.

This image contains both contaners. `LGL` and `Lab` so we can do the full fukan system emulation with it.

---

# Install and use LGL

## Documentation

- [Official Web (Adai)](http://lgl.sourceforge.net/#Download)
- [Opte Project GitHub](https://github.com/TheOpteProject/LGL)
- [Random tutorial](http://clairemcwhite.github.io/lgl-guide/)

## Setting-up LGL

Spin up a server and ssh into it.

```bash
# Install basic libraries 
sudo apt-get install build-essential
sudo apt-get install g++

# Get LGL from GitHub
git clone [https://github.com/TheOpteProject/LGL.git](https://github.com/TheOpteProject/LGL.git)

# Add this path to the perl @INC
**export PERL5LIB=/root/LGL/perls

# Download legacy library. LGL only works with Boost 1.55**
wget [http://archive.ubuntu.com/ubuntu/pool/main/b/boost1.55/boost1.55_1.55.0.orig.tar.bz2](http://archive.ubuntu.com/ubuntu/pool/main/b/boost1.55/boost1.55_1.55.0.orig.tar.bz2)
tar xjf boost1.55_1.55.0.orig.tar.bz2

# Compile LGL
cd LGL
**env CPLUS_INCLUDE_PATH=/root/boost_1_55_0 ./setup.pl -i**
```

When we install LGL from the Opte Project, we need to create folders and set parameters as follows:

```bash
# Create a folder for the results
mkdir lgl-results
chmod -R 777 lgl-results

# Change the bin path in the lgl.pl file
cd bin
nano lgl.pl

# There, find '$LGLDIR' and set it to '/root/LGL/bin'
# `ctrl+o` and `enter` to save
# `ctrl+x` to exit

# Create a config file
cd ..
./setup.pl -c my_config
nano my_config

# There, set 
# tmpdir = '/root/LGL/lgl-results'
# inputfile = '/root/LGL/lgl-results/file.ncol'
# Save and exit
```

To run LGL we need the .ncol file, which is the list of edges. Non-repeated, inverted, or incomplete edges are allowed in this file. The format is 2 columns with each edge, and an optional column with the weight. They are separated by `' '` a space. Not a tab. See the documentation for more explanations.

`<Node 1> <Node 2> [<weight>]`

If we already have that file in our server. In  `LGL/lgl-results/file.ncol` Then we can finally run LGL

```bash
# Run LGL from the LGL directory.
./bin/lgl.pl -c my_config
```

The results will be placed in the `lgl-results` directory.

---

## Create the ncol file

I have 2 options: 

1- Using the `mission.pairs.tsv` from Fukan System

2- Creating my own file with `direct_citations.R` 

```r
# Load Igraph
library(igraph)

# Load our ncol file (from Fukan or created by us)
test <- read.csv("mission.pairs.tsv", sep = "\t", header = FALSE, stringsAsFactors = FALSE)

# Create a network object, and simplify it.
# We need to simplify it because the Fukan pairs have repeated edges
g1 <- graph_from_data_frame(test, directed = FALSE)
g1 <- simplify(g1)

# Write the file
write_graph(g1, "file.ncol", format = "ncol")

# Optionally, check the file
test2 <- read.csv("file.ncol", sep = " ", header = FALSE, stringsAsFactors = FALSE)
```

---

## Plot using the LGL layout

Once run, LGL will output several files into the `lgl-results` folder. Bring them to Windows using WinSCP. Then, move to R.

```r
# Assuming we still have the `g1` object in the environment

# Test with igraph LGL layout
test_coords <- layout_with_lgl(g1)
plot(g1, layout = test_coords, vertex.size = 0, vertex.frame.color = NA, vertex.label = NA)

# Using original LGL layout from server
# The layout may come with a blank 4th column, and the nodes ordered differently to the 
# `g1` object. Thus, we correct that here.
coords <- read.csv(file.choose(), sep = " ", header = FALSE, stringsAsFactors = FALSE)
coords[,4] <- NULL
vv <- V(g1) %>% names %>% as.numeric
idx <- match(vv, coords$V1)
coords_edited <- coords[idx,]
coords_edited <- as.matrix(coords_edited[,c(2,3)])
plot(g1, layout = coords_edited, vertex.size = 0, vertex.frame.color = NA, vertex.label = NA)
```

---

## Using LGL container

I prepared an image with all the above environment so I don't need to do it every time.

check my docker hub: `cristianmejia00/lgl`

To run this image we need to mount a folder where to input the `file.ncol` and receive the outputs. Also, because I used the image from Rstudio we connect it to an open port. I put it in port 7777 because I'll use it with the laboratory container where I run RStudio Server on port 8787.

```bash
# Run the container
docker run --rm -dp 7777:8787 -e PASSWORD=tokyo2015 -v /root/container:/root/LGL/lgl-results --name lgl cristianmejia00/lgl

# Place the file.ncol in `container` in the host
# Then access the container, add the perls, and compute the layout. No need to change anything here.
docker exec -it lgl /bin/bash
cd root/LGL
**export PERL5LIB=/root/LGL/perls**
./bin/lgl.pl -c my_config
```

---

# Databases in R

In particular, how I'm using MySql for Paper Digest API to store everything there.

Here, I assume there is a managed MySql database already set in Digital Ocean.

First, lets be sure the Linux dependencies are installed in our host.

```bash
sudo apt-get install unixodbc-dev
sudo apt-get install libpq-dev
```

Then go to Rstudio server and install the necessary packages

```r
devtools::install_github("rstats-db/odbc")
install.packages("DBI")
```

Then go to Digital Ocean and run a managed database and get the credentials. They look like this:

```bash
username = doadmin
password = d83ie4th3l8ed1e6 hide
host = [db-postgresql-sgp1-81169-do-user-2699525-0.a.db.ondigitalocean.com](http://db-postgresql-sgp1-81169-do-user-2699525-0.a.db.ondigitalocean.com/)
port = 25060
database = defaultdb
sslmode = require
```

Then in the Rstudio server connect to the database:
[https://db.rstudio.com/getting-started/conect-to-database/](https://db.rstudio.com/getting-started/connect-to-database/)

Note: For Postgress follow this tutorial [https://phoenixnap.com/kb/how-to-install-postgresql-on-ubuntu](https://phoenixnap.com/kb/how-to-install-postgresql-on-ubuntu)

---

# Extract images from PDFs

My first attempt was trying again the [PDFNLT](https://github.com/KMCS-NII/PDFNLT-1.0/blob/master/pdfanalyzer/INSTALL.en) utility I saw during a NLP conference in Keio. In that reporsitory they explain how to install it, but unfortnatelly all dependencies are now outdated. Meaning that for installing it properly I had to figure out how to install such old dependencies by hand (because the normal way using apt-get bring the most updated one which wont work).

Then, I realized that they were using  [PDFFiugures](https://github.com/allenai/pdffigures) from `Allenai`. And, same as above their tools was developed more or less ate the same time that PDFNLT hence also shares the same dependency problems. Given that I know Allenai is good with academic articles I decided to focus on that instead.

## Pdffigures from Allenai

Here is how to install `pdffigures` in a new linux server. 

From Digital Ocean I spun a server running Ubuntu 20. Then, ssh to the server...

```bash
# Install basic libraries
sudo apt-get update
sudo apt-get install -y build-essential 
sudo apt-get install -y unzip
sudo apt-get install -y libfontconfig1-dev libjpeg-dev xfonts-scalable
sudo apt-get install -y xzgv

# Install Poppler 0.52
wget https://poppler.freedesktop.org/poppler-0.52.0.tar.xz
tar xJf poppler-0.52.0.tar.xz
cd poppler-0.52.0
./configure --enable-xpdf-headers
make
sudo make install

# Install leptonica 1.72
# Repo: https://github.com/DanBloomberg/leptonica/tree/v1.72
git clone --depth 1 --branch v1.72 https://github.com/DanBloomberg/leptonica.git
cd leptonica
sudo chmod +x configure
./configure
make
sudo make install

# Install pdffigures
git clone https://github.com/allenai/pdffigures.git
cd pdffigures
export PATH="/usr/local/lib:$PATH"
export PKG_CONFIG_PATH="/usr/local/lib"
make DEBUG=0
sudo cp pdffigures /usr/local/bin/

# Move the libraries to the correct path
sudo **cp /usr/local/lib/libpoppler.so.66 /usr/lib/**
sudo **cp /usr/local/lib/liblept.so.4 /usr/lib/**
```

**usage**

```bash
# Get the PDF page with a box drawn over the figure
pdffigures -a /folder/path/prefix /path/tofile.pdf

# Save figures and tables low resolution but fast
pdffigures -o /folder/path/prefix /path/tofile.pdf

# Save figures and tables high resolution but fast
pdffigures -c /folder/path/prefix /path/tofile.pdf

# Save a json file with the captions and pixel coordinates of each figure and table
pdffigures -j /folder/path/prefix /path/tofile.pdf

# Get help
pdffigure -help
```

And pretty much that's it. It does not offer more customization. We extract both figure and tables at once. Cannot select either of them.

The output directory path needs a "prefix". This prefix is a string that is prefixed to the name of each figure and table extracted. This means that when using something like `/root/output/mypdf`

It will save the images and tables in the `output` folder of the `root` directory. And each image and table is named `mypdf-image-1.png`; `mypdf-image-2.png`; `mypdf-table-1.png`; etc.

Finally, there is a `pdffigures2` also from Allenai, but this one is using Scala and binaries I don't know how to install. Maybe worth checking in the future. [Repository.](https://github.com/allenai/pdffigures2)

**current state (20200923)**

An small droplet in Digital Ocean is woring with this setting in the ip `188.166.226.28.`

That IP is added to Windows Terminal, Putty, and WinSCP as PDF figures. There I added the folders `/root/pdf` and `/root/images` for the pdf inputs and image outputs respectively. Those folder are meant to use with WinSCP. Once ssh, just run the **usage** commands as needed.

Next step is to conteinerize it and put it as a service with Flask and Docker.

## Extract with Python

I made a colab summarizing my attempts with Python. [Here.](https://colab.research.google.com/drive/1NlL-OAAxiKtdDT3i1eRQfndsFw18_nBz#scrollTo=N8wEgR_Dg_m_)

So far not so good. While presumably could extract image faster, they only extract images, not tables, nor captions and the image order needs to be inferred from the page number. Which can be problematic when multiple images appear in the same page. Also, they extract all images, including publisher logos that need to be filtered out.
