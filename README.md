# DeepEthogram Note

DeepEthogram is an open-source pipeline based on supervised machine learning for automatically classifying each frame of a video into a set of user-defined behavior labels. It enables researchers to quantify pre-defined behaviors-of-interest without time-consuming manual quantification. This note will show you how to get DEG running on the laptop and train all models using command line interface or GUI.

![enter image description here](https://user-images.githubusercontent.com/81632945/120920670-8757de00-c6f2-11eb-8c7e-76ef3ab5c365.jpg)

 - Written by Yi (Daniel) Lu 
 -  luyi2018@zju.edu.cn

# Instructions for getting started
## Installation
See [the installation documentation](https://github.com/jbohnslav/deepethogram/blob/master/docs/installation.md).
In brief:
-   `conda create --name deg python=3.7`
-   `conda activate deg`
-   [Install PyTorch](https://pytorch.org/)
-   `pip install deepethogram`

## Workflow
 - Run first behavior experiments, collect videos.
 - Make a DeepEthogram project
 -  Label videos on your laptop with the DeepEthogram GUI.
 -  Convert videos (optional)
 -  Train the flow generator 
 -  Train the feature extractor and run inference
 -  Train a sequence model and run inference
 -  The predictions are  generated. Use the GUI to re-label mistakes.
 -  Re-train your previous models with the new videos and labels
## Making a project
 -  Open your terminal window 
 -  Activate your `conda` environment by typing `conda activate deg`.
 -  Type   `deepethogram` or `python -m deepethogram` in the command line to open the GUI. 
 
![enter image description here](https://user-images.githubusercontent.com/81632945/120920668-84f58400-c6f2-11eb-882d-ad60505fbf05.png)

Go to  `file -> new project`. Select a location for the new project to be created. 
![enter image description here](https://user-images.githubusercontent.com/81632945/120920822-588e3780-c6f3-11eb-8026-918af3614897.png)
After selecting a location, a screen will appear with three fields:

-   `project name`: the name of your project. Examples might be:  `mouse_reach`,  `grooming`,  `itch_mutation_screen`, etc.
-   `name of person labeling`: your name. Currently unused, in the future it could be used to compare labels between humans.
-   `list of behaviors`: the list of behaviors you want to label. Think carefully since all previous models must need to be retrained when a behavior has been added or removed! Separate the behaviors with commas. Do not include  `none`,  `other`,  `background`,  `etc`, or  `misc`  or anything like that.

Press OK.

A directory will be created in the location you specified, with the name  `projectname_deepethogram`. It will initialize the file structure needed to run deepethogram

**Equivalent command line arguments:**

    from deepethogram import projects
    
    # this is a path to a base directory on your hard drive
    # this is just an example! change this to whereever you want, e.g. 'C:\DATA\movies`
    data_path = '/mnt/DATA'
    
    # pick a project name
    project_name = 'Daniel'
    
    # make a list of behaviors. try to choose your behaviors carefully! 
    # you'll have to re-train all your models if you add or remove behaviors. 
    behaviors = ['background', 
                'scratch', 
                'etc1', 
                'etc2']
    
    # this will create a folder called /mnt/DATA/open_field_deepethogram
    # there will be subdirectories called DATA and models
    # there will also be a project_config.yaml
    project_config = projects.initialize_project(data_path, project_name, behaviors)

## Add videos

When you add videos to DeepEthogram, we will  **copy them to the deepethogram project directory**  (*not move or use a symlink*). We highly recommend starting with at least 3 videos, so you have more than 1 to train, and 1 for validation (*roughly, videos are assigned to splits probabilistically, it's better to start with >10 decently sized videos*). When you add videos, DeepEthogram will automatically compute mean and standard deviation statistics and save them to disk. This might take a few moments (or minutes for very long videos). 

![enter image description here](https://user-images.githubusercontent.com/81632945/120922843-0272c180-c6fe-11eb-9a80-4829ffd969eb.png)

**Equivalent command line arguments:**

    # adding videos
    list_of_movies = ['/path/to/training 1.mov', 
                     '/path/to/assessment 1.mov'
                     '/path/to/assessment 2.mp4']
    mode = 'copy' # or 'symlink' or 'move'
    
    # depending on the mode, it will copy, symlink, or move each video file
    # it will also compute the mean and standard deviation of each RGB channel
    for movie_path in list_of_movies:
        projects.add_video_to_project(project_config, movie_path, mode=mode)
    
    # now, we have our new movie files properly in our deepethogram project
    new_list_of_movies = ['/mnt/DATA/Daniel_deepethogram/DATA/training 1.mov', 
                       '/mnt/DATA/Daniel_deepethogram/DATA/assessment 1.mov',
                      '/mnt/DATA/Daniel_deepethogram/DATA/assessment 2.mp4' ]
**Execution results:**

    [2021-06-06 19:41:02,694] INFO [deepethogram.gui.main.initialize_project:1007] cwd: C:\deepethogram\training_deepethogram\gui_logs\210606_194102
    [2021-06-06 19:41:02,778] INFO [deepethogram.gui.main.project_loaded_buttons:173] Number finalized labels: 3
    [2021-06-06 19:41:03,112] INFO [deepethogram.gui.main.initialize_video:221] Record for loaded video: {'flow': None, 'label': 'C:\\deepethogram\\training_deepethogram\\DATA\\training 1\\training 1_labels.csv', 'output': None, 'rgb': 'C:\\deepethogram\\training_deepethogram\\DATA\\training 1\\training 1.MOV', 'keypoint': None, 'key': 'training 1'}
    [2021-06-06 19:41:12,016] INFO [deepethogram.gui.main.initialize_video:236] Copying C:/DEG training videos/assessment.MOV to your DEG directory
    [2021-06-06 19:41:16,729] INFO [deepethogram.zscore.zscore_video:151] zscoring file: C:\deepethogram\training_deepethogram\DATA\assessment\assessment.MOV
      1%|█▋                                                                                                                                              | 70/5930 [00:32<48:48,  2.00it/s]
## Label a few videos

 -  **Label buttons**: These buttons are the ordered list of behaviors for your project. Each button corresponds to a row in the label viewer (**9ii**). Pushing this button will toggle the behavior. Background is a special class: it is mutually exclusive with the other behaviors. Pushing the keyboard number keys denoted in brackets will also toggle the behavior. 
 - **Label viewer**: This is where you can view your manual labels. The current video frame is denoted with the current frame indicator (**13**). Unlabeled frames will be partially transparent, with the "background" class pre-selected. Labeled frames will be opaque (see image). 
 - **Finalize labels**: When labeling a video, particularly with rare behaviors, the vast majority of frames will be "background". We want to be able to tell the difference between  _this frame is background_  and  _I have not looked at or labeled this frame yet_, so that you can partially label a video and then return to it. By default, when saving the project (**menu bar/file/save**), unlabeled frames are set to  `-1`  and the video is not used for training. When you've fully labeled a video, instead of affirmatively going through every frame and labeling them as  _background_, we will use this button. When you press this button, all unlabeled frames will be set to background, and the video will be considered fully labeled. This video will be used by DeepEthogram for training (**5i, 6i**)

For more information, see [using the GUI docs](https://github.com/jbohnslav/deepethogram/blob/master/docs/using_gui.md).

**Equivalent command line arguments:**

    # we also have a list of label files, created by some other means
    list_of_labels = ['/mnt/DATA/my_other_project/training 1/labels.csv', 
                      '/mnt/DATA/my_other_project/assessment 1/labels.csv',
                      '/mnt/DATA/my_other_project/assessment 2/labels.csv']
    
    for movie_path, label_path in zip(new_list_of_movies, list_of_labels):
        projects.add_label_to_project(label_path, movie_path)

## Download models

Rather than start from scratch, we will start with model weights pretrained on the Kinetics700 dataset. Go to To download the pretrained weights, please use  [this Google Drive link](https://drive.google.com/file/d/1ntIZVbOG1UAiFVlsAAuKEBEVCVevyets/view?usp=sharing). Unzip the files in your  `project/models`  directory. The path should be  `your_project/models/pretrained/{models 1:6}`.

![enter image description here](https://user-images.githubusercontent.com/81632945/120924173-f0e0e800-c704-11eb-9f10-3ad864a2838c.png)

## Train Flow Generator and Feature Extractor models

The flow generator is the model that computes frame-to-frame motion; the feature extractors are the big models that learn to compress the image into a set of low-dimensional vectors (relatively small lists of numbers) that represent the behaviors in the image.
![enter image description here](https://user-images.githubusercontent.com/81632945/120924170-ecb4ca80-c704-11eb-9dd7-82054c461869.png)

## Train Flow Generator
In the flow generator box, click `train`

 - **Train button**: Train the flow generator. It will use hyperparameters from your project configuration file (or defaults specified in  `deepethogram/conf`). See  [using configuration files for details](https://github.com/jbohnslav/deepethogram/blob/master/docs/using_config_files.md). This includes the model architecture (TinyMotionNet, MotionNet, TinyMotionNet3D). The weights to pre-load will be specified by the model selector (**4ii**)
 - **Model selector**: Choose the weights to pre-load. The default model might be TinyMotionNet.
![enter image description here](https://user-images.githubusercontent.com/81632945/120924933-bda05800-c708-11eb-917a-d34b49db9f90.png)

**Equivalent command line arguments:**
To train the flow generator with the larger MotionNet architecture and a batch size of 16 (*you can change the batch size in project_config.yaml, too large batch size might result in errors*):

    `python -m deepethogram.flow_generator.train project.config_file=path/to/project_config.yaml flow_generator.arch=MotionNet compute.batch_size=16`
**Execution results:**

    [2021-06-07 09:55:26,453] INFO [__main__.flow_generator_train:54] args: /home/huangyufei/.conda/envs/deg/lib/python3.7/site-packages/deepethogram/flow_generator/train.py project.config_file=/home/huangyufei/deg/training_deepethogram/project_config.yaml flow_generator.arch=TinyMotionNet compute.batch_size=2
    [2021-06-07 09:55:26,454] INFO [__main__.flow_generator_train:62] configuration used ~~~~~
    [2021-06-07 09:55:26,464] INFO [__main__.flow_generator_train:63] split:
      reload: true
      file: null
      train_val_test:
      - 0.8
      - 0.2
      - 0.0
    compute:
      fp16: false
      num_workers: 8
      batch_size: 2
      min_batch_size: 2
      max_batch_size: 512
      distributed: false
      gpu_id: 0
      dali: false
      metrics_workers: 0
    reload:
      overwrite_cfg: false
      latest: false
    notes: null
    log:
      level: info
    augs:
      brightness: 0.25
      contrast: 0.1
      hue: 0.1
      saturation: 0.1
      color_p: 0.5
      grayscale: 0.5
      crop_size: null
      resize:
      - 224
      - 224
      dali: false
      random_resize: false
      pad: null
      LR: 0.5
      UD: 0.0
      degrees: 10
      normalization:
        'N': 35734348800
        mean:
        - 0.5623775488272594
        - 0.5133215610229026
        - 0.3992611527834769
        std:
        - 0.1138012953521028
        - 0.09838669747141122
        - 0.14414765149547307
    train:
      lr: 0.0001
      scheduler: plateau
      num_epochs: 10
      steps_per_epoch:
        train: 1000
        val: 200
        test: 20
      min_lr: 5.0e-07
      stopping_type: learning_rate
      milestones:
      - 50
      - 100
      - 150
      - 200
      - 250
      - 300
      weight_loss: true
      patience: 3
      early_stopping_begins: 0
      viz_metrics: true
      viz_examples: 10
      reduction_factor: 0.1
      loss_weight_exp: 1.0
      loss_gamma: 1.0
      label_smoothing: 0.05
      oversampling_exp: 0.0
      regularization:
        style: l2_sp
        alpha: 1.0e-05
        beta: 0.001
    flow_generator:
      type: flow_generator
      flow_loss: MotionNet
      flow_max: 10
      input_images: 11
      flow_sparsity: false
      smooth_weight_multiplier: 1.0
      sparsity_weight: 0.0
      loss: MotionNet
      max: 5
      n_rgb: 11
      arch: TinyMotionNet
      weights: pretrained
    cmap: deepethogram
    control_arrow_jump: 31
    label_view_width: 31
    postprocessor:
      min_bout_length: 1
      type: min_bout_per_behavior
    prediction_opacity: 0.2
    project:
      class_names:
      - background
      - scratch
      config_file: /home/huangyufei/deg/training_deepethogram/project_config.yaml
      data_path: /home/huangyufei/deg/training_deepethogram/DATA
      labeler: null
      model_path: /home/huangyufei/deg/training_deepethogram/models
      name: training
      path: /home/huangyufei/deg/training_deepethogram
      pretrained_path: /home/huangyufei/deg/training_deepethogram/models/pretrained_models
    run:
      type: train
      model: flow_generator
      dir: /home/huangyufei/deg/training_deepethogram/models/210607_095526_flow_generator_train
    sequence:
      filter_length: 15
    unlabeled_alpha: 0.1
    vertical_arrow_jump: 3
    
    [2021-06-07 09:55:27,378] INFO [__main__.flow_generator_train:67] Total trainable params: 1,951,784
    [2021-06-07 09:55:27,652] INFO [deepethogram.projects.get_weightfile_from_cfg:1051] loading pretrained weights: /home/huangyufei/deg/training_deepethogram/models/pretrained_models/200221_115158_TinyMotionNet/checkpoint.pt
    reloading weights...
    [2021-06-07 09:55:27,652] INFO [deepethogram.utils.load_state:341] loading from checkpoint file /home/huangyufei/deg/training_deepethogram/models/pretrained_models/200221_115158_TinyMotionNet/checkpoint.pt...
    [2021-06-07 09:55:27,701] INFO [__main__.get_metrics:364] key metric is SSIM
    [2021-06-07 09:55:27,768] INFO [deepethogram.data.augs.get_gpu_transforms:256] GPU transforms: {'train': Sequential(
      (0): ToFloat()
      (1): VideoSequential(
        (0): RandomHorizontalFlip(p=0.5, p_batch=1.0, same_on_batch=False, return_transform=False)
        (1): RandomRotation(degrees=10, interpolation=BILINEAR, p=0.5, p_batch=1.0, same_on_batch=False, return_transform=False)
        (2): ColorJitter(brightness=0.25, contrast=0.1, saturation=0.1, hue=0.1, p=0.5, p_batch=1.0, same_on_batch=False, return_transform=False)
        (3): RandomGrayscale(p=0.5, p_batch=1.0, same_on_batch=False, return_transform=False)
      )
      (2): NormalizeVideo()
      (3): StackClipInChannels()
    ), 'val': Sequential(
      (0): ToFloat()
      (1): NormalizeVideo()
      (2): StackClipInChannels()
    ), 'test': Sequential(
      (0): ToFloat()
      (1): NormalizeVideo()
      (2): StackClipInChannels()
    ), 'denormalize': Sequential(
      (0): UnstackClip()
      (1): DenormalizeVideo()
    )}
    [2021-06-07 09:55:27,770] INFO [deepethogram.base.__init__:89] scheduler mode: min
    [2021-06-07 09:55:27,920] INFO [deepethogram.losses.get_regularization_loss:205] Regularization: L2_SP. Pretrained file: /home/huangyufei/deg/training_deepethogram/models/pretrained_models/200221_115158_TinyMotionNet/checkpoint.pt alpha: 1e-05 beta: 0.001
    [2021-06-07 09:55:27,939] INFO [deepethogram.flow_generator.losses.__init__:179] Using MotionNet Loss with settings: smooth_weights: [0.01, 0.02, 0.04, 0.08, 0.16] flow_sparsity: False sparsity_weight: 0.0
    GPU available: True, used: True
    TPU available: None, using: 0 TPU cores
    LOCAL_RANK: 0 - CUDA_VISIBLE_DEVICES: [0]
    [2021-06-07 09:55:33,630] INFO [deepethogram.base.configure_optimizers:221] learning rate: 0.0001
    
      | Name      | Type          | Params
    --------------------------------------------
    0 | model     | TinyMotionNet | 2.0 M 
    1 | criterion | MotionNetLoss | 0     
    --------------------------------------------
    2.0 M     Trainable params
    0         Non-trainable params
    2.0 M     Total params
    Epoch 0: 100%|█████████████████████████████████████████████████████████████████████████████| 1000/1000 [03:10<00:00,  5.26it/s, loss=0.0278, v_num=0{} {}dating: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████▌| 199/200 [01:06<00:00,  4.19it/s]
    Epoch 1: 100%|████████████████████████████████████████████████████████████████████████████▌| 1194/1200 [04:06<00:01,  4.84it/s, loss=0.0324, v_num=0{} {}dating:  99%|█████████████████████████████████████████████████████████████████████████████████████████████████ | 198/200 [01:05<00:00,  4.52it/s]
    Epoch 2: 100%|████████████████████████████████████████████████████████████████████████████▉| 1199/1200 [04:07<00:00,  4.84it/s, loss=0.0286, v_num=0]{} {}ating:  99%|█████████████████████████████████████████████████████████████████████████████████████████████████ | 198/200 [01:05<00:00,  4.63it/s]
    Epoch 3: 100%|████████████████████████████████████████████████████████████████████████████▉| 1199/1200 [04:06<00:00,  4.87it/s, loss=0.0261, v_num=0]{} {}ating:  99%|█████████████████████████████████████████████████████████████████████████████████████████████████ | 198/200 [01:05<00:00,  4.36it/s]
    Epoch 4: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████▊| 1198/1200 [04:05<00:00,  4.87it/s, loss=0.0279, v_num=0{} {}dating: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 200/200 [01:06<00:00,  4.60it/s]
    Epoch     5: reducing learning rate of group 0 to 1.0000e-05.
    Epoch 5: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 1199/1200 [04:05<00:00,  4.88it/s, loss=0.0273, v_num=0]{} {}ating:  99%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▋ | 198/200 [01:06<00:00,  4.95it/s]
    Epoch 6: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████▌| 1195/1200 [04:06<00:01,  4.84it/s, loss=0.0268, v_num=0{} {}dating:  99%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▋ | 198/200 [01:05<00:00,  4.46it/s]
    Epoch 7: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████▊| 1198/1200 [04:08<00:00,  4.82it/s, loss=0.0249, v_num=0]{} {}ating:  98%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████  | 197/200 [01:06<00:00,  3.90it/s]
    Epoch 8: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████▋| 1197/1200 [04:06<00:00,  4.85it/s, loss=0.0276, v_num=0{} {}dating:  98%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████  | 197/200 [01:05<00:00,  4.87it/s]
    Epoch     9: reducing learning rate of group 0 to 1.0000e-06.
    Epoch 9: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████▍| 1194/1200 [04:06<00:01,  4.85it/s, loss=0.0264, v_num=0{} {}dating:  99%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▋ | 198/200 [01:06<00:00,  5.06it/s]
    Epoch 9: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1200/1200 [04:09<00:00,  4.82it/s, loss=0.0264, v_num=0Saving latest checkpoint...                                                                                                                                                              
    Epoch 9: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1200/1200 [04:09<00:00,  4.81it/s, loss=0.0264, v_num=0]
    

## Train Feature Extractor 
In the feature extractor box, click `train`. After feature extractor training is over, click `infer`

 - **Train button**: Train the feature extractor. It will use hyperparameters from your project configuration file. This will take a long time, perhaps overnight. To speed training potentially at the cost of model performance, set  `feature_extractor/curriculum = false`  in your project configuration file.
-  **Model selector**: A list of models in your  [models directory](https://github.com/jbohnslav/deepethogram/blob/master/docs/file_structure.md). These are the weights that will be loaded and fine-tuned when you train (**5i**) and used to run inference (**5ii**). The default model might be ResNet18 (hidden_two_stream_kinetics_degf)
![enter image description here](https://user-images.githubusercontent.com/81632945/120924929-b9743a80-c708-11eb-8b5c-bde9170c32a8.png)
**Equivalent command line arguments:**
To train the feature extractor with the ResNet18 base, without the curriculum training, with an initial learning rate of 1e-5:    

         python -m deepethogram.feature_extractor.train compute.num_workers=8 project.path=/path/to/project/training_deepethogram flow_generator.weights=latest
         
 **Execution results:**

    [2021-06-07 10:47:12,304] INFO [__main__.feature_extractor_train:66] args: /home/huangyufei/.conda/envs/deg/lib/python3.7/site-packages/deepethogram/feature_extractor/train.py compute.num_workers=2 project.path=/home/huangyufei/deg/training_deepethogram flow_generator.weights=latest
    [2021-06-07 10:47:12,305] INFO [__main__.feature_extractor_train:71] configuration used ~~~~~
    [2021-06-07 10:47:12,316] INFO [__main__.feature_extractor_train:72] split:
      reload: true
      file: null
      train_val_test:
      - 0.8
      - 0.2
      - 0.0
    compute:
      fp16: false
      num_workers: 2
      batch_size: 4
      min_batch_size: 2
      max_batch_size: 512
      distributed: false
      gpu_id: 0
      dali: false
      metrics_workers: 0
    reload:
      overwrite_cfg: false
      latest: false
    notes: null
    log:
      level: info
    augs:
      brightness: 0.25
      contrast: 0.1
      hue: 0.1
      saturation: 0.1
      color_p: 0.5
      grayscale: 0.5
      crop_size: null
      resize:
      - 224
      - 224
      dali: false
      random_resize: false
      pad: null
      LR: 0.5
      UD: 0.0
      degrees: 10
      normalization:
        'N': 35734348800
        mean:
        - 0.5623775488272594
        - 0.5133215610229026
        - 0.3992611527834769
        std:
        - 0.1138012953521028
        - 0.09838669747141122
        - 0.14414765149547307
    train:
      lr: 0.0001
      scheduler: plateau
      num_epochs: 20
      steps_per_epoch:
        train: 1000
        val: 1000
        test: null
      min_lr: 5.0e-07
      stopping_type: learning_rate
      milestones:
      - 50
      - 100
      - 150
      - 200
      - 250
      - 300
      weight_loss: true
      patience: 3
      early_stopping_begins: 0
      viz_metrics: true
      viz_examples: 10
      reduction_factor: 0.1
      loss_weight_exp: 1.0
      loss_gamma: 1.0
      label_smoothing: 0.05
      oversampling_exp: 0.0
      regularization:
        style: l2_sp
        alpha: 1.0e-05
        beta: 0.001
    flow_generator:
      type: flow_generator
      flow_loss: MotionNet
      flow_max: 10
      input_images: 11
      flow_sparsity: false
      smooth_weight_multiplier: 1.0
      sparsity_weight: 0.0
      loss: MotionNet
      max: 5
      n_rgb: 11
      arch: TinyMotionNet
      weights: latest
    feature_extractor:
      arch: resnet18
      fusion: average
      sampler: null
      final_bn: false
      sampling_ratio: null
      final_activation: sigmoid
      dropout_p: 0.25
      n_flows: 10
      n_rgb: 1
      curriculum: false
      inputs: both
      weights: pretrained
    cmap: deepethogram
    control_arrow_jump: 31
    label_view_width: 31
    postprocessor:
      min_bout_length: 1
      type: min_bout_per_behavior
    prediction_opacity: 0.2
    project:
      class_names:
      - background
      - scratch
      config_file: project_config.yaml
      data_path: /home/huangyufei/deg/training_deepethogram/DATA
      labeler: null
      model_path: /home/huangyufei/deg/training_deepethogram/models
      name: training
      path: /home/huangyufei/deg/training_deepethogram
      pretrained_path: /home/huangyufei/deg/training_deepethogram/models/pretrained_models
    run:
      type: train
      model: feature_extractor
      dir: /home/huangyufei/deg/training_deepethogram/models/210607_104712_feature_extractor_train
    sequence:
      filter_length: 15
    unlabeled_alpha: 0.1
    vertical_arrow_jump: 3
    
    [2021-06-07 10:47:12,516] INFO [deepethogram.projects.get_weightfile_from_cfg:1070] loading LATEST weights: /home/huangyufei/deg/training_deepethogram/models/210607_095526_flow_generator_train/lightning_checkpoints/epoch=5-step=5999.ckpt
    [2021-06-07 10:47:12,516] INFO [__main__.feature_extractor_train:79] loading flow generator from file /home/huangyufei/deg/training_deepethogram/models/210607_095526_flow_generator_train/lightning_checkpoints/epoch=5-step=5999.ckpt
    [2021-06-07 10:47:12,516] INFO [deepethogram.utils.load_state:341] loading from checkpoint file /home/huangyufei/deg/training_deepethogram/models/210607_095526_flow_generator_train/lightning_checkpoints/epoch=5-step=5999.ckpt...
    [2021-06-07 10:47:13,742] INFO [deepethogram.data.utils.make_loss_weight:114] Class counts: [115532   1145]
    [2021-06-07 10:47:13,743] INFO [deepethogram.data.utils.make_loss_weight:115] Pos weight: [  0.00991067 100.90131004]
    [2021-06-07 10:47:13,743] INFO [deepethogram.data.utils.make_loss_weight:116] Pos weight, weighted: [  0.00991067 100.90131   ]
    [2021-06-07 10:47:13,743] INFO [deepethogram.data.utils.make_loss_weight:117] Softmax weight: [0.00981342 0.99018658]
    [2021-06-07 10:47:13,743] INFO [deepethogram.data.utils.make_loss_weight:118] Softmax weight transformed: [0.00981342 0.9901866 ]
    [2021-06-07 10:47:13,984] INFO [deepethogram.projects.get_weightfile_from_cfg:1051] loading pretrained weights: /home/huangyufei/deg/training_deepethogram/models/pretrained_models/200415_125824_hidden_two_stream_kinetics_degf/checkpoint.pt
    [2021-06-07 10:47:13,985] INFO [__main__.build_model_from_cfg:220] feature extractor weightfile: /home/huangyufei/deg/training_deepethogram/models/pretrained_models/200415_125824_hidden_two_stream_kinetics_degf/checkpoint.pt
    [2021-06-07 10:47:14,238] INFO [deepethogram.feature_extractor.models.CNN.get_cnn:91] Custom bias: Parameter containing:
    tensor([ 4.6141, -4.6141], requires_grad=True)
    [2021-06-07 10:47:14,238] INFO [deepethogram.utils.load_feature_extractor_components:630] loading component spatial from file /home/huangyufei/deg/training_deepethogram/models/pretrained_models/200415_125824_hidden_two_stream_kinetics_degf/checkpoint.pt
    [2021-06-07 10:47:14,311] WARNING [deepethogram.utils.load_state_from_dict:249] 1.weight has different size: pretrained:torch.Size([700, 512]) model:torch.Size([2, 512])
    [2021-06-07 10:47:14,311] WARNING [deepethogram.utils.load_state_from_dict:249] 1.bias has different size: pretrained:torch.Size([700]) model:torch.Size([2])
    [2021-06-07 10:47:14,582] INFO [deepethogram.feature_extractor.models.CNN.get_cnn:91] Custom bias: Parameter containing:
    tensor([ 4.6141, -4.6141], requires_grad=True)
    [2021-06-07 10:47:14,582] INFO [deepethogram.utils.load_feature_extractor_components:630] loading component flow from file /home/huangyufei/deg/training_deepethogram/models/pretrained_models/200415_125824_hidden_two_stream_kinetics_degf/checkpoint.pt
    [2021-06-07 10:47:14,658] WARNING [deepethogram.utils.load_state_from_dict:249] 1.weight has different size: pretrained:torch.Size([700, 512]) model:torch.Size([2, 512])
    [2021-06-07 10:47:14,658] WARNING [deepethogram.utils.load_state_from_dict:249] 1.bias has different size: pretrained:torch.Size([700]) model:torch.Size([2])
    [2021-06-07 10:47:14,877] INFO [deepethogram.projects.get_weightfile_from_cfg:1070] loading LATEST weights: /home/huangyufei/deg/training_deepethogram/models/210607_095526_flow_generator_train/lightning_checkpoints/epoch=5-step=5999.ckpt
    [2021-06-07 10:47:14,878] INFO [deepethogram.utils.load_state:341] loading from checkpoint file /home/huangyufei/deg/training_deepethogram/models/210607_095526_flow_generator_train/lightning_checkpoints/epoch=5-step=5999.ckpt...
    [2021-06-07 10:47:14,892] INFO [deepethogram.utils.load_feature_extractor_components:630] loading component fusion from file /home/huangyufei/deg/training_deepethogram/models/pretrained_models/200415_125824_hidden_two_stream_kinetics_degf/checkpoint.pt
    [2021-06-07 10:47:14,944] INFO [__main__.get_metrics:631] key metric: f1_class_mean_nobg
    [2021-06-07 10:47:15,822] INFO [deepethogram.data.utils.make_loss_weight:114] Class counts: [115532   1145]
    [2021-06-07 10:47:15,823] INFO [deepethogram.data.utils.make_loss_weight:115] Pos weight: [  0.00991067 100.90131004]
    [2021-06-07 10:47:15,823] INFO [deepethogram.data.utils.make_loss_weight:116] Pos weight, weighted: [  0.00991067 100.90131   ]
    [2021-06-07 10:47:15,824] INFO [deepethogram.data.utils.make_loss_weight:117] Softmax weight: [0.00981342 0.99018658]
    [2021-06-07 10:47:15,824] INFO [deepethogram.data.utils.make_loss_weight:118] Softmax weight transformed: [0.00981342 0.9901866 ]
    [2021-06-07 10:47:15,826] INFO [deepethogram.feature_extractor.losses.__init__:96] Focal loss: gamma 1.00 smoothing: 0.05
    [2021-06-07 10:47:15,900] INFO [deepethogram.losses.get_regularization_loss:205] Regularization: L2_SP. Pretrained file: /home/huangyufei/deg/training_deepethogram/models/pretrained_models/200415_125824_hidden_two_stream_kinetics_degf/checkpoint.pt alpha: 1e-05 beta: 0.001
    [2021-06-07 10:47:15,978] INFO [deepethogram.data.augs.get_gpu_transforms:256] GPU transforms: {'train': Sequential(
      (0): ToFloat()
      (1): VideoSequential(
        (0): RandomHorizontalFlip(p=0.5, p_batch=1.0, same_on_batch=False, return_transform=False)
        (1): RandomRotation(degrees=10, interpolation=BILINEAR, p=0.5, p_batch=1.0, same_on_batch=False, return_transform=False)
        (2): ColorJitter(brightness=0.25, contrast=0.1, saturation=0.1, hue=0.1, p=0.5, p_batch=1.0, same_on_batch=False, return_transform=False)
        (3): RandomGrayscale(p=0.5, p_batch=1.0, same_on_batch=False, return_transform=False)
      )
      (2): NormalizeVideo()
      (3): StackClipInChannels()
    ), 'val': Sequential(
      (0): ToFloat()
      (1): NormalizeVideo()
      (2): StackClipInChannels()
    ), 'test': Sequential(
      (0): ToFloat()
      (1): NormalizeVideo()
      (2): StackClipInChannels()
    ), 'denormalize': Sequential(
      (0): UnstackClip()
      (1): DenormalizeVideo()
    )}
    [2021-06-07 10:47:15,979] INFO [deepethogram.base.__init__:89] scheduler mode: max
    [2021-06-07 10:47:15,979] INFO [deepethogram.base.get_train_sampler:165] not using oversampling
    GPU available: True, used: True
    TPU available: None, using: 0 TPU cores
    LOCAL_RANK: 0 - CUDA_VISIBLE_DEVICES: [0]
    [2021-06-07 10:47:22,288] INFO [deepethogram.base.configure_optimizers:221] learning rate: 0.0001
    
      | Name       | Type               | Params
    --------------------------------------------------
    0 | model      | HiddenTwoStream    | 24.4 M
    1 | activation | Sigmoid            | 0     
    2 | criterion  | ClassificationLoss | 0     
    --------------------------------------------------
    22.4 M    Trainable params
    2.0 M     Non-trainable params
    24.4 M    Total params
    Epoch 0: 100%|███████████████████████████████████████████████████████████████████████████████| 1000/1000 [05:09<00:00,  3.23it/s, loss=1.28, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [06:52<00:00,  2.23it/s]
    Epoch 1: 100%|██████████████████████████████████████████████████████████████████████████████▉| 1999/2000 [12:17<00:00,  2.71it/s, loss=1.25, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████▉| 999/1000 [06:42<00:00,  1.60it/s]
    Epoch 2: 100%|██████████████████████████████████████████████████████████████████████████████▉| 1999/2000 [11:34<00:00,  2.88it/s, loss=1.61, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [06:26<00:00,  2.84it/s]
    Epoch 3: 100%|█████████████████████████████████████████████████████████████████████████████▉| 1999/2000 [11:33<00:00,  2.88it/s, loss=0.758, v_num=0]{} {}ating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████▊| 998/1000 [06:26<00:00,  2.35it/s]
    Epoch 4: 100%|█████████████████████████████████████████████████████████████████████████████▉| 1998/2000 [11:39<00:00,  2.86it/s, loss=0.849, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [06:34<00:00,  2.46it/s]
    Epoch     5: reducing learning rate of group 0 to 1.0000e-05.
    Epoch 5: 100%|███████████████████████████████████████████████████████████████████████████████▉| 1999/2000 [11:46<00:00,  2.83it/s, loss=0.9, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [06:26<00:00,  3.64it/s]
    Epoch 6: 100%|█████████████████████████████████████████████████████████████████████████████▉| 1999/2000 [11:38<00:00,  2.86it/s, loss=0.809, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [06:24<00:00,  2.63it/s]
    Epoch 7: 100%|█████████████████████████████████████████████████████████████████████████████▉| 1999/2000 [11:45<00:00,  2.83it/s, loss=0.698, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [06:27<00:00,  3.30it/s]
    Epoch 8: 100%|██████████████████████████████████████████████████████████████████████████████| 2000/2000 [11:33<00:00,  2.88it/s, loss=0.843, v_num=0]{} {}ating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████▉| 999/1000 [06:21<00:00,  2.44it/s]
    Epoch 9: 100%|█████████████████████████████████████████████████████████████████████████████▉| 1999/2000 [11:43<00:00,  2.84it/s, loss=0.761, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [06:28<00:00,  3.54it/s]
    Epoch    10: reducing learning rate of group 0 to 1.0000e-06.
    Epoch 10: 100%|████████████████████████████████████████████████████████████████████████████▉| 1998/2000 [11:41<00:00,  2.85it/s, loss=0.778, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [06:29<00:00,  2.35it/s]
    Epoch 11: 100%|████████████████████████████████████████████████████████████████████████████▉| 1999/2000 [11:44<00:00,  2.84it/s, loss=0.696, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [06:30<00:00,  2.38it/s]
    Epoch 12: 100%|████████████████████████████████████████████████████████████████████████████▉| 1998/2000 [11:39<00:00,  2.86it/s, loss=0.777, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████▉| 999/1000 [06:24<00:00,  3.03it/s]
    Epoch 13: 100%|████████████████████████████████████████████████████████████████████████████▉| 1999/2000 [11:28<00:00,  2.90it/s, loss=0.746, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████▉| 999/1000 [06:25<00:00,  2.06it/s]
    Epoch    14: reducing learning rate of group 0 to 5.0000e-07.
    Epoch 14:  50%|██████████████████████████████████████▌                                      | 1000/2000 [05:08<05:08,  3.24it/s, loss=0.692, v_num=0]Reached learning rate 5e-07, stopping...                                                                                                             
    [2021-06-07 13:44:29,463] INFO [deepethogram.callbacks.on_train_epoch_end:248] Stopping criterion reached! setting trainer.should_stop=True
    Epoch 14: 100%|████████████████████████████████████████████████████████████████████████████▉| 1999/2000 [11:32<00:00,  2.89it/s, loss=0.692, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [06:23<00:00,  3.50it/s]
    Epoch 14: 100%|█████████████████████████████████████████████████████████████████████████████| 2000/2000 [11:35<00:00,  2.87it/s, loss=0.692, v_num=0Saving latest checkpoint...                                                                                                                           
    Epoch 14: 100%|█████████████████████████████████████████████████████████████████████████████| 2000/2000 [11:35<00:00,  2.87it/s, loss=0.692, v_num=0] 

## Run inference 
 - **Inference button**: Run inference using the feature extractor models. Clicking this button will open a list of videos in your project with check boxes. Videos without output files ([see file structure](https://github.com/jbohnslav/deepethogram/blob/master/docs/file_structure.md)) will be pre-selected. For each video you select, the feature extractors will run frame-by-frame and extract spatial features, flow features, and predictions and save them to disk. These will be loaded as inputs to the sequence models (below). This may take a long time, as inference takes around 30-60FPS depending on video resolution and model complexity.
 
![enter image description here](https://user-images.githubusercontent.com/81632945/120924923-b1b49600-c708-11eb-91b6-68f7737df4e5.png)
**Equivalent command line arguments:**

      python -m deepethogram.feature_extractor.inference feature_extractor.weights=latest flow_generator.weights=latest inference.overwrite=True inference.ignore_error=False compute.num_workers=8 project.path=/path/to/config/training_deepethogram
 **Execution results:**

    [2021-06-07 16:09:57,756] INFO [__main__.feature_extractor_inference:474] args: /home/huangyufei/.conda/envs/deg/lib/python3.7/site-packages/deepethogram/feature_extractor/inference.py feature_extractor.weights=latest flow_generator.weights=latest inference.overwrite=True inference.ignore_error=False compute.num_workers=8 project.path=/home/huangyufei/deg/training_deepethogram
    [2021-06-07 16:09:57,757] INFO [__main__.feature_extractor_inference:476] configuration used in inference: 
    [2021-06-07 16:09:57,766] INFO [__main__.feature_extractor_inference:477] split:
      reload: true
      file: null
      train_val_test:
      - 0.8
      - 0.2
      - 0.0
    compute:
      fp16: false
      num_workers: 8
      batch_size: 4
      min_batch_size: 2
      max_batch_size: 512
      distributed: false
      gpu_id: 0
      dali: false
      metrics_workers: 0
    reload:
      overwrite_cfg: false
      latest: false
    notes: null
    log:
      level: info
    augs:
      brightness: 0.25
      contrast: 0.1
      hue: 0.1
      saturation: 0.1
      color_p: 0.5
      grayscale: 0.5
      crop_size: null
      resize:
      - 224
      - 224
      dali: false
      random_resize: false
      pad: null
      LR: 0.5
      UD: 0.0
      degrees: 10
      normalization:
        'N': 35734348800
        mean:
        - 0.5623775488272594
        - 0.5133215610229026
        - 0.3992611527834769
        std:
        - 0.1138012953521028
        - 0.09838669747141122
        - 0.14414765149547307
    feature_extractor:
      arch: resnet18
      fusion: average
      sampler: null
      final_bn: false
      sampling_ratio: null
      final_activation: sigmoid
      dropout_p: 0.25
      n_flows: 10
      n_rgb: 1
      curriculum: false
      inputs: both
      weights: latest
    train:
      steps_per_epoch:
        train: 1000
        val: 200
        test: 20
      num_epochs: 10
      loss_weight_exp: 1.0
    flow_generator:
      type: flow_generator
      flow_loss: MotionNet
      flow_max: 10
      input_images: 11
      flow_sparsity: false
      smooth_weight_multiplier: 1.0
      sparsity_weight: 0.0
      loss: MotionNet
      max: 5
      n_rgb: 11
      arch: TinyMotionNet
      weights: latest
    inference:
      directory_list: all
      ignore_error: false
      overwrite: true
      use_loaded_model_cfg: true
    postprocessor:
      type: min_bout_per_behavior
      min_bout_length: 1
    cmap: deepethogram
    control_arrow_jump: 31
    label_view_width: 31
    prediction_opacity: 0.2
    project:
      class_names:
      - background
      - scratch
      config_file: project_config.yaml
      data_path: /home/huangyufei/deg/training_deepethogram/DATA
      labeler: null
      model_path: /home/huangyufei/deg/training_deepethogram/models
      name: training
      path: /home/huangyufei/deg/training_deepethogram
      pretrained_path: /home/huangyufei/deg/training_deepethogram/models/pretrained_models
    run:
      type: inference
      model: feature_extractor
      dir: /home/huangyufei/deg/training_deepethogram/models/210607_160957_feature_extractor_inference
    sequence:
      filter_length: 15
    unlabeled_alpha: 0.1
    vertical_arrow_jump: 3
    
    [2021-06-07 16:09:57,767] INFO [__main__.feature_extractor_inference:482] Latent name used in HDF5 file: resnet18
    [2021-06-07 16:09:57,778] INFO [deepethogram.data.augs.get_gpu_transforms:256] GPU transforms: {'train': Sequential(
      (0): ToFloat()
      (1): VideoSequential(
        (0): RandomHorizontalFlip(p=0.5, p_batch=1.0, same_on_batch=False, return_transform=False)
        (1): RandomRotation(degrees=10, interpolation=BILINEAR, p=0.5, p_batch=1.0, same_on_batch=False, return_transform=False)
        (2): ColorJitter(brightness=0.25, contrast=0.1, saturation=0.1, hue=0.1, p=0.5, p_batch=1.0, same_on_batch=False, return_transform=False)
        (3): RandomGrayscale(p=0.5, p_batch=1.0, same_on_batch=False, return_transform=False)
      )
      (2): NormalizeVideo()
      (3): StackClipInChannels()
    ), 'val': Sequential(
      (0): ToFloat()
      (1): NormalizeVideo()
      (2): StackClipInChannels()
    ), 'test': Sequential(
      (0): ToFloat()
      (1): NormalizeVideo()
      (2): StackClipInChannels()
    ), 'denormalize': Sequential(
      (0): UnstackClip()
      (1): DenormalizeVideo()
    )}
    [2021-06-07 16:09:57,779] INFO [__main__.feature_extractor_inference:516] gpu_transform: Sequential(
      (0): ToFloat()
      (1): NormalizeVideo()
      (2): StackClipInChannels()
    )
    [2021-06-07 16:09:57,984] INFO [deepethogram.projects.get_weightfile_from_cfg:1070] loading LATEST weights: /home/huangyufei/deg/training_deepethogram/models/210607_105404_feature_extractor_train/lightning_checkpoints/epoch=5-step=5999.ckpt
    [2021-06-07 16:09:58,246] INFO [deepethogram.projects.get_weightfile_from_cfg:1062] loading specified weights: /home/huangyufei/deg/training_deepethogram/models/210607_105404_feature_extractor_train/lightning_checkpoints/epoch=5-step=5999.ckpt
    [2021-06-07 16:09:58,246] INFO [deepethogram.feature_extractor.train.build_model_from_cfg:220] feature extractor weightfile: /home/huangyufei/deg/training_deepethogram/models/210607_105404_feature_extractor_train/lightning_checkpoints/epoch=5-step=5999.ckpt
    [2021-06-07 16:09:58,508] INFO [deepethogram.utils.load_feature_extractor_components:630] loading component spatial from file /home/huangyufei/deg/training_deepethogram/models/210607_105404_feature_extractor_train/lightning_checkpoints/epoch=5-step=5999.ckpt
    [2021-06-07 16:09:59,004] INFO [deepethogram.utils.load_feature_extractor_components:630] loading component flow from file /home/huangyufei/deg/training_deepethogram/models/210607_105404_feature_extractor_train/lightning_checkpoints/epoch=5-step=5999.ckpt
    [2021-06-07 16:09:59,470] INFO [deepethogram.projects.get_weightfile_from_cfg:1070] loading LATEST weights: /home/huangyufei/deg/training_deepethogram/models/210607_095526_flow_generator_train/lightning_checkpoints/epoch=5-step=5999.ckpt
    [2021-06-07 16:09:59,470] INFO [deepethogram.utils.load_state:341] loading from checkpoint file /home/huangyufei/deg/training_deepethogram/models/210607_095526_flow_generator_train/lightning_checkpoints/epoch=5-step=5999.ckpt...
    [2021-06-07 16:09:59,485] INFO [deepethogram.utils.load_feature_extractor_components:630] loading component fusion from file /home/huangyufei/deg/training_deepethogram/models/210607_105404_feature_extractor_train/lightning_checkpoints/epoch=5-step=5999.ckpt
    [2021-06-07 16:09:59,592] INFO [__main__.feature_extractor_inference:546] best epoch from loaded file: 5
    [2021-06-07 16:09:59,597] INFO [__main__.feature_extractor_inference:553] thresholds: [0.01       0.02019849]
    [2021-06-07 16:10:05,771] INFO [__main__.extract:383] inference batch size: 4
      0%|                                                                                                                          | 0/3 [00:00<?, ?it/s[2021-06-07 16:10:07,954] INFO [__main__.print_debug_statement:128] images shape: torch.Size([4, 33, 224, 224])             | 0/14825 [00:00<?, ?it/s]
    [2021-06-07 16:10:07,955] INFO [__main__.print_debug_statement:129] logits shape: torch.Size([4, 2])
    [2021-06-07 16:10:07,955] INFO [__main__.print_debug_statement:130] spatial_features shape: torch.Size([4, 512])
    [2021-06-07 16:10:07,955] INFO [__main__.print_debug_statement:131] flow_features shape: torch.Size([4, 512])
    [2021-06-07 16:10:07,956] INFO [__main__.print_debug_statement:133] spatial: min 0.0 mean 0.5858678221702576 max 2.509967088699341 shape torch.Size([4, 512])
    [2021-06-07 16:10:07,956] INFO [__main__.print_debug_statement:135] flow   : min 0.0 mean 1.200002670288086 max 9.126016616821289 shape torch.Size([4, 512])
    [2021-06-07 16:10:07,960] INFO [__main__.print_debug_statement:144] channel min:  tensor([-0.0140, -0.0358, -0.0221, -0.0140, -0.0358, -0.0221, -0.0140, -0.0358,
            -0.0221, -0.0140, -0.0358, -0.0221, -0.0140, -0.0358, -0.0221, -4.7350,
            -5.2174, -2.7698, -4.7695, -5.2174, -2.7698, -4.7695, -5.2174, -2.7698,
            -4.7695, -5.2174, -2.7698, -4.8039, -5.2174, -2.7698, -4.8039, -5.2174,
            -2.7698], device='cuda:0')
    [2021-06-07 16:10:07,962] INFO [__main__.print_debug_statement:145] channel mean: tensor([-0.0140, -0.0358, -0.0221, -0.0140, -0.0358, -0.0221, -0.0140, -0.0358,
            -0.0221, -0.0140, -0.0358, -0.0221, -0.0140, -0.0358, -0.0221,  0.2550,
             0.1441, -0.1613,  0.2594,  0.1467, -0.1589,  0.2579,  0.1478, -0.1613,
             0.2654,  0.1534, -0.1563,  0.2598,  0.1523, -0.1600,  0.2667,  0.1559,
            -0.1547], device='cuda:0')
    [2021-06-07 16:10:07,965] INFO [__main__.print_debug_statement:146] channel max : tensor([-0.0140, -0.0358, -0.0221, -0.0140, -0.0358, -0.0221, -0.0140, -0.0358,
            -0.0221, -0.0140, -0.0358, -0.0221, -0.0140, -0.0358, -0.0221,  2.0880,
             2.1565,  1.7191,  2.0880,  2.1565,  1.6646,  2.1225,  2.1963,  1.6646,
             2.2259,  2.3159,  1.6646,  2.0880,  2.1565,  1.6646,  2.0880,  2.1963,
             1.6646], device='cuda:0')
    [2021-06-07 16:10:07,967] INFO [__main__.print_debug_statement:147] channel std : tensor([0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000,
            0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.8353, 0.9088, 0.8733,
            0.8364, 0.9073, 0.8747, 0.8369, 0.9078, 0.8760, 0.8373, 0.9097, 0.8787,
            0.8376, 0.9089, 0.8770, 0.8376, 0.9092, 0.8799], device='cuda:0')
                                                                                                                                                        [2021-06-07 16:17:43,160] INFO [__main__.extract:416] probabilities: torch.Size([59297, 2])                                                           
    [2021-06-07 16:17:43,160] INFO [__main__.extract:416] logits: torch.Size([59297, 2])
    [2021-06-07 16:17:43,160] INFO [__main__.extract:416] spatial_features: torch.Size([59297, 512])
    [2021-06-07 16:17:43,160] INFO [__main__.extract:416] flow_features: torch.Size([59297, 512])
    [2021-06-07 16:17:43,161] INFO [__main__.extract:416] debug: torch.Size([59297])
    [2021-06-07 16:17:43,259] INFO [__main__.extract:438] macro F1: 0.21008494869073102
     33%|█████████████████████████████████████▋                                                                           | 1/3 [07:37<15:15, 457.72s/it[2021-06-07 16:24:54,395] INFO [__main__.extract:438] macro F1: 0.5345990241155272                                                                    
     67%|███████████████████████████████████████████████████████████████████████████▎                                     | 2/3 [14:48<07:22, 442.10s/it[2021-06-07 16:32:15,058] INFO [__main__.extract:438] macro F1: 0.3507124144354564                                                                    
    100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 3/3 [22:09<00:00, 443.18s/it]

## Train a sequence model
In the sequence box, click `train`, After sequence model training is over, click `infer`

- **Train button**: Train the sequence model. It will use hyperparameters from your project configuration file.
(*There shouldn't be any sequence models by default; each sequence model will be trained from scratch. These buttons will un-gray-out when you've trained a feature extractor, and run inference; that saves the features to disk, which are the inputs to the sequence model.*)

![enter image description here](https://user-images.githubusercontent.com/81632945/120924939-c2650c00-c708-11eb-9ac4-614acbf0f28a.png)
**Equivalent command line arguments:**

      python -m deepethogram.sequence.train project.config_file=/path/to/config/project_config.yaml compute.batch_size=16
      
**Execution results:**  

    [2021-06-05 19:07:21,110] INFO [__main__.sequence_train:43] args: /home/huangyufei/.conda/envs/deg/lib/python3.7/site-packages/deepethogram/sequence/train.py project.config_file=/home/huangyufei/deepethogram/deepethogram/training_deepethogram/project_config.yaml compute.batch_size=4
    [2021-06-05 19:07:21,111] INFO [__main__.sequence_train:49] Configuration used: 
    [2021-06-05 19:07:21,123] INFO [__main__.sequence_train:50] split:
      reload: true
      file: null
      train_val_test:
      - 0.8
      - 0.2
      - 0.0
    compute:
      fp16: false
      num_workers: 8
      batch_size: 4
      min_batch_size: 8
      max_batch_size: 512
      distributed: false
      gpu_id: 0
      dali: false
      metrics_workers: 0
    reload:
      overwrite_cfg: false
      latest: false
    notes: null
    log:
      level: info
    feature_extractor:
      arch: resnet18
      fusion: average
      sampler: null
      final_bn: false
      sampling_ratio: null
      final_activation: sigmoid
      dropout_p: 0.25
      n_flows: 10
      n_rgb: 1
      curriculum: false
      inputs: both
      weights: pretrained
    train:
      steps_per_epoch:
        train: 1000
        val: 1000
        test: 20
      num_epochs: 100
      lr: 0.0001
      scheduler: plateau
      min_lr: 5.0e-07
      stopping_type: learning_rate
      milestones:
      - 50
      - 100
      - 150
      - 200
      - 250
      - 300
      weight_loss: true
      patience: 5
      early_stopping_begins: 0
      viz_metrics: true
      viz_examples: 10
      reduction_factor: 0.1
      loss_weight_exp: 1.0
      loss_gamma: 1.0
      label_smoothing: 0.05
      oversampling_exp: 0.0
      regularization:
        style: l2
        alpha: 0.01
        beta: 0.001
    sequence:
      arch: tgmj
      latent_name: resnet18
      output_name: null
      sequence_length: 180
      dropout_p: 0.5
      num_layers: 3
      hidden_size: 64
      nonoverlapping: true
      filter_length: 15
      input_dropout: 0.5
      n_filters: 8
      tgm_reduction: max
      c_in: 1
      c_out: 8
      soft_attn: true
      num_features: 128
      bidirectional: false
      rnn_style: lstm
      hidden_dropout: 0.0
      weights: null
      nonlinear_classification: true
      final_bn: true
      input_type: features
    augs:
      LR: 0.5
      UD: 0.0
      brightness: 0.25
      contrast: 0.1
      crop_size: null
      degrees: 10
      grayscale: 0.5
      hue: 0.1
      normalization:
        'N': 35734348800
        mean:
        - 0.5623775488272594
        - 0.5133215610229026
        - 0.3992611527834769
        std:
        - 0.1138012953521028
        - 0.09838669747141122
        - 0.14414765149547307
      pad: null
      random_resize: false
      resize:
      - 224
      - 224
      saturation: 0.1
    cmap: deepethogram
    control_arrow_jump: 31
    label_view_width: 31
    postprocessor:
      min_bout_length: 1
      type: min_bout_per_behavior
    prediction_opacity: 0.2
    project:
      class_names:
      - background
      - scratch
      config_file: /home/huangyufei/deepethogram/deepethogram/training_deepethogram/project_config.yaml
      data_path: /home/huangyufei/deepethogram/deepethogram/training_deepethogram/DATA
      labeler: null
      model_path: /home/huangyufei/deepethogram/deepethogram/training_deepethogram/models
      name: training
      path: /home/huangyufei/deepethogram/deepethogram/training_deepethogram
      pretrained_path: /home/huangyufei/deepethogram/deepethogram/training_deepethogram/pretrained_models
    run:
      type: train
      model: sequence
      dir: /home/huangyufei/deepethogram/deepethogram/training_deepethogram/models/210605_190721_sequence_train
    unlabeled_alpha: 0.1
    vertical_arrow_jump: 3
    
    [2021-06-05 19:07:22,208] INFO [deepethogram.data.utils.make_loss_weight:114] Class counts: [115532   1145]
    [2021-06-05 19:07:22,208] INFO [deepethogram.data.utils.make_loss_weight:115] Pos weight: [  0.00991067 100.90131004]
    [2021-06-05 19:07:22,208] INFO [deepethogram.data.utils.make_loss_weight:116] Pos weight, weighted: [  0.00991067 100.90131   ]
    [2021-06-05 19:07:22,209] INFO [deepethogram.data.utils.make_loss_weight:117] Softmax weight: [0.00981342 0.99018658]
    [2021-06-05 19:07:22,209] INFO [deepethogram.data.utils.make_loss_weight:118] Softmax weight transformed: [0.00981342 0.9901866 ]
    TGMJ(
      (input_dropout): Dropout(p=0.5, inplace=False)
      (output_dropout): Dropout(p=0.5, inplace=False)
      (tgm_layers): Sequential(
        (0): TGMLayer()
        (1): TGMLayer()
        (2): TGMLayer()
      )
      (h): Conv1d(1024, 128, kernel_size=(1,), stride=(1,))
      (h2): Conv1d(1024, 128, kernel_size=(1,), stride=(1,))
      (classify1): Sequential(
        (0): Conv1d(128, 2, kernel_size=(1,), stride=(1,), bias=False)
        (1): BatchNorm1d(2, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
      )
      (classify2): Sequential(
        (0): Conv1d(128, 2, kernel_size=(1,), stride=(1,), bias=False)
        (1): BatchNorm1d(2, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
      )
    )
    [2021-06-05 19:07:22,571] WARNING [deepethogram.projects.get_weightfile_from_cfg:1073] no sequence weights found...
    [2021-06-05 19:07:22,572] INFO [__main__.sequence_train:63] Total trainable params: 264,206
    [2021-06-05 19:07:22,573] INFO [deepethogram.feature_extractor.train.get_metrics:631] key metric: f1_class_mean
    [2021-06-05 19:07:22,576] INFO [deepethogram.feature_extractor.losses.__init__:96] Focal loss: gamma 1.00 smoothing: 0.05
    [2021-06-05 19:07:22,577] INFO [deepethogram.losses.get_regularization_loss:188] Regularization: L2. alpha: 0.01 
    [2021-06-05 19:07:22,596] INFO [deepethogram.base.__init__:84] scheduler mode: max
    GPU available: True, used: True
    TPU available: None, using: 0 TPU cores
    LOCAL_RANK: 0 - CUDA_VISIBLE_DEVICES: [0]
    [2021-06-05 19:07:28,324] INFO [deepethogram.base.configure_optimizers:216] learning rate: 0.0001
    
      | Name       | Type               | Params
    --------------------------------------------------
    0 | model      | TGMJ               | 264 K 
    1 | activation | Sigmoid            | 0     
    2 | criterion  | ClassificationLoss | 0     
    --------------------------------------------------
    264 K     Trainable params
    0         Non-trainable params
    264 K     Total params
    Epoch 0: 100%|██████████████████████████████████████████████████████████████████████████████████| 163/163 [00:30<00:00,  5.34it/s, loss=732, v_num=0{} {}dating:  96%|████████████████████████████████████████████████████████████████████████████████████████████████▏   | 75/78 [00:12<00:00, 32.03it/s]
    Epoch 1:  99%|█████████████████████████████████████████████████████████████████████████████████▎| 239/241 [00:28<00:00,  8.33it/s, loss=687, v_num=0]{} {}ating:  95%|██████████████████████████████████████████████████████████████████████████████████████████████▊     | 74/78 [00:02<00:00, 33.92it/s]
    Epoch 2: 100%|█████████████████████████████████████████████████████████████████████████████████▋| 240/241 [00:29<00:00,  8.19it/s, loss=413, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████| 78/78 [00:03<00:00, 30.49it/s]
    Epoch 3:  99%|█████████████████████████████████████████████████████████████████████████████████▎| 239/241 [00:29<00:00,  8.15it/s, loss=955, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████| 78/78 [00:02<00:00, 33.39it/s]
    Epoch 4: 100%|█████████████████████████████████████████████████████████████████████████████████▋| 240/241 [00:29<00:00,  8.14it/s, loss=589, v_num=0{} {}dating:  99%|██████████████████████████████████████████████████████████████████████████████████████████████████▋ | 77/78 [00:03<00:00, 36.54it/s]
    Epoch 5: 100%|█████████████████████████████████████████████████████████████████████████████████▋| 240/241 [00:29<00:00,  8.16it/s, loss=427, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████| 78/78 [00:03<00:00, 33.88it/s]
    Epoch 6: 100%|█████████████████████████████████████████████████████████████████████████████████▋| 240/241 [00:30<00:00,  7.99it/s, loss=695, v_num=0]{} {}ating:  96%|████████████████████████████████████████████████████████████████████████████████████████████████▏   | 75/78 [00:02<00:00, 30.92it/s]
    Epoch     7: reducing learning rate of group 0 to 1.0000e-05.
    Epoch 7: 100%|█████████████████████████████████████████████████████████████████████████████████▋| 240/241 [00:29<00:00,  8.23it/s, loss=526, v_num=0]{} {}ating:  96%|████████████████████████████████████████████████████████████████████████████████████████████████▏   | 75/78 [00:02<00:00, 29.47it/s]
    Epoch 8: 100%|█████████████████████████████████████████████████████████████████████████████████▋| 240/241 [00:29<00:00,  8.23it/s, loss=553, v_num=0]{} {}ating:  96%|████████████████████████████████████████████████████████████████████████████████████████████████▏   | 75/78 [00:02<00:00, 33.54it/s]
    Epoch 9: 100%|█████████████████████████████████████████████████████████████████████████████████▋| 240/241 [00:28<00:00,  8.34it/s, loss=789, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████| 78/78 [00:02<00:00, 33.27it/s]
    Epoch 10:  99%|████████████████████████████████████████████████████████████████████████████████▎| 239/241 [00:48<00:00,  4.88it/s, loss=986, v_num=0{} {}dating:  97%|█████████████████████████████████████████████████████████████████████████████████████████████████▍  | 76/78 [00:12<00:00, 37.34it/s]
    Epoch 11: 100%|████████████████████████████████████████████████████████████████████████████████▋| 240/241 [00:29<00:00,  8.20it/s, loss=432, v_num=0]{} {}ating:  97%|█████████████████████████████████████████████████████████████████████████████████████████████████▍  | 76/78 [00:03<00:00, 33.00it/s]
    Epoch 12: 100%|████████████████████████████████████████████████████████████████████████████████▋| 240/241 [00:29<00:00,  8.20it/s, loss=383, v_num=0]{} {}ating:  97%|█████████████████████████████████████████████████████████████████████████████████████████████████▍  | 76/78 [00:02<00:00, 35.09it/s]
    Epoch    13: reducing learning rate of group 0 to 1.0000e-06.
    Epoch 13: 100%|████████████████████████████████████████████████████████████████████████████████▋| 240/241 [00:29<00:00,  8.20it/s, loss=852, v_num=0]{} {}ating:  96%|████████████████████████████████████████████████████████████████████████████████████████████████▏   | 75/78 [00:03<00:00, 31.23it/s]
    Epoch 14: 100%|████████████████████████████████████████████████████████████████████████████████▋| 240/241 [00:29<00:00,  8.16it/s, loss=649, v_num=0]{} {}ating:  96%|████████████████████████████████████████████████████████████████████████████████████████████████▏   | 75/78 [00:02<00:00, 29.45it/s]
    Epoch 15: 100%|████████████████████████████████████████████████████████████████████████████████▋| 240/241 [00:29<00:00,  8.14it/s, loss=512, v_num=0]{} {}ating:  97%|█████████████████████████████████████████████████████████████████████████████████████████████████▍  | 76/78 [00:02<00:00, 30.52it/s]
    Epoch 16: 100%|███████████████████████████████████████████████████████████████████████████▋| 240/241 [00:29<00:00,  8.15it/s, loss=1.17e+03, v_num=0{} {}dating:  99%|██████████████████████████████████████████████████████████████████████████████████████████████████▋ | 77/78 [00:03<00:00, 34.47it/s]
    Epoch 17: 100%|████████████████████████████████████████████████████████████████████████████████▋| 240/241 [00:29<00:00,  8.12it/s, loss=720, v_num=0{} {}dating:  99%|██████████████████████████████████████████████████████████████████████████████████████████████████▋ | 77/78 [00:03<00:00, 35.00it/s]
    Epoch 18: 100%|████████████████████████████████████████████████████████████████████████████████▋| 240/241 [00:29<00:00,  8.09it/s, loss=730, v_num=0]{} {}ating:  97%|█████████████████████████████████████████████████████████████████████████████████████████████████▍  | 76/78 [00:02<00:00, 34.95it/s]
    Epoch    19: reducing learning rate of group 0 to 5.0000e-07.
    Epoch 19:  68%|███████████████████████████████████████████████████▍                        | 163/241 [00:25<00:12,  6.45it/s, loss=1.32e+03, v_num=0]Reached learning rate 5e-07, stopping...                                                                                                             
    [2021-06-05 19:17:58,492] INFO [deepethogram.callbacks.on_train_epoch_end:248] Stopping criterion reached! setting trainer.should_stop=True
    Epoch 19: 100%|███████████████████████████████████████████████████████████████████████████▋| 240/241 [00:29<00:00,  8.26it/s, loss=1.32e+03, v_num=0{} {}dating: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████| 78/78 [00:03<00:00, 31.39it/s]
    Epoch 19: 100%|████████████████████████████████████████████████████████████████████████████| 241/241 [00:29<00:00,  8.12it/s, loss=1.32e+03, v_num=0Saving latest checkpoint...                                                                                                                           
    Epoch 19: 100%|████████████████████████████████████████████████████████████████████████████| 241/241 [00:29<00:00,  8.12it/s, loss=1.32e+03, v_num=0]

## Run inference 
- **Inference button**: Run inference with your sequence model. Clicking this button will load a list of videos in your project that already have some features extracted. Output files that do not have any predictions from the currently selected (**6iii**) sequence architecture will be automatically pre-selected. If you don't see a video here, you need to run inference with your feature extractor first (**5ii**). This runs extremely fast, and should only take a few seconds for any number of videos.
![enter image description here](https://user-images.githubusercontent.com/81632945/120924926-b6794a00-c708-11eb-84f1-24d510fd3f4b.png)


**Equivalent command line arguments:**

    python -m deepethogram.sequence.inference sequence.weights=latest compute.num_workers=2 inference.overwrite=True inference.ignore_error=False
**Execution results:**

    [2021-06-05 19:21:47,005] INFO [__main__.sequence_inference:230] args: /home/huangyufei/.conda/envs/deg/lib/python3.7/site-packages/deepethogram/sequence/inference.py project.config_file=/home/huangyufei/deepethogram/deepethogram/training_deepethogram/project_config.yaml sequence.weights=latest inference.overwrite=True inference.ignore_error=False
    [2021-06-05 19:21:47,006] INFO [__main__.sequence_inference:232] configuration used: 
    [2021-06-05 19:21:47,017] INFO [__main__.sequence_inference:233] split:
      reload: true
      file: null
      train_val_test:
      - 0.8
      - 0.2
      - 0.0
    compute:
      fp16: false
      num_workers: 8
      batch_size: 32
      min_batch_size: 8
      max_batch_size: 512
      distributed: false
      gpu_id: 0
      dali: false
      metrics_workers: 0
    reload:
      overwrite_cfg: false
      latest: false
    notes: null
    log:
      level: info
    augs:
      brightness: 0.25
      contrast: 0.1
      hue: 0.1
      saturation: 0.1
      color_p: 0.5
      grayscale: 0.5
      crop_size: null
      resize:
      - 224
      - 224
      dali: false
      random_resize: false
      pad: null
      LR: 0.5
      UD: 0.0
      degrees: 10
      normalization:
        'N': 35734348800
        mean:
        - 0.5623775488272594
        - 0.5133215610229026
        - 0.3992611527834769
        std:
        - 0.1138012953521028
        - 0.09838669747141122
        - 0.14414765149547307
    feature_extractor:
      arch: resnet18
      fusion: average
      sampler: null
      final_bn: false
      sampling_ratio: null
      final_activation: sigmoid
      dropout_p: 0.25
      n_flows: 10
      n_rgb: 1
      curriculum: false
      inputs: both
      weights: pretrained
    train:
      steps_per_epoch:
        train: 1000
        val: 1000
        test: null
      num_epochs: 100
      regularization:
        style: l2
        alpha: 0.01
      patience: 5
      loss_weight_exp: 1.0
    sequence:
      arch: tgmj
      latent_name: null
      output_name: null
      sequence_length: 180
      dropout_p: 0.5
      num_layers: 3
      hidden_size: 64
      nonoverlapping: true
      filter_length: 15
      input_dropout: 0.5
      n_filters: 8
      tgm_reduction: max
      c_in: 1
      c_out: 8
      soft_attn: true
      num_features: 128
      bidirectional: false
      rnn_style: lstm
      hidden_dropout: 0.0
      weights: latest
      nonlinear_classification: true
      final_bn: true
      input_type: features
    inference:
      directory_list: all
      ignore_error: false
      overwrite: true
      use_loaded_model_cfg: true
    cmap: deepethogram
    control_arrow_jump: 31
    label_view_width: 31
    postprocessor:
      min_bout_length: 1
      type: min_bout_per_behavior
    prediction_opacity: 0.2
    project:
      class_names:
      - background
      - scratch
      config_file: /home/huangyufei/deepethogram/deepethogram/training_deepethogram/project_config.yaml
      data_path: /home/huangyufei/deepethogram/deepethogram/training_deepethogram/DATA
      labeler: null
      model_path: /home/huangyufei/deepethogram/deepethogram/training_deepethogram/models
      name: training
      path: /home/huangyufei/deepethogram/deepethogram/training_deepethogram
      pretrained_path: /home/huangyufei/deepethogram/deepethogram/training_deepethogram/pretrained_models
    run:
      type: inference
      model: sequence
      dir: /home/huangyufei/deepethogram/deepethogram/training_deepethogram/models/210605_192146_sequence_inference
    unlabeled_alpha: 0.1
    vertical_arrow_jump: 3
    
    [2021-06-05 19:21:47,396] INFO [deepethogram.projects.get_weightfile_from_cfg:1070] loading LATEST weights: /home/huangyufei/deepethogram/deepethogram/training_deepethogram/models/210605_190721_sequence_train/lightning_checkpoints/epoch=0-step=162.ckpt
    [2021-06-05 19:21:47,475] INFO [__main__.sequence_inference:263] latent name used for running sequence inference: resnet18
    TGMJ(
      (input_dropout): Dropout(p=0.5, inplace=False)
      (output_dropout): Dropout(p=0.5, inplace=False)
      (tgm_layers): Sequential(
        (0): TGMLayer()
        (1): TGMLayer()
        (2): TGMLayer()
      )
      (h): Conv1d(1024, 128, kernel_size=(1,), stride=(1,))
      (h2): Conv1d(1024, 128, kernel_size=(1,), stride=(1,))
      (classify1): Sequential(
        (0): Conv1d(128, 2, kernel_size=(1,), stride=(1,), bias=False)
        (1): BatchNorm1d(2, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
      )
      (classify2): Sequential(
        (0): Conv1d(128, 2, kernel_size=(1,), stride=(1,), bias=False)
        (1): BatchNorm1d(2, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
      )
    )
    [2021-06-05 19:21:47,484] INFO [__main__.sequence_inference:286] model: TGMJ(
      (input_dropout): Dropout(p=0.5, inplace=False)
      (output_dropout): Dropout(p=0.5, inplace=False)
      (tgm_layers): Sequential(
        (0): TGMLayer()
        (1): TGMLayer()
        (2): TGMLayer()
      )
      (h): Conv1d(1024, 128, kernel_size=(1,), stride=(1,))
      (h2): Conv1d(1024, 128, kernel_size=(1,), stride=(1,))
      (classify1): Sequential(
        (0): Conv1d(128, 2, kernel_size=(1,), stride=(1,), bias=False)
        (1): BatchNorm1d(2, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
      )
      (classify2): Sequential(
        (0): Conv1d(128, 2, kernel_size=(1,), stride=(1,), bias=False)
        (1): BatchNorm1d(2, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
      )
    )
    [2021-06-05 19:21:47,484] INFO [deepethogram.utils.load_state:341] loading from checkpoint file /home/huangyufei/deepethogram/deepethogram/training_deepethogram/models/210605_190721_sequence_train/lightning_checkpoints/epoch=0-step=162.ckpt...
    [2021-06-05 19:21:47,498] INFO [__main__.sequence_inference:294] best epoch from loaded file: 0
    [2021-06-05 19:21:47,504] INFO [__main__.sequence_inference:303] thresholds: [0.01 0.5 ]
      0%|                                                                                                                          | 0/3 [00:00<?, ?it/s][2021-06-05 19:21:53,822] INFO [__main__.extract:196] running inference on /home/huangyufei/deepethogram/deepethogram/training_deepethogram/DATA/assessment 1/assessment 1_outputs.h5. latent name: resnet18 output name: tgmj...
     33%|██████████████████████████████████████                                                                            | 1/3 [00:06<00:12,  6.37s/it][2021-06-05 19:22:00,188] INFO [__main__.extract:196] running inference on /home/huangyufei/deepethogram/deepethogram/training_deepethogram/DATA/assessment 2/assessment 2_outputs.h5. latent name: resnet18 output name: tgmj...
     67%|████████████████████████████████████████████████████████████████████████████                                      | 2/3 [00:11<00:05,  5.55s/it][2021-06-05 19:22:05,161] INFO [__main__.extract:196] running inference on /home/huangyufei/deepethogram/deepethogram/training_deepethogram/DATA/training 1/training 1_outputs.h5. latent name: resnet18 output name: tgmj...
    100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 3/3 [00:16<00:00,  5.65s/it]

 

## Export predictions to .csv files
In the predictions box, click  `export predictions to CSV files`

**Equivalent codes:**
   
    from deepethogram import configuration, postprocessing, projects
    import  os
    import  pandas  as  pd
    import  random  
   
    project_path = "/path/to/project/training_deepethogram"
    cfg = configuration.make_postprocessing_cfg(project_path=project_path) 
    postprocessing.postprocess_and_save(cfg)
    records = projects.get_records_from_datadir(os.path.join(project_path, 'DATA'))
    animal = random.choice(list(records.keys()))
    record = records[animal]
    
    # figure out the filename
    predictions_filename = os.path.join(os.path.dirname(record['rgb']), record['key'] + '_predictions.csv')
    assert  os.path.isfile(predictions_filename)
    # read csv  
    df = pd.read_csv(predictions_filename, index_col=0)
    # display outputs 
    print(df.head())
![enter image description here](https://user-images.githubusercontent.com/81632945/120946323-a431f700-c76e-11eb-85df-acdca8e38726.png)
![enter image description here](https://user-images.githubusercontent.com/81632945/120986303-07dc1480-c7af-11eb-96f3-2ce660d0f0c4.png)
## Add more videos and Re-train models

Using the GUI, add videos to your project. After you add the videos, as depicted in the workflow figure above, extract features to disk. This will take about 30-60 frames per second, depending on the model and your video resolution. Then run inference using the pretrained sequence model (should be instantaneous).

Now, for your newly added videos, you have probabilities and predictions for every video frame. Use the  `import predictions as labels`  button on the GUI, and it will move the predictions to your labeling box. There, you can quickly edit them for errors.

Now that you have even more videos and labels, we want to incorporate these into our models. Use the "train models" workflow above to re-train your previous models with the new data.

## Next steps

If this works OK for your project, the next steps should be to get a workstation with a good GPU locally (*workstation is a user-friendly desktop system for laptop and PCs which allows GUI operation*). According to Jim, purchasing a proper GPU really depends on the budget. it would be great to get an Nvidia 30-series card, such as a 3070, 3080 or 3090 . However, Nvidia 2080 or 2080 Tis would work well too.

## Acknowledgements
We thank Harvey lab members James P.  Bohnslav for sharing us with COLAB notebook on how to upload the data and train models. If you have any issues, or can't get Deepethogram running, or just have questions, please raise an issue on Github.[https://github.com/jbohnslav/deepethogram/issues](https://github.com/jbohnslav/deepethogram/issues)











