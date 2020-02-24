
## Step 1: Prepare the data and model
We experimented on the [ActEv dataset](https://actev.nist.gov) and the
[ETH & UCY dataset](https://graphics.cs.ucy.ac.cy/research/downloads/crowd-data).
The original ActEv annotations can be downloaded from [here](https://next.cs.cmu.edu/data/actev-v1-drop4-yaml.tgz).
*Please do obtain the data copyright and download the raw videos from their website.*
You can download our prepared features from the [project page](https://next.cs.cmu.edu)
by running the script `bash scripts/download_prepared_data.sh`.
This will download the following data,
and will require about 31 GB of disk space:

- `next-data/final_annos/`: This folder includes extracted features and
annotations for both experiments. Data format notes are [here](NOTES.md#prepared-data).
- `next-data/actev_personboxfeat/`: This folder includes person appearance
features for ActEv experiments
- `next-data/ethucy_personboxfeat/`: This folder includes person appearance
features for ETH/UCY experiments

Then download the pretrained model following instructions
[from here](README.md#pretrained-models).

## Step 2: Preprocess - ActEv
Preprocess the data for training and testing.
The following is for ActEv experiments.

```
python code/preprocess.py next-data/final_annos/actev_annos/virat_2.5fps_resized_allfeature/ \
  actev_preprocess --obs_len 8 --pred_len 12 --add_kp --kp_path next-data/final_annos/actev_annos/anno_kp/ \
  --add_scene --scene_feat_path next-data/final_annos/actev_annos/ade20k_out_36_64/ \
  --scene_map_path next-data/final_annos/actev_annos/anno_scene/ \
  --scene_id2name next-data/final_annos/actev_annos/scene36_64_id2name_top10.json \
  --scene_h 36 --scene_w 64 --video_h 1080 --video_w 1920 --add_grid \
  --add_person_box --person_box_path next-data/final_annos/actev_annos/anno_person_box/ \
  --add_other_box --other_box_path next-data/final_annos/actev_annos/anno_other_box/ \
  --add_activity --activity_path next-data/final_annos/actev_annos/anno_activity/ \
  --person_boxkey2id_p next-data/final_annos/actev_annos/person_boxkey2id.p
```

## Step 3: Test the models - ActEv
Run testing with our pretrained single model for ActEv experiments.

```python
python code/test.py /home/richardkxu/DATA/NEXT-DATA/actev_preprocess \
  next-models/actev_single_model model --runId 1 --load_best \
  --load_from next-models/actev_single_model/model/01/best.ckpt --is_actev --add_kp --add_activity \
  --person_feat_path /home/richardkxu/DATA/NEXT-DATA/next-data/actev_personboxfeat --multi_decoder
```

The evaluation result should be:
<table>
  <tr>
    <td>Activity mAP</td>
    <td>ADE</td>
    <td>FDE</td>
  </tr>
  <tr>
    <td>0.199</td>
    <td>17.979</td>
    <td>37.176</td>
  </tr>
</table>

Test results:
```
100%|##########| 535/535 [18:39<00:00,  2.09s/it]
performance:
0000_ade, 18.5173
0000_fde, 38.880512
0002_ade, 12.5554085
0002_fde, 25.474648
0400_ade, 13.4572315
0400_fde, 28.538671
0401_ade, 22.815294
0401_fde, 47.292492
0500_ade, 23.5591
0500_fde, 47.287983
act_ap, 0.1996404821024035
ade, 17.978905
fde, 37.175632
grid1_acc, 0.29725467289719626
grid2_acc, 0.3974883177570093
mov_ade, 20.28942
mov_fde, 42.356396
static_ade, 14.221087
static_fde, 28.749632
traj_class_accuracy, 0.9351375073142189
traj_class_accuracy_0, 0.8822806208698325
traj_class_accuracy_1, 0.9676367759614476
0000_ade 0000_fde 0002_ade 0002_fde 0400_ade 0400_fde 0401_ade 0401_fde 0500_ade 0500_fde act_ap ade fde grid1_acc grid2_acc mov_ade mov_fde static_ade static_fde traj_class_accuracy traj_class_accuracy_0 traj_class_accuracy_1
18.5173 38.880512 12.5554085 25.474648 13.4572315 28.538671 22.815294 47.292492 23.5591 47.287983 0.1996404821024035 17.978905 37.175632 0.29725467289719626 0.3974883177570093 20.28942 42.356396 14.221087 28.749632 0.9351375073142189 0.8822806208698325 0.9676367759614476

```

## Step 4: Preprocess - ETH/UCY
Preprocess the data for training and testing. The following is for ETH/UCY
experiments. We conduct leave-one-scene-out experiment therefore we need to
preprocess the data once for each scene.

```
for dataset in {eth,hotel,univ,zara1,zara2};
  do
    python code/preprocess.py next-data/final_annos/ucyeth_annos/original_trajs/${dataset}/ ethucy_exp/preprocess_${dataset} \
    --person_boxkey2id next-data/final_annos/ucyeth_annos/${dataset}_person_boxkey2id.p \
    --obs_len 8 --pred_len 12 --min_ped 1 --add_scene \
    --scene_feat_path next-data/final_annos/ucyeth_annos/ade20k_e10_51_64/ \
    --scene_map_path next-data/final_annos/ucyeth_annos/scene_feat/ \
    --scene_id2name next-data/final_annos/ucyeth_annos/scene51_64_id2name_top10.json \
    --scene_h 51 --scene_w 64 --video_h 576 --video_w 720 --add_grid --add_person_box \
    --person_box_path next-data/final_annos/ucyeth_annos/person_box/ --add_other_box \
    --other_box_path next-data/final_annos/ucyeth_annos/other_box/ \
    --feature_no_split --reverse_xy --traj_pixel_lst \
    next-data/final_annos/ucyeth_annos/traj_pixels.lst ;
  done
```

## Step 4: Test the models - ETH/UCY
Run testing with our pretrained single model for each scene
in the ETH/UCY experiments.

```
for dataset in {eth,hotel,univ,zara1,zara2};
  do
    python code/test.py ethucy_exp/preprocess_${dataset} next-models/ethucy_single_model/${dataset}/ model \
      --runId 1 --load_best --person_feat_path next-data/ethucy_personboxfeat/${dataset}/ \
      --scene_h 51 --scene_w 64 ;
  done
```

```python
python code/test.py /home/richardkxu/DATA/NEXT-DATA/ethucy_exp/preprocess_eth \
  next-models/ethucy_single_model/eth model --runId 1 --load_best --scene_h 51 --scene_w 64 \
  --load_from next-models/ethucy_single_model/eth/model/01/best.ckpt \
  --person_feat_path /home/richardkxu/DATA/NEXT-DATA/next-data/ethucy_personboxfeat/eth 
```

The evaluation result should be:
<table>
  <tr>
    <td>Scene</td>
    <td>ADE</td>
    <td>FDE</td>
  </tr>
  <tr>
    <td>ETH</td>
    <td>0.86</td>
    <td>1.94</td>
  </tr>
  <tr>
    <td>HOTEL</td>
    <td>0.36</td>
    <td>0.74</td>
  </tr>
  <tr>
    <td>UNIV</td>
    <td>0.62</td>
    <td>1.32</td>
  </tr>
  <tr>
    <td>ZARA1</td>
    <td>0.42</td>
    <td>0.91</td>
  </tr>
  <tr>
    <td>ZARA2</td>
    <td>0.34</td>
    <td>0.74</td>
  </tr>
</table>

eth test results:
```
100%|##########| 3/3 [00:07<00:00,  2.63s/it]
performance:
act_ap, None
ade, 0.85683197
fde, 1.9422761
grid1_acc, 0.20833333333333334
grid2_acc, 0.28125
act_ap ade fde grid1_acc grid2_acc
None 0.85683197 1.9422761 0.20833333333333334 0.28125

```

