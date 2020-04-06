
#include <bits/stdc++.h>
using namespace std;
struct Process_Data
{
	int Num;
	int Pid;  
	int C_time; 
	int D_time; 
	int Priority; 
	int F_time; 
	int R_time; 
	int W_time; 
	int S_time;
	int Turnaround_time;

};
struct Process_Data current;
typedef struct Process_Data P_d ;
bool idsort(const P_d& x , const P_d& y)
{
	return x.Pid < y.Pid;
}
bool arrivalsort( const P_d& x ,const P_d& y)
{
	if(x.C_time < y.C_time)
		return true;
	else if(x.C_time > y.C_time)
		return false;
	if(x.Priority < y.Priority)
		return true;
	else if(x.Priority > y.Priority)
		return false;
	if(x.Pid < y.Pid)
		return true;

	return false;
}
bool Numsort( const P_d& x ,const P_d& y)
{
	return x.Num < y.Num;
}
struct comPare
{
	bool operator()(const P_d& x ,const P_d& y)
	{
		if( x.Priority > y.Priority )
			return true;
		else if( x.Priority < y.Priority )
			return false;
		if( x.Pid > y.Pid )
			return true;
		return false;	
	}	
};
int main()
{
	int i;
	float twt,tat;
	vector< P_d > input;
	vector<P_d> input_copy;
	P_d temp;
	int pq_process = 0; 
	int rq_process = 0; 
	int C_time;
	int D_time;
	int Pid;
	int Priority;
	int n;
	int clock;
	int total_exection_time = 0;
	cout<<"Enter the no of processes.\n";
	cin>>n;
	cout<<"Enter PID,arrivaltime,execution time & priority for "<<n;cout<<" processes:\n";
	for( i= 0; i< n; i++ )
	{
		cin>>Pid>>C_time>>D_time>>Priority;
		temp.Num = i+1;
		temp.C_time = C_time;
		temp.D_time = D_time;
		temp.R_time = D_time;
		temp.Pid = Pid;
		temp.Priority = Priority;
		input.push_back(temp);
	}
	input_copy = input;
	sort( input.begin(), input.end(), arrivalsort );
    total_exection_time = total_exection_time + input[0].C_time;
    for( i= 0 ;i< n; i++ )
    {
    	if( total_exection_time >= input[i].C_time )
    	{
    		total_exection_time = total_exection_time +input[i].D_time;
    	}
    	else
    	{
    		int diff = (input[i].C_time - total_exection_time);
    		total_exection_time = total_exection_time + diff + D_time;

    	}
    }
	int Ghant[total_exection_time]={0}; 
	for( i= 0; i< total_exection_time; i++ )
	{
		Ghant[i]=-1;
	}
	priority_queue < P_d ,vector<Process_Data> ,comPare> pq; 
	queue< P_d > rq; 
	int cpu_state = 0; 
	int quantum = 4 ; 
	current.Pid = -2;
	current.Priority = 999999;
	for ( clock = 0; clock< total_exection_time; clock++ )
	{
		for( int j = 0; j< n ; j++ )
		{
			if(clock == input[j].C_time)
			{
				pq.push(input[j]);
			}
		}
		if(cpu_state == 0) 
		{
			if(!pq.empty())
			{
				current = pq.top();
				cpu_state = 1;
				pq_process = 1;
				pq.pop();
				quantum = 4; 
			}
			else if(!rq.empty())
			{
				current = rq.front();
				cpu_state = 1;
				rq_process = 1;
				rq.pop();
				quantum = 4;
			}
		}
		else if(cpu_state == 1) 
		{
			if(pq_process == 1 && (!pq.empty()))
			{
				if(pq.top().Priority < current.Priority ) 
				{
					rq.push(current); 
					current = pq.top();
					pq.pop();
					quantum = 4; 
				}
			}
			else if(rq_process == 1 && (!pq.empty())) 
			{
				rq.push(current);
				current = pq.top();
				pq.pop();
				rq_process = 0;
				pq_process = 1;
				quantum = 4 ;
			}
		}
		if(current.Pid != -2) 
		{
			current.R_time--;
			quantum--;
			Ghant[clock] = current.Pid;
			if(current.R_time == 0) 
			{
				cpu_state = 0 ;
				quantum = 4 ;
				current.Pid = -2;
				current.Priority = 999999;
				rq_process = 0;
				pq_process = 0;
			}
			else if(quantum == 0 )  
			{
				rq.push(current);
				current.Pid = -2;
				current.Priority = 999999;
				rq_process = 0;
				pq_process = 0;
				cpu_state=0;
			}
		}	
	}
	sort( input.begin(), input.end(), idsort );
	for(int i=0;i<n;i++)
	{
		for(int k=total_exection_time;k>=0;k--)
		{
			if(Ghant[k]==i+1)
			{
				input[i].F_time=k+1;
				break;
			}
		}
	}
	for(int i=0;i<n;i++)
	{
		for(int k=0;k<total_exection_time;k++)
		{
			if(Ghant[k]==i+1)
			{
				input[i].S_time=k;
				break;
			}
		}
	}
	sort( input.begin(), input.end(), Numsort );
	for(int i=0;i<n;i++)
	{
		input[i].W_time=(input[i].F_time-input[i].C_time)-input[i].D_time;
		input[i].Turnaround_time=input[i].W_time+input[i].D_time;
	}
	cout<<"PID Waiting_time Finish_time Turnaround_time\n";
	for(int i=0;i<n;i++)
	{
		cout<<" "<<input[i].Pid<<"     "<<input[i].W_time<<"           "<<input[i].F_time<<"            "<<input[i].Turnaround_time<<endl;
		twt+=input[i].W_time;
		tat+=input[i].Turnaround_time;	
	}
	cout<<"Average Waiting time="<<twt/n<<endl;
	cout<<"Average Turnaround time="<<tat/n<<endl;	
	return 0;
}
