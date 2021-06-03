[TOC]



# 1 题目10 正则表达式匹配

## 1.1 题目描述

![image-20210503084601627](file:///Users/cuiguangsong/go/src/docs/Algorithm/images/image-20210503084601627.png?lastModify=1622442044)

![image-20210503084610308](file:///Users/cuiguangsong/go/src/docs/Algorithm/images/image-20210503084610308.png?lastModify=1622442044)

## 1.2 解答

**思路：**

![image-20210503092215819](file:///Users/cuiguangsong/go/src/docs/Algorithm/images/image-20210503092215819.png?lastModify=1622442044)

![image-20210503092232203](file:///Users/cuiguangsong/go/src/docs/Algorithm/images/image-20210503092232203.png?lastModify=1622442044)

![image-20210503092245735](file:///Users/cuiguangsong/go/src/docs/Algorithm/images/image-20210503092245735.png?lastModify=1622442044)

![image-20210503092258595](file:///Users/cuiguangsong/go/src/docs/Algorithm/images/image-20210503092258595.png?lastModify=1622442044)

```java
class Solution {
    public boolean isMatch(String s, String p) {
        if (s == null || p == null) {
            return false;
        }
        char[] sArr = s.toCharArray();
        char[] pArr = p.toCharArray();
        int sLen = sArr.length;
        int pLen = pArr.length;
        boolean[][] dp = new boolean[sLen + 1][pLen + 1];
        for (boolean[] a : dp) {
            for (boolean e : a) {
                e = false;
            }
        }

//        base case
        dp[0][0] = true;
        for (int j = 1; j < pLen + 1; j++) {
            if (pArr[j - 1] == '*') {
                dp[0][j] = dp[0][j - 2];
            }
        }
//        迭代
        for (int i = 1; i < sLen + 1; i++) {
            for (int j = 1; j < pLen + 1; j++) {
                //情况1：s[i-1] 和 p[j−1] 是匹配的
                if (sArr[i - 1] == pArr[j - 1] || pArr[j - 1] == '.') {
                    dp[i][j] = dp[i - 1][j - 1];
                } else if (pArr[j - 1] == '*') {//情况2：s[i-1] 和 p[j−1] 是不匹配的
                    //p[j−1]=="∗"，且 s[i-1] 和 p[j−2] 匹配
                    if (sArr[i - 1] == pArr[j - 2] || pArr[j - 2] == '.') {
                        if (dp[i][j - 2]) {//*让p[j-2]重复0次，此时需要比较s(0,i-1)和p(0,j-3)
                            dp[i][j] = dp[i][j - 2];
                        } else if (dp[i - 1][j - 2]) {//*让p[j-2]重复1次,此时需要比较s(0,i-2)和p(0,j-3)
                            dp[i][j] = dp[i - 1][j - 2];
                        } else {//*让p[j-2]重复≥2次,拿出一个a,让它和s[i-1]抵消，此时需要比较s(0,i-2)和p(0,j-1)
                            dp[i][j] = dp[i - 1][j];
                        }

                    } else {//p[j−1]=="∗"，但 s[i-1] 和 p[j−2] 不匹配,让*干掉p[j-2],继续考察p[j-3]
                        dp[i][j] = dp[i][j - 2];
                    }

                }
            }

        }
        return dp[sLen][pLen];
    }
}
```

使用go解法

```go
func isMatch(s string, p string) bool {
  sLen, pLen := len(s), len(p)
  dp := make([][]bool, sLen+1)
  for i := 0; i < len(dp); i++ {
    dp[i] = make([]bool, pLen+1)
  }
  //base case
  dp[0][0] = true
  for j := 1; j < pLen+1; j++ {
    if p[j-1] == '*' {
      dp[0][j] = dp[0][j-2]
    }
  }
  for i := 1; i < sLen+1; i++ {
    for j := 1; j < pLen+1; j++ {
      //情况1:s[i-1]和p[j-1]匹配
      if s[i-1] == p[j-1] || p[j-1] == '.' {
        dp[i][j] = dp[i-1][j-1]
      } else if p[j-1] == '*' { //情况2:s[i-1]和p[j-1]不匹配,但是p[j-1]是*
        //s[i-1]和p[j-2]匹配
        if s[i-1] == p[j-2] || p[j-2] == '.' {
          //情况2.1:*让p[j-2]出现0次，此时需要比较s(0,i-1)和p(0,j-3)
          if dp[i][j-2] {
            dp[i][j] = dp[i][j-2]
          } else if dp[i-1][j-1] { //情况2.2:*让p[j-2]出现1次,此时需要比较s(0,i-2)和p(0,j-2)
            dp[i][j] = dp[i-1][j-1]
          } else {
            //情况2.3:*让p[j-2]出现≥2次,拿出一个抵消s[i-1],此时需要比较s(0,i-2)和p(0,j-1)
            dp[i][j] = dp[i-1][j]
          }

        } else {
          //s[i-1]和p[j-2]不匹配,让*把p[j-2]干掉，继续考察s[i-1]和p[j-3]是否匹配
          dp[i][j] = dp[i][j-2]

        }
      }
    }
  }
  return dp[sLen][pLen]
}
```

# 2 题目32 最长有效括号

## 2.1 题目描述

![image-20210503114246313](file:///Users/cuiguangsong/go/src/docs/Algorithm/images/image-20210503114246313.png?lastModify=1622442044)

## 2.2 解答

![image-20210503120950530](file:///Users/cuiguangsong/go/src/docs/Algorithm/images/image-20210503120950530.png?lastModify=1622442044)



```
public class Solution {
    public int longestValidParentheses(String s) {
        int maxans = 0;
        int[] dp = new int[s.length()];
        for (int i = 1; i < s.length(); i++) {
            if (s.charAt(i) == ')') {
                if (s.charAt(i - 1) == '(') {
                    dp[i] = (i >= 2 ? dp[i - 2] : 0) + 2;
                } else if (i - dp[i - 1] > 0 && s.charAt(i - dp[i - 1] - 1) == '(') {
                    dp[i] = dp[i - 1] + ((i - dp[i - 1]) >= 2 ? dp[i - dp[i - 1] - 2] : 0) + 2;
                }
                maxans = Math.max(maxans, dp[i]);
            }
        }
        return maxans;
    }
}
```

go的写法

```
func longestValidParentheses(s string) int {
  maxans := 0
  dp := make([]int, len(s))
  for i := 1; i < len(s); i++ {
    if s[i] == ')' {
      if s[i-1] == '(' {
        if i >= 2 {
          dp[i] = dp[i-2] + 2
        } else {
          dp[i] = 2
        }

      } else if i-dp[i-1] > 0 && s[i-dp[i-1]-1] == '(' {
        if i-dp[i-1] >= 2 {
          dp[i] = dp[i-1] + dp[i-dp[i-1]-2] + 2
        } else {
          dp[i] = dp[i-1] + 2
        }
      }
    }
    if dp[i] > maxans {
      maxans = dp[i]
    }
  }

  return maxans

}
```