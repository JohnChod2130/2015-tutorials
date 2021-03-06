##Changing the default parameters in QIIME
Authored by Siobhan Cusack for EDAMAME 2015

[EDAMAME-2015 wiki](https://github.com/edamame-course/2015-tutorials/wiki)

***
EDAMAME tutorials have a CC-BY [license](https://github.com/edamame-course/2015-tutorials/blob/master/LICENSE.md). _Share, adapt, and attribute please!_
***

When using a pipeline such as QIIME, you may want to change the default parameters of a script. To do this, you will need to create a parameters file. QIIME has a good overview of parameters files [here](http://qiime.org/documentation/qiime_parameters_files.html).

As an example, when using [pick_open_reference_otus.py](http://qiime.org/scripts/pick_open_reference_otus.html), I may want to change the OTU cutoff to 99% instead of the default 97%. Let's try this.

Start up an EC2 instance with the QIIME AMI (ami-1918ff72) and grab the combined_seqs_smaller.fna file, which is a file containing just a fraction of our full sequences for demonstration purposes:

```
curl -O https://raw.githubusercontent.com/edamame-course/2015-tutorials/master/QIIME_files/combined_seqs_smaller.fna

```

To make sure that the workflow script is running properly, it's a good idea to inspect the log file carefully. To get an idea of what the log file will look like when we use the default parameters, let's run it as we did in the QIIME tutorial.
If you don't already have usearch installed, do that now:
```
curl -O https://raw.githubusercontent.com/edamame-course/2015-tutorials/master/QIIME_files/usearch5.2.236_i86linux32
curl -O https://raw.githubusercontent.com/edamame-course/2015-tutorials/master/QIIME_files/usearch6.1.544_i86linux32
sudo cp usearch5.2.236_i86linux32 /usr/local/bin/usearch
sudo chmod +x /usr/local/bin/usearch
sudo cp usearch6.1.544_i86linux32 /usr/local/bin/usearch61
sudo chmod +x /usr/local/bin/usearch61
```
In the home directory (containing the combined_seqs_smaller.fna file), run the following:

```
pick_open_reference_otus.py -i combined_seqs_smaller.fna -m usearch61 -o usearch61_openref_97/ -f

```
This will take just a couple of minutes to run. Once it has finished, navigate into the usearch61_openref_97 directory and use ```more``` to inspect the log file fully.

![img1](../img/similarity_97.jpg)

Navigate back to the home directory and grab the parameters file:
```
curl -O https://raw.githubusercontent.com/edamame-course/2015-tutorials/master/QIIME_files/poro_parameters.txt
```
Use nano to inspect it.

![img2](../img/parameters.jpg)

The parameters file specifies to the workflow script that when it gets to the pick_otus step, it should use a cutoff of 99% instead of 97%. It also specifies that when it gets to the assign_taxonomy step, it should use a file from QIIME that was created using a 99% rather than a 97% cutoff. 

The format here is to specify the script, the option, and the setting we want to use. It's similar to running a script with a flag (in the case of pick_otus here, "similarity" would be the flag) and the option that we would normally put after the flag goes after that. Important note: when making your parameters file, write out the script (without .py), then a colon, then the option, without any spaces. Then hit tab, and type your specification. The space between the option and your setting IS important.

Now we're going to run pick_open_reference OTUs with our new parameters file from the home directory.  

```
pick_open_reference_otus.py -i combined_seqs_smaller.fna -m usearch61 -o usearch61_openref_99/ -f -p poro_parameters.txt
```
Once it finishes running, inspect the log file using ```more```. 

![img3](../img/similarity_99.jpg)

That's it! You can now change the OTU cutoff to anything your heart desires!


Now for something more complicated; say I want to change the database used for clustering and aligning. We'll try Silva.
Go to Silva's [QIIME-compatible download page](http://www.arb-silva.de/no_cache/download/archive/qiime/), and click Silva_108_release_curated.tgz.

Save this in a place where you can find it, like the desktop, use scp to transfer the file to your Amazon instance, stick it in the home directory, and unzip it:

```
scp -i [your key file] Silva_108_release_curated.tgz ubuntu@ec2-[your DNS]
mv Silva_108_release_curated.tgz /home/ubuntu
cd /home/ubuntu
tar -xvzf Silva_108_release_curated.tgz

```
This will generate a new directory called "Silva_108_database_curated". Navigate there and inspect the new files if you'd like to. The format notes file can help answer a lot of questions you may have about how the database was created.

![img5](../img/silva_unzipped.jpg)

When you are ready to run the command, navigate back into the home directory and make a new parameters file.

```
nano parameters_Silva.txt
```
Copy and paste these lines, making sure to add the correct file path if you unzipped the Silva database somewhere other than the home directory:

```
align_seqs:template_fp    /home/ubuntu/Silva_108_database_curated/97_rep_set_Silva_108_aligned.fasta 

assign_taxonomy:id_to_taxonomy_fp   /home/ubuntu/Silva_108_database_curated/97_taxa_map_Silva_108.txt

assign_taxonomy:reference_seqs_fp   /home/ubuntu/Silva_108_database_curated/97_rep_set_Silva_108.fasta

```
Exit and save the file. We need to specify one more thing, which is the reference database to use during the actual pick_otus.py step. It is not possible to add this specification to the parameters file (you can, but it won't run), so we will add a -r flag to specify this.

```
pick_open_reference_otus.py -i combined_seqs_smaller.fna -m usearch61 -o usearch61_openref_Silva/ -f -p parameters_Silva.txt -r /home/ubuntu/Silva_108_database_curated/97_rep_set_Silva_108.fasta 

```
Let it run, then inspect the log file to ensure that the correct database was used:
![img4](../img/silva2.jpg)

Hooray! You can now use any database you like so long as the files are formatted correctly and you specify them in the command and parameters file.

#Resources and help
## QIIME
  - QIIME documentation on [parameters files](http://qiime.org/documentation/qiime_parameters_files.html).
  - [QIIME](qiime.org) offers a suite of developer-designed [tutorials](http://www.qiime.org/tutorials/tutorial.html).
  - [Documentation](http://www.qiime.org/scripts/index.html) for all QIIME scripts.
  - There is a very active [QIIME Forum](https://groups.google.com/forum/#!forum/qiime-forum) on Google Groups.  This is a great place to troubleshoot problems, responses often are returned in a few hours!
  - The [QIIME Blog](http://qiime.wordpress.com/) provides updates like bug fixes, new features, and new releases.
  - QIIME development is on [GitHub](https://github.com/biocore/qiime).
  - Remember that QIIME is a workflow environment, and the original algorithms/software that are compiled into QIIME must be referenced individually (e.g., PyNAST, RDP classifier, FastTree etc...)



