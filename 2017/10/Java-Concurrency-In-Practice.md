
#### 安全性
#### 原子性
``` 
public class A{

    public int a;

    public void incre(){
        a++;
    }
}
```
编译并通过javap查看代码
```
...
  public void incre();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=1, args_size=1
         0: aload_0
         1: dup
         2: getfield      #2                  // Field a:I
         5: iconst_1
         6: iadd
         7: putfield      #2                  // Field a:I
        10: return
...
```
a++操作实际进行了三步：读取（getfield）- 修改（iadd）- 写入（putfield）

竞态条件（Race Condition）：由于不恰当的执行时序而出现不正确的结果