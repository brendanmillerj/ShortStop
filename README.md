![[Pasted image 20240202153118.png]]
## 1. What is ShortStop?
ShortStop, written in Python, is a computational tool that compares putative microproteins to randomly generated counterparts. 

- **Core Problem Addressed**: 
	- By generating random/decoy microproteins matched to a putative set, ShortStop offers a solution to the void of robust null hypotheses in the microprotein field. 
- **Functionality**:
    - Random/decoy microprotein generation
	    - Generates random/decoy sequences from user-inputted putative microprotein sequences (e.g., from Ribo-Seq). This establishes a null hypothesis.
	- Predict putative microproteins
		- ShortStop offers a prediction algorithm to assign microproteins as random, intracellular, or secreted — the "ShortStop standard prediction model."
    - Custom training platform based on user-specific hypotheses 
	    - Allows users to input experimentally validated microproteins/short proteins (e.g., annotated by Gene Ontology) to establish the alternative hypothesis.
	    - Inputs gene feature information in a neural network to classify sequences as null/random or alternative/functional.
	    - 
- **Benefits for Biologists**:
    - Filters Ribo-Seq smORFs by answering the question, "Which smORFs look the least like a random gene?"
    - Refines search space for proteogenomics analyses
    - Is a medium for computational experiments by training custom models (e.g., which smORFs look like a certain class of short proteins?)
    
## 2. What can users do with Shortstop?
- **Random/Decoy Generation**: Creates decoy microproteins from user-inputted putative sequences (e.g., smORFs identified via Ribo-Seq). The user can then use this list outside the ShortStop predictive scaffold (e.g., generate custom features for custom training)
- **Feature Extraction**: Automatically extracts default gene features from random/decoy, putative, and experimentally validated microproteins for training. These features include 4-mer composition of upstream, CDS, and downstream nucleotides, as well as the protein features CTD, CKSAAP, APAAC, and QSO.
- - **Offers the `**Standard ShortStop Predictive Model**`**:  for assigning class probabilities to putative microproteins. This model is based on deciphering putative microproteins from random ones generated from the GENCODE smORF Phase 1 catalog. 
- **Neural Network Training**: Uses extracted features to train a neural network for default class prediction (i.e., Intracellular, Secreted, Random/Decoy).
- **User Customization**: Allows for the input of custom features and classes beyond the default options for tailored model creation. 
## 3. Installation
Typical installation time will vary depending on how many dependencies requirements are already met. Installation of all dependencies should take no more than 15 minutes.

1. Download the latest version of the pipeline from [insert link]
2. ``cd`` to the directory containing the ShortStop.tar.gz file and run ``tar -xzvf ShortStop.tar.gz`` in your terminal terminal.
3. Install required Python packages
	1. ``$ cd`` into the ``ShortStop`` directory
	2. Run ``$ pip install -r requirements.txt
#### Testing the installation through demo mode
We provide a *demo* mode for the user to check if the installation is working. The necessary data sets to run *demo* mode are in the subdirectory called ``demo_data``. *Demo* mode will take approximately 2-5 minutes to run. 

*Demo* mode will check the 2 main modes: ``training`` and ``predict``. To run, enter the following:

1. Download require reference genomes. This will take up about 4GB of space. 
	1. ``cd`` into the ``demo_data`` directory
	2. Run ``./download_refgenomes.sh`` 
	3. Run `python ShortStop.py demo`

After initiating *demo*, it will perform all the steps necessary to train a predictive model, except it will terminate early during label propagation and tuning to save run time. 

Lastly, *demo* will predict a short dataset of putative microproteins according to the `**Standard ShortStop Predictive Model**.`After this step, which completes *demo* mode, the output directory will be deleted to free up memory. The default output directory is ``shortstop_output``. 

A successful *demo* run will show the following message:
``✅ Demo has been successfuly completed! You are ready to use ShortStop.``
## 4. Train Model
The following will train a prediction model using the same input data that we used to train the `**Standard Predictive Model**.
`
**Note**: This time it takes to complete train mode will vary per computer. Expect the mode to last between 10-30 minutes. With a standard Apple M1 Pro chip, the time took approximately 15 minutes.

1. Enter ``$ ShortStop.py training -h`` at the terminal to print this message:
```Training mode options:
  --positive_gtf POSITIVE_GTF
                        Provide the reference gtf file (default: demo_datasets/gencode.v43.primary_assembly.basic.annotation.gtf)
  --positive_ids POSITIVE_IDS
                        Provide the positive data frame (default: demo_datasets/uniprot_entry_ensembl_ids.csv)
  --positive_functions POSITIVE_FUNCTIONS
                        Provide the positive functional protein assignment data frame, such as cellular localization (default: demo_datasets/shortstop_training_cc.csv)
  --genome GENOME       genome fasta file (default: demo_datasets/hg_38_primary.fa)
  --putative_smorfs_gtf PUTATIVE_SMORFS_GTF
                        Provide the to_be_predicted gtf file (default: demo_datasets/gencode_smorfs.gtf)
  --utr_length UTR_LENGTH
                        Provide the length of UTRs. Note the memory substantially increases beyond 25 bases (default: 25)
  --n_random_smORFs N_RANDOM_SMORFS
                        Provide the number of random smORFs you want to generate. Consider class balancing. (default: 2500)
  --kmer KMER           Provide the kmer size for nucleotide data processing. (default: 4)
  --n_neighbors N_NEIGHBORS
                        Number of neighbors for UMAP (default: 50)
  --gamma GAMMA         Gamma initialization for label spreading. Increase the number for greater strictness. Deafult set at 1. (default: 1)
  --normalization NORMALIZATION
                        If considering protein length normalziation, options include inverse, sqrt, and log transformation of feature values. (default: none)
```


2. Enter ``$ ShortStop.py training `` to initiate training mode.
	**Note**: You *do not* have to input any of the arguments showing by ``-h`` . The default arguments is sufficient to initiate training and will pull from the `demo_data` directory for training. Therefore, please make sure you do not delete `demo_data`. 
	
3. Follow progress updates throughout the training process. Observe the descriptive statistics, notably the average microprotein length of the putative smORFs along with the standard deviation, in the following example.
```
▶️You have successfully initiated training.
✅ GTF for positive ORFs completed.
🔔: Don't forget double check the CDS order of your GTF file. This is important for concatenating the CDS sequences, specifically on the reverse strand. Failure to do so could result in incorrect CDS sequences.
✅ Amino acids and DNA for each ORF extracted.
⏳Generating random decoy sequences based on your putative smORFs...
     Here is the average length and standard deviation of your smORFs that the random/decoy generator is using:
         -- Mean length of amino acid sequence:  56.359274289806635
         -- 1.25 Standard deviations of length of amino acid sequence:  30.934958617229217
(9535, 16)
```

4. Observe the label propagation results. 
	**Note:** Label propagation may not be necessary nor recommended. Label propagation here involves the utilization of UMAP. The primary motivation of UMAP is visual representation of putative microproteins in reduced space. In some cases, if you are using your own data, you may not want to label propagate to assign labels to putative smORFs, given that visual patterns in reduced space do not appear distinct, and thus it would be unclear whether you can further maximize information given your putative smORFs in this shrunken space. In this case, it is strongly suggested to simply enter the following command to bypass updating the labels of putative microproteins you are using to generate random/decoys:
	`` python ShortStop.py training --keep 
	
```✅ ORF feature extraction completed.
⏳Initiating label propagation to give your putative smORF a label for training...
UMAP took 73.39 seconds
Final gamma value:  50
```

5. Observe the neural network training results. 
```
⏳Starting neural network training...
Number of DNA features: 921
Number of AA features: 719
Starting hyperparameter tuning...
Best score: 0.8986437916755676
Best params: {'batch_size': 32, 'dropout_rate': 0.2, 'epochs': 24, 'learning_rate': 0.0001, 'nodes_layer_one': 512, 'nodes_layer_two': 32, 'validation_split': 0.2}
58/58 [==============================] - 0s 995us/step - loss: 0.2834 - accuracy: 0.8965
Test accuracy: 0.8964950442314148
58/58 [==============================] - 0s 1000us/step
Confusion Matrix:
 [[1348   61    3]
 [  76  233   14]
 [  11   24   56]]

Precision: [0.93937282 0.7327044  0.76712329]
Recall: [0.95467422 0.72136223 0.61538462]
F1 Score: [0.94696171 0.72698908 0.68292683] 

✅ Neural network training completed.
```

6. At the conclusion of ``training`` mode, the following directories will be built and stored with the sequentially-generated output files:
```
shortstop_output/
├── features/
	   └── extracted_features_of_smorfs.csv
	   └── label_spreading_confusion_matrix.csv
	   └── label_spreading_umap_data.csv	
├── models/
	   └── aa_features.txt
	   └── dna_features.txt
	   └── best_model.h5
	   └── scaler.save
	   └── model_accuracy_loss.csv
	   └── orfs_features_in_training_model.csv
   ├── plots/
	   └── database_sequence_length_distribution.png
	   └── model_accuracy.png
	   └── model_architecture.png
	   └── umap_3d_scatter_reduced_features_propagated.html
	   └── umap_3d_scatter_reduced_features.html
   ├── predictions/
   ├── sequences/
	   └── positive_proteins.gtf
	   └── positive_unknown_random_sequences.csv
	   └── positives_unknown_sequences.csv
```

## 5. Predict Putative Microproteins
The following will use the `**ShortStop Standard Predictive Model**` to assign one of three labels to putative microproteins: 1) Random/decoy, 2) Intracellular, or 3) Secreted. 

The only file type that a user must provide outside the ShortStop directory is a GTF file containing transcripts and CDS for each putative smORF. 

We will use chr1 smORFs provided by GENCODE smORF Phase 1 catalog to test ``predict`` mode. 

1. Enter ``$ ShortStop.py predict -h`` at the terminal to print this message:
Predict mode options:
  ```
Predict mode options:

  --genome GENOME       genome fasta file (default: demo_data/hg_38_primary.fa)

  --putative_smorfs_gtf PUTATIVE_SMORFS_GTF

                        Provide the to_be_predicted gtf file (default: demo_data/chr1_smorfs.gtf)

  --utr_length UTR_LENGTH

                        Provide the length of UTRs. Note the memory substantially increases beyond 25 bases (default: 25)

  --kmer KMER           Provide the kmer size for nucleotide data processing. (default: 4)

  --normalization NORMALIZATION

                        If considering protein length normalziation, options include inverse, sqrt, and log transformation of feature values. (default: none)

  --orfs_features_in_training_model ORFS_FEATURES_IN_TRAINING_MODEL

                        Feature file of the orfs that were used to train the model. The default naming convention is: orfs_features.csv. This will be output from the training mode. (default:

                        standard_prediction_model/orfs_features_in_training_model.csv)

  --orfs_to_be_predicted ORFS_TO_BE_PREDICTED

                        Feature file of the orfs that will be predicted. The default naming convention is: orfs_to_be_predicted.csv (default: shortstop_output/features/extracted_features_of_smorfs.csv)

  --model_scaler MODEL_SCALER

                        Scaler file of the model. The default naming convention is: scaler_params.csv. This will be output from the training mode. (default: standard_prediction_model/scaler.save)

  --model MODEL         Model file. The default naming convention is: best_model.h5. This will be output from the training mode. (default: standard_prediction_model/best_model.h5)
```
2. Enter ``$ python ShortStop.py predict`` to initiate predict mode.
	**Note**:  The default arguments is sufficient to initiate prediction and will pull from the `demo_data` directory for training. Therefore, please make sure you do not delete `demo_data`. 
	
	 In sequential order, the chr1_smorfs.gtf file provided in ``demo_data`` will undergo sequence conversion, followed by feature extraction, which will be thrown into the `**ShortStop Standard Predictive Model**`. The progress and completion will look like this example:
```
⏳Throwing features into the prediction algorithm...
Confirmed: Your data is appropriately scaled based on the original model's training platform.
Number of DNA features being used to predict: 912
Number of AA features being used to predict: 719
298/298 [==============================] - 0s 878us/step
✅ Predictions out and completed!
```
3. The following prediction files will be generated:
	**Note**:  The ``all_class_percentages.csv`` file contains all tested genes. The ``random_vs_decoy_percentages.csv`` contains the sum of the intracellular and secreted prediction percentage.  
```
shortstop_output/
   ├── predictions/
	   └── secreted_smorfs.csv
	   └── all_class_percentages.csv
	   └── decoy_percentages_smorfs.csv
	   └── intracellular_smorfs.csv 
	   └── random_vs_decoy_percentages.csv
```
