## 细读golang底层各种锁源码
#### **互斥锁**:    
互斥锁作为是并发操作中多个程序对共享资源的访问控制的必要手段，（竞争）而golang则定义了mutex和rwmutex，互斥锁和读写互斥锁来实现。     
### 我们首先来看一段程序进而理解锁的重要性。     
```
package bank
var balance int
func _deposit(amount int){
balance+=amount
}
func _balance() int{
  return balance
}
```
我们首先定义了一个银行，次银行定义了用户余额，以及两个方法_deposit和_balance方法，其中_deposit表示我们用户向balance进行存款，balance是我们查询用户的余额。其实我们初看此程序是没有什么问题的，我们有钱存进来我们就进行存储，然后我们还可以查询余额。但是这段代码是有着很大的漏洞的。    
*我们首先考虑一种情况*:     
我们现在定义两个goroutine：  
```
//A进行存款并读取账户余额
go func(){
  bank._deposit(200)
  fmt.Println(bank._balance(balance))
}()
//B进行存款操作
go _deposit(100)
```


现在当我们A在存款中我们将执行balance+amount操作，此时是写操作，并且还未进行更新，而此时我们的B也在进行存款，即发生在写操作之后，更新操作之前，此时我们的balance读到的还是A写入操作的“200”，而我们的B的操作就会直接被忽略，因为此时我们即便B的操作正常的顺利执行了，也就相当于balance被更新到了100；但是我们的这是发生在A写之后和A更新之前的，这样当我们读到A更新操作时，我们的balance还是会被赋值为200；即最终我们的balance还是200；这对于我们的客户是极其不公平的。         
所以上述有两种方法，一种是设置channel通道信号量（本文不介绍），一种是使用互斥锁的原理，下面我们将详细介绍锁底层原理，希望有助于各位读者理解