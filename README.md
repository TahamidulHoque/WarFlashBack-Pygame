# WarFlashBack-Pygame
/* Tahamidul Hoque
period:7
Galton board project*/
#include <iostream>
#include<cstdlib>
#include<ctime>
using namespace std;
int main(){
int slots[30]={0};
srand(time(0));
int location;
int balls=5000;
//Number of balls
for(int i=0; i<=balls; i++){
	 int position=0;
	//number generator per each ball
	for(int r=0; r<30; r++){
		location=rand()%2+1;
		//will check where the ball goes to
		if(location%2==1)
			position=position+1;
		}
		//add the ball to that position
	slots[position]+=1;
	
}
//showcase the amount of balls dropped
for (int i=0; i<30; i++){
	cout<<i<<": "<<slots[i]<<"\t";
	if(slots[i]>5){
		for(int r=1; r<=slots[i]/5; r++){
			cout<<" *";
		}
	}
	else{
		for(int e=1; e<=slots[i]; e++)
			cout<<" *";
	}
	
	cout<<endl;	
}
		
cout<<"Balls dropped: "<<balls;


}	

















	

















