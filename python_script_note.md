### python notebook
**numpy,pandas**

```#!/bin/py
"""argv[0] means script
argv[1] means probe file
argv[2] means input expression file
argv[3] means target patnoid
argv[4] means output file"""

from itertools import islice
from sys import * ##sys.argv[0] 表示脚本本身，argv[1]代表传入的第一个数据
import numpy as np
import pandas as pd

probe_list = []
for line in open(argv[1], 'r'): ##the first input file
	line = line.strip()
	probe_list.append(line)
row_number = len(probe_list)
print(row_number)

with open(argv[2]) as inputfile:
	header = inputfile.readline().strip().split('\t') ## get the first row from input file获取文件第一行
	col_number = len(header)## get column numbers of file
print(col_number)
header_changed = []
for item in header:
	#if item=='geneid':
	#	header_changed.append(item)
	#else:
	#	header_changed.append(item.split('.')[0][1:])

	header_changed.append(item)
probe_exp = []

for line in islice(open(argv[2], 'r'), 1, None): ##the seconde input file
	line1 = line.strip().split('\t') ##return a list
	if line1[0] in probe_list:
		probe_exp.append(line1)
	else:
		pass

print(len(probe_exp))
probe_array = []
for item in probe_exp:
	probe_array = np.append(probe_array,item) ##save to np array存入数组中

#probe_array = np.concatenate(probe_exp[:])
probe = probe_array.reshape(row_number,col_number) ## list was changed to array;#print(probe);#print(probe.shape)

probe_dataframe = pd.DataFrame(data=probe, columns=header_changed) ##np.array to pd.DataFrame; 

#print(probe_dataframe.columns)
probe_dataframe.index = probe_dataframe.ix[:,0]
#probe_dataframe.to_csv(argv[4], header=True, index=False) #probe_dataframe.columns = header
target_patno = []
for line in open(argv[3], 'r'):
	line = line.strip()
	target_patno.append(line)

target_patno_dataframe = probe_dataframe.loc[:, target_patno]
target_patno_dataframe.to_csv(argv[4])```

####pd.DataFrame
pd.set_option('precision', n) ##是个数据小数点的位数

df = pd.read_table(argv[1], sep="\t", header=0, index_col=0) ##header=0指定第一行为列名，index_col=0指定第一列为行名

df = df.ix[1:10,['column1','column2']] ##只取第二行至第九行，第二列和第三列的数据

df = df.drop(index=["pat","mat"], axis=0) ##去掉指定行
df = df.drop(columns=[

