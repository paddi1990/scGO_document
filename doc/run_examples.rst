.. _run_examples:

Run examples
==================================
This section demostrates how to use scGO with examples.

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

Normalize data and filter out genes that expressed in fewer cells. The number of genes to be retained can be customized by using the argument ``--num_genes``. The default value is 2000. The normalized and filtered data will be saved in the ``./demo/`` folder.
::
    python scripts/data_processing.py norm_and_filter --gene_expression_matrix demo/baron_data.csv --num_genes 2000 --output demo/baron_data_filtered.csv

**2. Building network connections**

Build network connections between gene layer, TF layer and GO layer according to GO annotations and TF annotations.
::
    python scripts/data_processing.py build_network --gene_expression_matrix demo/baron_data_filtered.csv --GO_annotation demo/goa_human.gaf  --TF_annotation demo/TF_annotation_hg38.demo.tsv

**3. Model training**

To train the scGO model using your own dataset from scratch, you can run the ``scGO.py`` script with the command ``train``. scGO accepts both single-cell gene expression matrix (.csv) as input. Meta data (.csv) is needed for the training process. The column cell_type in the meta data is used as the label for training. You can specify the model save path by using the argument ``--model``. The model's training epochs can be defined using the argument ``--epochs``, and the model states will be saved at the end of each epoch. The training process duration can vary, depending on the size of your dataset and the computational capacity, and may range from minutes to several hours. Here is an example of the training process using the demo dataset.
::
    python scripts/scGO.py train --gene_expression_matrix demo/baron_data_filtered.csv --meta_data demo/baron_meta_data.csv --model models/scGO.demo.pkl

During the training process, the following information can be used to monitor and evaluate the performance of the model:
::
    epoch 0         accuracy:        0.8    loss:    0.6765657464663187
    epoch 1         accuracy:        1.0    loss:    0.21495116502046585 
    epoch 2         accuracy:        1.0    loss:    0.045270659029483795
    epoch 3         accuracy:        1.0    loss:    0.01799739959339301
    epoch 4         accuracy:        1.0    loss:    0.008456315845251083
    epoch 5         accuracy:        1.0    loss:    0.0038305727454523244
    epoch 6         accuracy:        1.0    loss:    0.0015001039331158001
    epoch 7         accuracy:        1.0    loss:    0.0006393008710195621
    epoch 8         accuracy:        1.0    loss:    0.0003799065889324993
    epoch 9         accuracy:        1.0    loss:    0.00025623222851815325
    epoch 10        accuracy:        1.0    loss:    0.0001992921606870368
    epoch 11        accuracy:        1.0    loss:    0.0001639570424837681
    epoch 12        accuracy:        1.0    loss:    0.00013759526094266525
    epoch 13        accuracy:        1.0    loss:    0.0001203383071697317



Application of pretrained scGO to predict new data
********************

**Predicting new data**

After the completion of the training process, the model file ``scGO.demo.pkl`` will be stored in the ``./models/`` folder. This trained model can be employed to make predictions on new data. Use the ``predict`` command to predict new data, and assign the predicted results using the ``--output`` argument.
::
    python scripts/scGO.py predict --gene_expression_matrix demo/baron_data.csv  --model models/scGO.demo.pkl --output demo/baron_data_filtered.predicted.csv

The prediction results include a predicted cell type label along with the confidence probability of the predicted cell type. The following serves as an example of the prediction results.::

                            cell_id       predicted cell_type      probability
    0   human1_lib1.final_cell_0123       Epsilon cells            0.999998
    1   human1_lib1.final_cell_0288       Macrophages              1.000000
    2   human1_lib1.final_cell_0309       Epsilon cells            0.999936
    3   human1_lib1.final_cell_0323       Epsilon cells            0.999969
    4   human1_lib1.final_cell_0417       Macrophages              0.999999
    ..  ...                               ...                      ...
    68  human4_lib1.final_cell_0349       Macrophages              0.999999
    69  human4_lib1.final_cell_0579       Macrophages              1.000000
    70  human4_lib3.final_cell_0064       Macrophages              0.999999
    71  human4_lib3.final_cell_0215       Macrophages              1.000000
    72  human4_lib3.final_cell_0574       Macrophages              1.000000


**Reporting novel cell type**


scGO provided the configuration to indiate novel cell type by setting the argument ``--indicate_novel_cell_type`` to ``True``. The predictions with low confident probability will be asigned as novel cell type. The following serves as an example of the prediction results with novel cell type.
::
    python scripts/scGO.py predict --gene_expression_matrix demo/baron_data.csv  --model models/scGO.demo.pkl --indicate_novel_cell_type True --output demo/baron_data_filtered_novel.predicted.csv


                            cell_id        predicted cell_type     probability
    0   human4_lib1.final_cell_0035        Macrophages             0.999773
    1   human3_lib3.final_cell_0413        Macrophages             1.000000
    2   human1_lib1.final_cell_0428        novel cell type         0.515880
    3   human3_lib3.final_cell_0819        Epsilon cells           0.880849
    4   human3_lib3.final_cell_0621        Epsilon cells           0.998823
    ..  ...                                ...                     ...
    93  human2_lib1.final_cell_0399        Epsilon cells           0.999883
    94  human2_lib1.final_cell_0544        Macrophages             0.999957
    95  human4_lib1.final_cell_0326        Epsilon cells           0.999983
    96  human2_lib3.final_cell_0147        Macrophages             1.000000
    97  human4_lib1.final_cell_0295        Macrophages             0.999955




Train a regression model that predict a continous value
********************

In addition to discrete cell types, we provided a regression mode (set the ``task`` argument to ``regression``) to predict a continous cell status. The demo dataset contains a meta data ``baron_meta_data_senescence_score.csv`` with a column ``senescence_score`` under the ``demo`` directory. The ``senescence_score`` is a continous value. We can train a regression model to predict the ``senescence_score``. The following serves as an example of the training process using the demo dataset. The data processing and connections building are similar to the classification model. The sole distinction lies in setting the ``task`` argument to ``regression`` and specifying the ``label`` argument to correspond to a column in the metadata that you aim to predict.

**1. Data normalization and filtering**

Normalize data and filter out genes that expressed in fewer cells. The number of genes to be retained can be customized by using the argument ``--num_genes``. The default value is 2000. The normalized and filtered data will be saved in the ``./demo/`` folder.
::
    python scripts/data_processing.py norm_and_filter --gene_expression_matrix demo/baron_data.csv --num_genes 2000 --output demo/baron_data_filtered.csv

**2. Building network connections**

Build network connections between gene layer, TF layer and GO layer according to GO annotations and TF annotations.
::
    python scripts/data_processing.py build_network --gene_expression_matrix demo/baron_data_filtered.csv --GO_annotation demo/goa_human.gaf  --TF_annotation demo/TF_annotation_hg38.demo.tsv

**3. Training regression model**

Set the ``task`` argument to ``regression`` and specify the ``label`` argument to correspond to a column in the metadata that you aim to predict.
::
    python scripts/scGO.py train --gene_expression_matrix demo/baron_data_filtered.csv --task regression --epoch 100 --batch_size 8 --meta_data demo/baron_meta_data_senescence_score.csv --label senescence_score --model models/scGO.senescence_score.demo.pkl

**4. Predicitng new data**

Load pre-trained model and predict new data. The predicted results include a predicted cell type label along with the predicted value. 
::
    python scripts/scGO.py predict --gene_expression_matrix demo/baron_data.csv --task regression --model models/scGO.senescence_score.demo.pkl --output demo/baron_meta_data_senescence_score.predicted.csv

After the data processing and model training, the following files should be generated by scGO. The trained model will be saved in the ``./models/`` folder. You can utilize this model for making predictions in the future.
::
    demo
    ├── baron_data.csv
    ├── baron_data_filtered.csv
    ├── baron_data_filtered.predicted.csv
    ├── baron_meta_data.csv
    ├── baron_meta_data_senescence_score.csv
    ├── baron_meta_data_senescence_score.predicted.csv
    ├── feature
    ├── gene_TF_dict
    ├── gene_to_TF_transform_matrix
    ├── goa_human.gaf.zip
    ├── GO_mask
    ├── GO_TF_mask
    ├── test_data.csv
    ├── TF_annotation_hg38.demo.tsv
    ├── TF_gene_dict
    └── TF_mask
    models
    ├── scGO.demo.pkl
    └── scGO.senescence_score.demo.pkl



The execution time for each demonstration is estimated to be approximately 0-3 minutes.