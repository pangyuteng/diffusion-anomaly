
--

pip3 install visdom
visdom -port 8850

--

docker build -t pangyuteng/diffusion-anomaly .
docker push pangyuteng/diffusion-anomaly

docker run -it --network=host -u $(id -u):$(id -g) -v /mnt:/mnt -w $PWD pangyuteng/diffusion-anomaly bash

python preprocess.py \
    /mnt/hd2/data/brats2020/BraTS2020_TrainingData/MICCAI_BraTS2020_TrainingData \
    /mnt/scratch/databrats2020-diffusion-anomal


visdom -port 8850

export TRAIN_FLAGS="--lr 1e-4 --batch_size 10"
export CLASSIFIER_FLAGS="--image_size 256 --classifier_attention_resolutions 32,16,8 --classifier_depth 4 --classifier_width 32 --classifier_pool attention --classifier_resblock_updown True --classifier_use_scale_shift_norm True"

CUDA_VISIBLE_DEVICES=0 \
    python scripts/classifier_train.py \
    --data_dir /mnt/scratch/data/brats2020-diffusion-anomal/training \
    --dataset brats $TRAIN_FLAGS $CLASSIFIER_FLAGS

export MODEL_FLAGS="--image_size 256 --num_channels 128 --class_cond True --num_res_blocks 2 --num_heads 1 --learn_sigma True --use_scale_shift_norm False --attention_resolutions 16"
export DIFFUSION_FLAGS="--diffusion_steps 1000 --noise_schedule linear --rescale_learned_sigmas False --rescale_timesteps False"
export TRAIN_FLAGS="--lr 1e-4 --batch_size 10"

CUDA_VISIBLE_DEVICES=1 \
    python scripts/image_train.py \
    --data_dir /mnt/scratch/data/brats2020-diffusion-anomal/training \
    --dataset brats  $MODEL_FLAGS $DIFFUSION_FLAGS $TRAIN_FLAGS