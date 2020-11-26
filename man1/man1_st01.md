- [Main page](../index.md)

- [Next step](man1_st02.md)

  

## Vibroseis decompression

### Theory



### Implentation



<span style="color:black"> **Source code used to reproduce Figure 4**: </span> <br>
<span style="color:blue"> *Dependency:* </span> [Seismic Unix](https://github.com/JohnWStockwellJr/SeisUnix). <br>
<span style="color:blue"> *Data availability:* </span> Input data is not yet available. <br>



**(1.1) Decompress on shot gather **

_Xdecon_weight.scr:_

```sh
#!/bin/sh
#
# This scipt is desinged to CC or deconvolution vibrator data
# with a master sweep trace weighted by 4 traces (49,50,51,52).
# This sweep trace can also be a recorded reference trace. 
#
# Original Author: Shohei Minato
# new 10-08-2017 
# recheck at 18-05-2019
# Usage: ./Xdecon_weight.scr $fldr
# Used by X01_moveFiles.scr

field_id=$1

sweep_length=3700 # ms
record_length=4400 # ms
ntaper_smpl=10 # sample points for tapering before sweep ends
dt=0.5 # sampling interval in ms

# 'file_id.dat'  will first be converted to 'field_id.sgy'
$CWPROOT/src/Third_Party/seg2segy/seg2segy $field_id 1

# then convert 'field_id.sgy' to 'field_id.su' for further processing
segyread tape=${field_id}.sgy \
         |sushw key=trid a=1 \
         |segyclean > ${field_id}orig.su

rm -f ${field_id}.sgy binary header

# ch69(w1=64.156) ,ch70(w2=17.595) ,ch71(w3=7.119), ch72(w4=11.13)
# are extracted, weighted , summed to give a master sweep function,
# which can then be used to implement Crosscorrelation(suxcor)
# and Deconvolution(sucddecon).

# ch53 ?
# ch54, reference ch
# ch55,ch56 are blank traces.
ch1=49; w1=64.156
ch2=50; w2=17.595
ch3=51; w3=1.0  # 7.119
ch4=52; w4=11.13
#ch1=70; w1=1.
#ch2=70; w2=1.
#ch3=70; w3=1.
#ch4=70; w4=1.

suwind < ${field_id}orig.su key=tracf min=$ch3 max=$ch3 |
sugain scale=$w1 > ch_${ch1}w.su

suwind < ${field_id}orig.su key=tracf min=$ch3 max=$ch3 |
sugain scale=$w2 > ch_${ch2}w.su

suwind < ${field_id}orig.su key=tracf min=$ch3 max=$ch3 |
sugain scale=$w3 > ch_${ch3}w.su

suwind < ${field_id}orig.su key=tracf min=$ch3 max=$ch3 |
sugain scale=$w4 > ch_${ch4}w.su

# Summation of weighted traces
cat ch_${ch1}w.su ch_${ch2}w.su ch_${ch3}w.su ch_${ch4}w.su \
    |sustack key=dt normpow=0 > sweep_weighted.su
rm -f ch_*.su

# ============CC and Decon parameters==============
# output record length in sec
output_length_sec=$(echo $record_length $sweep_length | awk '{print ($1-$2)*0.001}')

# output record length in sample
output_length_ns=$(echo $output_length_sec $dt | awk '{print ($1/($2*0.001))}' \
       |awk '{print int($1)+1}')
echo output_length=$output_length_sec
echo output_ns=$output_length_ns

# sweep length in sec
sweep_length_sec=$(echo $sweep_length | awk '{print $1*0.001}')
echo sweep_length_sec=$sweep_length_sec

sumute < sweep_weighted.su \
       key=tracf xmute=1 tmute=0.01 ntaper=$ntaper_smpl \
       |sumute key=tracf xmute=1 tmute=$sweep_length_sec \
        mode=1 ntaper=$ntaper_smpl  > ${field_id}_sweep.su
rm -f sweep_weighted.su
rm -f temp00*

# needed to change if field data are recorded with different chanels. 
# raw traces range from [47-64] and [73-122] use 'tracf' as trace index
suwind < ${field_id}orig.su \
        key=tracf min=1 max=48 > temp001_01.su
suwind < ${field_id}orig.su \
        key=tracf min=57 max=96 > temp001_02.su
suwind < ${field_id}orig.su \
        key=tracf min=97 max=176 |
sugain scale=1.0 > temp001_03.su

cat temp001_01.su >> temp002.su
cat temp001_02.su >> temp002.su
cat temp001_03.su >> temp002.su

< temp002.su sushw key=tracf,tracl a=1,0 b=1,0 > temp003.su
 
suwind < temp003.su suwind key=tracf min=1 max=96 > temp004_01.su
suwind < temp003.su suwind key=tracf min=97 max=168 |
sugain scale=-1.0 > temp004_02.su

cat temp004_01.su >> temp005.su
cat temp004_02.su >> temp005.su

< temp005.su sushw key=tracf,tracl a=1,0 b=1,0 > ${field_id}.su 

# ============= Convolution ===================
# Positive time only (causal)
suxcor < ${field_id}.su sufile=${field_id}_sweep.su \
         vibroseis=$output_length_ns > xcor_w${field_id}.su

# ============ Deconvolution ================
sucddecon < ${field_id}.su sufile=${field_id}_sweep.su \
            pnoise=0.001 > temp002_decon.su
rm -f ${field_id}_sweep.su

# Positive time only (causal)
half_record_length_sec=$(echo $record_length | awk '{print $1*0.5*0.001}')
t1_sec=$(echo $half_record_length_sec $output_length_sec | awk '{print $1+$2}')

#suwind < temp002_decon.su key=tracf min=1 \
#         tmin=$half_record_length_sec \
#         tmax=$t1_sec > temp003_decon.su

# debuged by Shohe, take the uncausal part instead
suwind < temp002_decon.su itmax=$output_length_ns > temp003_decon.su

sushw < temp003_decon.su key=delrt a=0 |
sufilter f=3,10,200,220 amp=0,1,1,0 > decon_w${field_id}.su

rm -f ${field_id}orig.su ${field_id}.su ${field_id}.dat
rm -f temp00*
rm -f header binary

exit
```



**(1.2) Processing many shot gathers**

_X01_moveFiles.scr:_

```sh
#!/bin/sh
#
# Decompress many shot gathers.
#
# 25-11-2020, J. Liu 
# Usage: ./X01_moveFiles.scr

fd1=6; fd2=150

dir=../Data/ouwerkerk/oudekerk
for ((fldr=$fd1; fldr<=$fd2; fldr++)) do
     cp $dir/${fldr}.dat .
    ./Xdecon_weight.scr $fldr
done
```



<a href="#top">Back to top</a>