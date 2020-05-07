---
layout: note
title: 2D vector in cpp 
---

{% highlight js %}
// C++ code to demonstrate 2D vector 
#include <iostream> 
#include <vector> // for 2D vector 
using namespace std; 
int main() 
{ 
    // Initializing 2D vector "vect" with 
    // values
{% raw %} 
    vector<vector<int> > vect{{ -9,   -9,   -9,    1,   1,  1 }, 
                              { 0,    -9,    0,    4,   3,  2 }, 
                              { -9,   -9,   -9,    1,   2,  3 },
                              { 0,     0,    8,    6,   6,  0 }, 
                              { 0,     0,    0,   -2,   0,  0 },
                              { 0,     0,    1,    2,   4,  0 }};
{% endraw %}  

    // Displaying the 2D vector
    cout << "vect size = " << vect.size() << endl; 
    cout << "vect capacity = " << vect.capacity() << endl;

    int sum = 0;
    int largest_i, largest_j;

    for (int i = 0; i < 4; i++) { 
        for (int j = 0; j < 4; j++) {
            int newSum =     vect[i][j] + vect[i][j+1] + vect[i][j+2]
                     + vect[i+1][j+1]
                     + vect[i+2][j] + vect[i+2][j+1] + vect[i+2][j+2];

            if(newSum > sum) {
                sum = newSum;
                largest_i = i;
                largest_j = j;
            }

        }
    } 
  
    cout << "largest hourglass sum = " << sum << " at [" << largest_i << "][" << largest_j << "]" << endl;
    return 0; 
} 
{% endhighlight %}
