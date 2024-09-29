---
title: php根据权重随机方法
date: 2024-09-29 16:21:37
tags: [php]
categories: [php]
---
```
public static function lotto($weight = array())
    {
        $roll = sprintf("%.2f", mt_rand() / mt_getrandmax() * (array_sum($weight)));
        $_tmpW = 0;
        $rollnum = 0;
        foreach ($weight as $k => $v) {
            $min = $_tmpW;
            $_tmpW += $v;
            $max = $_tmpW;
            if ($roll > $min && $roll <= $max) {
                $rollnum = $k;
                break;
            }
        }
        if ($rollnum == 0 && !is_string($rollnum)) {
            return self::lotto($weight);
        }
        return $rollnum;
    }
$lottoArr[1] = 40; //要随机的id =》权重
$lottoArr[2] = 20;
$lottoArr[3] = 40;
$randId = self::lotto($lottoArr); //开始随机
```

