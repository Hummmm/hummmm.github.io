depth filter流程
```flow
st=>start: Start
e=>end: End
op1=>operation: new frame

op2=>operation: next seed

op3=>operation: erase seed

op4=>operation: update seed

op5=>operation: Flag the grid cell as occupied

op6=>operation: change to candidate point

op7=>operation: outlier++

cond1=>condition:  seed is too old

cond2=>condition:  seed is visible

cond3=>condition:  find epipolar match

cond4=>condition:  is keyframe

cond5=>condition:  seed is converged

io=>inputoutput: catch something...
st->op1->cond1
cond1(no)->cond2
cond1(yes)->op3->op2(left)->cond1

cond2(yes)->cond3

cond2(no,left)->op2


cond3(yes)->op4(right)->cond4->cond5

cond3(no,left)->op7->op2

cond4(yes,right)->op5->cond5

cond4(no)->cond5(yes)->op6

cond5(no,left)->op2

```



depth filter的f值分析
如果要混合分布里高斯分布产生显著效应，那么组间方差大于组内方差
