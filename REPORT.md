# Table
It is uploaded into the repo – [`results.csv`](https://github.com/kovarski-c/mlr-ex08/blob/main/results.csv) contain all three seeds, [`results_agg.csv`](https://github.com/kovarski-c/mlr-ex08/blob/main/results_agg.csv) contains averages of three seeds and the variance, ordered by best in camera_like accuracy.

# Baseline
Task 3 LeafNet, 101 train / 28 val photos, 15 epochs, Adam 1e-3, three seeds. Val: 52.4 ± 10.3 %, camera-like 35.7 ± 0.0 %.

# Noise
28 val images → one image = 3.6 pp; the baseline alone differs ±10.3 % between seeds.

# What helped
On the camera-like set: 
* pretraining (+24 % over the baseline)
* geometric augmentation (+13.1 %)
* dropout (+13.1 %), GAP (+9.5 % with 3.2× fewer parameters) 
* colour jitter (+3.6 pp, weakest)

On the clean val set: BatchNorm (+9.5 %) and pretraining.

# What did not help
* AdamW (+4.8 camera, +1.3 images — noise)
* label smoothing (nothing)
* colour jitter beyond noise
* combo_scratch

BatchNorm gains on the clean val set and nothing on the camera set, augmentation and dropout do the opposite. The clean set rewards memorizing those particular 28 photographs.

# Which model goes on the phone
`mnv3_frozen`: camera-like accuracy is the best and the steadiest one (63.1 ± 2.1 %) except of `mnv3_full_aug`, but this one takes like 10 times more time to train, that's why I choose `mnv3_frozen`.
!(screenshot_task4.png)[screenshot_task4.png]