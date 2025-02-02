---
layout: default
permalink: /slide/gaudio/music/
---

### Lightsaft: Memory-efficient Latent Source Attentive Frequency Transformation for Conditioned Source Separation


[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/lasaft-latent-source-attentive-frequency/music-source-separation-on-musdb18)](https://paperswithcode.com/sota/music-source-separation-on-musdb18?p=lasaft-latent-source-attentive-frequency)

[return to slide](/slide/gaudio/)

<audio controls="" class="audio-player" preload="metadata"><source src="{{ baseurl }}/audios/whatawonderfulworld.wav" type="audio/wav"></audio>

**Model Configuration**

```python
from lasaft.source_separation.\
  conditioned.cunet.models.\
  dcun_tfc_gpocm_lightsaft \
  import DCUN_TFC_GPoCM_LightSAFT_Framework


args = {}

# FFT params
args['n_fft'] = 4096
args['hop_length'] = 1024
args['num_frame'] = 128

# SVS Framework
args['spec_type'] = 'complex'
args['spec_est_mode'] = 'mapping'

# Other Hyperparams
args['optimizer'] = 'adam'
args['lr'] = 0.001
args['dev_mode'] = False
args['train_loss'] = 'spec_mse'
args['val_loss'] = 'raw_l1'

# DenseNet Hyperparams

args ['n_blocks'] = 9
args ['input_channels'] = 4
args ['internal_channels'] = 24
args ['first_conv_activation'] = 'relu'
args ['last_activation'] = 'identity'
args ['t_down_layers'] = None
args ['f_down_layers'] = None
args ['tif_init_mode'] = None

# TFC_TDF Block's Hyperparams
args['n_internal_layers'] = 5
args['kernel_size_t'] = 3
args['kernel_size_f'] = 3
args['tfc_tdf_activation'] = 'relu'
args['bn_factor'] = 16
args['min_bn_units'] = 16
args['tfc_tdf_bias'] = True
args['num_tdfs'] = 16
args['dk'] = 64

args['control_vector_type'] = 'embedding'
args['control_input_dim'] = 4
args['embedding_dim'] = 32
args['condition_to'] = 'decoder'

args['control_n_layer'] = 4
args['control_type'] = 'dense'
args['pocm_type'] = 'matmul'
args['pocm_norm'] = 'batch_norm'


model = DCUN_TFC_GPoCM_LightSAFT_Framework(**args)
```