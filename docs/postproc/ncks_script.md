# Full ncks post-processing script

```
#!/bin/bash
#SBATCH --qos=standard
#SBATCH --job-name=mcnk_1m
#SBATCH --time=01:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --account=n01
#SBATCH --partition=standard
# Created by: mkslurm -S 0 -s 0 -m 2 -C  12 -c  8 
module -s restore /work/n01/shared/acc/n01_modules/ucx_env
export OMP_NUM_THREADS=1
export PATH=/work/n01/n01/acc/TOOLS/bin:$PATH
export LD_LIBRARY_PATH=/work/n01/n01/acc/TOOLS/lib:$LD_LIBRARY_PATH
#
dstdir=/work/n01/n01/acc/NEMO/OUT_eORCA1/A006/monthly
dtadir=/work/n01/n01/acc/NEMO/r4.0.3/dev_r4.0.3_NERC/cfgs/ORCA1_MEDUSA/EXP_A001/MEANSA6
#####
ystart=2015
yend=2044
#####
tsklst=$dstdir/tasks.conf
tskdir=`pwd`
#
# cd to the data directory
#
cd $dtadir
#
# loop over years
#
for y in `seq $ystart 1 $yend`
do
 #
 # loop over filetypes
 #
 for typ in diaptr2D diaptr3D icemod grid_T grid_U grid_V grid_W diad_T ptrc_T
 do
  if [ -f $tsklst ] ; then rm $tsklst ; fi
  n=0
  #
  # loop over months
  #
  for m in 01 02 03 04 05 06 07 08 09 10 11 12
  do
   #
   # loop over matching files (there should only be a single match)
   #
   for f in `ls -1 e*_1m_*${typ}*${y}${m}.nc`
   do
     if [ ! -d ${dstdir}/$y ] ; then mkdir ${dstdir}/$y ; fi
     #
     # construct a more concise filename
     #
     if [ $f == ${f/icemod} ] && [ $f == ${f/diaptr} ]; then
       ff=`echo $f | sed -e's/\(.*\)1m_\(........_........\)\(_[a-zA-Z]*_[a-zA-Z]*\)_.*-\(....\)\(..\).nc/\1y\4m\5\3.nc/'`
     else
       ff=`echo $f | sed -e's/\(.*\)1m_\(........_........\)\(_[a-zA-Z23]*\)_.*-\(....\)\(..\).nc/\1y\4m\5\3.nc/'`
     fi
     if [ ! -f ${dstdir}/$y/$ff ] ; then
       #
       # add to the task list
       #
       echo $n   ncks --no_abc --cnk_dmn x,64 --cnk_dmn y,64 --cnk_dmn deptht,1 --cnk_dmn time_counter,1 --cnk_dmn depthu,1 --cnk_dmn depthv,1 --cnk_dmn depthw,1 --dfl_lvl 1 $f ${dstdir}/$y/$ff >> $tsklst
       n=$(( $n + 1 ))
     fi
   done  # end file loop
  done  # end month loop
  nt=`wc -l $tsklst | sed -e 's/ .*//'`
  #
  # if there are tasks, then run them
  #
  if test $nt -gt 0 ; then
   srun --ntasks=$nt --mem-bind=local --cpu-bind=v,map_cpu:00,0x8,0x10,0x18,0x20,0x28,0x30,0x38,0x40,0x48,0x50,0x58 --multi-prog $tsklst
  fi
 done   # end filetype loop
done   # end year loop
#
cd $tskdir
```
