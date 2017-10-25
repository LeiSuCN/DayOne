
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

#### 重排序（Reordering）
可充分利用多核处理器性能；在缺少同步情况下，Java内存模型允许编译器以及CPU对操作顺序进行重排序，并将数值缓存在寄存器中。
CPU与存储设备之间的运算速度有几个数量级的差距，所以现代计算机系统都不得不加入一层读写尽可能接近CPU速度的高速缓存(Cache)，作为内存与CPU间的缓冲。将运算需要的数据复制到缓冲，让运算快速进行，完后将缓存同步到内存。如此，CPU就无需等待缓慢的内存读写。