# Predicting Molecular Properties: Overview
Kaggle Competition Prediction Molecular Properties https://www.kaggle.com/c/champs-scalar-coupling

In this competition, you will be predicting the scalar_coupling_constant between atom pairs in molecules, given the two atom types (e.g., C and H), the coupling type (e.g., 2JHC), and any features you are able to create from the molecule structure (xyz) files.

For this competition, you will not be predicting all the atom pairs in each molecule rather, you will only need to predict the pairs that are explicitly listed in the train and test files. For example, some molecules contain Fluorine (F), but you will not be predicting the scalar coupling constant for any pair that includes F.

The training and test splits are by molecule, so that no molecule in the training data is found in the test data.

# Data Description
* train.csv - the training set, where the first column (molecule_name) is the name of the molecule where the coupling constant originates (the corresponding XYZ file is located at ./structures/.xyz), the second (atom_index_0) and third column (atom_index_1) is the atom indices of the atom-pair creating the coupling and the fourth column (scalar_coupling_constant) is the scalar coupling constant that we want to be able to predict
* test.csv - the test set; same info as train, without the target variable
* sample_submission.csv - a sample submission file in the correct format
* structures.zip - folder containing molecular structure (xyz) files, where the first line is the number of atoms in the molecule, followed by a blank line, and then a line for every atom, where the first column contains the atomic element (H for hydrogen, C for carbon etc.) and the remaining columns contain the X, Y and Z cartesian coordinates (a standard format for chemists and molecular visualization programs)
* structures.csv - this file contains the same information as the individual xyz structure files, but in a single file

# Additional Data
NOTE: additional data is provided for the molecules in Train only!

* dipole_moments.csv - contains the molecular electric dipole moments. These are three dimensional vectors that indicate the charge distribution in the molecule. The first column (molecule_name) are the names of the molecule, the second to fourth column are the X, Y and Z components respectively of the dipole moment.
* magnetic_shielding_tensors.csv - contains the magnetic shielding tensors for all atoms in the molecules. The first column (molecule_name) contains the molecule name, the second column (atom_index) contains the index of the atom in the molecule, the third to eleventh columns contain the XX, YX, ZX, XY, YY, ZY, XZ, YZ and ZZ elements of the tensor/matrix respectively.
* mulliken_charges.csv - contains the mulliken charges for all atoms in the molecules. The first column (molecule_name) contains the name of the molecule, the second column (atom_index) contains the index of the atom in the molecule, the third column (mulliken_charge) contains the mulliken charge of the atom.
* potential_energy.csv - contains the potential energy of the molecules. The first column (molecule_name) contains the name of the molecule, the second column (potential_energy) contains the potential energy of the molecule.
* scalar_coupling_contributions.csv - The scalar coupling constants in train.csv (or corresponding files) are a sum of four terms. scalar_coupling_contributions.csv contain all these terms. The first column (molecule_name) are the name of the molecule, the second (atom_index_0) and third column (atom_index_1) are the atom indices of the atom-pair, the fourth column indicates the type of coupling, the fifth column (fc) is the Fermi Contact contribution, the sixth column (sd) is the Spin-dipolar contribution, the seventh column (pso) is the Paramagnetic spin-orbit contribution and the eighth column (dso) is the Diamagnetic spin-orbit contribution.

# Code
Codes in lgb folder contains the work done during the competition.
* Feature_Explore_Data_Gen: Couping type specific feature engineering and also outputs the additional datasets that are used in the later modeling.
* Mole_Model: General molecular level feature engineering and outputs datasets that are used in the later modeling.
* Mole_Type_Model: Merges generated datasets from Feature_Explore_Data_Gen and Mole_Model. Creates lgb models for different types of coupling.

I also will also try reproducing top solutions with sequence models. Code will be uploaded in the future. Special thanks to [Andres Torrubia](https://www.kaggle.com/antorsae).

# Feature Engineering
Thanks to the competition host, as far as I can remember, we are provided with around one million rows of data. And the data is fairly clean. Therefore during the competition, I didn't do much data cleaning. Since I had no experience with neural network especially sequence models during that time, I decided to focus on feature engineering.

The inspiration of feature engineering comes from domain knowledge. First inspiration comes after watching some [online course](https://www.khanacademy.org/science/organic-chemistry/spectroscopy-jay/proton-nmr/v/coupling-constant). I got the idea that the coupling constant depends on the structure of the molecule. For example, the geometry of the molecule and types of atoms that atom_0 and atom_1 are connected to. Second inspiration is that I realized the number in the type of coupling (e.g. '2' in '2JHC') stands for the number of bonds in between. 

Based on two points above, I performed feature engineering for different types. Coupling types with the same number share a lot of features. For different coupling types, the aim of feature engineering is to start from the provided atoms atom_0 and atom_1, and try figuring out the structure of the molecule. This involves understanding what type of atoms atom_0 and atom_1 are connected, the angle and distance between different atoms, mulliken charge and bonding information on each atom.

# Modeling
Models are created for each type of coupling. In my experience, lgb model seemed to be the best choice due to its fast speed. Since we have a large amount of data and limited time, the number of columns have to be limited. My training sets have around 120 columns. 40 of them are molecular level features used in every coupling type and the other 80 are features specific to different coupling type. Features are selected by iteratively adding a feature into the feature subset which gives the best result in each iteration.

Stacking is used during the modeling. The coupling constant has 4 components given in scalar_coupling_contributions.csv (only for training set). Models are first trained to predict components. Then the component predictions are used as the secondary features, together with the other 120 features, to predict the final coupling constant.

# What works
* Domain knowledge and feature engineering
* Stacking
* Hyperparameter tuning (More cross-validation folds, 'huber' loss)
* Blending

# What doesn't work
* Fail to learn tools, packages or libraries commonly used in computational chemistry
* xgboost
* Fully connected neural network

# Things to improve
* Multiprocessing and other techniques to deal with large dataset.
* Sequence model, transformer model.
