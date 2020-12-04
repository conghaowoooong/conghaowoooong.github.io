---
layout: post
title:  "BGM: Evaluation on ETH-UCY by Our Already Trained Weights"
date:   2020-12-04 19:45:37 +0800
categories: publications
---

<!--
 * @Author: Conghao Wong
 * @Date: 2020-12-04 19:03:16
 * @LastEditors: Conghao Wong
 * @LastEditTime: 2020-12-04 19:54:03
 * @Description: Usage of already trained models
-->

# Usage of Already Trained ETH-UCY Weights
This note is about how to use our already trained BGM models.
For more BGM's details, please see our [🔗homepage]({% post_url 2020-12-03-bgm %}).

## Download Model Weights
Please download the zip file [🔗HERE](https://baidu.com).
It contains 5 folders, including:

```bash
20200905-202720NEW_500MRR3bgm0
20200905-202722NEW_500MRR3bgm1 
20200905-202724NEW_500MRR3bgm2 
20200905-202726NEW_500MRR3bgm3 
20200905-202727NEW_500MRR3bgm4
```

Each dataset folder contains 3 items: 
(a) model weights file `MODEL_NAME.h5`;
(b) model args file `MODEL_NAMEargs.npy`;
and (c) test data `MODEL_NAMEtest.npy`.
Note that the arg configuration in the arg `*arg.npy` file should match the model structure in the weights `*.h5` file, and ***DO NOT*** copy them to your models directly.

For example:

```bash
20200905-202727NEW_500MRR3bgm4
    NEW_500MRR3.h5
    NEW_500MRR3args.npy
    NEW_500MRR3test.npy
```

Before run the test, please unzip the downloaded file and move all of these folders to `BGM_prediction/logs/`.
(If there is no `logs` dir, please create it manually.)

---

## Evaluation
For simply test these models on **ETH-UCY** datasets (`eth, hotel, zara1, zara2, univ`), please run the below commands:

```bash
cd ~/BGM_prediction     # Your path of `BGM_prediction`
for set_dir in  20200905-202720NEW_500MRR3bgm0/NEW_500MRR3 \
                20200905-202722NEW_500MRR3bgm1/NEW_500MRR3 \
                20200905-202724NEW_500MRR3bgm2/NEW_500MRR3 \
                20200905-202726NEW_500MRR3bgm3/NEW_500MRR3 \
                20200905-202727NEW_500MRR3bgm4/NEW_500MRR3
do
    python main.py --load ./logs/$set_dir
done
```

---

## Visualized Results
To save the visualized images on one of `eth, hotel, zara1, zara2` datasets, please first download their videos [🔗HERE](https://baidu.com), unzip them and put them in `BGM_prediction/videos/`.
(If there is no `videos` dir, please create it manually.)

Thus, you can save the visualized results by adding args `--draw_results 1` to the evaluation command above as:
```bash
python main.py --load ./logs/$set_dir --draw_results 1
```

Results figures will be saved at `BGM_prediction/logs/SET_DIR/VisualTrajs/`.