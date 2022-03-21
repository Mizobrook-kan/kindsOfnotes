**思路**

rust实现相比其他语言实现的精妙之处，在这道题中就是Option代数数据类型的语义了。None就是裸指针语义，Some对应不是空指针。

对于rust实现，因为ListNode没有Copy特性，所以参数传递是move语义，如果像我直接用原来的参数在函数内调用其他move语义的函数，报E0382错误。

**错误记录**

[自己的实现错误](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=10db129fcc914a461d6d2970e8a90a42)，E0382错误，想用unwrap解封，但unwrap系都不是借用，而且类型中只要有一个没实现Copy特性，在传递参数给其他函数的时候是move语义。
