# MRF
This is the repository for MRF related projects of the Setsompop Lab at Stanford University. The code here is used to read and reconstruct MRF data and accompanying calibration scans.

## Usage
Each directory in `src` can be built into a Docker container that performs one task in the pipeline. By navigating into each directory and running `make` the docker containers will be built with all dependencies. The code can also be run natively, but then you need to manage all dependencies as listed in the Dockerfile for each task. For each task certain input arguments have to be set. The `main.py` file for each task lists input arguments and what format they should be in. Example calls are provided below.

1. `00_io` manages reading ScanArchive files form GE and can write Dicoms with metadata from the ScanArchives.

### Example call to save raw k-space data from ScanArchive to Numpy Array
```
docker run -v $(pwd):$(pwd):z setsompop/scan_archive_io \
        --scn $(pwd)/rel/path/to/scanarchive.h5 \
        --ksp $(pwd)/rel/path/to/ksp
```
Running the above creates a file called `ksp.npy`

2. `01_calib` is used to calibrate the MRF reconstruction with data from a pre-scan. It can pre-calculate the coil compression matrix using RoVir from a large FOV acquisition and shift the FOV so that the brain is within the smaller FOV for MRF.

<b>Example call to calculate shifts and coil compressionmatrix from large FOV GRE data</b>
```
docker run --gpus all -v /:/mnt/:z MRF/calib 
        --ksp /mnt/FULL PATH TO GRE NUMPY ARRAY
        --ccm /mnt/FULL PATH TO SAVE LOCATION OF COILS COMPRESSION MATRIX NUMPY ARRAY
        --shf /mnt/FULL PATH TO SAVE LOCATION OF SHIFTS NUMPY ARRAY
        --nrc 40
        --nsv 10
```
3. `02_recon` is used to reconstruct the MRF data . It has many optional input arguments for pre-calculated data (e.g. density compensation and coil sensitivity maps) for improved pipeline optimization avoiding recalculating the same thing multiple times. Note that GPU access is important for fast reconstruction.

<b>Example call to reconstruct MRF data using a subspace reconstruction</b>
```
docker run --gpus all -v /:/mnt/:z MRF/recon 
        -p
        --trj /mnt/FULL PATH TO TRAJECTORY FILE
        --ksp /mnt/FULL PATH TO KSPACE DATA FILE
        --res /mnt/FULL PATH TO SAVE LOCATION OF RECONSTRUCTED IMAGES NUMPY ARRAY
        --phi /mnt/FULL PATH TO SUBSPACE BASIS FILE
        --ccm /mnt/FULL PATH TO COIL COMPRESSION MATRIX
        --shf /mnt/FULL PATH TO SHIFTS 
        --mal jsense
        --mtx 256 --ptt 10 --dev 0
        --pdg 0 --blk 8 --lam 5e-5 --mit 40 
```
## Example data
TODO!
