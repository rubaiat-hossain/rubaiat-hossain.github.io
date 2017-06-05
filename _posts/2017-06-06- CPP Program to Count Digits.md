---
published: true
---
Often programmers are faced with the task of **counting digits** for a number with some special phenomena.
Like, _how many digit's there for the factorial of a number **n** in base **b**_?? Here, the last part (factorial of n, in base b) represents the phenomena; for this specific attribute we need to count the number of digits present.

The simple task of counting digits for a given _positive_ number can de done with the following cpp code. This code demonstrates a very simple approach, feel free to examine and modify the code in order to better understand it.

{%highlight cpp%}
/* this program counts the no. i of digits
          in a given number num          */
#include<bits/stdc++.h>
using namespace std;

int main(){
  int num, i=0;
  cin>>num;  //get the number

  while(num>0){
    num= num/10;
    i++;

  }
  cout<<i<<"\n";  //i no. of digits
}

{%endhighlight cpp%}

So, after you've determined the specific type of number you're looking for, you can use this code count the digits. In case of any confusion, feel free to google it and do some research.
