
# EDGE Bioinformatics

This is docker image for **2.4.1 beta** version of EDGE Bioinformatics, a product of collaboration between Los Alamos National Laboratory and the Naval Medical Research Center sponsored by the Defense Threat Reduction Agency.

EDGE is a highly adaptable bioinformatics platform that allows laboratories to quickly analyze and interpret genomic sequence data. The bioinformatics platform allows users to address a wide range of use cases including assay validation and the characterization of novel biological threats, clinical samples, and complex environmental samples.

# How to use this image

### Install Docker (Docker Desktop for Mac and Windows)

See Docker at https://www.docker.com/

For Windows, it also needs Windows subsystem for Linux (WSL) which can be installed through [Microsoft store](https://apps.microsoft.com/detail/9pdxgncfsczv?rtc=1&hl=en-us&gl=US). 

### Obtain the docker image (use terminal in Mac/Linux or WSL in Windows)

    $ docker pull bioedge/edge_24_1_ubuntu18

### Obtain inital mysql database or use data volume container

    $ wget https://ref-db.edgebioinformatics.org/EDGE/Docker/edge_ubuntu_init_mysql_EDGE_input_for_edge_docker.tgz
    
    Or 

    $ docker pull bioedge/edge_ubuntu_mysql
    $ docker create --name mysql_data --volume /var/lib/mysql  bioedge/edge_ubuntu_mysql

### Download EDGE database from web server 

#### You will need ~500GB disk space for all databases

      ## Pipeline database is ~17Gb and contains the other databases needed for EDGE
      wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_dev_pipeline_databases.tgz

      ## BWA index is ~41Gb and contains the databases for bwa taxonomic identification pipeline
      wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_dev_bwa_index.tgz

      ## HOST genomes BWA index is ~41Gb for Host removal, including human, bacteria, phiX, viruses, invertebrate vectors of human pathogens
      wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_dev_HostIndex.tgz

      ## NCBI Genomes is ~21Gb and contain the full genomes for prokaryotes and some viruses
      wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_dev_NCBI_genomes.tgz

      ## GOTTCHA database is ~16Gb and contains the custom databases for the GOTTCHA taxonomic identification pipeline
      wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_dev_GOTTCHA_db.tgz

      ## NT database is ~25Gb and contains the NCBI nt database for contig identification
      wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_dev_nt_20160426.tgz

      ## ShortBRED database is ~27Mb and contains the databases used by ShortBRED for virulence factors and read based antibiotic resistance analysis
      wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_dev_ShortBRED_Database.tgz

      ## Diamond database is ~16Gb and contains the databases from RefSeq for protein based taxonomic identification
      wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_dev_diamond_db.tgz

     ## MetaPhlAn4 database is 14Gb file contains the databases used for the MetaPhlAn4 taxonomic identification pipeline
     wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_dev_metaphlan4DB.tgz

     ## GOTTCHA2 databases is 38Gb file and contains the custom databases for the GOTTCHA2 taxonomic identification pipeline
     wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_GOTTCHA2_db_20190729.tgz

     ## Kraken2 database is 39Gb file contains the databases used for the Kraken2 taxonomic identification pipeline
     wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_Kraken2_db_20211216.tgz

     ## Centrifuge database is 20G file contains the databases used for the Centrifuge taxonomic identification pipeline
     wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_Centrifuge_db_20200329.tgz

     ## PanGIA database is 35G file for PanGIA taxonomic identification pipeline
     wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_dev_PanGIA_db.tgz

     ## MICCR database is 48GB contains the databases used for the contig taxonomic identification pipeline
     wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_ContigTax_db_20190114.tgz

     ## CheckM database is 275MB contains the databases used for the Metagenome Binned contig quality assessment.
     wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_checkM_db_20190213.tgz

     ## Qiime2 database is 1.4GB contains 16s,18s and ITS db.
     wget -c  https://ref-db.edgebioinformatics.org/EDGE/dev/edge_qiime2_db_20230719.tgz

     ## AntiSmash database is 3.2GB contains pfam resfam tigrfam can clusterblast db for antismash version 6
     wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_dev_AntiSmash6.tgz

     (Optional)
     ## Other Host bwa index ~18Gb for host removal, including pig, sheep, cow, monkey, hamster. and goat.
     wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_dev_otherHostIndex.tgz

     ## For machine with < 32Gb memory, we suggest to use the smaller BWA index (~14Gb) and contains the databases for bwa taxonomic identification pipeline
     wget -c https://ref-db.edgebioinformatics.org/EDGE/dev/edge_dev_bwa_mini_index.tgz

     # Decompressed each tar.gz file (tar -xzvf) for being used later

### Start EDGE bioinformatics instance

```
    $ docker run -d --privileged=true --security-opt "seccomp:unconfined" \
    --cap-add=SYS_ADMIN --cap-add=SYS_PTRACE  \
    -v /path/to/mysql:/var/lib/mysql \
    -v /path/to/database:/home/edge/database \
    -v /path/to/EDGE_output:/home/edge/EDGE_output \
    -v /path/to/EDGE_input:/home/edge/EDGE_input \
    -v /path/to/EDGE_report:/home/edge/EDGE_report \
    -p 80:80 -p 8080:8080 --name edge bioedge/edge_24_1_ubuntu18

```
    
Wait for few seconds for the docker image to start EDGE service and Open http://localhost/ on the browser to start experience EDGE. 

* The -v /path/to/mysql:/var/lib/mysql part of the command mounts the /my/own/mysql (obtain from the link above and chown to mysql:mysql if needed.) directory from the underlying host system as /var/lib/mysql inside the container, where MySQL by default will write its data files. Using this to persist the database data in the host.  You can use data volume instead and it described in the "Note" section .
* The -v /path/to/database:/home/edge/database mounts the databse obtained from the above download step. 
* The -v /path/to/EDGE_input:/home/edge/EDGE_input mounts the EDGE input directory structure (obtain from the git clone above) to persist the input/upload files/user projects in the host. 
* The -v /path/to/EDGE_output:/home/edge/EDGE_output mounts the EDGE output directory to persist the output files in the host. 
* The -v /path/to/EDGE_report:/home/edge/EDGE_report mounts the EDGE report directory to persist the report files in the host. 
* The -p host:container bind the host port 80 and 8080 to container port 80 and 8080 inside the container. You can change the 80 and 8080 to fit your host system requirements.
* In some linux environment, you need to set the /path/to/mysql directory into 0777 mode. Please try opening up the directory permission if you run into trouble. Or use data volume method below.

###  Default credentials

* EDGE user: admin_docker@my.edge/admin_docker

* For security, you may need update the credentials if the server will be used by others or public. 

### Note
* The image size is around 36.1GB: If you have issue pulling this image, you may increase the basesize when launch docker daemon or using different [Storage Driver](https://goo.gl/8YUeUA). see [issue](https://github.com/moby/moby/issues/8971).  
* This image is built on top of offical Ubuntu 18.04.3 LTS Base Image, and is officially supported on Docker Engine version 19.03.5.
* The user management can be accessed by http://localhost:8080/userManagement if host port is 8080
* There is a permission issue when using docker in Windows 10+ + WSL 2 [notes](https://github.com/LANL-Bioinformatics/EDGE/issues/63). The WSL metadata support need to turn on for Windows drives by editing /etc/wsl.conf and add following section.
```
    [automount]
    options = "metadata". 
```
* There is an issue to mount host volume using MAC OSX.  https://github.com/boot2docker/boot2docker/issues/581.  However, there is a workaround to use data volume container instead.

```
    * Use data volume container (a.k.a: OS independent.)
    $ docker pull bioedge/edge_ubuntu_mysql
    $ docker create --name mysql_data --volume /var/lib/mysql  bioedge/edge_ubuntu_mysql
    $ docker run -d-privileged=true --security-opt "seccomp:unconfined" --cap-add=SYS_ADMIN --cap-add=SYS_PTRACE --volumes-from mysql_data -v /path/to/database:/home/edge/database -v /path/to/EDGE_output:/home/edge/EDGE_output -v /path/to/EDGE_input:/home/edge/EDGE_input/public -p 80:80 -p 8080:8080 --name edge bioedge/edge_24_1_ubuntu18
```
### Commands for checking status and error log

* Check the mysql status in container:
```  
    $ docker exec edge service mysql status
```
where "edge" is the container name when user `docker run` it with --name flag 

* And user management system service status: 
```
    $ docker exec edge service tomcat7 status
```
* For the Apache web server status and log: 
```    
    $ docker exec edge service apache2 status 
    $ docker exec edge tail /var/log/apache2/error.log
    $ docker exec edge tail /var/log/apache2/access.log
```

### Customize the docker0 bridge

Docker is hard coded to look for 172.17.0.1. If the ip address conflict with the subnet of your wifi, you may edit the /etc/docker/daemon.json as describe [here](https://forums.docker.com/t/change-default-docker0-bridge-ip-address/30470/9) 

### Contact Info
Chien-Chi Lo:  <chienchi@lanl.gov>

### Citation
Po-E Li, Chien-Chi Lo, Joseph J. Anderson, Karen W. Davenport, Kimberly A. Bishop-Lilly, Yan Xu, Sanaa Ahmed, Shihai Feng, Vishwesh P. Mokashi, Patrick S.G. Chain; Enabling the democratization of the genomics revolution with a fully integrated web-based bioinformatics platform, Nucleic Acids Research, Volume 45, Issue 1, 9 January 2017, Pages 67–80, https://doi.org/10.1093/nar/gkw1027

### Notes

Docker Desktop gRPC FUSE issue:
https://github.com/docker/for-mac/issues/4964
https://stackoverflow.com/questions/67521547/docker-mac-error-running-ls-on-large-directory

sed -i inplace replacement not work on VirtioFS. need to use sed 4.8 which can be installed using conda install sed:
https://forums.docker.com/t/sed-couldnt-open-temporary-file-xyz-permission-denied-when-using-virtiofs/125473/4

