- [Main content](ch3_main.md)
- [Prev figure](ch3_fig10.md)
- [Next figure](ch3_fig12.md)

![Figure 11](Figs/ch3_fig11.png).    
**Figure 11:** Two-dimensional subsurface $V_S$ model obtained by elastic full-waveform inversion of the sledge-hammer data. 

<span style="color:black"> **Source code used to reproduce Figure 11**: </span> <br>
<span style="color:blue"> *Dependency:* </span> [Seismic Unix](https://github.com/JohnWStockwellJr/SeisUnix). <br>
<span style="color:blue"> *Data availability:* </span> Input data is not yet available.

```sh
% wo-dimensional subsurface $V_S$ model obtained by FWI
% Author: Jianhuan Liu
% 20-01-2020

#!/bin/bash

WID=4.5 # 4.5
HEI=2.5 #2.5

# - read-white-blue -
BRGB="1.000,0.000,0.000"
GRGB="1.000,1.000,1.000"
WRGB="0.000,0.000,1.000"

clip="perc=97"

# (1) make the null data
sunull nt=120 dt=0.05 ntr=628 > temp/null.su
sustrip < temp/null.su head=temp/null.h > temp/null.d

supaste < modelTest_vs_stage_6.bin > inv.su ns=120 head=temp/null.h 

< inv.su suwind key=tracl min=17 max=612 |
supsimage d1=0.05 d2=0.05 f2=0.001  \
brgb=$BRGB grgb=$GRGB wrgb=$WRGB \
n1tic=5 n2tic=5 \
grid1=dash grid2=dash gridcolor=white gridwidth=1 \
width=$WID height=$HEI xbox=0.0 ybox=0.0 \
legend=1 lstyle=vertright lheight=$HEI \
units="S-wave velocity (m/s)" \
label1="Depth (m)" label2="Location (m)" labelsize=18 \
d1num=2 d2num=5 $clip \
bps=24 > temp/fig10_new_model.ps

# merge into one file
scale=0.6
psmerge translate=0.,0. scale=$scale,$scale in=temp/fig10_new_model.ps > figs/fig10_new_merge.eps

open figs/fig10_new_merge.eps &

```

<a href="#top">Back to top</a>
