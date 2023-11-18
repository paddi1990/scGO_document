.. _installation:

Installation
==================================
The following modules are needed to run scGO.


.. list-table:: Required modules
   :widths: 50 50
   :header-rows: 1

   * - module
     - version
   * - python 
     - 3.8.13
  * - torch
     - 1.9.1


Conda is recommended for package management, you can create a new conda environment and then install the packages. Here's an example of how you can do it. Create a new conda environment::
    
    conda create -n scGO python=3.8.13

Activate the newly created environment::

    conda activate scGO

Install the required modules::

    pip install torch==1.9.1
    pip install scanpy==1.9.3
    



The entire installation will take about 1-5 minutes. After installing all the essential packages,  reset the environment's state by deactivating and reactivating the environment:
::
    conda deactivate
    conda activate scGO

We have also provided a yaml file in the repository so you can install the dependencies through the configuration file::

    conda env create -f scGO.yaml


The source code and data processing scripts are available on `GitHub <https://github.com/yulab2021/scGO>`_. You can download them by using the git clone command::

    git clone https://github.com/yulab2021/scGO.git

TandemMod offers three modes: de novo training, transfer learning, and prediction. Researchers can train from scratch, fine-tune pre-trained models, or apply existing models for predictions. It provides a user-friendly solution for studying RNA modifications.

In the provided repository, the pretrained models are located under the ``./models`` directory, and the data processing scripts and the main script are located under the ``./scripts`` directory:: 

    .
    ├── data
    │   ├── A_test.tsv
    │   ├── A_train.tsv
    │   ├── m5C
    │   ├── m6A
    │   ├── m6A_test.tsv
    │   └── m6A_train.tsv
    ├── demo
    │   ├── fast5
    │   │   └── batch_0.fast5
    │   ├── files.txt
    │   ├── guppy
    │   │   ├── fail
    │   │   │   └── fastq_runid_71d544d3bd9e1fe7886a5d176c756a576d30ed50_0_0.fastq
    │   │   ├── guppy_basecaller_log-2023-06-06_09-58-28.log
    │   │   ├── pass
    │   │   │   └── fastq_runid_71d544d3bd9e1fe7886a5d176c756a576d30ed50_0_0.fastq
    │   │   ├── sequencing_summary.txt
    │   │   ├── sequencing_telemetry.js
    │   │   └── workspace
    │   │       └── batch_0.fast5
    ├── models
    │   ├── hm5C_transfered_from_m5C.pkl
    │   ├── m1A_train_on_rice_cDNA.pkl
    │   ├── m5C_train_on_rice_cDNA.pkl
    │   ├── m6A_train_on_rice_cDNA.pkl
    │   ├── m7G_transfered_from_m5C.pkl
    │   ├── psU_transfered_from_m5C.pkl
    │   ├── test.model
    │   └── test.pkl
    ├── plot
    ├── README.md
    ├── scripts
    │   ├── extract_feature_from_signal.py
    │   ├── extract_signal_from_fast5.py
    │   ├── __init__.py
    │   ├── models.py
    │   ├── TandemMod.py
    │   ├── train_test_split.py
    │   ├── transcriptome_loci_to_genome_loci.py
    │   └── utils.py
    └── TandemMod.yaml
