---
published: true
author: Rubaiat
---

##Solution for the LightOJ problem 1001: "Opposite Task"

Here, I'm giving the solve of LightOJ prob. 1001. Please, try hard first before checking the solution and try to come up with your own unique logic to solve it

{%highlight cpp%}
#include<bits/stdc++.h>
using namespace std;

int main(){
  int t, n, i;
  cin>>t;

  for(i=0; i<t; i++){

    cin>>n;
    int temp= n/2;
    n=n%2 + n/2;

    cout<<n<<" "<<temp<<"\n";
  }

  return 0;
}

{%endhighlight cpp%}
