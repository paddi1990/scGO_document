Usage
=====


Source code
********************

The source code and data processing scripts are available on `GitHub <https://github.com/yulab2021/scGO>`_. You can download them by using the git clone command::

    git clone https://github.com/yulab2021/scGO.git


In the provided repository, the pretrained models are located under the ``./models`` directory, and the data processing scripts and the main script are located under the ``./scripts`` directory. The demo data is located under the ``./demo`` directory.::

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
        ├── test.ipynb
        └── utils.py




Training cell type annotation model
********************
Train scGO model using `Baron dataset <https://www.cell.com/cell-systems/fulltext/S2405-4712(16)30266-6?_returnURL=https%3A%2F%2Flinkinghub.elsevier.com%2Fretrieve%2Fpii%2FS2405471216302666%3Fshowall%3Dtrue>`_.
********************
The Baron dataset covers a range of cell types found in the pancreas, including acinar cells, activated stellate cells, alpha cells, beta cells, delta cells, ductal cells, endothelial cells, epsilon cells, gamma cells, macrophages, mast cells, quiescent stellate cells, and schwann cells. Baron dataset is available at `GSE84133 <https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE84133>`_. In this demo, a subsets of the the baron dataset was taken for demonstration purposes due to the large size of the original datasets. The demo dataset was located under ``./demo/`` directory.
::
    
    demo
    ├── baron_data.csv
    ├── baron_meta_data.csv
    ├── goa_human.gaf
    └── TF_annotation_hg38.demo.tsv


**1. Data normalization and filtering**

Normalize data and filter out genes that expressed in fewer cells using script ``data_processing.py`` with command ``norm_and_filter``. The number of genes to be retained can be customized by using the argument ``--num_genes``. The default value is 2000. The normalized and filtered data will be saved in the ``./demo/`` folder.
::
    usage: data_processing.py norm_and_filter [-h] [--gene_expression_matrix GENE_EXPRESSION_MATRIX] [--num_genes NUM_GENES] [--output OUTPUT]
    optional arguments:
        -h, --help            show this help message and exit
        --gene_expression_matrix GENE_EXPRESSION_MATRIX       Path to the input data
        --num_genes NUM_GENES      Number of top genes to keep
        --output OUTPUT        Path to the output data

    expample:
    python scripts/data_processing.py norm_and_filter --gene_expression_matrix demo/baron_data.csv --num_genes 2000 --output demo/baron_data_filtered.csv

**2. Building network connections**

Build network connections between gene layer, TF layer and GO layer according to GO annotations and TF annotations using script ``data_processing.py`` with command ``build_network``. The human GO annotation used in this study is downloaded from the `Gene Ontology knowledgebase <https://doi.org/10.5281/zenodo.7504797>`_. The connections between genes and TFs are builded according to the DAP-seq TF annotation data. The DAP-seq data is downloaded from the `Remap database <https://remap2022.univ-amu.fr/>`_. The demo data offers a subset of the processed DAP-seq file for illustrative purposes.  The full processed DAP-seq file has been uploaded to `google drive <https://drive.google.com/file/d/1VPSDyNbs4lBITm2VoPD2eJ3BZGcdkrdC/view?usp=drive_link>`_. 
::
    usage: data_processing.py build_network [-h] [--gene_expression_matrix GENE_EXPRESSION_MATRIX] [--GO_annotation GO_ANNOTATION] [--TF_annotation TF_ANNOTATION]
    optional arguments:
        -h, --help            show this help message and exit
        --gene_expression_matrix GENE_EXPRESSION_MATRIX     Path to the input data
        --GO_annotation GO_ANNOTATION         Path to the GO annotation file
        --TF_annotation TF_ANNOTATION         Path to the TF annotation file
    
    example:
    python scripts/data_processing.py build_network --gene_expression_matrix demo/baron_data_filtered.csv --GO_annotation demo/goa_human.gaf  --TF_annotation demo/TF_annotation_hg38.demo.tsv

**3. Model training**

To train the scGO model using your own dataset from scratch, you can run the ``scGO.py`` script with the command ``train``. scGO accepts both single-cell gene expression matrix (.csv) as input. Meta data (.csv) is needed for the training process. The column cell_type in the meta data is used as the label for training. You can specify the model save path by using the argument ``--model``. The model's training epochs can be defined using the argument ``--epochs``, and the model states will be saved at the end of each epoch. The training process duration can vary, depending on the size of your dataset and the computational capacity, and may range from minutes to several hours. Here is an example of the training process using the demo dataset.
::
    
    usage: scGO.py train [-h] --gene_expression_matrix GENE_EXPRESSION_MATRIX [--meta_data META_DATA] --model MODEL [--task TASK] [--epoch EPOCH] [--batch_size BATCH_SIZE] [--learning_rate LEARNING_RATE]  [--label LABEL]

    optional arguments:
        -h, --help            show this help message and exit  
        --gene_expression_matrix GENE_EXPRESSION_MATRIX      Train data file.
        --meta_data META_DATA        Meta data, including cell type labels.
        --model MODEL         Output model file.
        --task TASK           Task type, classification or regression.
        --epoch EPOCH         Number of training epochs.
        --batch_size BATCH_SIZE      Batch size.
        --learning_rate LEARNING_RATE       Learning rate.
        --label LABEL         Column in meta_data to be predicted.

    expample:
    python scripts/scGO.py train --gene_expression_matrix demo/baron_data_filtered.csv --meta_data demo/baron_meta_data.csv --model models/scGO.demo.pkl


Training regression model to predict a continuous cell status
********************

Set the ``task`` argument to ``regression`` and specify the ``label`` argument to correspond to a column in the metadata that you aim to predict.
::
    python scripts/scGO.py train --gene_expression_matrix demo/baron_data_filtered.csv --task regression --epoch 100 --batch_size 8 --meta_data demo/baron_meta_data_senescence_score.csv --label senescence_score --model models/scGO.senescence_score.demo.pkl

To predict new data, load the pre-trained model, and generate predictions for the new data. The results should include both the predicted cell type label and the associated predicted value.
::
    python scripts/scGO.py predict --gene_expression_matrix demo/baron_data.csv --task regression --model models/scGO.senescence_score.demo.pkl --output demo/baron_meta_data_senescence_score.predicted.csv