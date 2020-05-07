---
layout: note
title: Solution to minimum swaps problem from HackerRank 
---

{% highlight js %}
#include <iostream>
#include <limits>

using namespace std;

void printArray(int arr[], int size) {
	cout << "array: [ ";
	for(int i = 0; i < size; i++) {
		cout << arr[i] << " | ";
	}
	cout << " ]" << endl;
}

void swap(int arr[], int i, int j) {
	cout << "swap " << i << " with " << j << endl;
	auto tmp = arr[i];
	arr[i] = arr[j];
	arr[j] = tmp;
	cout << "newArray: ";
	printArray(arr, 7);
}

void sort(int arr[], int size) {
	int swaps = 0;
	bool swapped = false;
	//unordered_map<int, int> vistedPairs; //pairs of visited indices/values that fit in them

	for(int i = 0; i < size; i++) {
		if(swapped) {
			i--;
			swapped = false;
		}
		int minDiff = INT_MAX;
		int swapWith = i;

		for(int j = i; j < size; j++) {
			if(arr[j] < arr[i]) {
				int diff = arr[i] - arr[j];
				if(diff < minDiff) {
					minDiff = diff;
					swapWith = j;
					cout << "minDiff: " << minDiff << " swapWith: " << swapWith << " i: " << i << " j: " << j << endl;

				}
			}
		}
		if(i != swapWith) {
			swap(arr, i, swapWith);
			swaps++;
			swapped = true;
		}
		
	}
	cout << "total swaps: " << swaps << endl;
}

int main() {
	int arr[] = {7, 1, 3, 2, 4, 5, 6};
	int length = (sizeof(arr)/sizeof(*arr));

	printArray(arr, length);
	sort(arr, length);

	for(int i = 0; i < length; i++) {
		cout << arr[i] << " ";
	}

	cout << "" << endl;

	return 0;
}


{% endhighlight %}

