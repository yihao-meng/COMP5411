
#GROUP ID: 5

#Group members:


Yihao Meng 20972376  ymengas@connect.ust.hk


Lei Han 21068134 lhanao@connect.ust.hk

# Code generate from scratch:
All the file in the project/   directory, including

```

bezier.py
dynamicTypography.py
environment.yml
losses.py
model_utils.py
packages.py
painter_dTypo.py
positional_encoding.py
save_svg.py
ttf.py
utils.py
```

The implementation of the differentiable rasterizer part was referenced from https://github.com/BachiLi/diffvg. But we try different methods for realizing differential, the default code use Monte Carlo Sampling, while we try analytical prefiletering which is slower but more accurate.


The implementation of the different rasterizers was referenced from https://github.com/diegonehab/stroke-to-fill#. But we encode the code to our workflow, and generate different results using different rasterizers.

# Workload Breakdown:
Yihao Meng(50%): Implement of different rasterizers, mesh-based structure preservation and skeleton code. Write half of the final report.


Lei Han(50%): Implement of differentiable rasterizer, including understand the thoery and implementation detail,  try using analytical prefiltering to replace the Monte Carlo Sampling. Write half of the final report.


Overall, the contribution of each member to this project is 50%.
## Requirements:
All our animation samples are generated with a single H800 GPU with 80GB VRAM. To generate a text animation with 20 or more frames, a GPU with at least 24GB VRAM is required.

## Setup
```
git clone https://github.com/zliucz/animate-your-word.git
cd animate-your-word
```

## Environment
All the tests are conducted in Linux. We suggest running our code in Linux. To set up our environment in Linux, please run:
```
conda env create -f environment.yml
```
Next, you need to install diffvg:
```bash
conda activate dTypo
git clone https://github.com/BachiLi/diffvg.git
cd diffvg
git submodule update --init --recursive
python setup.py install
```

## Generate Your Animation!
To animate a letter within a word, run the following command:
```
CUDA_VISIBLE_DEVICES=0 python dynamicTypography.py \
        --word "<The Word>" \
        --optimized_letter "<The letter to be animated>" \
        --caption "<The prompt that describes the animation>" \
        --use_xformer --canonical --anneal \
        --use_perceptual_loss --use_conformal_loss  \
        --use_transition_loss
```
For example:
```
CUDA_VISIBLE_DEVICES=0 python dynamicTypography.py \
        --word "father" --optimized_letter "h" \
        --caption "A tall father walks along the road, holding his little son with his hand" \
        --use_xformer --canonical --anneal \
        --use_perceptual_loss --use_conformal_loss \
        --use_transition_loss
```

The output animation will be saved to "videos". The output includes the network's weights, SVG frame logs, and their rendered .mp4 files (under svg_logs and mp4_logs respectively). We save both the in-context and the sole letter animation.
At the end of training, we output a high-quality gif render of the last iteration (HG_gif.gif). <br>

We provide many example run scripts in `scripts`, the expected resulting gifs are in `example_gifs`. More results can be found on our [project page](https://animate-your-word.github.io/demo/).

## Tips:

By default, a 24-frame video will be generated, requiring about 28GB of VRAM. If there is not enough VRAM available, the number of frames can be reduced by using the `--num_frames` parameter.

If your animation remains the same with/deviates too much from the original letter's shape, please set a lower/higher `--perceptual_weight`.

If you want the animation to be less/more geometrically similar to the original letter, please set a lower/higher `--angles_w`.

If you want to further enforce appearance consistency between frames, please set a higher `--transition_weight`. But please keep in mind that this will reduce the motion amplitude.

Small visual artifacts can often be fixed by changing the `--seed`.

# Different rasterizers:
If you want to generate results with different rasterizers, you should follow the following procedure the set up the environment.

```
cd different_rasterizers

```


## Installation of general dependencies

On Ubuntu 20.04

```
    sudo apt-get install build-essential git subversion wget libb64-dev libboost-dev libcairo2-dev libfreetype6-dev libharfbuzz-dev libicu-dev libpng-dev liblapacke-dev libreadline-dev zlib1g-dev libegl1-mesa-dev libglew-dev libgsl-dev libwebp-dev clang-9 libjpeg-turbo8-dev qt5-default maven openjdk-8-jre openjdk-11-jre

    sudo mkdir -p /usr/local/lib/pkgconfig
    sudo cp b64.pc /usr/local/lib/pkgconfig/

    wget http://www.lua.org/ftp/lua-5.3.5.tar.gz
    tar -zxvf lua-5.3.5.tar.gz
    cd lua-5.3.5
    patch -p1 < ../luapp.patch
    make CC=g++ MYCFLAGS="-x c++ -fopenmp" MYLIBS="-lgomp" linux
    sudo make install
    sudo ln -s /usr/local/bin/luapp5.3 /usr/local/bin/luapp
    sudo cp ../luapp53.pc /usr/local/lib/pkgconfig/
    cd ..
    \rm -rf lua-5.3.5

```

On macOS Catalina using macports


```
    sudo port install clang-9.0 git subversion wget boost pkgconfig libb64 libpng qt5 cairo freetype harfbuzz-icu gsl mesa

    sudo mkdir -p /usr/local/lib/pkgconfig
    sudo cp b64.pc /usr/local/lib/pkgconfig/

    wget http://www.lua.org/ftp/lua-5.3.5.tar.gz
    tar -zxvf lua-5.3.5.tar.gz
    cd lua-5.3.5
    patch -p1 < ../luapp.patch
    make CC=clang++-mp-9.0 MYCFLAGS="-x c++ -fopenmp" MYLIBS="-L/opt/local/lib/libomp -lgomp" macosx
    sudo make install
    sudo ln -s /usr/local/bin/luapp5.3 /usr/local/bin/luapp
    sudo mkdir -p /usr/local/lib/pkgconfig
    sudo cp ../luapp53.pc /usr/local/lib/pkgconfig/
    cd ..
    \rm -rf lua-5.3.5
```

## Installation of third-party stroker dependencies

Go through each directory in

```
    strokers/agg
    strokers/cairo
    strokers/gs
    strokers/livarot
    strokers/mupdf
    strokers/openvg-ri
    strokers/skia
```

and read the README file for instructions to build the dependencies.

> **WARNING: Please be careful: don't blindly execute the commands.
> These commands worked for us but are only meant as an illustration.**

Now you can build the entire framework

```
    export vg_build_strokers=yes
    export vg_build_distroke=yes
    export vg_build_skia=yes
    # Optionally, enable other drivers
    # export vg_build_qt5=yes
    # export vg_build_cairo=yes
    # export vg_build_nvpr=yes
    make
```

The main command for the stroke tests is test-strokers.lua.
Run

```
    luapp test-strokers.lua -help
```

for the options.

If you want to generate the same result of different rasterizers, you can run the following code as an example:
```
**luapp test-strokers.lua -test:yihao -stroker:mvpg output yihao/mpvg_1.svg**

```

There are also a few scripts inside the strokers/ directory.
We used them to create the animations in the supplemental meterials.

## Acknowledgment:
Our implementation is based on [word-as-image](https://github.com/Shiriluz/Word-As-Image) , [live-sketch](https://github.com/yael-vinker/live_sketch) and [stroke-to-fill](https://github.com/diegonehab/stroke-to-fill#). Thanks for their remarkable contribution and released code.
