---
published: true
author: Rubaiat
---
Read the problem several times to understand the input constraints present. By observing deeply we can know for sure that, given the input size if small (the largest factorial we will need is **20!**) so here we can generate all the possible factorials and use **bitmasking (or recursion)** to pre-calculate all possible values and **cache** them ... then for every case we just _output the answer_.

I've solved it with a greedy approach. Since, **fact(n) > fact(0)+fact(1)+...+fact(n-1)** (_this is true for each n > 2_) we can iterate from the greatest factorial(20!) to the smallest one, if fact[i] is less than or equal to N we must take it, decrease N by fact[i] and proceed with the next factorial. At the end, if N is 0 we found the solution, otherwise it is impossible to obtain N adding factorials.

{%highlight cpp%}
#include<bits/stdc++.h>
using namespace std;

int main(){
	
	int t;
	long long n;
	unsigned long long fact[22];
	fact[0]=1;
	
	for(int i=1; i<=20; i++){
		fact[i]= fact[i-1] * i;
	}
	cin>>t;
	
	for(int j=1; j<=t; j++){
		scanf("%lld", &n);
		stack <int> q;
		int i=20;
		
		while(i>=0){
			if(fact[i] <= n){
				n -=fact[i];
				q.push(i);
			}
			i--;
		}

		if(n == 0){
			cout<<"Case "<<j<<": ";

			while(q.size() !=1){
				cout<<q.top()<<"!+";
				q.pop();
			}

			cout<<q.top()<<"!\n";
			q.pop();
		}
		else{
			cout<<"Case "<<j<<": impossible\n";
		}
	}
}
	
{%endhighlight cpp%}


![source_code]({{site.baseurl}}/_posts/upload.png)
