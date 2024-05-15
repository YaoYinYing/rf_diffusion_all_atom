Code for RFDiffusion AA
--------------------
<p align="center">
  <img src="./img/RFDiffusionAA.png" alt="alt text" width="600px"/>
</p>

### Setup/Installation (MacOS, M1 chips)
1. Clone the package
```
git clone https://github.com/YaoYinYing/rf_diffusion_all_atom.git
cd rf_diffusion_all_atom
git switch pip-installable # switch to pip installable branch
```
2. Download the model weights.
  ```
  wget http://files.ipd.uw.edu/pub/RF-All-Atom/weights/RFDiffusionAA_paper_weights.pt
  ```
3. use the exact conda env of RF2AA:
   ```shell
   conda activate rf2aa # suppose you have created the conda env and installed the RF2AA
   ```

4. Install the remaining dependencies:
   ```shell
   pip3 install torch torchvision torchaudio --force-reinstall 
   pip install 'libclang>=13.0.0' 'protobuf<3.20,>=3.9.2' 
   pip install dgl -f https://data.dgl.ai/wheels/repo.html --force-reinstall 
   pip install -r requirements.txt   
   ```
5. Install RFdiffusionAA:
   ```shell
   pip install .   
   ```

### Inference
#### Small molecule binder design

To generate a binder to the ligand OQO from PDB 7v11, run the following:


Example (ligand binder):
```shell
HYDRA_FULL_ERROR=1  rfdaa_inference inference.ckpt_path=/path/to/weights/RFdiffusionAA/RFDiffusionAA_paper_weights.pt inference.deterministic=True diffuser.T=100 inference.output_prefix=output/ligand_only/sample inference.input_pdb=input/7v11.pdb 'contigmap.contigs=[150-150]' inference.ligand=OQO inference.num_designs=1 inference.design_startnum=0
```



Explanation of arguments:
- `inference.ckpt_path` specifies the path to the checkpoint file
- `inference.deterministic=True` seeds the random number generators used so that results are reproducible.  i.e. running with inference.design_startnum=X will produce the same reusults.  Note that torch does not guarantee reproducibility across CPU/GPU architectures: https://pytorch.org/docs/stable/notes/randomness.html
- `inference.num_designs=1` specifies that 1 design will be generated
- `'contigmap.contigs=[150-150]'` (Please mind the single quotes) specifies that the length of the generated protein should be 150
- `diffuser.T=100` specifies the number of denoising steps taken.

Expected outputs:
- `output/ligand_only/sample_0.pdb` The design PDB
- `output/ligand_only/sample_0_Xt-1_traj.pdb` The partially denoised intermediate structures
- `output/ligand_only/sample_0_X0-1_traj.pdb` The predictions of the ground truth made by the network at each step

Note that the sequences associated with these structure have no meaning, apart from the given motif.  LigandMPNN or similar must be used to generate sequences for the backbones if they are to be used for structure prediction / expression.

To include protein residues A430-435 in the motif, use the argument contigmap.contigs.  e.g. `contigmap.contigs=[\'10-120,A84-87,10-120\']` tells the model to design a protein containing the 4 residue motif A84-87 with 10-120 residues on either side.

#### Small molecule binder design with protein motif
Example (ligand binder with protein motif):
```shell
HYDRA_FULL_ERROR=1  rfdaa_inference inference.ckpt_path=/path/to/weights/RFdiffusionAA/RFDiffusionAA_paper_weights.pt inference.deterministic=True diffuser.T=200 inference.output_prefix=output/ligand_protein_motif/sample inference.input_pdb=input/1haz.pdb 'contigmap.contigs=[10-120,A84-87,10-120]' contigmap.length="150-150" inference.ligand=CYC inference.num_designs=1 inference.design_startnum=0

```

An end-to-end design pipeline illustrating the design of heme-binding proteins using RFdiffusionAA, proteinMPNN, AlphaFold2, LigandMPNN and PyRosetta is available at: https://github.com/ikalvet/heme_binder_diffusion

