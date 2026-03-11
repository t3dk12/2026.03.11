# 2026.03.11
I-ReaxFF to train for ReaxFF-NN

Scripts for processing VASP DFT data and training a Message Passing Neural Network (MPNN) ReaxFF potential for Lithium metal.

## Files

* **build_dataset_duplicate.py**: The main pipeline script. Parses OUTCAR/vasprun.xml files, handles the train/test split, and outputs a master summary CSV. Use the `--duplicate on` flag to automatically supercell small geometries so they don't violate the minimum image convention based on the cutoff in the ffield file.
* **train_without_force_Li_NN.py**: The actual neural network training script. Optimises the ReaxFF/MPNN energy, bond-order, and many-body parameters.
* **ffield.json**: The starting force field parameter file. Generated from previous I-ReaxFF run and 2024vD data.

## Usage

First, process the DFT data and duplicate the undersized cells:

```bash
python build_dataset_duplicate.py --duplicate on
```

Any crashed VASP runs or original undersized cells get dumped into the traj/unused_data/ folder automatically so they don't mess up the training.

Once the data is ready, run the training script. Use the following command-line toggles to configure the training loop and freeze/optimise specific classical physics terms:

```bash
python train_without_force_Li_NN.py --step 10000 --lr 0.0001 --pr 10 --writelib 1000 --batch 50 --t 1 --h 0 --a 1 --f 1 --bo 1 --vdw 1
```

# Training Toggles (`train_without_force_Li_NN.py`)

### Hyperparameters:
* `--step (-s)`: Total training iterations (epochs). Default is `10000`.
* `--lr`: Learning rate for gradient updates. Default is `0.0001`.
* `--writelib`: Step interval for saving the parameter library to disk. Default is `1000`.
* `--pr`: Step interval for console output/logging of loss metrics. Default is `10`.
* `--batch`: Number of molecular configurations processed per batch. Default is `50`.
* `--ffield`: Path to the initial ReaxFF/MPNN force-field file. Default is `ffield.json`.
* `--train_dir`: Directory containing the training `.traj` files. Default is `traj/train`.

### Physics Control Flags (1 = Optimise, 0 = Freeze):
* `--t`: Three-body (angle) term optimisation. Default is `1`.
* `--h`: Hydrogen bond term optimisation. Default is `0`.
* `--a`: Angle term supervision (general bending). Default is `1`.
* `--f`: Four-body (torsion) term optimisation. Default is `1`.
* `--bo`: Bond order/energy optimisation. Default is `1`.
* `--vdw`: Van der Waals (non-bonded) energy optimisation. Default is `1`.

## Dataset Generation Toggles (`build_dataset_duplicate.py`)

### Core Directories & Files:
* `--input_dir`: Directory containing raw VASP output files. Default is `Li-metal_OUTCARs`.
* `--output_dir`: Directory where the processed `.traj` files will be saved. Default is `traj`.
* `--csv_name`: Name of the generated master summary CSV. Default is `dft_summary.csv`.

### Dataset Splitting:
* `--split_ratio`: Fraction of valid data to reserve for the test set. Default is `0.10` (10%).
* `--seed`: Random seed for reproducible train/test splitting. Default is `42`.

### Supercell Duplication (Minimum Image Convention):
* `--duplicate`: Set to `on` to enable automatic supercell duplication for undersized geometries. Set to `off` to extract as-is. Default is `off`.
* `--ffield`: Path to your force field parameter file to dynamically extract the `rcut` value. Default is `ffield.json`.
* `--cutoff`: Fallback cutoff radius (in Å) to use if `ffield.json` is missing or unreadable. Default is `4.0`. 
