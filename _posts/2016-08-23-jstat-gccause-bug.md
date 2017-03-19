---
layout : post
title : jdk6 下面 jstat 结果不准确的一个bug
tags : ['gc','java','jstat']
category : java
---

之前从`jstat -gcuitl`改到`jstat -gccause` ， gccause比gcutil多出来一个last gc reanson，可以看到上一次的gc由什么原因触发。但是实际在服务器上jdk6u45上使用时，新生代的结果全是 `Unknown GCCause`，导致我很困惑。

一度怀疑是jvm参数禁用了s区导致，所以动手实验了一下。发现s区正常的情况下，LGCC依然打印的是`Unknown GCCause`.

```
./jstat -gccause 15628 1000 1000
  S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC
  0.00   0.00   4.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00   4.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00   4.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00   4.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00   4.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00   4.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00   4.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00   4.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00   4.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00   4.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00   4.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00   4.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00   4.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  41.70   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  41.70   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  41.70   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  41.70   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  41.70   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  79.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  79.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  79.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  79.20   0.00  14.45      0    0.000     0    0.000    0.000 No GC                No GC
  0.00  19.56  37.50  30.01  14.45      1    0.005     0    0.000    0.005 unknown GCCause      No GC
  0.00  19.56  39.50  30.01  14.47      1    0.005     0    0.000    0.005 unknown GCCause      No GC
  0.00  19.56  39.50  30.01  14.47      1    0.005     0    0.000    0.005 unknown GCCause      No GC
  0.00  19.56  39.50  30.01  14.47      1    0.005     0    0.000    0.005 unknown GCCause      No GC
  0.00  19.56  39.50  30.01  14.47      1    0.005     0    0.000    0.005 unknown GCCause      No GC
  0.00  19.56  39.50  30.01  14.47      1    0.005     0    0.000    0.005 unknown GCCause      No GC
  0.00  19.56  78.05  30.01  14.47      1    0.005     0    0.000    0.005 unknown GCCause      No GC
  0.00  19.56  78.05  30.01  14.47      1    0.005     0    0.000    0.005 unknown GCCause      No GC
  0.00  19.56  78.05  30.01  14.47      1    0.005     0    0.000    0.005 unknown GCCause      No GC
  0.00  19.56  78.05  30.01  14.47      1    0.005     0    0.000    0.005 unknown GCCause      No GC
  0.00  19.56  78.05  30.01  14.47      2    0.005     0    0.000    0.009 unknown GCCause      No GC
 18.13   0.00  37.50  60.02  14.47      2    0.009     1    0.000    0.009 CMS Initial Mark     No GC
 18.13   0.00  37.50  60.02  14.47      2    0.009     1    0.000    0.009 CMS Initial Mark     No GC
 18.13   0.00  37.50  60.02  14.47      2    0.009     1    0.000    0.009 CMS Initial Mark     No GC
 18.13   0.00  37.50  60.02  14.47      2    0.009     1    0.000    0.009 CMS Initial Mark     No GC
 18.13   0.00  37.50  60.02  14.47      2    0.009     1    0.000    0.009 CMS Initial Mark     No GC
 18.13   0.00  75.69  60.02  14.47      2    0.009     2    0.001    0.010 CMS Final Remark     No GC
 18.13   0.00  75.69  60.02  14.47      2    0.009     2    0.001    0.010 CMS Final Remark     No GC
 18.13   0.00  75.69  60.02  14.47      2    0.009     3    0.001    0.010 CMS Initial Mark     No GC
 18.13   0.00  75.69  60.02  14.47      2    0.009     3    0.001    0.010 CMS Initial Mark     No GC
 18.13   0.00  75.69  90.03  14.47      3    0.013     3    0.001    0.014 CMS Final Remark     No GC
  0.00  19.83  37.50  90.03  14.47      3    0.013     4    0.001    0.014 CMS Final Remark     No GC
  0.00  19.83  37.50  90.03  14.47      3    0.013     5    0.002    0.015 CMS Initial Mark     No GC
  0.00  19.83  37.50  90.03  14.47      3    0.013     6    0.002    0.015 CMS Final Remark     No GC
  0.00  19.83  37.50  90.03  14.47      3    0.013     6    0.002    0.015 CMS Final Remark     No GC
  0.00  19.83  37.50  90.03  14.47      3    0.013     8    0.003    0.016 CMS Final Remark     No GC
  0.00  19.83  37.50  90.03  14.47      3    0.013     8    0.003    0.016 CMS Final Remark     No GC
  0.00  19.83  37.50  90.03  14.47      3    0.013    10    0.004    0.017 CMS Final Remark     No GC
  0.00  19.83  37.50  90.03  14.47      3    0.013    10    0.004    0.017 CMS Final Remark     No GC
  0.00  19.83  37.50  90.03  14.47      3    0.013    12    0.004    0.017 CMS Final Remark     No GC
  0.00  19.83  37.50  90.03  14.47      3    0.013    12    0.004    0.017 CMS Final Remark     No GC
  0.00  19.83  75.46   0.03  14.47      3    0.013    14    0.005    0.018 CMS Final Remark     No GC
  0.00  19.83  75.46   0.03  14.47      3    0.013    14    0.005    0.018 CMS Final Remark     No GC
  0.00  19.83  75.46   0.03  14.47      3    0.013    14    0.005    0.018 CMS Final Remark     No GC
  0.00  19.83  75.46   0.03  14.47      3    0.013    14    0.005    0.018 CMS Final Remark     No GC
 19.79   0.00   0.00  15.04  14.47      4    0.014    14    0.005    0.019 unknown GCCause      No GC
 19.79   0.00  37.50  15.04  14.47      4    0.014    14    0.005    0.019 unknown GCCause      No GC
 19.79   0.00  37.50  15.04  14.47      4    0.014    14    0.005    0.019 unknown GCCause      No GC
 19.79   0.00  37.50  15.04  14.47      4    0.014    14    0.005    0.019 unknown GCCause      No GC
 19.79   0.00  37.50  15.04  14.47      4    0.014    14    0.005    0.019 unknown GCCause      No GC
 19.79   0.00  39.50  15.04  14.51      4    0.014    14    0.005    0.019 unknown GCCause      No GC
```

fgc的次数是我在sleep导致的，不清楚算不算jstat的另外一个bug

搜索了一下，发现果然是一个bug.[JDK-7015169 : GC Cause not always set](http://bugs.java.com/view_bug.do?bug_id=7015169)


这个bug的描述是这样的

```
The GC Cause is not always set up. This can be verified by running:

jstat -gccause <PID> 100

"unknown GCCause" is often listed in the LGCC column.
```

这个bug大概在 2011-02-11 修补，具体的修改在[这里](http://hg.openjdk.java.net/jdk7u/jdk7u-dev/hotspot/rev/c798c277ddd1)可以看到，之后的版本就没有了这个问题。

切换到jdk8，把相同的case跑了一下，果然就没问题了。


```
$ jstat -gccause 8964 1000 1000
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  12.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  49.51   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  49.51   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  49.51   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  49.51   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  49.51   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  87.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  87.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  87.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  87.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00   0.00  87.01   0.00  17.18  19.75      0    0.000     0    0.000    0.000 No GC                No GC
  0.00  55.55  37.50  30.01  55.17  57.22      1    0.005     0    0.000    0.005 Allocation Failure   No GC
  0.00  55.55  37.50  30.01  55.17  57.22      1    0.005     0    0.000    0.005 Allocation Failure   No GC
  0.00  55.55  37.50  30.01  55.17  57.22      1    0.005     0    0.000    0.005 Allocation Failure   No GC
  0.00  55.55  37.50  30.01  55.17  57.22      1    0.005     0    0.000    0.005 Allocation Failure   No GC
  0.00  55.55  37.50  30.01  55.17  57.22      1    0.005     0    0.000    0.005 Allocation Failure   No GC
  0.00  55.55  76.95  30.01  55.17  57.22      1    0.005     0    0.000    0.005 Allocation Failure   No GC
  0.00  55.55  76.95  30.01  55.17  57.22      1    0.005     0    0.000    0.005 Allocation Failure   No GC
  0.00  55.55  76.95  30.01  55.17  57.22      1    0.005     0    0.000    0.005 Allocation Failure   No GC
  0.00  55.55  76.95  30.01  55.17  57.22      1    0.005     0    0.000    0.005 Allocation Failure   No GC
  0.00  55.55  76.95  30.01  55.17  57.22      1    0.005     0    0.000    0.005 Allocation Failure   No GC
  0.00   0.00  37.50  62.82  55.17  57.22      2    0.011     1    0.000    0.011 CMS Initial Mark     No GC
  0.00   0.00  37.50  62.82  55.17  57.22      2    0.011     1    0.000    0.011 CMS Initial Mark     No GC
  0.00   0.00  37.50  62.82  55.17  57.22      2    0.011     1    0.000    0.011 CMS Initial Mark     No GC
  0.00   0.00  37.50  62.82  55.17  57.22      2    0.011     1    0.000    0.011 CMS Initial Mark     No GC
  0.00   0.00  37.50  62.82  55.17  57.22      2    0.011     1    0.000    0.011 CMS Initial Mark     No GC
  0.00   0.00  76.62  62.76  55.17  57.22      2    0.011     2    0.002    0.013 CMS Final Remark     No GC
  0.00   0.00  76.62  62.76  55.17  57.22      2    0.011     2    0.002    0.013 CMS Final Remark     No GC
  0.00   0.00  76.62  62.76  55.17  57.22      2    0.011     3    0.002    0.013 CMS Initial Mark     No GC
  0.00   0.00  76.62  62.76  55.17  57.22      2    0.011     3    0.002    0.013 CMS Initial Mark     No GC
  0.00   0.00  76.62  62.76  55.17  57.22      2    0.011     3    0.002    0.013 CMS Initial Mark     No GC
  0.00   0.00  37.50  92.76  55.17  57.22      3    0.015     4    0.004    0.020 CMS Final Remark     No GC
  0.00   0.00  37.50  92.76  55.17  57.22      3    0.015     4    0.004    0.020 CMS Final Remark     No GC
  0.00   0.00  37.50  92.76  55.17  57.22      3    0.015     6    0.006    0.021 CMS Final Remark     No GC
  0.00   0.00  37.50  92.76  55.17  57.22      3    0.015     6    0.006    0.021 CMS Final Remark     No GC
  0.00   0.00  37.50  92.76  55.17  57.22      3    0.015     8    0.008    0.023 CMS Final Remark     No GC
  0.00   0.00  37.50  92.54  55.18  57.22      3    0.015    11    0.017    0.033 CMS Final Remark     No GC
  0.00   0.00  37.50  92.54  55.18  57.22      3    0.015    11    0.017    0.033 CMS Final Remark     No GC
  0.00   0.00  37.50  92.54  55.18  57.22      3    0.015    13    0.020    0.035 CMS Final Remark     No GC
  0.00   0.00  37.50  92.54  55.18  57.22      3    0.015    13    0.020    0.035 CMS Final Remark     No GC
  0.00   0.00  37.50  92.54  55.18  57.22      3    0.015    15    0.021    0.036 CMS Final Remark     No GC
  0.00   0.00  76.74  92.54  55.18  57.22      3    0.015    15    0.021    0.036 CMS Final Remark     No GC
  0.00   0.00  76.74   2.54  55.18  57.22      3    0.015    17    0.023    0.038 CMS Final Remark     No GC
  0.00   0.00  76.74   2.54  55.18  57.22      3    0.015    17    0.023    0.038 CMS Final Remark     No GC
  0.00   0.00  76.74   2.54  55.18  57.22      3    0.015    17    0.023    0.038 CMS Final Remark     No GC
  0.00   0.00  76.74   2.54  55.18  57.22      3    0.015    17    0.023    0.038 CMS Final Remark     No GC
  0.00   0.00  37.50  17.55  55.18  57.22      4    0.017    17    0.023    0.040 Allocation Failure   No GC
  0.00   0.00  37.50  17.55  55.18  57.22      4    0.017    17    0.023    0.040 Allocation Failure   No GC
  0.00   0.00  37.50  17.55  55.18  57.22      4    0.017    17    0.023    0.040 Allocation Failure   No GC
  0.00   0.00  37.50  17.55  55.18  57.22      4    0.017    17    0.023    0.040 Allocation Failure   No GC
  0.00   0.00  37.50  17.55  55.18  57.22      4    0.017    17    0.023    0.040 Allocation Failure   No GC
```

再贴一个源码的可能的gccause，这个代码是7u40版本的，因为手头正好没有其他版本源码了：

```
const char* GCCause::to_string(GCCause::Cause cause) {
  switch (cause) {
    case _java_lang_system_gc:
      return "System.gc()";

    case _full_gc_alot:
      return "FullGCAlot";

    case _scavenge_alot:
      return "ScavengeAlot";

    case _allocation_profiler:
      return "Allocation Profiler";

    case _jvmti_force_gc:
      return "JvmtiEnv ForceGarbageCollection";

    case _gc_locker:
      return "GCLocker Initiated GC";

    case _heap_inspection:
      return "Heap Inspection Initiated GC";

    case _heap_dump:
      return "Heap Dump Initiated GC";

    case _no_gc:
      return "No GC";

    case _allocation_failure:
      return "Allocation Failure";

    case _tenured_generation_full:
      return "Tenured Generation Full";

    case _permanent_generation_full:
      return "Permanent Generation Full";

    case _cms_generation_full:
      return "CMS Generation Full";

    case _cms_initial_mark:
      return "CMS Initial Mark";

    case _cms_final_remark:
      return "CMS Final Remark";

    case _cms_concurrent_mark:
      return "CMS Concurrent Mark";

    case _old_generation_expanded_on_last_scavenge:
      return "Old Generation Expanded On Last Scavenge";

    case _old_generation_too_full_to_scavenge:
      return "Old Generation Too Full To Scavenge";

    case _adaptive_size_policy:
      return "Ergonomics";

    case _g1_inc_collection_pause:
      return "G1 Evacuation Pause";

    case _g1_humongous_allocation:
      return "G1 Humongous Allocation";

    case _last_ditch_collection:
      return "Last ditch collection";

    case _last_gc_cause:
      return "ILLEGAL VALUE - last gc cause - ILLEGAL VALUE";

    default:
      return "unknown GCCause";
  }
  ShouldNotReachHere();
}
```