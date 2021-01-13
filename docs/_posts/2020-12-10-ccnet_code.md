---
layout: post
title:  "CCNet: Usage of Codes"
date:   2020-12-10 09:38:55
categories: publications
---

<!--
 * @Author: Conghao Wong
 * @Date: 2020-11-09 20:13:21
 * @LastEditors: Conghao Wong
 * @LastEditTime: 2021-01-13 20:33:03
 * @Description: file content
-->

This note is about how to use CCNet's code to train or test your models on your datasets.
For more about CCNet's details, please see [🔗 homepage]({% post_url 2020-12-09-ccnet %}).


## Installation

### Download
Welcome to clone our code by the following command:

```bash
git clone what.git
```

### Requirements
We developed all codes with Python 3.8 on NVIDIA GPUs.
Detailed packages used can be seen in `./requirements.txt`.
You can install all of these python packages by:

```bash
pip install -r ./requirements.txt
```


## Released Models
Please see another post [🔗 post]() to download our released model weights, and evaluate them on ETH-UCY or SDD or your datasets.

Test results should be the same as follows if there are not any errors:

<table>
    <thead>
        <tr>
            <th colspan=2>Dataset</th>
            <th>Model Name</th>
            <th>ADE</th>
            <th>FDE</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan=6>ETH-UCY</td>
            <td>eth</td>
            <td>CCNet_eth</td>
            <td>0.517</td>
            <td>1.052</td>
        </tr>
        <tr>
            <td>hotel</td>
            <td>CCNet_hotel</td>
            <td>0.222</td>
            <td>0.427</td>
        </tr>
        <tr>
            <td>zara1</td>
            <td>CCNet_zara1</td>
            <td>0.381</td>
            <td>0.817</td>
        </tr>
        <tr>
            <td>zara2</td>
            <td>CCNet_zara2</td>
            <td>0.318</td>
            <td>0.687</td>
        </tr>
        <tr>
            <td>univ</td>
            <td>CCNet_univ</td>
            <td>0.470</td>
            <td>1.021</td>
        </tr>
        <tr>
            <td>AVERAGE</td>
            <td>-</td>
            <td>0.381</td>
            <td>0.800</td>
        </tr>
        <tr>
            <td colspan=2>SDD</td>
            <td>CCNet_sdd</td>
            <td>14.63</td>
            <td>29.91</td>
        </tr>
    </tbody>
</table>


## Train New Models on ETH-UCY or SDD Datasets
In this section we will introduce how to train your CCNet model on ETH-UCY or SDD datasets.

### Datasets
The original ETH-UCY datasets (Copied from [🔗 Social GAN](https://github.com/agrimgupta92/sgan)) (contains ETH-eth, ETH-hotel, UCY-zara1,2,3 and UCY-univ1,3,examples) are included in `./data/eth` and `./data/ucy` with the real world position in `.csv` format.

The Stanford Drone Dataset (SDD) is not included in this repository due to the size limitation.
You can download the full SDD with videos in [🔗 Stanford]() or download our processed trajectory files in `.csv` format (the same definition as ETH-UCY `.csv` files) [🔗 Here]().
After downloading, put these `.csv` files in `./data/sdd` folder.
For example, put `bookstore-video0`'s file `true_pos_.csv` in `./data/sdd/bookstore/video0/true_pos_.csv`.

### Training
You can refer to the following example command to train your CCNet model on ETH-UCY or SDD datasets:

```bash
python main.py \
    --dataset ethucy \
    --test_set zara1 \
    --epochs 500 \
    --batch_size 5000 \
    --lr 0.001 \
    --gpu 0 \
    --model_name MY_MODEL
```

Your new model will be saved at `./logs/YOUR_TRAINING_TIME_YOUR_MODEL_NAME/`.
(For example `./logs/20200909-120030example_model`.)

See more descriptions of all available args in section `Options`.
Attention:
- You should specify the test dataset by passing `--test_set TEST_SET` arg when training on ETH-UCY datasets.
(Refer to the *Leave-One-Off* training strategy used in Social LSTM (CVPR2016), Social GAN (CVPR2018), etc.)

- The split of training sets, val sets, and test sets of SDD is the same as SimAug (ECCV2020), where:
    - Test sets: hyang7, hyang11, bookstore6, nexus3, deathCircle4, hyang6, hyang3, little1, hyang13, gates8, gates7, hyang2;
    - Val sets: nexus7, coupa1, gates4, little2, bookstore3, little3, nexus4, hyang4, gates3, quad2, gates1, hyang9;
    - Training sets: All other sdd datasets except the above test sets and val sets.

    You can also change the split in `./modules/models/_managers/_datasetManager.py`.


## Train New Models on Your Datasets
### Prepare datasets
If you want to train CCNet on your dataset, please process your data into several `.csv` files.
Note that the data matrix's size should be `4 * Number_of_Trajectory_Positions`, and each row should as follows:

- The first row contains all the frame numbers;
- The second row contains all the pedestrian IDs;
- The third row contains all the y-coordinates;
- The fourth row contains all the x-coordinates.

Rename these `.csv` files as `true_pos_.csv`.
Then, add your dataset's infomation to `./modules/models/_managers/_datasetManager.py` in `__init__()` of `class PredictionDatasetManager`.
For example if you have you dataset `a, b, c, d` and their `.csv` file's save folder `./a, ./b, ./c, ./d`, you can add them by:

```python
...
class PredictionDatasetManager():
    def __init__(self):
        self.datasets = dict()
        self.dataset_list = dict()
        for dataset_name, dataset_path in zip(['a', 'b', 'c', 'd'], ['./a', './b', './c', './d'])
            self.datasets[dataset_name] = Dataset(
                # name of your dataset
                dataset     =   dataset_name,
                # save path of your dataset      
                dataset_dir =   dataset_path,
                # x-y order of your csv file        
                order       =   [1, 0],
                # leave the below four items as `[], '', [], 1` if your dataset have no videos
                # paras are [sample_rate, frame_rate] of your video 
                paras       =   [1, 30],
                # video path of your dataset
                video_path  =   dataset_path + '/video.mp4',
                # transform weights [wx, bx, wy, by] from csv file to real videos  
                weights     =   [1.0, 0.0, 1.0, 0.0],
                # scale when draw visualized results
                # visualized_shape = original_video_shape/scale
                scale       =   2,
            )

        self.dataset_list[YOUR_DATASET] = ['a', 'b', 'c', 'd']
        self.train_list[YOUR_DATASET] = ['a', 'b']
        self.val_list[YOUR_DATASET] = ['c']
        self.test_list[YOUR_DATASET] = ['d']
    ...
```

### Run training

Commands used to train your model are the same as them used to train on the ETH-UCY dataset.
When training your model on you dataset named `'my_dataset'` you can use the below command:

```bash
python main.py --dataset my_dataset
```


## Evaluate Models

### Prepare model weights and data

After training, these files will be saved in your log folder:
- model weights file `YOUR_MODEL_NAME.tf` (Support both `.h5` and `.tf` format);
- model arguments file `args.npy`.
  
Make sure your log folder contains at least this two files before evaluation.

### Run evaluation

You can run the evaluation of your model by the following commands:

```bash
python main.py --load YOUR_LOG_DIR --draw_results 0
```

You can draw the visualized results on the video to see the model's performance if your test dataset have videos.
Set the arg `--draw_results` to `1` to enable saving.
**Note** that make sure you have put the videos of your test dataset correctly, and you have already config your datasets' videos' path in `./modules/models/_managers/_datasetManager.py`.
See above *Train New Models on Your Datasets* section for details.
Default saving path of these visualized results is `YOUR_LOG_DIR/VisualTrajs/`.


## Model Options
You can custom these options both on model components and training or evaluation.
Note that all items that need a bool type of inputs should take integer `0` and `1` instead of `False` and `True`.
To passing your customed args, please use the below format:

```bash
python main.py --ARGS_YOU_WANT_TO_CHANGE ARGS_VALUE
```

To get help, you can run the following commands:

```bash
python main.py -h
```

Detailed description of all available arguments are list below:

- `--add_noise`:
Controls if add noise to training data
Default is `0`.

- `--avoid_size`:
Avoid size in grids.
Default is `15`.

- `--batch_size`:
Training batch_size.
Default is `5000`.

- `--dataset`:
Dataset. Can be `ethucy` or `sdd`.
Default is `ethucy`.

- `--diff_weights`:
Parameter of linera prediction.
Default is `0.95`.

- `--draw_results`:
Controls if draw visualized results on videoframes. Make sure that you have put video files.
Default is `0`.

- `--dropout`:
Dropout rate.
Default is `0.5`.

- `--epochs`:
Training epochs.
Default is `500`.

- `--force_set` *or* `-fs`:
Force test dataset. Only works on ETH-UCY dataset when arg `load` is not `null`.
Default is `null`.

- `--gpu` *or* `-g`:
Speed up training or test if you have at least one nvidia GPU. Use `_` to separate if you want to use more than one gpus. If you have no GPUs or want to run the code on your CPU, please set it to `-1`.
Default is `0`.

- `--init_position`:
***DO NOT CHANGE THIS***.
Default is `10000`.

- `--interest_size`:
Interest size in grids.
Default is `20`.

- `--load` *or* `-l`:
Log dir to load model. If set to `null`,it will start training new models according to arguments.
Default is `null`.

- `--log_dir`:
Log dir for saving logs. If set to `null`,logs will save at save_base_dir/current_model.
Default is `null`.

- `--lr`:
Learning rate.
Default is `0.001`.

- `--map_half_size`:
Local map's size.
Default is `50`.

- `--model`:
Model used to train. Canbe `l` (linear prediction) or `ccnet` (CCNet).
Default is `ccnet`.

- `--model_name`:
Model's name when saving.
Default is `model`.

- `--obs_frames`:
Observation frames for prediction.
Default is `8`.

- `--pred_frames`:
Prediction frames.
Default is `12`.

- `--prepare_type`:
Prepare argument. Do Not Change it.
Default is `test`.

- `--rotate`:
Rotate dataset to obtain more available training data.This arg is the time of rotation, for example set to `1` will rotatetraining data 180 degree once; set to `2` will rotate them 120 degreeand 240 degree.
Default is `0`.

- `--save_base_dir`:
Base saving dir of logs.
Default is `./logs`.

- `--save_best`:
Controls if save the best model when val.
Default is `1`.

- `--save_format`:
Model save format, canbe `tf` or `h5`.
Default is `tf`.

- `--save_model`:
Controls if save the model.
Default is `1`.

- `--start_test_percent`:
Set when to start val during training.Range of this arg is [0.0, 1.0]. The val will start at epoch = args.epochs * args.start_test_percent.
Default is `0.0`.

- `--step`:
Frame step for obtaining training data.
Default is `4`.

- `--test`:
Controls if run test.
Default is `1`.

- `--test_mode`:
Test settings, canbe `one` or `all` or `mix`.When set to `one`, it will test the test_set only;When set to `all`, it will test on all test datasets of this dataset;When set to `mix`, it will test on one mix dataset that made up of alltest datasets of this dataset.
Default is `one`.

- `--test_set` *or* `-s`:
Test dataset. Only works on ETH-UCY dataset.
Default is `zara1`.

- `--test_step`:
Val step in epochs.
Default is `3`.

- `--train_percent` *or* `-tp`:
Percent of training data used in training datasets. Split with `_` if you want to specify each dataset, for example `0.5_0.9_0.1`.
Default is `1.0_`.

- `--verbose` *or* `-v`:
Set if print logs
Default is `1`.

- `--window_size_expand_meter`:
***DO NOT CHANGE THIS***.
Default is `10.0`.

- `--window_size_guidance_map`:
Resolution of map.(grids per meter)
Default is `10`.