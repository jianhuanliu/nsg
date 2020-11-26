- [Main page](../index.md)
- [Prev step](man1_st02.md)



**(3) Combine separate shot gathers into one file **

_X03_combineShots.scr:_

```sh
#!/bin/sh
#
# Combine separate shot gathers into one file 
#
# 20-01-2017, J. Liu 
# Usage: ./X03_combineShots.scr

dataout=Zeeland_A_L1_vibro_sh

rm -f temp_*
for ((fldr=1; fldr<=36; fldr++)) do
  cat stack_xcor_${fldr}.su >> temp_xcor_shots.su
  cat stack_decon_${fldr}.su >> temp_decon_shots.su
done

< temp_xcor_shots.su sushw key=fldr a=1 c=1 j=168 |
    sushw key=tracl a=1 b=1 |
    suchw key1=offset key2=gx key3=sx b=1 c=-1 > ${dataout}_xcor.su

< temp_decon_shots.su sushw key=fldr a=1 c=1 j=168 |
    sushw key=tracl a=1 b=1 |
    suchw key1=offset key2=gx key3=sx b=1 c=-1 > ${dataout}_decon.su

segyhdrs < ${dataout}_xcor.su
segywrite tape=${dataout}_xcor.sgy < ${dataout}_xcor.su

segyhdrs < ${dataout}_decon.su
segywrite tape=${dataout}_decon.sgy < ${dataout}_decon.su

rm -f temp_* stack_* xcor_* decon_*
rm -f binary header
```



<a href="#top">Back to top</a>

