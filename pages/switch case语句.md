- switch (表达式){
  case 值1 : 语句1
  break;
- case 值2 : 语句2
  break;
- ...
  default : 语句n
  break;
  }
- 从表达式值等于某个case语句后的值开始，它下方的所有语句都会一直运行，直到遇到一个break为止。随后，switch语句将结束，程序从switch结束大括号之后的第一个语句继续执行，并忽略其他case。
  假如任何一个case语句的值都不等于表达式的值，就运行可选标签default之下的语句。
  假如表达式的值和任何一个case标签都不匹配，同时没有发现一个default标签，程序会跳过整个switch语句，从它的结束大括号之后的第一个语句继续执行。