## University of California, Santa Cruz Genomics Institute
### Guide: Running the CGL HGVM project VG Pipeline using Toil

This guide attempts to walk the user through running this pipline using the VG framework from start to finish.
If there are any questions please contact Charles Markello (cmarkell@ucsc.edu). If you find any errors or corrections please
send feedback or feel free to make a pull request. 

## Basic start-to-finish run of the VG-toil pipeline on Amazon EC2

### Install and setup local cgcloud

* `sudo apt-get install python-virtualenv`
* `virtualenv ~/cgcloud`
* Edit your ~/.profile or ~/.bash_profile and add the following lines:
    * `export CGCLOUD_ZONE=us-west-2a`
    * `export CGCLOUD_PLUGINS="cgcloud.toil:$CGCLOUD_PLUGINS"`
* Setup credentials for your AWS account in ~/.aws/credentials:
    * `[default]`
    * `aws_access_key_id=PASTE_YOUR_FOO_ACCESS_KEY_ID_HERE`
    * `aws_secret_access_key=PASTE_YOUR_FOO_SECRET_KEY_ID_HERE`
    * `region=us-west-2`
* `source ~/cgcloud/bin/activate`
* `pip install cgcloud-toil`
* `cgcloud register-key ~/.ssh/id_rsa.pub`

For more information on the cgcloud core tools, you can find them [here](https://github.com/BD2KGenomics/cgcloud/blob/master/README.md).
For more information on the cgcloud plugin for Toil, you can read about it [here](https://github.com/BD2KGenomics/cgcloud/blob/master/toil/README.rst).

### Startup a Toil EC2 cluster

- `cgcloud create -IT toil-box`
- `cgcloud create-cluster toil -s 3 -t m4.xlarge --cluster-name toil-setup-test`
- `cgcloud ssh --admin -c toil-setup-test toil-leader 'sudo apt-get install git && sudo apt-get install python-virtualenv && sudo pip install awscli'`
- `cgcloud ssh-cluster --admin --cluster-name toil-setup-test toil 'sudo pip install toil[aws,mesos]==3.5.0a1.dev241'`
- `cgcloud ssh -c toil-setup-test toil-leader`

### Set up dependencies

- `virtualenv --system-site-packages toilvenv`
- `source toilvenv/bin/activate`
- `git clone --recursive https://github.com/cmarkello/toil-lib.git`
- `pip install /home/mesosbox/toil-lib/`
- `git clone --recursive https://github.com/BD2KGenomics/toil-vg.git`
- `pip install /home/mesosbox/toil-vg/`

For more information on the latest release of Toil, you can find the documentation [here](http://toil.readthedocs.io/en/latest/).

### Get test input files from s3

- `mkdir /home/mesosbox/debug_eval_input/`
- `aws s3 cp s3://cmarkello-vgtoil-test--files/BRCA1_BRCA2_may6.vg /home/mesosbox/debug_eval_input/BRCA1_BRCA2_may6.vg`
- `aws s3 cp s3://cmarkello-vgtoil-test--files/NA12877.brca1.brca2.bam.fq /home/mesosbox/debug_eval_input/NA12877.brca1.brca2.bam.fq`

### Example run of VG toil-pipeline for variant calling on chromosome 13 for sample NA12877

- `toil clean aws:us-west-2:cmarkello-s3-jobstore`
- `toil-vg --batchSystem mesos --mesosMaster=mesos-master:5050 --realTimeLogging --logError --logDebug --edge_max 5 --kmer_size 16 --num_fastq_chunks 3 --call_chunk_size 10000 --overwrite --index_mode gcsa-mem --include_primary 'aws:us-west-2:cmarkello-s3-jobstore' '/home/mesosbox/debug_eval_input/BRCA1_BRCA2_may6.vg' '/home/mesosbox/debug_eval_input/NA12877.brca1.brca2.bam.fq' 'NA12877' '/home/mesosbox/debug_eval_output' 'aws:us-west-2:cmarkello-hgvmdebugtest-input' 'aws:us-west-2:cmarkello-hgvmdebugtest-output' --path_name 13 --path_size 84989`

### Example run of VG toil-pipeline for variant calling on chromosome 17 for sample NA12877

- `toil clean aws:us-west-2:cmarkello-s3-jobstore`
- `toil-vg --batchSystem mesos --mesosMaster=mesos-master:5050 --realTimeLogging --logError --logDebug --edge_max 5 --kmer_size 16 --num_fastq_chunks 3 --call_chunk_size 10000 --overwrite --index_mode gcsa-mem --include_primary 'aws:us-west-2:cmarkello-s3-jobstore' '/home/mesosbox/debug_eval_input/BRCA1_BRCA2_may6.vg' '/home/mesosbox/debug_eval_input/NA12877.brca1.brca2.bam.fq' 'NA12877' '/home/mesosbox/debug_eval_output' 'aws:us-west-2:cmarkello-hgvmdebugtest-input' 'aws:us-west-2:cmarkello-hgvmdebugtest-output' --path_name 17 --path_size 81189`

### Example run of VG toil-pipeline for variant calling on both chromosome 13 and 17 for sample NA12877

- `toil clean aws:us-west-2:cmarkello-s3-jobstore`
- `toil-vg --batchSystem mesos --mesosMaster=mesos-master:5050 --realTimeLogging --logError --logDebug --edge_max 5 --kmer_size 16 --num_fastq_chunks 3 --call_chunk_size 10000 --overwrite --index_mode gcsa-mem --include_primary 'aws:us-west-2:cmarkello-s3-jobstore' '/home/mesosbox/debug_eval_input/BRCA1_BRCA2_may6.vg' '/home/mesosbox/debug_eval_input/NA12877.brca1.brca2.bam.fq' 'NA12877' '/home/mesosbox/debug_eval_output' 'aws:us-west-2:cmarkello-hgvmdebugtest-input' 'aws:us-west-2:cmarkello-hgvmdebugtest-output' --path_name 13 17 --path_size 84989 81189`

### Get test output files from s3

- `aws s3 cp s3://cmarkello-vgtoil-test--files/normal_13.vcf /home/mesosbox/debug_eval_output/normal_13.vcf`
- `aws s3 cp s3://cmarkello-vgtoil-test--files/normal_17.vcf /home/mesosbox/debug_eval_output/normal_17.vcf`
- `aws s3 cp s3://cmarkello-vgtoil-test--files/normal_vcf_NA12877.vcf /home/mesosbox/debug_eval_output/normal_vcf_NA12877.vcf`

### Test consistancy of chr 13 variant call output

- `cd /home/mesosbox/debug_eval_output/`
- `gzip -d 13.vcf.gz`
- `diff 13.vcf normal_13.vcf`

### Test consistancy of chr 17 variant call output

- `cd /home/mesosbox/debug_eval_output/`
- `gzip -d 17.vcf.gz`
- `diff 17.vcf normal_17.vcf`

### Test consistancy of chr 13 and chr 17 variant call output

- `cd /home/mesosbox/debug_eval_output/`
- `gzip -d NA12877.vcf.gz`
- `diff NA12877.vcf normal_vcf_NA12877.vcf`

#### Delete cgcloud cluster

- `cgcloud terminate-cluster --cluster-name toil-setup-test toil`

## Basic start-to-finish run of the VG-toil pipeline on Microsoft Azure

### Startup a Toil Azure cluster

- Go to the toil azure readme instructions [here](https://github.com/BD2KGenomics/toil/tree/master/contrib/azure#mesos-cluster-with-toil).
- Click on the `Deploy to Azure` button to use the toil Azure template for setting up a toil Azure cluster.
- Follow the instructions on setup. You will need to specify the following parameters as defined [here](https://github.com/BD2KGenomics/toil/tree/master/contrib/azure#template-parameters).
- Login to the master node of the Azure cluster by running `ssh <your_user_name>@<your_cluster_name>.<your_zone>.cloudapp.azure.com -p 2211`.

### Set up dependencies

- `virtualenv --system-site-packages toilvenv`
- `source toilvenv/bin/activate`
- `git clone --recursive https://github.com/cmarkello/toil-lib.git`
- `pip install ~/toil-lib/`
- `git clone --recursive https://github.com/BD2KGenomics/toil-vg.git`
- `pip install ~/toil-vg/`

### Get test input files from s3

- `mkdir ~/test_vg_input/`
- `pip install awscli`
- Setup aws credentials by running `aws configure` and filling in the prompted parameters.
- `aws s3 cp s3://cmarkello-vgtoil-test--files/BRCA1_BRCA2_may6.vg ~/test_vg_input/BRCA1_BRCA2_may6.vg`
- `aws s3 cp s3://cmarkello-vgtoil-test--files/NA12877.brca1.brca2.bam.fq ~/test_vg_input/NA12877.brca1.brca2.bam.fq`

### Example run of VG toil-pipeline for variant calling on chromosome 13 for sample NA12877

- `toil clean azure:hgvm:cmarkello-toilvg-jobstore`
- `toil-vg --batchSystem mesos --mesosMaster=10.0.0.5:5050 --realTimeLogging --logError --logDebug --edge_max 5 --kmer_size 16 --num_fastq_chunks 8 --call_chunk_size 10000 --overwrite --index_mode gcsa-mem --include_primary --index_cores 16 --alignment_cores 16 --calling_cores 16 'azure:hgvm:cmarkello-toilvg-jobstore' ${PWD}/test_vg_input/BRCA1_BRCA2_may6.vg ${PWD}/test_vg_input/NA12877.brca1.brca2.bam.fq 'NA12877' ${PWD}/test_vg_output 'azure:hgvm:cmarkello-toilvg-input' 'azure:hgvm:cmarkello-toilvg-output' --path_name 13 --path_size 84989`

### Test consistency of chr 13 variant call output

- `gzip -d ~/test_vg_output/13.vcf.gz`
- `aws s3 cp s3://cmarkello-vgtoil-test--files/normal_13.vcf ~/test_vg_output/normal_13.vcf`
- `diff ~/test_vg_output/13.vcf ~/test_vg_output/normal_13.vcf`


