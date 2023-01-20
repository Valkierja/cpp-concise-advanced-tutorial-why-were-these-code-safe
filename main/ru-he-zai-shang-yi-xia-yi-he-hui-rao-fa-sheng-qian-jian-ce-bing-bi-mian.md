# 如何在上溢、下溢和回绕发生前检测并避免

解决方法其实很简单，即对数据的封装，将初始的int类封装在自己的类里面，然后重载加法运算符等运算符

加减比较简单，只需要一个不等式的检验

```
#include <limits.h>

int a = <something>;
int x = <something>;
if (x > 0 && a > INT_MAX - x) // `a + x` would overflow
if (x < 0 && a < INT_MIN - x) // `a + x` would underflow
```

乘法的判定相对复杂

```
#include <limits.h>

int a = <something>;
int x = <something>;
// There may be a need to check for -1 for two's complement machines.
// If one number is -1 and another is INT_MIN, multiplying them we get abs(INT_MIN) which is 1 higher than INT_MAX
if (a == -1 && x == INT_MIN) // `a * x` can overflow
if (x == -1 && a == INT_MIN) // `a * x` (or `a / x`) can overflow
// general case
if (x != 0 && a > INT_MAX / x) // `a * x` would overflow
if (x != 0 && a < INT_MIN / x) // `a * x` would underflow
```

更建议的方法是利用库

采用C23标准中的

```
bool ckd_add(type1 *result, type2 a, type3 b);
bool ckd_sub(type1 *result, type2 a, type3 b);
bool ckd_mul(type1 *result, type2 a, type3 b);
```
