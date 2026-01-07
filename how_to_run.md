# How to Run TransPro

This guide explains how to use the TransPro repository for testing pre-trained models on your own data.

## Environment Setup

1. Create and activate a conda environment:
```bash
conda create -n octa python=3.6
conda activate octa
```

2. Install requirements:
```bash
pip install -r requirements.txt
```

## Data Preparation

### Format Requirements

This repository works with 3D OCT and OCTA volumes in NumPy (.npy) format.

### Directory Structure

Organize your data in the following structure:
```
your_data_directory/
├── test/
    ├── A/  # Input OCT images (3D .npy volumes)
    │   ├── volume1.npy
    │   ├── volume2.npy
    │   └── ...
    └── B/  # For testing, you can use empty placeholders or ground truth OCTA
        ├── volume1.npy
        ├── volume2.npy
        └── ...
```

**Important Notes:**
- Both folders A and B must have the same number of files with corresponding names
- Each .npy file should contain a 3D NumPy array
- The model expects input size of 304×304×304 (as used during training)

## Using Pre-trained Models

### Download Weights

Make sure you have the pre-trained weights in the correct location:
1. Download the weights from:
   - [VPG](https://drive.google.com/file/d/1dUf45500QKoO9h9VEDOvFGlN2rxD_853/view?usp=share_link)
   - [HCG](https://drive.google.com/file/d/1eAIt3feAIsr1Wn_f_mnPmYf6iVwwLmyk/view?usp=share_link)
2. Place both files in the `pretrain-weights` folder (should already contain vpg.pth and hcg.pth)

### Run Training
To train the model, run:
```bash

python train3d.py \
    --dataroot /home/omerfarukaydin/Desktop/transpro-dataset \
    --name transpro_3M \
    --model TransPro \
    --netG unet_256 \
    --direction AtoB \
    --lambda_A 10 \
    --lambda_C 5 \
    --dataset_mode alignedoct2octa3d \
    --norm batch \
    --pool_size 0 \
    --load_size 304 \
    --input_nc 1 \
    --output_nc 1 \
    --display_port 6031 \
    --gpu_ids 0 \
    --no_flip \
    --continue_train \
    --epoch 20
```


### Run Testing

To test the model on your data, run:
```bash
python test3d.py \
    --dataroot /home/omerfarukaydin/Desktop/ad-transpro-dataset \
    --name transpro_3M \
    --test_name ad-oct-to-octa \
    --model TransPro \
    --netG unet_256 \
    --direction AtoB \
    --lambda_A 10 \
    --lambda_C 5 \
    --dataset_mode alignedoct2octa3d \
    --norm batch \
    --input_nc 1 \
    --output_nc 1 \
    --gpu_ids 0 \
    --num_test 323 \
    --which_epoch latest \
    --load_iter 164 
```

Replace the following:
- `/path/to/your/data`: Path to your data directory (containing the test folder)
- `your_test_name`: A name for this test run
- `NUMBER_OF_TEST_SAMPLES`: The number of samples to test (default is 15200, set to a value matching your dataset size)

### GPU/CPU Options

- With GPU: Use `--gpu_ids 0` (for first GPU)
- Without GPU: Use `--gpu_ids -1` to run on CPU

## Output Results

Test results will be saved in:
```
./results/your_test_name/test_164/
```

The output directory will contain:
- Generated OCTA volumes
- HTML page with result visualizations

## Troubleshooting

1. **Memory issues**: If you encounter memory problems, try using smaller volumes or processing fewer samples at once.

2. **File format errors**: Ensure all your .npy files are valid NumPy arrays:
   ```python
   # Check a sample file
   import numpy as np
   sample = np.load('/path/to/sample.npy')
   print(sample.shape)  # Should show dimensions of your 3D volume
   ```

3. **Missing files**: The code expects both A and B folders to have the same number of files with matching names. For testing, you can create placeholder files in the B folder if needed.

4. **Path errors**: Ensure all paths are absolute or correctly relative to your working directory.

## Data Format Example

Your input NumPy files should contain 3D volumes. Here's an example of how to create a test file:

```python
import numpy as np

# Example: Create a dummy OCT volume (replace with your actual data)
dummy_oct = np.random.rand(304, 304, 304)  # Create random 3D volume
np.save('/path/to/your/data/test/A/sample1.npy', dummy_oct)

# For test B, you can create an empty placeholder if you don't have ground truth
dummy_octa = np.zeros((304, 304, 304))  # Empty volume
np.save('/path/to/your/data/test/B/sample1.npy', dummy_octa)
```