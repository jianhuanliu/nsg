- [Main content](ch5_main.md)
- [Prev figure](ch5_fig02.md)
- [Next figure](ch3_fig03.md)

![Figure 02](Figs/ch5_fig02.png).
**Figure 2:** Source-receiver geometry used to acquire 2D seismic data along the Line A in a roll-along mode. The triangles represent horizontal, 10 Hz, single-component component geophones we used to recored the seismic data. Every fouth source and receiver positions are displayed.
    

<span style="color:black"> **Source code used to reproduce Figure 2**: </span> <br>
<span style="color:blue"> *Dependency:* </span> [Seismic Unix](https://github.com/JohnWStockwellJr/SeisUnix). <br>
<span style="color:blue"> *Data availability:* </span> Input data is not yet available.


```sh
#!/bin/bash
#
# display acquisition geometry for a fieldwork 
# 28-09-2020, J.Liu

# size of short Line 
SWIDTH=4.0; SHEIGHT=4.0
short="wbox=$SWIDTH hbox=$SHEIGHT"

dir=firstLine/Preprocessed_Line01
data1=$dir/LIN01_shots_raw.su

# geometry of shortLine
< $data1 suchw key1=sx key2=sx b=0.0001 |
suchw key1=gx key2=gx b=0.0001 |
suchart key1=gx key2=sx outpar=pfile > temp/plot_data

psgraph < temp/plot_data par=pfile  \
    xbox=0.0 ybox=0.0 $short \
    x1beg=-1 x1end=50 d1num=10 n1tic=2 \
    x2beg=-5 x2end=55 d2num=10 n2tic=2 \
    grid1=dot grid2=dot label1="Receivers (m)" label2="Sources (m)" \
    labelsize=20 title= \
    linewidth=0 marksize=4 mark=6 linecolor=red > temp/tmp_fig02_a.eps

# calculate (x,y) positions of each subfigs
scale=0.5; dX=0.5
# 1st row
x1=0; y1=0
x2=$(echo $SWIDTH $scale $dX | awk '{print $1 * $2 + $3}');
y2=0

# merge into one figure
psmerge translate=$x1,$y1 scale=$scale,$scale in=temp/tmp_fig02_a.eps  > figs/fig02_merge.eps

open figs/fig02_merge.eps & 

```

<a href="#top">Back to top</a>