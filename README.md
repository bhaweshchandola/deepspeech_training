# Training a new SpeechtoText model with Deepspeech

**Make sure the deepspeech version are same for training and running**


1. clone the deepspeech via- https://github.com/mozilla/DeepSpeech
2. then `git checkout v 0.5.1`
3. cd DeepSpeech `pip install -r requirements.txt`
4. Download ngram processing tool kenlm.

```$ git clone https://github.com/kpu/kenlm   
$sudo apt install zlib1g-dev libbz2-dev liblzma-dev libeigen3-dev libboost1.65-all-dev cmake
$mkdir build
$cd build
$cmake ..
$sudo make install
```
5. Install native_client.
This is the pre-processing tool that comes with deepSpeech and can help with pre-processing.
runs in the root directory of deepSpeech: `python3 util/taskcluster.py --arch gpu --target ./native_client`

6. Prepare dataset. There should be three folders train, test, and dev. Each folder should contain all the audio files with csv. Which should have three columns. wav_file, wav_size, wav_transcript.

7. Create the alphabet, vocubulary and lm.binary file.

8. Save the below code as train.sh and run in from the deepspeech root directory.


```#!/bin/sh
set -xe
if [ ! -f DeepSpeech.py ]; then
    echo "Please make sure you run this from DeepSpeech's top level directory."
    exit 1
fi;

python3.5 -u DeepSpeech.py \
  --train_files /usr/workspace/pykaldi/deepspeech/vietnam/deepspeech_training/train/train.csv \
  --test_files /usr/workspace/pykaldi/deepspeech/vietnam/deepspeech_training/test/test.csv \
  --train_batch_size 80 \
  --test_batch_size 40 \
  --n_hidden 375 \
  --epoch 3 \
  --validation_step 1 \
  --early_stop True \
  --earlystop_nsteps 6 \
  --estop_mean_thresh 0.1 \
  --estop_std_thresh 0.1 \
  --dropout_rate 0.22 \
  --learning_rate 0.00095 \
  --report_count 100 \
  --use_seq_length False \
  --export_dir /usr/workspace/pykaldi/deepspeech/vietnam/deepspeech_training/results/model_export/ \
  --checkpoint_dir /usr/workspace/pykaldi/deepspeech/vietnam/deepspeech_training/results/checkout/ \
  --decoder_library_path /home/nvidia/tensorflow/bazel-bin/native_client/libctc_decoder_with_kenlm.so \
  --alphabet_config_path /usr/workspace/pykaldi/deepspeech/vietnam/deepspeech_training/alphabet.txt \
  --lm_binary_path /usr/workspace/pykaldi/deepspeech/vietnam/deepspeech_training/lm.bin \
  --lm_trie_path /usr/workspace/pykaldi/deepspeech/vietnam/deepspeech_training/trie \
  "$@"
```
9. running the model
`deepspeech --model model_data/output_graph.pb --alphabet model_data/alphabet.txt --lm model_data/lm.bin --trie model_data/tri --audio model_data/x.wav`


### Useful links
https://github.com/mozilla/DeepSpeech/blob/d3168391b3e9ee041bf8c517237cbacbd0c23da2/README.md#installing-prerequisites-for-training

http://www.programmersought.com/article/1886530040/
