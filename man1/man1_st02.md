- [Main page](../index.md)

- [Prev step](man1_st01.md)

- [Next step](man1_st03.md)

  

**(2.1) Stacking shots at the same locations **

_Xstacking.scr:_

```sh
#!/bin/sh
#
# This script is created to stack four shots acuired at the same position,
# with the purpose of improving S/N ratio.
#
# 20-01-2017, J. Liu 
# Usage: ./Xstacking.scr $ffid1 $ffid2 $sx $fldr xcor
# Used by X02_stacking_all.scr
 
for ((EP=$1; EP<=$2; EP++)) do

# Set shot point (sx) and receiver point (gx)

        < $5_w${EP}.su sushw key=scalco a=100 |
            sushw key=sx a=$3 | 
            sushw key=gx a=0 b=25 >temp003.su  

# Rename individual shot 
# Make sure that there are 4 shots to stack,
# or some change shoud be made.
	if ((${EP} % 4==1))
	  then
	     mv temp003.su shot1.su
	elif ((${EP} % 4==2))
	   then
	      mv temp003.su shot2.su
	elif ((${EP} % 4==3))
	    then
              mv temp003.su shot3.su
	elif ((${EP} % 4==0))
	     then
              mv temp003.su shot4.su
	fi

done

# Stacking, with last two shots reversed
suop2 shot1.su shot2.su op=sum > shot5.su
suop2 shot3.su shot4.su op=sum > shot6.su

suop2 shot5.su shot6.su w1=0.5 w2=0.5 op=sum > shot7.su
mv shot7.su stack_$5_$4.su

rm -f shot* temp003.su
```



**(2.2) Stacking for all the acquired positions**

_X02_stacking_all.scr:_

```sh
#!/bin/sh
#
# Stack shot gathers at different locations.
#
# 20-01-2017, J. Liu 
# Usage: ./X02_stacking_all.scr

fd1=6; fd2=150

for ((fldr=1; fldr<=8; fldr++)) do
    ffid1=$(echo 6 4  $fldr | awk '{print $1 + $2*($3-1)}')
    ffid2=$(echo 9 4 $fldr | awk '{print $1 + $2*($3-1)}')
    sx=$(echo -600 150 $fldr | awk '{print $1 + $2*($3-1)}')

    echo "FFID ($ffid1 ~ $ffid2), for shot at $sx cm"

   ./Xstacking.scr $ffid1 $ffid2 $sx $fldr xcor
   ./Xstacking.scr $ffid1 $ffid2 $sx $fldr decon
done

for ((fldr=9; fldr<=36; fldr++)) do
    ffid1=$(echo 39 4  $fldr | awk '{print $1 + $2*($3-9)}')
    ffid2=$(echo 42 4 $fldr | awk '{print $1 + $2*($3-9)}')
    sx=$(echo 600 150 $fldr | awk '{print $1 + $2*($3-9)}')

    echo "FFID ($ffid1 ~ $ffid2), for shot at $sx cm"

   ./Xstacking.scr $ffid1 $ffid2 $sx $fldr xcor
   ./Xstacking.scr $ffid1 $ffid2 $sx $fldr decon
done

```



<a href="#top">Back to top</a>