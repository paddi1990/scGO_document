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
   * - scanpy
     - 1.9.3
   * - scikit-learn
     - 1.3.2
   * - scipy
     - 1.10.1


Conda is recommended for package management, you can create a new conda environment and then install the packages. Here's an example of how you can do it. Create a new conda environment::
    
    conda create -n scGO python=3.8.13

Activate the newly created environment::

    conda activate scGO

Install the required modules::

    pip install torch==1.9.1
    pip install scanpy==1.9.3
    pip install scikit-learn==1.3.2
    pip install scipy==1.10.1
    



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

  scGO
  ├── data
  │   ├── goa_human.gaf.zip
  │   └── TF_annotation_hg38.demo.tsv
  ├── demo
  │   ├── baron_data.csv
  │   ├── baron_data_filtered.csv
  │   ├── baron_data_filtered.predicted.csv
  │   ├── baron_meta_data.csv
  │   ├── baron_meta_data_senescence_score.csv
  │   ├── baron_meta_data_senescence_score.predicted.csv
  │   ├── feature
  │   ├── gene_TF_dict
  │   ├── gene_to_TF_transform_matrix
  │   ├── goa_human.gaf.zip
  │   ├── GO_mask
  │   ├── GO_TF_mask
  │   ├── test_data.csv
  │   ├── TF_annotation_hg38.demo.tsv
  │   ├── TF_gene_dict
  │   └── TF_mask
  ├── models
  │   ├── scGO.demo.pkl
  │   └── scGO.senescence_score.demo.pkl
  ├── README.md
  ├── results_reproduce
  ├── scGO.yml
  └── scripts
      ├── data_processing.py
      ├── __init__.py
      ├── models.py
      ├── __pycache__
      │   ├── models.cpython-38.pyc
      │   └── utils.cpython-38.pyc
      ├── rds_to_csv.r
      ├── scGO.py
      ├── test.csv
      ├── test.ipynb
      └── utils.py

