---
published: true
author: Rubaiat
---
**Problem source: LightOJ**

**Problem no: 1069-Lift**

This is not a tough problem, just slightly tricky. I've commented out some error prone codes to demonstrate the tricky section.

I suggest you to try the problem yourself first. You should only see the solution when you're certain that you can't solve it. Below is the code I've written and used.

{%hightlight cpp%}
#include<bits/stdc++.h>
using namespace std;

int lift_cal(int me, int li){
  // 4 sec to move, 3 sec to open/close, 5 sec to enter/exit
  int move, open, close, enter, exit;
  move=4; open=3; close=3; enter=5; exit=5;
  int time;
  //time=(li-me)*move +(open+enter+close)+ (me*move) + (open+exit);
  time=(int)abs((double)me-li) *move +(open+enter+close)+ (me*move) + (open+exit);
  return time;
}

int main(){
  int t, i, a, b;
  cin>>t;
  for(i=0;i<t;i++){
    cin>>a>>b;
    cout<<"Case "<<i+1<<": "<<lift_cal(a,b)<<"\n";
  }
  return 0;
}

{%endhighlight cpp%}

You should watch the code carefully and understand every bit of it. Try tweaking the logic and shorten the code. Feel free to educate yourself from google.
