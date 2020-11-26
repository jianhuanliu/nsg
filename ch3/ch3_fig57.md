- [Main content](ch3_main.md)
- [Prev figure](ch3_fig04.md)
- [Next figure](ch3_fig06.md)

![Figure 05](Figs/ch3_fig05.png).    
**Figure 5:** Steps for revealing weak diffractions dominated by strong Love waves. (a) A synthetic SH shot gather computed for the model shown in Figure4; (b) retrieved Love waves from the gather as shown in Figure 5a; (c) result after convolution of the data in (b) with the matching filter (d); (d) mean filter coefficients estimated by minimizing the difference between (a) and (b), using the regularized non-stationary regression method.

![Figure 07](Figs/ch3_fig07.png).   
**Figure 7:** (a) Result after Love-wave suppression in the data in Figure 5a by $f-k$ filtering; (b) result after adaptive subtraction of Figure 5b from Figure 5a; (c) result after applying SVI to the result in (b) for diffraction enhancement; (d) reference shot gather showing only diffracted arrivals obtained from subtraction of modeled data using models with and without diffractors.

<span style="color:black"> **Source code used to reproduce Figure 4**: </span> <br>
<span style="color:blue"> *Dependency:* </span> [Seismic Unix](https://github.com/JohnWStockwellJr/SeisUnix). <br>
<span style="color:blue"> *Data availability:* </span> Input data is not yet available.

**Source code (1):**
```sh
% Mean filter coefficients
% Author: Jianhuan Liu
% 23-11-2020

#!/bin/bash

WID=4.
HEI=4.

# - read-white-blue -
BRGB="1.000,0.000,0.000"
GRGB="1.000,1.000,1.000"
WRGB="0.000,0.000,1.000"

# [-0.1 ~ 0.5]
dir=updatedVassenSyn_2th_revision/c02as_csg_noise_fa

tmax=0.2

psimage < $dir/flt_09.bin n1=401 d1=0.0005 d2=1 f2=1 \
brgb=$BRGB grgb=$GRGB wrgb=$WRGB units= \
width=4 height=4 xbox=0.0 ybox=0.0 \
x1beg=0 x1end=$tmax  d1num=0.05 n1tic=5 \
x2beg=1 x2end=40 f2num=0  \
d2num=10 n2tic=5 \
legend=1 lstyle=vertright lheight=3 \
lbeg= lend= \
labelsize=18 bps=24 perc=99 > temp/fig05_coeff.eps

open temp/fig05_coeff.eps &

```

**Source code (2):**
```sh
% Steps for revealing weak diffraction by SI+AS+SVI
% Author: Jianhuan Liu
% 23-11-2020

#!/bin/bash

WID=4.
HEI=4.

# - read-white-blue -
BRGB="1.000,0.000,0.000"
GRGB="1.000,1.000,1.000"
WRGB="0.000,0.000,1.000"

# Figure 2, (a)
fldr=9

size="wbox=4 hbox=4"
tmax=0.2
clip="perc=99"
filt="sufilter f=3,10,95,100 amp=0,1,1,0"

dir=updatedVassenSyn_2th_revision
dir1=$dir/c02as_csg_noise_fa

< $dir1/shots_full.su suwind key=fldr min=$fldr max=$fldr |
suwind tmax=$tmax | sunormalize norm=max |
supswigp xbox=0.0 ybox=0.0 $size $clip  \
x1beg=0 x1end=$tmax  d1num=0.05 n1tic=5 \
d2num=10 n2tic=5 \
labelsize=18 label2="Receiver number" label1="Time (s)" > temp/fig06_raw.eps 

< $dir1/shots_diff.su suwind key=fldr min=$fldr max=$fldr |
suwind tmax=$tmax | sunormalize norm=max |
supswigp xbox=0.0 ybox=0.0 $size $clip \
x1beg=0 x1end=$tmax  d1num=0.05 n1tic=5 \
d2num=10 n2tic=5 \
labelsize=18 label2= > temp/fig06_ref.eps 

# FK
< $dir1/shots_fk.su suwind key=fldr min=$fldr max=$fldr |
suwind tmax=$tmax | sunormalize norm=max |
supswigp xbox=0.0 ybox=0.0 $size $clip key=tracf \
x1beg=0 x1end=$tmax  d1num=0.05 n1tic=5 \
d2num=10 n2tic=5 labelsize=18 \
label1="Time (s)" label2="Receiver number" > temp/fig06_fk.eps 

< $dir1/datasw/shot_sw_${fldr}.su \
suwind tmax=$tmax | sunormalize norm=max |
supswigp xbox=0.0 ybox=0.0 $size $clip \
x1beg=0 x1end=$tmax  d1num=0.05 n1tic=5 \
d2num=10 n2tic=5 \
labelsize=18 label1="Time (s)" > temp/fig06_sw.eps 

((fd1=2*$fldr-11))
< $dir1/shots_si.su suwind key=fldr min=$fd1 max=$fd1 |
suwind tmax=$tmax | sunormalize norm=max |
supswigp xbox=0.0 ybox=0.0 $size $clip \
x1beg=0 x1end=$tmax  d1num=0.05 n1tic=5 \
d2num=10 n2tic=5 f2=1 \
labelsize=18 label2="Receiver number" > temp/fig06_si.eps 

((fd=$fldr-6))
< $dir1/shots_as.su suwind key=fldr min=$fd max=$fd |
suwind tmax=$tmax | 
supswigp xbox=0.0 ybox=0.0 $size $clip \
x1beg=0 x1end=$tmax  d1num=0.05 n1tic=5 \
d2num=10 n2tic=5 \
labelsize=18 label1= label2="Receiver number" > temp/fig06_as.eps 

< $dir1/shots_svi.su suwind key=fldr min=$fd max=$fd |
suwind tmax=$tmax | 
supswigp xbox=0.0 ybox=0.0 $size $clip \
x1beg=0 x1end=$tmax  d1num=0.05 n1tic=5 \
d2num=10 n2tic=5 \
labelsize=18 label1="Time (s)" > temp/fig06_svi.eps 


# merge into one file
psmerge translate=0.,0. scale=0.5,0.5 in=temp/fig06_sw.eps \
translate=2.5,0.0 scale=0.5,0.5 in=temp/fig05_coeff.eps \
translate=0.0,2.5 scale=0.5,0.5 in=temp/fig06_raw.eps \
translate=2.5,2.5 scale=0.5,0.5 in=temp/fig06_si.eps > figs/fig05_merge.eps

psmerge translate=0.,0. scale=0.5,0.5 in=temp/fig06_svi.eps \
translate=2.5,0.0 scale=0.5,0.5 in=temp/fig06_ref.eps \
translate=0.0,2.5 scale=0.5,0.5 in=temp/fig06_fk.eps \
translate=2.5,2.5 scale=0.5,0.5 in=temp/fig06_as.eps > figs/fig06_merge.eps

open figs/fig05_merge.eps &
open figs/fig06_merge.eps &


```

<a href="#top">Back to top</a>
