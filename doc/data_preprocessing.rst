.. _data_preprocessing:

Data preprocessing
==================================
This section involves processing single cell gene expression matrix and meta_data to train and test data. The preprocessing procedure typically consists of several stages.

* Data normalization and filtering.

* Build network structure using GO annotations.

* Build network structure using TF-gene relationships.

* Dataset creation.




Data normalization and filtering
********************

The scGO takes as input a single cell gene expression matrix and meta_data. The gene expression matrix is a matrix of size (n_cells, n_genes), where n_genes is the number of genes and n_cells is the number of cells.  The following is a gene expression matrix example.

.. csv-table:: Gene expression matrix
   :header: "", "A1BG", "A1CF", "A2M", "A2ML1", "A4GALT", "A4GNT", "AA06", "AAAS", "AACS", "AACSP1", "...", "ZWILCH", "ZWINT", "ZXDA", "ZXDB", "ZXDC", "ZYG11B", "ZYX", "ZZEF1", "ZZZ3"
   :widths: 30,5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5

   "human1_lib1.final_cell_0001", 0, 4, 0, 0, 0, 0, 0, 0, 0, 0, "...", 0, 0, 0, 0, 0, 0, 2, 0, 0
   "human1_lib1.final_cell_0002", 0, 0, 0, 0, 0, 0, 0, 0, 2, 0, "...", 0, 0, 0, 0, 0, 1, 4, 0, 1
   "human1_lib1.final_cell_0003", 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, "...", 0, 0, 0, 0, 0, 0, 0, 0, 0
   "human1_lib1.final_cell_0004", 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, "...", 1, 0, 0, 0, 0, 1, 3, 1, 0

The mata_data is a matrix of size (n_cells, n_meta_data), where n_meta_data is the number of meta_data. The meta_data should have a column named ``cell_type``. The following is a meta_data matrix example.

.. csv-table:: Meta data
   :header: "", "donor", "cell_type"
   :widths: 30, 30, 30

   "human1_lib1.final_cell_0001", "GSM2230757", "Acinar cells"
   "human1_lib1.final_cell_0002", "GSM2230757", "Acinar cells"
   "human1_lib1.final_cell_0003", "GSM2230757", "Acinar cells"
   "human1_lib1.final_cell_0004", "GSM2230757", "Acinar cells"

Normalization is a crucial step in the analysis of single-cell RNA sequencing (scRNA-seq) data. scGO employs total counts normalization, wherein each cell's gene expression values are normalized by dividing them by the counts per ten thousand (CP10K) of that cell. Other normalization methods are also effective in conjunction with scGO. If the gene expression matrix is already normalized, the normalization step can be skipped. Following that, scGO retains the top genes expressed in the majority of cells, with a recommended range of 2000-6000 genes for input. This process was implemented in the ``norm_and_filter`` command from the data processing script. The following is an example of the usage::

    python data_processing.py norm_and_filter --gene_expression_matrix ../demo/baron_data.tsv --num_genes 2000 --output ../demo/baron_data_filtered.tsv


Biological knowledge utilization
********************

In scGO, GO knowledge is employed to establish connections between gene nodes and GO nodes. A gene node is connected to a GO node if the gene is annotated with the GO term. The human GO annotation used in this study is downloaded from the `Gene Ontology knowledgebase <https://doi.org/10.5281/zenodo.7504797>`_. The connections between genes and TFs are builded according to the DAP-seq TF annotation data. The DAP-seq data is downloaded from the `Remap database <https://remap2022.univ-amu.fr/>`_. The demo data offers a subset of the processed DAP-seq file for illustrative purposes.  The full processed DAP-seq file has been uploaded to `google drive <https://drive.google.com/file/d/1VPSDyNbs4lBITm2VoPD2eJ3BZGcdkrdC/view?usp=drive_link>`_ The following is an example of the usage::

    python data_processing.py build_network  --go_annotation ../demo/go_annotation.tsv --tf_annotation ../demo/tf_annotation.tsv 


Data integration
********************
In scRNA-seq, batch effects can significantly impact the interpretation of scRNA-seq data and may lead to incorrect conclusions if not properly addressed. Data from various sources was harmonized using the Seurat `data integration pipeline <https://satijalab.org/seurat/articles/integration_mapping.html>`_ in this work. Initially, log normalization was applied, and variable features were identified independently for each dataset. Subsequently, the 'anchors,' representing matching cell populations across the individual datasets, were identified to combine these datasets into a unified Seurat object. This integration process aligns and harmonizes the data, allowing for coherent analysis across diverse sources.

