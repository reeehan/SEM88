#include <iostream>
#include <omp.h>
using namespace std;

int main(){
    int n;
    cout << "Enter number of data points: ";
    cin >> n;

    double *x = new double[n];
    double *y = new double[n];
    double sum = 0;

    cout << "Enter x and y values:\n";
    for(int i=0;i<n;i++){
        cout << "Data " << i+1 << ": ";
        cin >> x[i] >> y[i];
    }

    double w = 1.0;  // weight
    double b = 0.0;  // bias

    // Parallel MSE Loss Calculation
    #pragma omp parallel for reduction(+:sum)
    for(int i=0;i<n;i++){
        double pred = w * x[i] + b;
        sum += (pred - y[i]) * (pred - y[i]);
    }

    cout << "\nLoss = " << sum/n << endl;

    delete[] x;
    delete[] y;

    return 0;
}