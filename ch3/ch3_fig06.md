- [Main content](ch3_main.md)
- [Prev figure](ch3_fig57.md)
- [Next figure](ch3_fig08.md)

![Figure 06](Figs/ch3_fig06.png).    
**Figure 6:** (a) The frequency-wavenumber ($f-k$) spectrum of the synthetic SH shot gather from Figure 5a;  (b) the $f-k$ spectrum of the same shot gather, but after rejecting Love-wave energy  indicated by the black arrows in (a).

<span style="color:black"> **Source code used to reproduce Figure 6**: </span> <br>
<span style="color:blue"> *Dependency:* </span> [Seismic Unix](https://github.com/JohnWStockwellJr/SeisUnix). <br>
<span style="color:blue"> *Data availability:* </span> Input data is not yet available.

```sh
% frequency-wavenumber ($f-k$) spectrum comparison
% Author: Jianhuan Liu
% 20-01-2020

!/bin/bash

WIDTH=4.; HEIGHT=5.
size="width=$WIDTH height=$HEIGHT"
LHEIGHT=$(echo $HEIGHT | awk '{print $1 * 0.7}')
LWIDTH=0.15

# sample colormap:  red, green, blue
cmap="brgb=1.0,0.0,0.0 grgb=1.0,1.0,1.0 wrgb=0.0,0.0,1.0"

clip="perc=99"

fldr=9

dir=updatedVassenSyn_2th_revision/c02fk_csg

rawData=$dir/shots_full.su
fkData=$dir/shots_fk.su


tmax=0.2

# check the frequency interval 
< $rawData suwind key=fldr min=$fldr max=$fldr |
suwind tmax=$tmax | 
suspecfk dx=0.5 |
suwind itmax=30 | surange

itmax=20

< $rawData suwind key=fldr min=$1 max=$1 |
suwind tmax=$tmax | 
suspecfk dx=0.5 |
suwind itmax=$itmax | sushw key=trid a=130 |
supsimage  d1=6.493506 d2=0.05 f1=0.0 f2=-1.0 \
n1tic=5 n2tic=5 d1num=20 f1num=0.0 d2num=0.5 f2num=-1. \
$cmap units= \
$size xbox=0.0 ybox=0.0 \
legend=1 lstyle=vertright lheight=$LHEIGHT lbeg=0 lend=1.25 \
label1="Frequency (Hz)" label2="Wavenumber (1/m)" labelsize=24 \
bps=24 $clip title= > temp/fig05b_a.eps

< $fkData suwind key=fldr min=$1 max=$1 |
suwind tmax=$tmax | 
suspecfk dx=0.5 |
suwind itmax=$itmax |
supsimage  d1=6.493506 d2=0.05 f1=0.0 f2=-1.0 \
n1tic=5 n2tic=5 d1num=20 f1num=0.0 d2num=0.5 f2num=-1. \
$cmap units="Amplitude" \
$size xbox=0.0 ybox=0.0 \
legend=1 lstyle=vertright lheight=$LHEIGHT lbeg=0 lend=0.125 \
label1= label2="Wavenumber (1/m)" labelsize=24 \
bps=24 $clip title= > temp/fig05b_b.eps

# calculate (x,y) positions of each subfigs
scale=0.5; dX=1.0; dY=0.35
# 1st row
x1=0; y1=0
x2=$(echo $WIDTH $scale $dX| awk '{print $1 * $2 + $3}'); y2=$y1


# merge into one file
psmerge translate=$x1,$y1 scale=$scale,$scale in=temp/fig05b_a.eps \
translate=$x2,$y2 scale=$scale,$scale in=temp/fig05b_b.eps > figs/fig05b_merge.eps

open figs/fig05b_merge.eps &

```

<a href="#top">Back to top</a>
