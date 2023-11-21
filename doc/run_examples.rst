.. _run_examples:

Run examples
==================================
This section demostrates how to use scGO with examples.

Train scGO model using `Baron dataset <https://www.cell.com/cell-systems/fulltext/S2405-4712(16)30266-6?_returnURL=https%3A%2F%2Flinkinghub.elsevier.com%2Fretrieve%2Fpii%2FS2405471216302666%3Fshowall%3Dtrue>`_.
********************
The Baron dataset covers a range of cell types found in the pancreas, including acinar cells, activated stellate cells, alpha cells, beta cells, delta cells, ductal cells, endothelial cells, epsilon cells, gamma cells, macrophages, mast cells, quiescent stellate cells, and schwann cells. Baron dataset is available at `GSE84133 <https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE84133>`_. In this demo, a subsets of the the baron dataset was taken for demonstration purposes due to the large size of the original datasets. The demo dataset was located under ``../demo/`` directory.
::
    
    demo
    ├── baron_data.csv
    └── baron_meta_data.csv


**1. Data normalization and filtering**

Normalize data and filter out genes that expressed in fewer cells. The number of genes to be retained can be customized by using the argument ``--num_genes``. The default value is 2000. The normalized and filtered data will be saved in the ``./demo/`` folder.
::
    python scripts/data_processing.py norm_and_filter --gene_expression_matrix demo/baron_data.csv --num_genes 2000 --output demo/baron_data_filtered.csv

**2. Building network connections**

Build network connections between gene layer, TF layer and GO layer according to GO annotations and TF annotations.
::
    python scripts/data_processing.py build_network --gene_expression_matrix demo/baron_data_filtered.csv --GO_annotation demo/goa_human.gaf  --TF_annotation demo/TF_annotation_hg38.demo.tsv

**3. Model training**

In this step, the sequence obtained by basecalling is aligned or mapped to a reference genome or a known sequence. Then the corrected sequence is then associated with the corresponding current signals. The resquiggling process is typically performed in-place. No separate files are generated in this step.
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

**Predict new data**
After the completion of the training process, the model file ``scGO.demo.pkl`` will be stored in the ``./models/`` folder. This trained model can be employed to make predictions on new data. Use the provided command to predict new data, and assign the predicted results using the ``--output`` argument.
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


**Report novel cell type**


scGO provided the configuration to indiate novel cell type by setting the argument ``--indicate_novel_cell_type`` to ``True``. The prediction with low confident probability will be asigned as novel cell type. The following serves as an example of the prediction results.::

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

To train the TandemMod model using your own dataset from scratch, you can set the ``--run_mode`` argument to "train". TandemMod accepts both modified and unmodified feature files as input. Additionally, test feature files are necessary to evaluate the model's performance. You can specify the model save path by using the argument ``--new_model``. The model's training epochs can be defined using the argument ``--epochs``, and the model states will be saved at the end of each epoch. TandemMod will preferentially use the ``GPU`` for training if CUDA is available on your device; otherwise, it will utilize the ``CPU`` mode. The training process duration can vary, depending on the size of your dataset and the computational capacity, and may last for several hours. 
::
    python scripts/scGO.py train --gene_expression_matrix demo/baron_data_filtered.csv --task regression --epoch 100 --batch_size 8 --meta_data demo/baron_meta_data_senescence_score.csv --label senescence_score --model models/scGO.senescence_score.demo.pkl

Predict:
::
    python scripts/scGO.py predict --gene_expression_matrix demo/baron_data.csv --task regression --model models/scGO.senescence_score.demo.pkl --output demo/baron_meta_data_senescence_score.predicted.csv

After the data processing and model training, the following files should be generated by TandemMod. The trained model ``m6A.demo.IVET.pkl`` will be saved in the ``./demo/model/`` folder. You can utilize this model for making predictions in the future.
::
    demo
    ├── IVET
    │   ├── IVET_m6A
    │   ├── IVET_m6A.fastq
    │   ├── IVET_m6A_guppy
    │   ├── IVET_m6A_guppy_single
    │   ├── IVET_m6A.sam
    │   ├── IVET_unmod
    │   ├── IVET_unmod.fastq
    │   ├── IVET_unmod_guppy
    │   ├── IVET_unmod_guppy_single
    │   ├── IVET_unmod.sam
    │   ├── m6A.feature.tsv
    │   ├── m6A.signal.tsv
    │   ├── m6A.test.feature.tsv
    │   ├── m6A.train.feature.tsv
    │   ├── unmod.feature.tsv
    │   ├── unmod.signal.tsv
    │   ├── unmod.test.feature.tsv
    │   └── unmod.train.feature.tsv
    ├── IVET_reference.fa
    └── model
           └── m6A.demo.IVET.pkl


Train m6A model using curlcake m6A dataset
********************
Curlcake datasets are publicly available at the GEO database under the accession code `GSE124309 <https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE124309>`_. In this demo, subsets of the curcake datasets (m6A-modified and unmodified) were taken for demonstration purposes due to the large size of the original datasets. The demo datasets were located under ``./demo/curlcake/`` directory.
::
    demo
    └── curlcake
        ├── curlcake_m6A
        │   └── curlcake_m6A.fast5
        └── curlcake_unmod
            └── curlcake_unmod.fast5

**1. Guppy basecalling**

Basecalling converts the raw signal generated by Oxform Nanopore sequencing to DNA/RNA sequence. Guppy is used for basecalling in this step. In some nanopore datasets, the sequence information is already contained within the FAST5 files. In such cases, the basecalling step can be skipped as the sequence data is readily available.
::
    #m6A 
    guppy_basecaller -i demo/curlcake/curlcake_m6A -s demo/curlcake/curlcake_m6A_guppy --num_callers 40 --recursive --fast5_out --config rna_r9.4.1_70bps_hac.cfg
    
    #unmodified
    guppy_basecaller -i demo/curlcake/curlcake_unmod -s demo/curlcake/curlcake_unmod_guppy --num_callers 40 --recursive --fast5_out --config rna_r9.4.1_70bps_hac.cfg

**2. Multi-reads FAST5 files to single-read FAST5 files**

Convert multi-reads FAST5 files to single-read FAST5 files. If the data generated by the sequencing device is already in the single-read format, this step can be skipped.
::
    #m6A 
    multi_to_single_fast5 -i demo/curlcake/curlcake_m6A_guppy -s demo/curlcake/curlcake_m6A_guppy_single --recursive
    
    #unmodified
    multi_to_single_fast5 -i demo/curlcake/curlcake_unmod_guppy -s demo/curlcake/curlcake_unmod_guppy_single --recursive

**3. Tombo resquiggling**

In this step, the sequence obtained by basecalling is aligned or mapped to a reference genome or a known sequence. Then the corrected sequence is then associated with the corresponding current signals. The resquiggling process is typically performed in-place. No separate files are generated in this step. Curlcake reference file can be download `here <https://www.ncbi.nlm.nih.gov/geo/download/?acc=GSE124309&format=file&file=GSE124309%5FFASTA%5Fsequences%5Fof%5FCurlcakes%2Etxt%2Egz>`_. 
::
    #m6A
    tombo resquiggle --overwrite --basecall-group Basecall_1D_001 demo/curlcake/curlcake_m6A_guppy_single  demo/curlcake_reference.fa --processes 40 --fit-global-scale --include-event-stdev
    
    #unmodified
    tombo resquiggle --overwrite --basecall-group Basecall_1D_001 demo/curlcake/curlcake_unmod_guppy_single  demo/curlcake_reference.fa --processes 40 --fit-global-scale --include-event-stdev

**4. Map reads to reference**

minimap2 is used to map basecalled sequences to reference transcripts. The output sam file serves as the input for the subsequent feature extraction step. 
::
    #m6A
    cat demo/curlcake/curlcake_m6A_guppy/pass/*.fastq >demo/curlcake/curlcake_m6A.fastq
    minimap2 -ax map-ont demo/curlcake_reference.fa demo/curlcake/curlcake_m6A.fastq >demo/curlcake/curlcake_m6A.sam

    #unmodified
    cat demo/curlcake/curlcake_unmod_guppy/pass/*.fastq >demo/curlcake/curlcake_unmod.fastq
    minimap2 -ax map-ont demo/curlcake_reference.fa demo/curlcake/curlcake_unmod.fastq >demo/curlcake/curlcake_unmod.sam

**5. Feature extraction**

Extract signals and features from resquiggled fast5 files using the following python scripts.
::
    #m6A
    python scripts/extract_signal_from_fast5.py -p 40 --fast5 demo/curlcake/curlcake_m6A_guppy_single --reference demo/curlcake_reference.fa --sam demo/curlcake/curlcake_m6A.sam --output demo/curlcake/m6A.signal.tsv --clip=10
    python scripts/extract_feature_from_signal.py  --signal_file demo/curlcake/m6A.signal.tsv --clip 10 --output demo/curlcake/m6A.feature.tsv --motif DRACH
    
    #unmodified
    python scripts/extract_signal_from_fast5.py -p 40 --fast5 demo/curlcake/curlcake_unmod_guppy_single --reference demo/curlcake_reference.fa --sam demo/curlcake/curlcake_unmod.sam --output demo/curlcake/unmod.signal.tsv --clip=10
    python scripts/extract_feature_from_signal.py  --signal_file demo/curlcake/unmod.signal.tsv --clip 10 --output demo/curlcake/unmod.feature.tsv --motif DRACH

In the feature extraction step, the motif pattern should be provided using the argument ``--motif``. The base symbols of the motif follow the IUB code standard. 


**6. Train-test split**

The train-test split is performed randomly, ensuring that the data points in each set are representative of the overall dataset. The default split ratios are 80% for training and 20% for testing. The train-test split ratio can be customized by using the argument ``--train_ratio`` to accommodate the specific requirements of the problem and the size of the dataset.

The training set is used to train the model, allowing it to learn patterns and relationships present in the data. The testing set, on the other hand, is used to assess the model's performance on new, unseen data. It serves as an independent evaluation set to measure how well the trained model generalizes to data it has not encountered before. By evaluating the model on the testing set, we can estimate its performance, detect overfitting (when the model performs well on the training set but poorly on the testing set) and assess its ability to make accurate predictions on new data.
::
    usage: train_test_split.py [-h] [--input_file INPUT_FILE]
                               [--train_file TRAIN_FILE] [--test_file TEST_FILE]
                               [--train_ratio TRAIN_RATIO]
    
    Split a feature file into training and testing sets.
    
    optional arguments:
      -h, --help                  show this help message and exit
      --input_file INPUT_FILE     Path to the input feature file
      --train_file TRAIN_FILE     Path to the train feature file
      --test_file TEST_FILE       Path to the test feature file
      --train_ratio TRAIN_RATIO   Ratio of instances to use for training (default: 0.8)

    #m6A
    python scripts/train_test_split.py --input_file demo/curlcake/m6A.feature.tsv --train_file demo/curlcake/m6A.train.feature.tsv --test_file demo/curlcake/m6A.test.feature.tsv --train_ratio 0.8
    
    #unmodified
    python scripts/train_test_split.py --input_file demo/curlcake/unmod.feature.tsv --train_file demo/curlcake/unmod.train.feature.tsv --test_file demo/curlcake/unmod.test.feature.tsv --train_ratio 0.8


**7. Train m6A model**

To train the TandemMod model using your own dataset from scratch, you can set the ``--run_mode`` argument to "train". TandemMod accepts both modified and unmodified feature files as input. Additionally, test feature files are necessary to evaluate the model's performance. You can specify the model save path by using the argument ``--new_model``. The model's training epochs can be defined using the argument ``--epochs``, and the model states will be saved at the end of each epoch. TandemMod will preferentially use the ``GPU`` for training if CUDA is available on your device; otherwise, it will utilize the ``CPU`` mode. The training process duration can vary, depending on the size of your dataset and the computational capacity, and may last for several hours. 
::
    python scripts/TandemMod.py --run_mode train \
      --new_model demo/model/m6A.demo.curlcake.pkl \
      --train_data_mod demo/curlcake/m6A.train.feature.tsv \
      --train_data_unmod demo/curlcake/unmod.train.feature.tsv \
      --test_data_mod demo/curlcake/m6A.test.feature.tsv \
      --test_data_unmod demo/curlcake/unmod.test.feature.tsv \
      --epoch 100

During training process, the following information can be used to monitor and evaluate the performance of the model:
::
    device= cpu
    train process.
    data loaded.
    start training...
    Epoch 0-0 Train acc: 0.482000,Test Acc: 0.788462,time0:00:07.666192
    Epoch 1-0 Train acc: 0.514000,Test Acc: 0.211538,time0:00:04.977504
    Epoch 2-0 Train acc: 0.496000,Test Acc: 0.211538,time0:00:05.498799
    Epoch 3-0 Train acc: 0.694000,Test Acc: 0.432692,time0:00:05.893204
    Epoch 4-0 Train acc: 0.814000,Test Acc: 0.639423,time0:00:06.149194
    Epoch 5-0 Train acc: 0.806000,Test Acc: 0.711538,time0:00:05.443221
    Epoch 6-0 Train acc: 0.828000,Test Acc: 0.831731,time0:00:05.706294
    Epoch 7-0 Train acc: 0.808000,Test Acc: 0.846154,time0:00:05.674450
    Epoch 8-0 Train acc: 0.804000,Test Acc: 0.822115,time0:00:05.956936


After the data processing and model training, the following files should be generated by TandemMod. The trained model ``m6A.demo.curlcake.pkl`` will be saved in the ``./demo/model/`` folder. You can utilize this model for making predictions in the future.
::
    demo
    ├── curlcake
    │   ├── curlcake_m6A
    │   ├── curlcake_m6A.fastq
    │   ├── curlcake_m6A_guppy
    │   ├── curlcake_m6A_guppy_single
    │   ├── curlcake_m6A.sam
    │   ├── curlcake_unmod
    │   ├── curlcake_unmod.fastq
    │   ├── curlcake_unmod_guppy
    │   ├── curlcake_unmod_guppy_single
    │   ├── curlcake_unmod.sam
    │   ├── m6A.feature.tsv
    │   ├── m6A.signal.tsv
    │   ├── m6A.test.feature.tsv
    │   ├── m6A.train.feature.tsv
    │   ├── unmod.feature.tsv
    │   ├── unmod.signal.tsv
    │   ├── unmod.test.feature.tsv
    │   └── unmod.train.feature.tsv
    ├── curlcake_reference.fa
    └── model
           └── m6A.demo.curlcake.pkl


Transfer m6A model to m7G using ELIGOS dataset
********************

To transfer the pretrained m6A model to an m7G prediction model using the ELIGOS dataset, you can follow these steps:

* Obtain the ELIGOS dataset: Download or access the ELIGOS m7G dataset, which consists of the necessary data (m7G-modified and unmodified) for training and testing.

* Prepare the data: Preprocess the ELIGOS dataset to extact features for transfer learning.

* Load the pretrained m6A model: Load the pretrained m6A model that you want to transfer to predict m7G modifications. This model should have been previously trained on a relevant m6A dataset.

* Train the modified model: Use the ELIGOS m7G dataset to fine-tune the model's parameters using transfer learning techniques.

* Evaluate the performance: Assess the performance of the transferred m7G model on the m7G testing set from the ELIGOS dataset.

By following these steps, you can transfer the knowledge gained from the pretrained m6A model to predict m7G modifications using the ELIGOS dataset.

ELIGOS datasets are publicly available at the SRA database under the accession code `SRP166020 <https://www.ncbi.nlm.nih.gov/sra/?term=SRP166020>`_. In this demo, subsets of the ELIGOS datasets (m7G-modified and unmodified) were taken for demonstration purposes due to the large size of the original datasets. The demo datasets were located under ``./demo/ELIGOS/`` directory.
::
    demo
    └── ELIGOS
        ├── ELIGOS_m7G
        │   └── ELIGOS_m7G.fast5
        └── ELIGOS_unmod
            └── ELIGOS_unmod.fast5

**1. Guppy basecalling**

Basecalling converts the raw signal generated by Oxform Nanopore sequencing to DNA/RNA sequence. Guppy is used for basecalling in this step. In some nanopore datasets, the sequence information is already contained within the FAST5 files. In such cases, the basecalling step can be skipped as the sequence data is readily available.
::
    #m7G 
    guppy_basecaller -i demo/ELIGOS/ELIGOS_m7G -s demo/ELIGOS/ELIGOS_m7G_guppy --num_callers 40 --recursive --fast5_out --config rna_r9.4.1_70bps_hac.cfg
    
    #unmodified
    guppy_basecaller -i demo/ELIGOS/ELIGOS_unmod -s demo/ELIGOS/ELIGOS_unmod_guppy --num_callers 40 --recursive --fast5_out --config rna_r9.4.1_70bps_hac.cfg

**2. Multi-reads FAST5 files to single-read FAST5 files**

Convert multi-reads FAST5 files to single-read FAST5 files. If the data generated by the sequencing device is already in the single-read format, this step can be skipped.
::
    #m7G 
    multi_to_single_fast5 -i demo/ELIGOS/ELIGOS_m7G_guppy -s demo/ELIGOS/ELIGOS_m7G_guppy_single --recursive
    
    #unmodified
    multi_to_single_fast5 -i demo/ELIGOS/ELIGOS_unmod_guppy -s demo/ELIGOS/ELIGOS_unmod_guppy_single --recursive

**3. Tombo resquiggling**

In this step, the sequence obtained by basecalling is aligned or mapped to a reference genome or a known sequence. Then the corrected sequence is then associated with the corresponding current signals. The resquiggling process is typically performed in-place. No separate files are generated in this step. ELIGOS reference file can be download `here <https://oup.silverchair-cdn.com/oup/backfile/Content_public/Journal/nar/49/2/10.1093_nar_gkaa620/1/gkaa620_supplemental_files.zip?Expires=1690555116&Signature=Mv7ppemTnplIZAvv6G3W-lob1eQwK5IvNeIIF-1GM8Jy93AdT6ALUynRjW3HQAyCMgkMW-0WnXktuVJfKDCUXiiwvjZ9z5iO5LksCl1e6yEA5dgRlr-FVUrDbj81NIfUJNhKReo5gxRYc~f7wbFZRcy9CcSB-D1DloUmv-4qdcydr35sM-YDKgfyNfaE-ZKnCZZ1KydDNtx7oRfYHCof-a3oHSNgxn5DFM9bGCq147cw6i9B1bCURAPLltdPzR4i7cBXmIRoNZuVkjLe8EktJPg47v9ElqlPUlZfAqoaESbmPtEs8NLoX~~82o~eMrjwomK4W5CzgwAZhJJIeelr7A__&Key-Pair-Id=APKAIE5G5CRDK6RD3PGA>`_. 
::
    #m7G
    tombo resquiggle --overwrite --basecall-group Basecall_1D_001 demo/ELIGOS/ELIGOS_m7G_guppy_single  demo/ELIGOS_reference.fa --processes 40 --fit-global-scale --include-event-stdev
    
    #unmodified
    tombo resquiggle --overwrite --basecall-group Basecall_1D_001 demo/ELIGOS/ELIGOS_unmod_guppy_single  demo/ELIGOS_reference.fa --processes 40 --fit-global-scale --include-event-stdev

**4. Map reads to reference**

minimap2 is used to map basecalled sequences to reference transcripts. The output sam file serves as the input for the subsequent feature extraction step. 
::
    #m7G
    cat demo/ELIGOS/ELIGOS_m7G_guppy/pass/*.fastq >demo/ELIGOS/ELIGOS_m7G.fastq
    minimap2 -ax map-ont demo/ELIGOS_reference.fa demo/ELIGOS/ELIGOS_m7G.fastq >demo/ELIGOS/ELIGOS_m7G.sam

    #unmodified
    cat demo/ELIGOS/ELIGOS_unmod_guppy/pass/*.fastq >demo/ELIGOS/ELIGOS_unmod.fastq
    minimap2 -ax map-ont demo/ELIGOS_reference.fa demo/ELIGOS/ELIGOS_unmod.fastq >demo/ELIGOS/ELIGOS_unmod.sam

**5. Feature extraction**

Extract signals and features from resquiggled fast5 files using the following python scripts.
::
    #m7G
    python scripts/extract_signal_from_fast5.py -p 40 --fast5 demo/ELIGOS/ELIGOS_m7G_guppy_single --reference demo/ELIGOS_reference.fa --sam demo/ELIGOS/ELIGOS_m7G.sam --output demo/ELIGOS/m7G.signal.tsv --clip=10
    python scripts/extract_feature_from_signal.py  --signal_file demo/ELIGOS/m7G.signal.tsv --clip 10 --output demo/ELIGOS/m7G.feature.tsv --motif NNGNN
    
    #unmodified
    python scripts/extract_signal_from_fast5.py -p 40 --fast5 demo/ELIGOS/ELIGOS_unmod_guppy_single --reference demo/ELIGOS_reference.fa --sam demo/ELIGOS/ELIGOS_unmod.sam --output demo/ELIGOS/unmod.signal.tsv --clip=10
    python scripts/extract_feature_from_signal.py  --signal_file demo/ELIGOS/unmod.signal.tsv --clip 10 --output demo/ELIGOS/unmod.feature.tsv --motif NNGNN

In the feature extraction step, the motif pattern should be provided using the argument ``--motif``. The base symbols of the motif follow the IUB code standard. 


**6. Train-test split**

The train-test split is performed randomly, ensuring that the data points in each set are representative of the overall dataset. The default split ratios are 80% for training and 20% for testing. The train-test split ratio can be customized by using the argument ``--train_ratio`` to accommodate the specific requirements of the problem and the size of the dataset.

The training set is used to train the model, allowing it to learn patterns and relationships present in the data. The testing set, on the other hand, is used to assess the model's performance on new, unseen data. It serves as an independent evaluation set to measure how well the trained model generalizes to data it has not encountered before. By evaluating the model on the testing set, we can estimate its performance, detect overfitting (when the model performs well on the training set but poorly on the testing set) and assess its ability to make accurate predictions on new data.
::
    usage: train_test_split.py [-h] [--input_file INPUT_FILE]
                               [--train_file TRAIN_FILE] [--test_file TEST_FILE]
                               [--train_ratio TRAIN_RATIO]
    
    Split a feature file into training and testing sets.
    
    optional arguments:
      -h, --help                  show this help message and exit
      --input_file INPUT_FILE     Path to the input feature file
      --train_file TRAIN_FILE     Path to the train feature file
      --test_file TEST_FILE       Path to the test feature file
      --train_ratio TRAIN_RATIO   Ratio of instances to use for training (default: 0.8)

    #m7G
    python scripts/train_test_split.py --input_file demo/ELIGOS/m7G.feature.tsv --train_file demo/ELIGOS/m7G.train.feature.tsv --test_file demo/ELIGOS/m7G.test.feature.tsv --train_ratio 0.8
    
    #unmodified
    python scripts/train_test_split.py --input_file demo/ELIGOS/unmod.feature.tsv --train_file demo/ELIGOS/unmod.train.feature.tsv --test_file demo/ELIGOS/unmod.test.feature.tsv --train_ratio 0.8


**7. Train m7G model**

To transfer the pretrained TandemMod model to new types of modifications, you can set the ``--run_mode`` argument to "transfer". TandemMod accepts both modified and unmodified feature files as input. Additionally, test feature files are necessary to evaluate the model's performance. You can specify the pretrained model by using the argument ``--pretrained_model`` and the new model save path by using the argument ``--new_model``. The model's training epochs can be defined using the argument ``--epochs``, and the model states will be saved at the end of each epoch. TandemMod will preferentially use the ``GPU`` for training if CUDA is available on your device; otherwise, it will utilize the ``CPU`` mode. The training process duration can vary, depending on the size of your dataset and the computational capacity, and may last for several hours. 
::
    usage: TandemMod.py [-h] --run_mode RUN_MODE
                        [--pretrained_model PRETRAINED_MODEL]
                        [--new_model NEW_MODEL] [--train_data_mod TRAIN_DATA_MOD]
                        [--train_data_unmod TRAIN_DATA_UNMOD]
                        [--test_data_mod TEST_DATA_MOD]
                        [--test_data_unmod TEST_DATA_UNMOD]
                        [--feature_file FEATURE_FILE]
                        [--predict_result PREDICT_RESULT] [--epoch EPOCH]
    
    TandemMod, multiple types of RNA modification detection.
    
    optional arguments:
      -h, --help                               show this help message and exit
      --run_mode RUN_MODE                      Run mode. Default is train
      --pretrained_model PRETRAINED_MODEL      Pretrained model file.
      --new_model NEW_MODEL                    New model file to be saved.
      --train_data_mod TRAIN_DATA_MOD          Train data file, modified.
      --train_data_unmod TRAIN_DATA_UNMOD      Train data file, unmodified.
      --test_data_mod TEST_DATA_MOD            Test data file, modified.
      --test_data_unmod TEST_DATA_UNMOD        Test data file, unmodified.
      --epoch EPOCH                            Training epoch

    python scripts/TandemMod.py --run_mode transfer \
      --pretrained_model demo/model/m6A.demo.IVET.pkl \
      --new_model demo/model/m7G.demo.ELIGOS.transfered_from_IVET_m6A.pkl \
      --train_data_mod demo/ELIGOS/m7G.train.feature.tsv \
      --train_data_unmod demo/ELIGOS/unmod.train.feature.tsv \
      --test_data_mod demo/ELIGOS/m7G.test.feature.tsv \
      --test_data_unmod demo/ELIGOS/unmod.test.feature.tsv \
      --epoch 100

During training process, the following information can be used to monitor and evaluate the performance of the transfered model:
::
    device= cpu
    transfer learning process.
    data loaded.
    start training...
    Epoch 0-0 Train acc: 0.544000,Test Acc: 0.489786,time0:00:08.688707
    Epoch 1-0 Train acc: 0.674000,Test Acc: 0.857939,time0:00:05.190997
    Epoch 2-0 Train acc: 0.748000,Test Acc: 0.813835,time0:00:05.426035
    Epoch 3-0 Train acc: 0.778000,Test Acc: 0.753946,time0:00:05.180632
    Epoch 4-0 Train acc: 0.854000,Test Acc: 0.776230,time0:00:05.236281
    Epoch 5-0 Train acc: 0.886000,Test Acc: 0.817549,time0:00:05.219122
    Epoch 6-0 Train acc: 0.926000,Test Acc: 0.889044,time0:00:05.470729



After the data processing and model training, the following files should be generated by TandemMod. The trained model ``m7G.demo.ELIGOS.transfered_from_IVET_m6A.pkl`` will be saved in the ``./demo/model/`` folder. You can utilize this fine-tuned model for making predictions in the future.
::
    demo
    ├── ELIGOS
    │   ├── ELIGOS_m7G
    │   ├── ELIGOS_m7G.fastq
    │   ├── ELIGOS_m7G_guppy
    │   ├── ELIGOS_m7G_guppy_single
    │   ├── ELIGOS_m7G.sam
    │   ├── ELIGOS_unmod
    │   ├── ELIGOS_unmod.fastq
    │   ├── ELIGOS_unmod_guppy
    │   ├── ELIGOS_unmod_guppy_single
    │   ├── ELIGOS_unmod.sam
    │   ├── m7G.feature.tsv
    │   ├── m7G.signal.tsv
    │   ├── m7G.test.feature.tsv
    │   ├── m7G.train.feature.tsv
    │   ├── unmod.feature.tsv
    │   ├── unmod.signal.tsv
    │   ├── unmod.test.feature.tsv
    │   └── unmod.train.feature.tsv
    ├── ELIGOS_reference.fa
    └── model
           ├── m6A.demo.IVET.pkl
           └── m7G.demo.ELIGOS.transfered_from_IVET_m6A.pkl



Predict m6A sites in human cell lines
********************

HEK293T nanopore data is publicly available and can be downloaded from the `SG-NEx project <https://groups.google.com/g/sg-nex-updates>`_. In this demo, subset of the HEK293T nanopore data was taken for demonstration purposes due to the large size of the original datasets. The demo datasets were located under ``./demo/HEK293T/`` directory.
::
    demo
    └── HEK293T
        └── HEK293T_fast5
            └── HEK293T.fast5

**1. Guppy basecalling**

Basecalling converts the raw signal generated by Oxform Nanopore sequencing to DNA/RNA sequence. Guppy is used for basecalling in this step. In some nanopore datasets, the sequence information is already contained within the FAST5 files. In such cases, the basecalling step can be skipped as the sequence data is readily available.
::
    guppy_basecaller -i demo/HEK293T/HEK293T_fast5 -s demo/HEK293T/HEK293T_fast5_guppy --num_callers 40 --recursive --fast5_out --config rna_r9.4.1_70bps_hac.cfg
    

**2. Multi-reads FAST5 files to single-read FAST5 files**

Convert multi-reads FAST5 files to single-read FAST5 files. If the data generated by the sequencing device is already in the single-read format, this step can be skipped.
::
    multi_to_single_fast5 -i demo/HEK293T/HEK293T_fast5_guppy -s demo/HEK293T/HEK293T_fast5_guppy_single --recursive


**3. Tombo resquiggling**

In this step, the sequence obtained by basecalling is aligned or mapped to a reference genome or a known sequence. Then the corrected sequence is then associated with the corresponding current signals. The resquiggling process is typically performed in-plac. No separate files are generated in this step. GRCh38 transcripts file can be download `here <https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000001405.40/>`_. 
::
    tombo resquiggle --overwrite --basecall-group Basecall_1D_001 demo/HEK293T/HEK293T_fast5_guppy_single  demo/GRCh38_subset_reference.fa --processes 40 --fit-global-scale --include-event-stdev


**4. Map reads to reference**

minimap2 is used to map basecalled sequences to reference transcripts. The output sam file serves as the input for the subsequent feature extraction step. 
::
    cat demo/HEK293T/HEK293T_fast5_guppy/pass/*.fastq >demo/HEK293T/HEK293T.fastq
    minimap2 -ax map-ont demo/GRCh38_subset_reference.fa demo/HEK293T/HEK293T.fastq >demo/HEK293T/HEK293T.sam


**5. Feature extraction**

Extract signals and features from resquiggled fast5 files using the following python scripts.
::
    python scripts/extract_signal_from_fast5.py -p 40 --fast5 demo/HEK293T/HEK293T_fast5_guppy_single --reference demo/GRCh38_subset_reference.fa --sam demo/HEK293T/HEK293T.sam --output demo/HEK293T/HEK293T.signal.tsv --clip=10
    python scripts/extract_feature_from_signal.py  --signal_file demo/HEK293T/HEK293T.signal.tsv --clip 10 --output demo/HEK293T/HEK293T.feature.tsv --motif DRACH
    


In the feature extraction step, the motif pattern should be provided using the argument ``--motif``. The base symbols of the motif follow the IUB code standard. 


**7. Predict m6A sites**

To predict m6A sites in HEK293T nanopore data using a pretrained model, you can set the ``--run_mode`` argument to "predict".  You can specify the pretrained model by using the argument ``--pretrained_model``. 
::
    python scripts/TandemMod.py --run_mode predict \
          --pretrained_model demo/model/m6A.demo.IVET.pkl \
          --feature_file demo/HEK293T/HEK293T.feature.tsv \
          --predict_result demo/HEK293T/HEK293T.prediction.tsv


During the prediction process, TandemMod generates the following files. The prediction result file is named "HEK293T.prediction.tsv". 
::
    demo
    └── HEK293T
        ├── HEK293T_fast5
        ├── HEK293T_fast5_guppy
        ├── HEK293T_fast5_guppy_single
        ├── HEK293T.fastq
        ├── HEK293T.feature.tsv
        ├── HEK293T.prediction.tsv
        ├── HEK293T.sam
        └── HEK293T.signal.tsv

The prediction result "demo/HEK293T/HEK293T.prediction.tsv" provides prediction labels along with the corresponding modification probabilities, which can be utilized for further analysis.
::
    transcript_id   site    motif   read_id                                 prediction   probability
    XM_005261965.4  10156   AAACA   60e0f6a3-2166-4730-9a10-8f8aaa750b37    unmod        0.1364245
    XM_005261965.4  10164   AAACT   60e0f6a3-2166-4730-9a10-8f8aaa750b37    unmod        0.034915127
    XM_005261965.4  10229   GAACC   60e0f6a3-2166-4730-9a10-8f8aaa750b37    unmod        0.4773725
    XM_005261965.4  10241   GGACC   60e0f6a3-2166-4730-9a10-8f8aaa750b37    unmod        0.11096856
    XM_005261965.4  10324   GGACT   60e0f6a3-2166-4730-9a10-8f8aaa750b37    mod          0.908553
    XM_005261965.4  10362   AAACA   60e0f6a3-2166-4730-9a10-8f8aaa750b37    unmod        0.2004475
    XM_005261965.4  10434   AGACA   60e0f6a3-2166-4730-9a10-8f8aaa750b37    unmod        0.1934688
    XM_005261965.4  10498   GGACC   60e0f6a3-2166-4730-9a10-8f8aaa750b37    unmod        0.1313666
    XM_005261965.4  10507   AAACA   60e0f6a3-2166-4730-9a10-8f8aaa750b37    unmod        0.030169742
    XM_005261965.4  10511   AAACT   60e0f6a3-2166-4730-9a10-8f8aaa750b37    unmod        0.020174831
    XM_005261965.4  10592   AGACT   60e0f6a3-2166-4730-9a10-8f8aaa750b37    mod          0.7666112

The execution time for each demonstration is estimated to be approximately 0-3 minutes.